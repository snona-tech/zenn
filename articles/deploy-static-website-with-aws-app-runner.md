---
title: "静的 Web サイトに認証をつけて AWS App Runner でデプロイしてみる"
emoji: "🛰"
type: "tech"
topics: ["docusaurus", "docker", "aws", "apprunner", "ecr"]
published: true
published_at: 2022-12-11 00:00
---

:::message
この記事は、🦌🛷[**カサレアル Advent Calender 2022**](https://qiita.com/advent-calendar/2022/casareal)🎄の **11** 日目のエントリです。
:::

# TL;DR

* [AWS App Runner](https://aws.amazon.com/jp/apprunner/) でコンテ化した認証機能付き静的 Web サイトをデプロイする
* AWS App Runner は設定が非常に簡単で、オートスケーリングやメトリクス監視などもデフォルト設定で扱える

本記事のソースコードは👇で公開しています。

https://github.com/casa-snona/hello-docusaurus

# はじめに

[前回のアドベントカレンダーの記事](https://zenn.dev/casa_snona/articles/nginx-with-oauth2-proxy)までで、Cognito と連携した認証付きのドキュメントサイトをコンテナ化しました。
いよいよ、[AWS App Runner](https://aws.amazon.com/jp/apprunner/) を使用して、ドキュサウルスをインターネットデビューさせてあげましょう✨

全体像は👇のイメージです。

![全体像](/images/deploy-static-website-with-aws-app-runner/architecture.drawio.png =500x)
*全体像*

:::message
上記のイメージでは、クライアントから App Runner に直接アクセスしているように見えますが、実際は App Runner 側で LB を用意してくれています。
ただ、こちら側からは App Runner の VPC や LB は隠蔽されているので、どのようなリソースが構築されているのかは見えません。
:::

# AWS App Runner って？

AWS App Runner は、コンテナ化された Web アプリケーションや API サービスをデプロイできる AWS のフルマネージドなサービスです。

簡単な設定で細かいインフラなどを意識しなくてもコンテナをデプロイできます！
個人的には非常に素晴らしいサービスだと思っています。

若干古いですが、👇の動画で App Runner の概要やデモンステレーションの説明を非常に丁寧にしてくれているので、まずはこちらをご覧いただくことをオススメします（他力本願です💦）

https://www.youtube.com/watch?v=8Tmf5J2hXT8

# Let's デプロイ！

それでは早速、デプロイをしていきましょう。

## 前提条件

以降の手順は、シリーズ化している記事で構築した環境を前提としています。
構築した環境なんて知らないよという方は、[GitHub で公開しているソースコード](https://github.com/casa-snona/hello-docusaurus)をクローンするか、前回までの記事をご覧ください🙏

https://zenn.dev/casa_snona/articles/vscode-docusaurus-local-env
https://zenn.dev/casa_snona/articles/docusaurus-is-good-here
https://zenn.dev/casa_snona/articles/containerize-docusaurus
https://zenn.dev/casa_snona/articles/nginx-with-oauth2-proxy

## AWS CLI を使えるようにする

後工程で AWS CLI が必要になるので、インストールしておきます。
開発環境のコンテナ内にインストールしてもよいのですが、コンテナイメージを再ビルドした際に使えなくなってしまいます。
開発環境を統一する意味でも、再現性があった方がよいです。

Dev Containers では、AWS CLI などのツールを簡単に使えるようにする仕組みがあります。

AWS CLI を使えるようにするには、`devcontainer.json` を編集します。

```diff json:.devcontainer/devcontainer.json
{
	"name": "Ubuntu",
	"image": "mcr.microsoft.com/devcontainers/base:jammy",
	"features": {
		"ghcr.io/devcontainers/features/docker-in-docker:2": {},
		"ghcr.io/devcontainers/features/git:1": {},
		"ghcr.io/devcontainers/features/node:1": {},
+		"ghcr.io/devcontainers/features/aws-cli:1": {}
	}
}
```

`"ghcr.io/devcontainers/features/aws-cli:1": {}` を一行追加して、イメージを再ビルドします。
イメージの再ビルドは、VSCode の左下の緑の `><` マークをクリックしてパネルを開き、`Rebuild Container` を選択します。

![パネルを開く](/images/deploy-static-website-with-aws-app-runner/open-panel.png)
*パネルを開く*


![Rebuild Container を選択](/images/deploy-static-website-with-aws-app-runner/rebuild-container.png)
*Rebuild Container を選択*

再ビルドが成功すると、次のように `aws` コマンドが使用できるようになっているはずです。

```
vscode ➜ /workspaces/hello-docusaurus $ aws --version
aws-cli/2.9.6 Python/3.9.11 Linux/5.15.79.1-microsoft-standard-WSL2 exe/x86_64.ubuntu.22 prompt/off
```

インストールコマンドをいちいち叩かなくても設定済みのイメージをビルドしてくれるなんて、本当に Dev Containers は素晴らしい拡張機能ですね！
この拡張機能が唯一無二の存在なので、筆者の開発環境はほぼ VSCode 一択になっています。

## イメージを ECR リポジトリに登録

[Amazon Elastic Container Registry (Amazon ECR)](https://docs.aws.amazon.com/ja_jp/AmazonECR/latest/userguide/what-is-ecr.html) は、AWS マネージドコンテナイメージレジストリサービスです。
ECR に今回作成した Docker イメージを保存しておきます。

今回は、`adventcalendar2022/my-website` という**プライベート**リポジトリを作成しました。

![作成した ECR リポジトリ](/images/deploy-static-website-with-aws-app-runner/ecr-repo.png)
*作成した ECR リポジトリ*

:::message
以降のコマンドは、AWS のクレデンシャル情報を環境変数に設定済みであることを前提としています。
予め、👇の設定を行ってください。
```bash
export AWS_ACCOUNT=AWSアカウントID
export AWS_ACCESS_KEY_ID=アクセスキーID
export AWS_SECRET_ACCESS_KEY=シークレットアクセスキー
```
:::

登録するイメージをビルドします。
タグは、作成したリポジトリ名をつけておきます。

```bash
DOCKER_BUILDKIT=1 docker build \
  --progress=plain \
  --no-cache \
  -t ${AWS_ACCOUNT}.dkr.ecr.ap-northeast-1.amazonaws.com/adventcalendar2022/my-website:latest .
```

ECR のリポジトリに接続できるように Docker ログインをします。

```bash
aws ecr get-login-password --region ap-northeast-1 | docker login --username AWS --password-stdin ${AWS_ACCOUNT}.dkr.ecr.ap-northeast-1.amazonaws.com
```

ログインに成功したら、イメージを Push します。

```bash
docker push ${AWS_ACCOUNT}.dkr.ecr.ap-northeast-1.amazonaws.com/adventcalendar2022/my-website:latest
```

## App Runner のサービスを作成

App Runner のサービスを作成していきます。

デプロイ対象の設定をします。
今回は ECR（プライベート）を選択しています。
コンテナイメージの URI は、タグも含めて指定します。（`latest` であっても省略できません）

![デプロイ対象の設定](/images/deploy-static-website-with-aws-app-runner/app-runner-setting-1.png =450x)
*デプロイ対象の設定*

```:コンテナイメージの URI
AWSアカウントID.dkr.ecr.ap-northeast-1.amazonaws.com/adventcalendar2022/my-website:latest
```

デプロイ設定で、手動で（任意のタイミングで）デプロイするか、自動で（ECR への Push トリガー）デプロイするかを選択できます。
今回は自動を選択しています。

![デプロイ設定](/images/deploy-static-website-with-aws-app-runner/app-runner-setting-2.png =450x)
*デプロイ設定*

:::message alert
手動デプロイを選択したとしても、**サービス作成時は必ずデプロイが実施されます**。
そのため、ECR に対象のイメージが存在しないとサービス作成に失敗します。
また、サービス作成に失敗すると、その後の設定変更や再デプロイができなくなるため、実質サービスを再作成するしかありません。
:::

環境変数を設定します。
設定値については、[前回の記事](https://zenn.dev/casa_snona/articles/nginx-with-oauth2-proxy#%E3%82%B3%E3%83%B3%E3%83%86%E3%83%8A%E3%81%AE%E8%B5%B7%E5%8B%95)をご確認ください。
リダイレクト URL については、本来は App Runner のドメインを指定しますが、サービス作成後でないとわからないため、一旦仮に localhost を設定しておきます。
ポートはデフォルトで `8080` が入力されていますが、今回のコンテナは `80` で Listen するので変更しておきます。

![環境変数の設定](/images/deploy-static-website-with-aws-app-runner/app-runner-setting-3.png =450x)
*環境変数の設定*

その他はデフォルト設定にしています。
サービスの作成とデプロイを開始すると、👇の画面に遷移します。
デフォルトドメインの URL で Web サイトが公開されます。（カスタムドメインも使用可能です）

![サービス作成とデプロイ](/images/deploy-static-website-with-aws-app-runner/app-runner-setting-4.png =450x)
*サービス作成とデプロイ*

正常にデプロイが完了すると次のように表示されます。（10 分ほどかかります）

![正常にデプロイが完了](/images/deploy-static-website-with-aws-app-runner/app-runner-setting-5.png =450x)
*正常にデプロイが完了*

:::message
今の状態でデフォルトドメインの URL にアクセスすると、OAuth2 Proxy の画面が表示されますが、認証画面へのリダイレクトの向き先が localhost になります。（環境変数の値によって）
リダイレクトの向き先を変更するため、環境変数の値を更新します。
:::

## App Runner のサービスを更新

作成した App Runner のサービス詳細画面で、「設定」タブ ＞ 「サービスを設定」の編集を開きます。

![環境変数の変更](/images/deploy-static-website-with-aws-app-runner/app-runner-update-1.png =450x)
*環境変数の変更*

リダイレクト URL の環境変数（`REDIRECT_URL`）の値を変更します。
`http://localhost` の部分をデフォルトドメインに変更します。

```diff:リダイレクト URL を変更
- http://localhost/oauth2/callback
+ https://**********.ap-northeast-1.awsapprunner.com/oauth2/callback
```

設定を変更するとすぐに更新が始まります。

![サービス更新](/images/deploy-static-website-with-aws-app-runner/app-runner-update-2.png =450x)
*サービス更新*

:::message
App Runner では、安全なブルーグリーンデプロイが行われるため、サービス更新中も公開している Web サイトには継続してアクセスできます。
:::

更新が完了すると、次のように表示されます。

![サービス更新完了](/images/deploy-static-website-with-aws-app-runner/app-runner-update-3.png =450x)
*サービス更新完了*

## Cognito のコールバック URL の追加

Cognito の認証後、ユーザをリダイレクトするコールバック URL を App Runner でデプロイした Web サイトの URL にする必要があります。
今回作成したサービスのデフォルトドメインのコールバック URL を変更します。

![許可するコールバック URL を追加](/images/deploy-static-website-with-aws-app-runner/cognito-add-callback-url.png)
*許可するコールバック URL を追加*

:::message
`http://localhost/oauth2/callback` を残しておけば、ローカルでコンテナを起動した際も認証画面の確認が可能です。
:::

## アクセス確認

これですべての準備が整いました！
App Runner のデフォルトドメインでアクセスしてみてください。

```:デフォルトドメイン
https://**********.ap-northeast-1.awsapprunner.com
```

![App Runner でデプロイしたドキュメントサイトにアクセス](/images/deploy-static-website-with-aws-app-runner/app-runner-service-login.gif)
*App Runner でデプロイしたドキュメントサイトにアクセス*

また、App Runner の「メトリクス」タブからリクエスト数やレスポンスコード、レイテンシー、アクティブなインスタンス数 などが確認できたりします。
自前で用意しなくていいので楽ですね。

![メトリクスの取得](/images/deploy-static-website-with-aws-app-runner/app-runner-metrics.png)
*メトリクスの取得*

VPCや VM、 K8s、ロードバランサーも自前で用意せずにここまでできました🌟
本当にすばらしいサービスです🛰

これで無事にドキュサウルスを世界の舞台に立たせることができましたね🌐🦖

# 最後に

いかがだったでしょうか？
App Runner の手軽さに驚愕していただけたのではないでしょうか。

他国に比べて日本ではまだまだコンテナ化が進んでいない現状です。
ひと度コンテナ化してしまえば、運用コストを軽減しつつ注力すべき開発に専念しやすくなります。
ぜひとも、この記事の投稿が今後の皆さまのお役に立てれば幸いです。
ドキュサウルスもきっとそう望んでくれていると思います！

さて、ドキュサウルスのゴリ押しにはそろそろ飽きてきたかと思いますので、これにて弊社アドベントカレンダーの勝手にシリーズ化は幕引きです。
弊社のアドベントカレンダーはまだまだ更新されていきますので、どうか引き続きお楽しみください。

最後までお読みいただき、ありがとうございました🙇‍♂
それではまたどこかでお会いしましょう😎✨
