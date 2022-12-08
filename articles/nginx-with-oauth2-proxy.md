---
title: "Nginx + OAuth2 Proxy で静的 Web サイトに認証機能を追加してみる"
emoji: "🔐"
type: "tech"
topics: ["vscode", "docusaurus", "docker", "nginx", "oauth"]
published: true
published_at: 2022-12-10 00:00
---

:::message
この記事は、🦌🛷[**カサレアル Advent Calender 2022**](https://qiita.com/advent-calendar/2022/casareal)🎄の **10** 日目のエントリです。
:::

# TL;DR

* [OAuth2 Proxy](https://oauth2-proxy.github.io/oauth2-proxy/) を使って Docusaurus で作成したドキュメントサイトに認証機能をつける
* OAuth2 Proxy は、認証と認可を外部の認証基盤に委譲するためのリバースプロキシサーバ
* 外部の認証基盤には、[Amazon Cognito](https://aws.amazon.com/jp/cognito/)（認証・認可およびユーザ管理機能を提供する AWS のマネージドサービス）を利用

# はじめに

[前回の記事](https://zenn.dev/casa_snona/articles/containerize-docusaurus)では、ドキュサウルスはコンテナ時代を航海するために、Docker の背中に乗り込みました。
とはいえ、いきなりインターネットに飛び出すのも気が引けますよね。
そこで、今回は信頼できるお友達とだけドキュサウルスに会えるようにしてあげましょう。

イメージとしては👇のような構成になります。

![ローカル環境で認証機能の確認をする](/images/nginx-with-oauth2-proxy/oauth2-proxy-image.drawio.png)
*ローカル環境で認証機能の確認をする*

:::message
単に認証機能をつけるだけであれば **OAuth2.0** や **リバースプロキシサーバ** などの事前知識はとくに不要ですが、より深く理解したい方は次に紹介する動画がわかりやすいです。
※ 紹介している動画のタイトル的に筆者がネットワークスペシャリストなのかと思われるかもしれませんが違います😅
:::

https://youtu.be/ySjGHYPypck

https://youtu.be/5rkYab-FgFk

# OAuth2 Proxy って？

認証と認可を外部の認証基盤に委譲するためのリバースプロキシサーバです。
MIT License の OSS なので誰でも自由に使えます。
Google や GitHub などの OIDC プロバイダーを利用して認証を提供し、アカウントを検証することができます。

詳細が知りたい方は、👇のドキュメントをご覧ください。

https://oauth2-proxy.github.io/oauth2-proxy/

あれ？、気づいちゃいましたか？
実は OAuth2 Proxy のドキュメントサイトは Docusaurus で作成されているんです🦖

# 認証するのは誰？

今回は、[Amazon Cognito](https://aws.amazon.com/jp/cognito/)（ユーザープール）を外部の認証基盤として使ってみます。
Cognito は、認証・認可およびユーザ管理機能を提供する AWS のマネージドサービスです。

:::message
Cognito の構築手順は割愛しますが、設定ポイントは👇です。
下記の設定以外はデフォルトで作成しました。
※ デモ用にセキュリティレベルを下げた設定なので、安全ではないです。

* Cognito ユーザープールのサインインオプション
  * [x] E メール
* 多要素認証
  * [x] MAF なし
* ユーザアカウントの復旧
  * [ ] セルフサービスのアカウントの復旧を有効化（チェックを外す）
* セルフサービスのサインアップ
  * [ ] 自己登録を有効化（チェックを外す）
* 属性検証とユーザーアカウントの確認
  * [ ] Cognito が検証と確認のためにメッセージを自動的に送信することを許可（チェックを外す）
* E メール
  * [x] Cognito で E メールを送信
* ユーザープール名
  * 任意のものを設定
* ホストされた認証ページ
  * [x] Cognito のホストされた UI を使用
* Cognito ドメイン
  * 重複しない利用可能なドメインプレフィックスを設定
* 最初のアプリケーションクライアント
  * [x] クライアントのシークレットを生成する
* 許可されているコールバック URL
  * `http://localhost/oauth2/callback`
* 高度なアプリケーションクライアントの設定
  * 許可されているサインアウト URL
    * `http://localhost/oauth2/sign_out`
:::

# 認証機能を追加してみよう

では、認証基盤の準備ができたので、各種設定ファイルを見ていきましょう。

## Nginx の設定ファイルを編集

次のように、OAuth2 Proxy 用のパス設定を追加しています。

```diff nginx:server-config/nginx.conf
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
+           auth_request /oauth2/auth;
+           error_page 401 = /oauth2/sign_in;
            root /usr/share/nginx/html;
            index index.html index.htm;
        }

+       location /oauth2/ {
+           proxy_pass http://127.0.0.1:4180;
+           proxy_set_header Host $host;
+           proxy_set_header X-Real-IP $remote_addr;
+           proxy_set_header X-Scheme $scheme;
+       }

+       location /oauth2/auth {
+           proxy_pass http://127.0.0.1:4180;
+           proxy_set_header Host $host;
+           proxy_set_header X-Real-IP $remote_addr;
+           proxy_set_header X-Scheme $scheme;
+           proxy_set_header Content-Length "";
+           proxy_pass_request_body off;
+       }
+   }
}
```

## OAuth2 Proxy の設定ファイルを作成

次に、`server-config` 配下に OAuth2 Proxy の設定ファイルのテンプレート（`oauth2proxy.conf.template`）を新規に作成します。
中身は次のようにします。（おそらく最小構成だと思います🙏）

```properties:server-config/oauth2proxy.conf.template
provider = "oidc"
http_address = "127.0.0.1:4180"
email_domains = ["*"]
scope = "openid"

cookie_secret = "${COOKIE_SECRET}"
cookie_secure = false
session_cookie_minimal = true

oidc_issuer_url = "https://cognito-idp.ap-northeast-1.amazonaws.com/${USER_POOL_ID}"

client_id = "${APPLICATION_CLIENT_ID}"
client_secret = "${APPLICATION_CLIENT_SECRET}"

redirect_url = "${REDIRECT_URL}"
```

ファイル内には、次の環境変数を埋め込んでいます。

| 環境変数                  | 設定値                                                   |
| :------------------------ | :------------------------------------------------------- |
| COOKIE_SECRET             | Cookie のシード文字列（後ほど、Python で生成します）     |
| USER_POOL_ID              | Cognito のユーザープール ID                              |
| APPLICATION_CLIENT_ID     | Cognito のアプリケーションクライアント ID                |
| APPLICATION_CLIENT_SECRET | Cognito のアプリケーションクライアントのシークレット     |
| REDIRECT_URL              | Cognito のアプリケーションクライアントのコールバック URL |

:::message
OAuth2 Proxy の設定ファイルをテンプレートで作成しているのは、コンテナ起動時にファイル内の環境変数を展開するためです。
環境変数の展開は、`envsubst` コマンドで行います。
:::

詳細な設定項目は次を参照してください。

https://oauth2-proxy.github.io/oauth2-proxy/docs/next/configuration/overview

## Supervisor の設定ファイルを作成

[Supervisor](http://supervisord.org/) は複数のプロセスを制御してくれるツールです。
今回は、コンテナ起動時に Nginx と OAuth2 Proxy の二つのプロセスを立ち上げますので、これらのプロセスをデーモン化する用途で使用します。

:::message
本来、1 コンテナ 1 プロセスが原則ですが大目に見てください🙇
:::

`server-config` 配下に Supervisor の設定ファイル（`supervisord.conf`）を新規に作成します。
中身は次のようにします。

```ini:server-config/supervisord.conf
[supervisord]
nodaemon=true

[program:nginx]
command=nginx -g "daemon off;"
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
startretries=10
autorestart=true

[program:oauth2_proxy]
command=/usr/local/bin/oauth2-proxy --config /etc/oauth2proxy.conf
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
startretries=10
autorestart=true
```

## コンテナ起動時のスタートアップスクリプトを作成

コンテナ起動時に実行するスタートアップスクリプト（`startup.sh`）を新規に作成します。
中身は次のようにします。

```bash:startup.sh
#!/bin/bash
envsubst < /etc/oauth2proxy.conf.template > /etc/oauth2proxy.conf
/usr/bin/supervisord -c /etc/supervisor/conf.d/supervisord.conf
```

:::message
コンテナ起動時に環境変数を渡し、`envsubst` コマンドで OAuth2 Proxy のテンプレート内の環境変数を展開することで設定ファイルを生成しています。
また、`supervisord` コマンドで Nginx と OAuth2 Proxy のプロセスを起動しています。
:::

## Dockerfile を編集

Dockerfile を次のように編集します。

```diff dockerfile:Dockerfile
FROM node:lts-alpine3.16 AS build
COPY ./my-website ./
RUN npm run build

FROM nginx:1.23.2

+ ARG CPU_ARCH=amd64
+ ARG OAUTH2_PROXY_VERSION=7.4.0

+ RUN set -x && \
+     apt update && \
+     apt install --no-install-recommends --no-install-suggests -y \
+     supervisor wget gettext-base

+ RUN wget https://github.com/oauth2-proxy/oauth2-proxy/releases/download/v${OAUTH2_PROXY_VERSION}/oauth2-proxy-v${OAUTH2_PROXY_VERSION}.linux-${CPU_ARCH}.tar.gz && \
+     tar xf oauth2-proxy-v${OAUTH2_PROXY_VERSION}.linux-${CPU_ARCH}.tar.gz -C /usr/local/bin/ --strip-components 1 && \
+     rm oauth2-proxy-v${OAUTH2_PROXY_VERSION}.linux-${CPU_ARCH}.tar.gz

COPY ./server-config/nginx.conf /etc/nginx/nginx.conf
+ COPY ./server-config/oauth2proxy.conf.template /etc/oauth2proxy.conf.template
+ COPY ./server-config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
+ COPY ./startup.sh /startup.sh
COPY --from=build /build /usr/share/nginx/html

- CMD [ "nginx", "-g", "daemon off;" ]
+ CMD [ "sh", "startup.sh" ]
```

アーキテクチャやバージョンを指定して OAuth2 Proxy をインストールしています。

```dockerfile:OAuth2 Proxy のインストール情報
ARG CPU_ARCH=amd64
ARG OAUTH2_PROXY_VERSION=7.4.0
・・・
RUN wget https://github.com/oauth2-proxy/oauth2-proxy/releases/download/v${OAUTH2_PROXY_VERSION}/oauth2-proxy-v${OAUTH2_PROXY_VERSION}.linux-${CPU_ARCH}.tar.gz && \
    tar xf oauth2-proxy-v${OAUTH2_PROXY_VERSION}.linux-${CPU_ARCH}.tar.gz -C /usr/local/bin/ --strip-components 1 && \
    rm oauth2-proxy-v${OAUTH2_PROXY_VERSION}.linux-${CPU_ARCH}.tar.gz
```

apt のインストールを高速化するようなオマジナイなんかがついていますが、必要なツールをインストールしているだけです。

* `supervisor` は、Nginx および OAuth2 Proxy をデーモン化するため
* `wget` は、OAuth2 Proxy のダウンロードで使用するため
* `gettext-base` は、テンプレートファイルの環境変数を展開する `envsubst` を使えるようにするため

```dockerfile:必要なツール群のインストール
RUN set -x && \
    apt update && \
    apt install --no-install-recommends --no-install-suggests -y \
    supervisor wget gettext-base
```

追加した設定ファイルおよびコンテナ起動時に実行するスタートアップスクリプトの COPY を追加しています。

```dockerfile:追加した設定ファイルおよびスタートアップスクリプトのコピー
COPY ./server-config/oauth2proxy.conf.template /etc/oauth2proxy.conf.template
COPY ./server-config/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./startup.sh /startup.sh
```

コンテナ起動時にスタートアップスクリプトを実行するようにします。

```dockerfile:コンテナ起動時にスタートアップスクリプトを実行
CMD [ "sh", "startup.sh" ]
```

# 認証機能の確認をしてみよう！

長かったですが、いよいよ認証機能の確認をしてみます。
Docker ビルドして、コンテナを起動してみましょう！

## Docker ビルド

それでは、Docker ビルドをしてみましょう。

```bash
DOCKER_BUILDKIT=1 docker build --progress=plain --no-cache -t my-website:1.0.0 .
```

## コンテナの起動

コンテナを起動してみます。
コンテナ起動時に Cognito のクレデンシャル情報等の環境変数の値を渡すため、`--env` オプションを指定します。

```bash
docker run --rm -p 80:80 \
  --env COOKIE_SECRET=`python3 -c 'import os,base64; print(base64.urlsafe_b64encode(os.urandom(32)).decode())'` \
  --env USER_POOL_ID=xxxxxxxxxxxxxx_xxxxxxxxx \
  --env APPLICATION_CLIENT_ID=xxxxxxxxxxxxxxxxxxxxxxxxxx \
  --env APPLICATION_CLIENT_SECRET=*************************************************** \
  --env REDIRECT_URL=http://localhost/oauth2/callback \
  my-website:1.0.0
```

:::message
Cognito のクレデンシャル情報は次の画面に表示されている値を設定します。

![USER_POOL_ID の値](/images/nginx-with-oauth2-proxy/user-pool-id.png =450x)
*USER_POOL_ID の値*

![APPLICATION_CLIENT_XXX の値](/images/nginx-with-oauth2-proxy/application-client-info.png =450x)
*APPLICATION_CLIENT_XXX の値*
:::

:::message
次のような出力であれば正常に起動しているはずです。

* `INFO success: nginx entered RUNNING state`
* `INFO success: oauth2_proxy entered RUNNING state`

```
2022-12-08 16:56:19,760 CRIT Supervisor is running as root.  Privileges were not dropped because no user is specified in the config file.  If you intend to run as root, you can set user=root in the config file to avoid this message.
2022-12-08 16:56:19,761 INFO supervisord started with pid 8
2022-12-08 16:56:20,764 INFO spawned: 'nginx' with pid 9
2022-12-08 16:56:20,766 INFO spawned: 'oauth2_proxy' with pid 10
[2022/12/08 16:56:20] [provider.go:55] Performing OIDC Discovery...
[2022/12/08 16:56:20] [oauthproxy.go:162] OAuthProxy configured for OpenID Connect Client ID: 1pf669r4moerc42vd2i1nat6de
[2022/12/08 16:56:20] [oauthproxy.go:168] Cookie settings: name:_oauth2_proxy secure(https):false httponly:true expiry:168h0m0s domains: path:/ samesite: refresh:disabled
2022-12-08 16:56:21,962 INFO success: nginx entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
2022-12-08 16:56:21,962 INFO success: oauth2_proxy entered RUNNING state, process has stayed up for > than 1 seconds (startsecs)
```
:::

`http://localhost` にアクセスして、認証画面が表示されれば成功です！
作成したユーザで早速ログインしてみましょう✨

![ログインしている様子](/images/nginx-with-oauth2-proxy/oauth2-proxy-login.gif)
*ログインしている様子*

:::message
追加手順は（簡単なので）割愛しましたが、予め Cognito のユーザプールに追加したユーザ（`test@test.com`）でログインしています。
ご自身でログインしてみる際は、ユーザの作成をお忘れなく。
:::

# 最後に

いかがだったでしょうか？
これで信頼できるお友達だけをドキュサウルスに会わせることができました。
一安心ですね😌

えっ？ローカル環境だから、同一ネットワーク内の人じゃないとドキュサウルスに会えないじゃないかって？
ですよね、それでは次回は認証付きのドキュメントサイトをインターネットに公開してみましょう！
そうすればドキュサウルスも一躍有名になれますね😎✨

ちなみに、今回は静的 Web サイトに認証機能をつけただけなのであまり面白味はないかもしれませんが、コンテナ化しているので当然 K8s 上で動く場合であっても OAuth2 Proxy が使えて、手軽に認証機能を組み込むことができます。
やっぱりコンテナは素晴らしいですね。

それではまた次回のアドベントカレンダー（勝手にシリーズ化）でお会いしましょう😉
