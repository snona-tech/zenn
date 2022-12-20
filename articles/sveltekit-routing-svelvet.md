---
title: "フロントエンドを知らない一般人がトレンディなフレームワーク SvelteKit を眺める - ルーティング編 -"
emoji: "🛁"
type: "tech"
topics: ["vscode", "devcontainer", "frontend", "svelte", "sveltekit"]
published: true
published_at: 2022-12-22 00:00
---

:::message
この記事は、🦌🛷[**カサレアル Advent Calender 2022**](https://qiita.com/advent-calendar/2022/casareal)🎄の **22** 日目のエントリです。
:::

# TL;DR

* [SvelteKit](https://kit.svelte.jp/) のルーティング機能を試す
* いい感じのフローをつくれる [Svelvet](https://www.svelvet.io/) を使ってみる

# はじめに

[前回の記事](https://zenn.dev/casa_snona/articles/vscode-sveltekit-local-env)では、SvelteKit のデモアプリを表示させてみました。
それだけでは味気ないので、SvelteKit のルーティング機能を使ってみようと思います。（一般人なのでほんの触りだけですが...）
また、テキストだけ変えて表示するのも退屈かもしれないので、いい感じのフローを作成できる [Svelvet](https://www.svelvet.io/) を導入してみたいと思います✨

# デモアプリにメニューを追加してみよう

前回の記事の環境を前提として進めます。
まだご覧頂いていない方は、先に👇の記事をみてみてください。

https://zenn.dev/casa_snona/articles/vscode-sveltekit-local-env

それでは早速、[SvelteKit のルーティング機能](https://kit.svelte.jp/docs/routing)を使ってデモアプリのメニューを追加してみます。
メニュー名は後ほど登場する `Svelvet` としておきます。

## ヘッダーメニューの HTML を変更

`Header.svelte` で、ヘッダーメニューの項目を表示している箇所を見つけて、`Svelvet` メニューを次のように追加してみます。

```diff jsx:my-app/src/routes/Header.svelte
    <li aria-current={$page.url.pathname === '/' ? 'page' : undefined}>
        <a href="/">Home</a>
    </li>
    <li aria-current={$page.url.pathname === '/about' ? 'page' : undefined}>
        <a href="/about">About</a>
    </li>
    <li aria-current={$page.url.pathname.startsWith('/sverdle') ? 'page' : undefined}>
        <a href="/sverdle">Sverdle</a>
    </li>
+   <li aria-current={$page.url.pathname === '/svelvet' ? 'page' : undefined}>
+       <a href="/svelvet">Svelvet</a>
+   </li>
```

上記追加して、デモアプリを表示してみると次のようにメニューが追加されているかと思います。

:::message
デモアプリは👇のコマンドで表示できます。
```bash
npm run dev -- --open
```

:::

![](/images/sveltekit-routing-svelvet/header-menue.png)

ここで、メニューをクリックするとページが見つからないよと言われてしまいます。
表示したい `/svelvet` へのルーティングを設定する必要があります。

![](/images/sveltekit-routing-svelvet/menue-not-found.png)

## ルーティングを設定する

[SvelteKit のルーティング](https://kit.svelte.jp/docs/routing)は、フォルダ階層やフォルダ名で設定することができます。
つまり、フォルダ構成を見ればどのような URL でアクセスできるのかがわかります。（Nuxt.js も同じようなスタイルでルーティングを行うようです）

では、ルーティングを設定してみます。
まずは、`my-app/src/routes` 配下に `svelvet` というフォルダを作成します。
さらに、作成した `svelvet` フォルダの中に `+page.svelte` というファイルを作成します。

この時点で `Svelvet` メニューにアクセスすると、先程の `404` エラーは消えているかと思います。
ただ、真っ白ですね😅
なので、見出しだけ追加してあげましょう。

```html:my-app/src/routes/svelvet/+page.svelte
<h1>Svelvet</h1>
```

![](/images/sveltekit-routing-svelvet/svelvet-title.png)

`http://localhost:5173/svelvet` でアクセスすると見出しが表示できたので、ちゃんとルーティングできていることがわかりますね！

# Svelvet でフローを表示する

https://www.svelvet.io/

Svelvet は、いい感じのフローを作成するための Svelte のライブラリです。
百聞は一見に如かずなので、まずは表示させてみましょう。

では、次のコマンドで Svelvet をインストールします。

```bash
npm install svelvet
```

次に、`+page.svelte` の中身を👇に書き換えます。

```html:my-app/src/routes/svelvet/+page.svelte
<script>
    import Svelvet from "svelvet";

    const initialNodes = [
      {
        id: 1,
        position: { x: 150, y: 50 },
        data: { label: "Landing Page" },
        width: 175,
        height: 40,
        bgColor: "white"
      },
      {
        id: 2,
        position: { x: 50, y: 150 },
        data: { label: "Register" },
        width: 150,
        height: 40,
        bgColor: "white"
      },
      {
        id: 3,
        position: { x: 300, y: 150 },
        data: { label: "Login" },
        width: 150,
        height: 40,
        bgColor: "white"
      },
      {
        id: 4,
        position: { x: 300, y: 250 },
        data: { label: "2-Step Verification" },
        width: 150,
        height: 40,
        bgColor: "white"
      },
      {
        id: 5,
        position: { x: 200, y: 350 },
        data: { label: "Account Page" },
        width: 150,
        height: 40,
        bgColor: "white"
      },
      {
        id: 6,
        position: { x: 400, y: 350 },
        data: { label: "Initialize purchase process" },
        width: 150,
        height: 50,
        bgColor: "white"
      },
      {
        id: 7,
        position: { x: 300, y: 450 },
        data: { label: "Finalize purchase" },
        width: 150,
        height: 50,
        bgColor: "#B8FFC6",
        borderColor: "#B8FFC6"
      },
      {
        id: 8,
        position: { x: 500, y: 450 },
        data: { label: "User Exits" },
        width: 150,
        height: 50,
        bgColor: "#FFB8B8",
        borderColor: "#FFB8B8"
      }
    ];

    const initialEdges = [
      { id: "e1-2", source: 1, target: 2 },
      { id: "e1-3", source: 1, target: 3, animate: true },
      { id: "e2-3", source: 2, target: 5 },
      { id: "e3-4", source: 3, target: 4, animate: true },
      { id: "e4-5", source: 4, target: 5 },
      { id: "e4-6", source: 4, target: 6, animate: true },
      { id: "e6-7", source: 6, target: 7, animate: true },
      { id: "e6-8", source: 6, target: 8 }
    ];
</script>

<h1>Svelvet</h1>

<Svelvet nodes={initialNodes} edges={initialEdges} width={1000} height={800} background />
```

:::message
script タグ内が長いですが構造は単純で、👇のように３段のタグ構成になっています。
`import Svelvet from "svelvet";` で Svelvet ライブラリをインポートすることで、`<Svelvet />` が使えるようになります。

```html
<script>・・・</script>
<h1>Svelvet</h1>
<Svelvet ・・・ />
```
:::

それでは表示してみましょう！

![](/images/sveltekit-routing-svelvet/svelvet.png)

キャンバスによくわからないけどいい感じのフローが出てきましたね✨
画像ではわからないですが、実物は破線がアニメーションなんかしちゃってますよ！

さらに、ノードの中に HTML なんかも書けたりするようです。
スクリプトで動的にフローを変化させることもできそうなので、依存関係のあるステップを表現する必要がある Web サービスにはうってつけな気がします。

![](/images/sveltekit-routing-svelvet/svelvet-with-html.png)
*ノードに HTML が書ける*

# 最後に

SvelteKit ではルーティングの設定をフォルダの階層で表現できることがおわかりいただけたかと思います。
これで少しフロントエンドの入門ができた気がしました。（気がするだけです🥹）

また、Svelte のライブラリの Svelvet もいい感じですね！
ぜひ皆さんも使ってみてください。

えっ？、Svelvet みたいなライブラリみたことあるって？
気づいちゃいましたか、、、~~これだから勘のいいピー＆＃☭〄；＄※♂☞？~~

実は、Svelvet は [React Flow](https://reactflow.dev/) にインスパイアされてつくられています。

![](/images/sveltekit-routing-svelvet/inspired-by-react-flow.png)

GitHub の README にちゃんと記載されていますね。
なので、React の経験がある方はそんなに感動しないかもしれませんね。

筆者はフロントエンドに触れる機会があまりなかったので、今回の執筆を通じて大変勉強になりました！
年の瀬も近づいてきて、弊社アドベントカレンダーも残り僅かですが、引き続きお楽しみいただければ幸いです🙏

それでは、またどこかでお会いしましょう。
_See you next_ バイバイ🤞
