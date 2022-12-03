---
title: "VSCode + Docusaurus で最高のドキュメント作成環境を構築する"
emoji: "🦖"
type: "tech"
topics: ["vscode", "docusaurus", "docker", "npm", "markdown"]
published: true
published_at: 2022-12-04 00:00
---

:::message
この記事は、🦌🛷[**カサレアル Advent Calender 2022**](https://qiita.com/advent-calendar/2022/casareal)🎄の **4** 日目のエントリです。
:::

# TL;DR

* VSCode で Docusaurus のローカル環境（Docker コンテナ）を構築する
* 開発環境は、[Dev Containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers)（VSCode の拡張機能）を使うと便利
* [Docusaurus](https://docusaurus.io/) は、Markdown(+[MDX](https://mdxjs.com/)) ベースでドキュメントやブログの静的 Web サイトを作成できるツール

# はじめに

みなさんは、ドキュメントを作成する時、何を使っていますか？
Microsoft Office や Google ドキュメントといったオフィススイートを使用していますか？
それとも、Confluence のような Web ベースの Wiki でしょうか。
GitHub リポジトリで Markdown を書いている方もいるかもしれませんね。

ドキュメントではないですが、弊社が提供している[クラウドネイティブ道場](https://www.casareal.co.jp/cs/service/cloudnativedojo)のチュートリアルでは、[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) を使用していたりします。
MkDocs は Python ベースで開発されている人気のドキュメントサイトジェネレータです。

今回は、Mkdocs よりもよさそうな [Docusaurus](https://docusaurus.io/) のローカル環境を構築してみます。
個人的にいいなと思う Docusaurus のポイントについては、別日のアドベントカレンダー内で投稿（勝手にシリーズ化）しようと思いますので、とりあえずは Docusaurus のデフォルトページの表示までを確認してみます。

# Docusaurus って？

Meta（旧：Facebook）が開発している Markdown(+[MDX](https://mdxjs.com/)) ベースでドキュメントやブログの静的 Web サイトを作成できるツールです。
ドキュサウルスかわいいですね💓

![docusaurus-icon](/images/vscode-docusaurus-local-env/docusaurus-icon.png =150x)
*キーボードで遊んでるドキュサウルス*

# VSCode + Docusaurus の構成内容

早くドキュサウルスに会いたいですが、その前に今回構築するローカル環境の構成内容について整理しておきます。

ローカル環境は、Dev Containers（VSCode の拡張機能：`ms-vscode-remote.remote-containers`）を使用して構築します。
大まかな流れは次の通りです。

1. Dev Container の設定ファイルを作成（プロンプトに従って作成）
2. 作成した設定ファイルに基づいて Docker イメージをビルドしてコンテナに接続（Dev Container が自動でやってくれます）
3. Docusaurus のインストールと初期化

![devcontainers](/images/vscode-docusaurus-local-env/devcontainers.drawio.png =500x)
*環境のイメージ*

Docker コマンドを自分で叩かなくてもイメージが作れて接続も簡単なので、初めて知ったときは感動しました。
設定ファイルを Git リポジトリ等で共有すれば誰でも同じ環境がつくれるので、今回に限らず非常に便利です。

## 前提条件

次の項目については実施済みであることを前提に構築します。

* VSCode がインストール済みであること
* Dev Containers（VSCode の拡張機能）がインストール済みであること
* Docker が使えること（Docker デーモンが起動していること）← Docker Desktop をインストールするのが一番楽です

:::message
基本は VSCode と Docker さえ使えれば OK です。
筆者の環境は👇のように若干尖った環境[^1]になっていますが、真似しなくても大丈夫です。

|    環境    | 補足                                                                                                                                                                                                                    |
| :--------: | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Windows 11 | VSCode と Docker が利用可能                                                                                                                                                                                             |
|   VSCode   | Windows 11 上にインストール                                                                                                                                                                                             |
|   Docker   | [WSL](https://learn.microsoft.com/ja-jp/windows/wsl/)（Windows 上で Linux カーネルを動かす仕組み）の Ubuntu ディストリビューション上にネイティブの Docker を起動<br>Dev Container で WSL の Docker を使用するように設定 |
:::

[^1]: 企業の規模等、特定の条件下では Docker Desktop は有料なので、有償ライセンスを回避するために WSL 上に Docker をインストールしています。

# ドキュサウルスに会いに行こう！

それでは環境の構築をはじめましょう。

## Dev Container の設定ファイル作成

まずは、空のフォルダを VSCode で開きましょう。
フォルダ名は任意のもので OK です。今回は、`hello-docusaurus` にしました。

VSCode でフォルダを開いたら、左下の緑の `><` マークをクリックしてパネルを開きます。

![パネルを開く](/images/vscode-docusaurus-local-env/open-panel.png)
*パネルを開く*

開いたパネルから、`Add Dev Container Configuration Files` を選択します。

![Dev Container の設定ファイルを追加](/images/vscode-docusaurus-local-env/add-devcontainer-config.png)
*Dev Container の設定ファイルを追加*

以下をチェックして、OK を選択します。

* [x] image: Ubuntu(Jammy)
* Features
  * [x] Git（Git 管理したい場合は選択）
  * [x] Node.js（Docusaurus で使用するので必須）
  * [x] Docker in Docker（アドベントカレンダーの別記事にて、コンテナ化するときに必須）

![イメージとインストールする機能を選択](/images/vscode-docusaurus-local-env/devcontainers-config-setting.gif)
*イメージとインストールする機能を選択*

すると、次のような JSON ファイルが生成されます。（コメント部分は省略）

```json:.devcontainer/devcontainer.json
{
	"name": "Ubuntu",
	"image": "mcr.microsoft.com/devcontainers/base:jammy",
	"features": {
		"ghcr.io/devcontainers/features/docker-in-docker:2": {},
		"ghcr.io/devcontainers/features/git:1": {},
		"ghcr.io/devcontainers/features/node:1": {}
	}
}
```

:::message
手動でこの JSON ファイルを作成しても OK です。
Features で選択したツール群がコンテナで使用できるようになります。
:::

## コンテナに接続

左下の緑の `><` マークをクリックしてパネルを開きます。
開いたパネルから、`Reopen in Container` を選択すると、イメージのビルドが行われ、起動したコンテナに接続します。

![イメージのビルドとコンテナの起動](/images/vscode-docusaurus-local-env/reopen-in-container.png)
*イメージのビルドとコンテナの起動*

:::message
２回目以降の接続は、ビルド済みのイメージからコンテナに接続します。
:::

正常にコンテナが起動すると、自動でコンテナに接続されます。
ターミナルで各種インストールされたツール群が使用できることが確認できます。

![コンテナへの接続](/images/vscode-docusaurus-local-env/container-connection.png)
*コンテナへの接続*

## Docusaurus をインストールしてデフォルトページを表示

[Docusaurus 公式のインストールガイド](https://docusaurus.io/docs/installation)に従ってインストールしてみます。

```bash
npx create-docusaurus@latest my-website classic
```

コマンドを実行すると、確認メッセージが出るので、`y` を入力して進みます。

```
vscode ➜ /workspaces/hello-docusaurus $ npx create-docusaurus@latest my-website classic
Need to install the following packages:
  create-docusaurus@2.2.0
Ok to proceed? (y)
```
:::message
`npm WARN` なんかが色々出ますが、ターミナル出力の中盤あたりに `[SUCCESS] Created my-website.` が出力されていれば問題ありません。
:::

それでは、ディレクトリを移動して、ドキュサウルスに会いに行きましょう！
次のコマンドを実行してしばらくすると、VSCode で勝手に Docusaurus のデフォルトポート（`3000` 番）にポートフォワードしてくれて、ローカルのブラウザが開きます。

```bash
cd my-website
npm start
```

![Docusaurus デフォルトページ](/images/vscode-docusaurus-local-env/docusaurus-default-page.png)
*Docusaurus デフォルトページ*

やったね🙌

# 最後に

とってもかわいいドキュサウルスに会えましたね✨
今回構築した環境でドキュサウルスといっぱい遊びましょう🦖
