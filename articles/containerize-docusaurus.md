---
title: "Nginx + Docusaurus でドキュメントをコンテナ化してみる"
emoji: "🐳"
type: "tech"
topics: ["vscode", "docusaurus", "docker", "nginx", "markdown"]
published: true
published_at: 2022-12-07 00:00
---

:::message
この記事は、🦌🛷[**カサレアル Advent Calender 2022**](https://qiita.com/advent-calendar/2022/casareal)🎄の **7** 日目のエントリです。
:::

# TL;DR

* Docusaurus の静的 Web サイトを Docker イメージに組み込んで Nginx 経由でアクセスできるようにする
* Docusaurus のビルドとデプロイはマルチステージビルドで行う

# はじめに

みなさん、ドキュサウルスとなかよくしてますか？
[前回のアドベントカレンダーの記事](https://zenn.dev/casa_snona/articles/docusaurus-is-good-here)を見て、少しでもドキュサウルスとお友達になってくれたらうれしいです！

ところで、時代はコンテナですよ🐳
ということで、ドキュサウルスと一緒にクジラに乗ってコンテナ時代を航海してみましょう！

まだドキュサウルスとお友達になれていないよって方は、👇の記事を見てから一緒に出かけましょう。

https://zenn.dev/casa_snona/articles/vscode-docusaurus-local-env
https://zenn.dev/casa_snona/articles/docusaurus-is-good-here

イメージとしては👇のようなことをします[^1]。

![環境イメージ](/images/containerize-docusaurus/containerize-image.drawio.png)
*環境イメージ*

[^1]: 厳密には、Docusaurus の開発環境も Docker コンテナで行うので、`Docker in Docker` の構成になります。

# Nginx の設定ファイルを作成

まずは、イメージに組み込む Nginx の設定ファイルを作っておきます。

`server-config` というフォルダを作成して、`nginx.conf` というファイルを作成します。
ファイルの中身は次のようにします。

```nginx:server-config/nginx.conf
events {
    multi_accept on;
}

http {
    sendfile on;
    tcp_nopush on;
    server_tokens off;
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    server {
        listen 80;
        server_name localhost;

        location / {
            root /usr/share/nginx/html;
            index index.html index.htm;
        }
    }
}
```

フォルダ構成は👇のようになります。

```
./hello-docusaurus
├── .devcontainer   # Dev Containers の設定フォルダ
├── my-website      # Docusaurus のプロジェクトフォルダ
└── server-config   # 今回作成するフォルダ
     └── nginx.conf # Nginx の設定ファイル
```

今回の Nginx の設定ファイルは開発向けの簡単な設定ですので、とくに難しい設定はしていません。
パフォーマンスを上げる設定（`tcp_nopush on;`）や、セキュリティの観点からバージョン情報を隠す設定（`server_tokens off;`）なんかを入れていますが趣味の問題です。

重要なポイントは `server` の設定で、`/` パスにアクセスがあったら `index.html` に誘導する設定です。
つまり、`http://localhost`（ポート：`80`）にアクセスすると、`/usr/share/nginx/html` ディレクトリ配下の静的 HTML を表示するという設定になります。

# Dockerfile を作成

Docusaurus のビルドをして、そのビルド結果を Nginx のイメージに配置してコンテナ化します。いわゆるマルチステージビルドです。
百聞は一見に如かずですが、Dockerfile 的には FROM が複数あるやつです。（雑な説明ですみません🙏）

では、`Dockerfile` を作成して、ファイルの中身は次のようにします。

```dockerfile:Dockerfile
FROM node:lts-alpine3.16 AS build
COPY ./my-website ./
RUN npm install
RUN npm run build

FROM nginx:1.23.2
COPY ./server-config/nginx.conf /etc/nginx/nginx.conf
COPY --from=build /build /usr/share/nginx/html
CMD [ "nginx", "-g", "daemon off;" ]
```

二段構成になっていて、一段階目は `node` という Node.js が利用できる Docker イメージで Docusaurus のビルドを実施しています。
Docusaurus のビルドをすると、`build` というフォルダが作成されて、その中に静的 HTML 等が展開されます。

```dockerfile:一段階目
# Node.js のイメージを Pull し、
# このコンテナ環境に build という名前で参照できるようにする
FROM node:lts-alpine3.16 AS build

# ローカルの my-website をコンテナにコピー
COPY ./my-website ./

# Docusaurus のビルドを実行
RUN npm install
RUN npm run build
```

二段階目は `nginx` という Nginx（Web サーバ） が利用できる Docker イメージで Web サーバを起動します。

```dockerfile:二段階目
# Nginx のイメージを Pull する
FROM nginx:1.23.2

# Nginx の設定ファイルおよび
# node のコンテナでビルドした結果をコンテナにコピー
COPY ./server-config/nginx.conf /etc/nginx/nginx.conf
COPY --from=build /build /usr/share/nginx/html

# Nginx の起動
CMD [ "nginx", "-g", "daemon off;" ]
```

:::message
このようにビルドとデプロイといった各ステージ毎に段階を踏むような Dockerfile の構成は、**マルチステージビルド**と呼ばれています。
:::

# Let's コンテナ化！

それでは、Dockerfile も出来上がったので、早速イメージをビルドしてみましょう！

わかりやすいようにイメージにタグ（`my-website:1.0.0`）をつけておきます。
また、実感できるほどではないかもしれませんが、パフォーマンスがよくなるように BuildKit を利用するようにします。

```bash
DOCKER_BUILDKIT=1 docker build -t my-website:1.0.0 .
```

エラーがなく正常に完了したら、作成されたイメージを確認してみましょう。

```bash
docker images
```

:::message
👇のように表示されていれば OK です。
```
$ docker images
REPOSITORY   TAG       IMAGE ID       CREATED       SIZE
my-website   1.0.0     7e2f0a71f3d7   4 hours ago   143MB
```
:::

では、いよいよコンテナを起動してみましょう！

```bash
docker run --rm -p 80:80 my-website:1.0.0
```

👆のコマンドを実行すると、VSCode のターミナルでは次のような出力が出てくるかと思います。
また、`ポート`タブを開くと自動で `80` ポートにアクセスできる URL が表示されているかと思います。（さすが、VSCode！）

![コンテナ起動](/images/containerize-docusaurus/docker-run.png)
*コンテナの起動*

`ポート`タブの🌐マークをクリックしてブラウザを起動してみましょう。

![VSCode のポートタブ](/images/containerize-docusaurus/vscode-port.png)
*VSCode のポートタブ*

Docker コンテナに乗ったドキュサウルスと会えましたね🐳🦖✨

# 最後に

Docusaurus をコンテナ化することで、Docker が動く環境であればどこでもドキュサウルスに会うことができるようになりましたね！
あらためてコンテナは素晴らしい技術ですよね。

移植性の高いコンテナ化ができたので、もっと世界中の人たちとドキュサウルスが触れ合えるように、どこからでもアクセスできるようにしたくなりませんか？
きっとドキュサウルスもそれを望んで Docker の背中に乗り込んだのだと思います。

でもその前に、信じたくないですが世の中には極悪非道な人たちも一定数は存在しています。
そんなところにドキュサウルスをいきなり放ってしまったら予期せぬ攻撃を受けてしまうかもしれません🥹
もしそうなったら、ドキュサウルスがかわいそうですよね？

ということで、次回は信頼するお友達とだけ触れ合えるように、認証機能を追加してみたいと思います。
それでは、次回のアドベントカレンダー（勝手にシリーズ化）の更新をお楽しみに～😉
