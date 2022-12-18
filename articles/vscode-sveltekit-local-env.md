---
title: "フロントエンドを知らない一般人がトレンディなフレームワーク SvelteKit を眺める - ローカル環境構築編 -"
emoji: "🔰"
type: "tech"
topics: ["vscode", "devcontainer", "frontend", "svelte", "sveltekit"]
published: true
published_at: 2022-12-20 00:00
---

:::message
この記事は、🦌🛷[**カサレアル Advent Calender 2022**](https://qiita.com/advent-calendar/2022/casareal)🎄の **20** 日目のエントリです。
:::

# TL;DR

* VSCode の Dev Containers で [SvelteKit](https://kit.svelte.jp/) のローカル環境を構築
* トレンディな SvelteKit の Welcom ページをブラウザで表示
* Welcom ページで [Wordle](https://www.nytimes.com/games/wordle/index.html) ライクなゲームで遊べる

# はじめに

世の中のトレンドは年々サイクルが早くなっているように感じています。
筆者が従事している[クラウドネイティブ推進支援サービス](https://www.casareal.co.jp/cs/)という事業は、その名の通りクラウドネイティブという広くそして変革の激しい分野です。
その一方で、今回触れるフロントエンドの分野に関しては、それ以上に早いサイクルで流行り廃りが移り変わっている気がしています。

もともと、HTML/CSS + JavaScript で開発されていた Web アプリケーションですが、JavaScript をより書きやすくした jQuery が登場したものの、今ではほとんど使われなくなってきているとか、JavaScript → TypeScript への移行であったり、React や Vue.js などの様々なフレームワーク（あっ、人気は落ち気味？ですが Angular もありますよね）が登場したりと、技術の幅が広がって開発者１人当たりのキャッチアップが大変なくらいに目まぐるしくなっています。

さらには、各種フレームワークに対して、より使いやすくするためのフレームワーク（React であれば Next.js、Vue.js なら Nuxt.js など）のメタフレームワーク（と言うらしい？）なんかも登場し始めていますよね。

フロントエンド初心者には何がなんだかですよ本当に🤯
と言いつつ、筆者はフロントエンドの実務経験もなければ初心者になるつもりもないです😓
が、この業界はいつどこでどんな知識が役に立つかわからないので、ほんの少しだけ入門してみてもいいかなぁという気持ちでやり始めましたというのが今回の記事のテーマです。

# SvelteKit って？

SvelteKit は、JavaScript のフレームワークである Svelte をより開発しやすくしたフレームワークです。
位置づけとしては、React で言う Next.js、Vue.js なら Nuxt.js と同じです。

先日（2022年12月中旬くらい）、バージョン `1.0.0` が正式にリリースされたことで話題になっているフレームワークなので、ご存じの方も多いかもしれません。

SvelteKit の[公式ドキュメント](https://kit.svelte.jp/docs/introduction)は日本語化されているので、英語に抵抗がある開発者にはありがたいですね。

https://kit.svelte.jp/

# とりあえずローカルで

ひとまず、Hello World 的なことをしたいので、ローカルの開発環境をつくりましょう。
VSCode の拡張機能である Dev Containers を使います。
Dev Containers を使ったことがない方は、👇の記事で使い方を軽く紹介しているのでご覧ください。

https://zenn.dev/casa_snona/articles/vscode-docusaurus-local-env#vscode-%2B-docusaurus-%E3%81%AE%E6%A7%8B%E6%88%90%E5%86%85%E5%AE%B9

今回は、`Node.js & TypeScript` のイメージに、Svelte 開発用の拡張機能（[Svelte for VS Code](https://marketplace.visualstudio.com/items?itemName=svelte.svelte-vscode)）と、ESLint の拡張機能を入れました。
`devcontainer.json` は👇のようにしました。

```json:.devcontainer/devcontainer.json
{
	"name": "Node.js & TypeScript",
	"image": "mcr.microsoft.com/devcontainers/typescript-node:0-18",
	"features": {
		"ghcr.io/devcontainers/features/git:1": {}
	},
	"extensions": [
		"svelte.svelte-vscode",
		"dbaeumer.vscode-eslint"
	]
}
```

# SvelteKit の Welcome ページを表示する

Dev Containers でコンテナに接続できたら、SvelteKit のプロジェクトを作成します。
[SvelteKit の公式ページ](https://kit.svelte.jp/)を見ると、`自分で試す場合` という項目に初期導入時のコマンドが記載されていますね！

![](/images/vscode-sveltekit-local-env/svelte-create-command.png)

表示されているコマンドをそのまま実行してみます。

```bash:SvelteKit プロジェクト初期化コマンド
npm create svelte@latest my-app
cd my-app
npm install
npm run dev -- --open
```

一行目のコマンドでいろいろと質問されますが、筆者は👇のようにしてデモアプリケーションを作成するようにしました。

![](/images/vscode-sveltekit-local-env/svelte-project.png =450x)

コマンドが正常に実行されると、ブラウザが起動して Welcome ページが表示されます。

![](/images/vscode-sveltekit-local-env/svelte-welcom-page.png =450x)

いやぁ、SvelteKit ってとても簡単ですね！（たぶん他のフレームワークもだいたいこんな感じだろうけど）

この Welcome ページのいいところは、Sverdle という [Wordle](https://www.nytimes.com/games/wordle/index.html) のクローンで遊べるところですね。
ルールは簡単で、5 文字の英単語を入力して、部分的に正解したスペルのヒントから一定回数以内に単語を当てるだけです。
👇のような感じです。

![](/images/vscode-sveltekit-local-env/svelte-game.png =300x)

Wordle には一時期ハマっていて、iOS 版のアプリをやっていましたが、無料版では一日に挑戦できる回数が制限されていたりしていてヤキモキしていました🤔
でも、この Welcome ページをホスティングしておけば、いつでも制限なく Sverdle で遊べますね✨

# 最後に

フロントエンドについてよく知らない筆者が SvelteKit に挑戦するために、ローカルの環境を整備しました。
駆け出しフロントエンドエンジニアのお役に立てれば幸いです！

えっ？、とくに何もしてないじゃないかって？
えぇ...まぁ、その通りなんですが。。。
初心者なんで責めないでください🙏

次回は SvelteKit のルーティング機能をほんの少し触ってみようかなぁ。
