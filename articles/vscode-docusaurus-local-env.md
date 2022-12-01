---
title: "VSCode + Docusaurus で最高のドキュメント作成環境を構築する"
emoji: "🦖"
type: "tech"
topics: ["vscode", "docusaurus", "docker", "npm", "markdown"]
published: false
# published_at: 2022-12-03 23:59
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

## コンテナに接続

## Docusaurus をインストールしてデフォルトページを表示

# 最後に

とってもかわいいドキュサウルスに会えましたね✨
今回構築した環境でドキュサウルスといっぱい遊びましょう🦖
