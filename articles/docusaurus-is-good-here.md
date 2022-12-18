---
title: "Docusaurus で作成するドキュメントはココがいいね！"
emoji: "🦖"
type: "tech"
topics: ["vscode", "docusaurus", "mkdocs", "npm", "markdown"]
published: true
published_at: 2022-12-06 00:00
---

:::message
この記事は、🦌🛷[**カサレアル Advent Calender 2022**](https://qiita.com/advent-calendar/2022/casareal)🎄の **6** 日目のエントリです。
:::

# TL;DR

* [Docusaurus](https://docusaurus.io/) の個人的ココがいいね！の紹介
  * 見た目がキレイ✨
  * ホームページ＋ドキュメント＋ブログ の構成がデフォルトで利用できる（いずれかの組み合わせで利用することも可能）
  * ドキュメントのバージョン切り替え機能の追加がとても簡単

# はじめに

[前回のアドベントカレンダーの記事](https://zenn.dev/casa_snona/articles/vscode-docusaurus-local-env)は楽しんでいただけましたでしょうか？
引き続き、今回は Docusaurus の個人的ココがいいね！の部分をちょっとだけ紹介してみたいと思います。

ちょっとだけじゃ足りないよという方は、ぜひ公式ドキュメントを見て、どんなことができるのかを確認してみてください！

https://docusaurus.io/docs

# ドキュメントを作成してみよう

## 前提条件

https://zenn.dev/casa_snona/articles/vscode-docusaurus-local-env

👆の環境構築が実施済みであることを前提に進めます。

## Docusaurus のフォルダ構成

主要なファイルのみ紹介します。

```
./my-website
├── blog # ブログ記事用のフォルダ
├── docs # ドキュメント用のフォルダ
├── src  # ホームページ用のフォルダ
│   ├── css
│   │   └── custom.css # グローバルなカスタム CSS
│   └── pages
│       ├── index.js         # トップページの HTML を生成する JS（React）
│       └── index.module.css # トップページ用の CSS
├── static
│  └── img # 画像ファイル用のフォルダ
└── docusaurus.config.js # Docusaurus のコンフィグファイル
```

## ドキュメントのページを追加

まずは、コンフィグファイルを編集して、サイトの言語を日本語に設定しておきます。

```diff js:docusaurus.config.js
  i18n: {
-   defaultLocale: 'en',
-   locales: ['en'],
+   defaultLocale: 'ja',
+   locales: ['ja'],
  },
```

また、今回はブログ部分を扱わないので、機能としてもオフにしておきます。
`blog` フォルダも不要なので削除して OK です。

```diff js:docusaurus.config.js
  presets: [
    [
      'classic',
      /** @type {import('@docusaurus/preset-classic').Options} */
      ({
        ・・・省略・・・
-       blog: {
-         showReadingTime: true,
-         // Please change this to your repo.
-         // Remove this to remove the "edit this page" links.
-         editUrl:
-           'https://github.com/facebook/docusaurus/tree/main/packages/create-docusaurus/templates/shared/',
-       },
+       blog: false,
        ・・・省略・・・
    ]
  ]
  ・・・省略・・・
  themeConfig:
    /** @type {import('@docusaurus/preset-classic').ThemeConfig} */
    ({
      navbar: {
        ・・・省略・・・
        items: [
          ・・・省略・・・
-         { to: '/blog', label: 'Blog', position: 'left' },
        ]
      },
  　　・・・省略・・・
    }),
```

`docs` 配下に、適当なマークダウンファイル（`fizzbuzz.md`としました）を作成します。
`npm run start` で起動している場合は、勝手に差分を検出してブラウザをリロードしてくれます。

マークダウンファイルの中身は次のようにしてみます。

~~~md:docs/fizzbuzz.md
# FizzBuzz 問題

FizzBuzz 問題のプログラム。

```python
for i in range(1, 101):
    if i % 3 == 0 and i % 5 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```

ワンライナーで書く。

```python
for i in range(1, 101):
    print("FizzBuzz" if i % 3 == 0 and i % 5 == 0 else "Fizz" if i % 3 == 0 else "Buzz" if i % 5 == 0 else i)
```
~~~

Tutorial のサイドバーにマークダウンファイルの H1 タグに相当する見出しで項目が追加されています。

![FizzBuzz 問題 のドキュメントを追加](/images/docusaurus-is-good-here/fizzbuzz-docs.png)
*FizzBuzz 問題 のドキュメントを追加*

ワンライナーで書かれたコードは見切れるくらい長いので、自動で横スクロールできるようになっています。
筆者は、基本マウスは使わずにトラックパッドで操作するので横スクロールもたやすいのですが、普通のマウスを愛用している方々は横スクロールするときはスクロールバーをいちいち掴んで移動させないといけないので大変ですよね。。。

でも安心してください！
なんと、Docusaurus では折り返しボタンが表示されるんです！
なので、読み手が右端で折り返すかどうかを切り替えることができます。

![コードを折り返す様子](/images/docusaurus-is-good-here/wrap-code.gif)
*コードを折り返す様子*

なんとフレンドリーな作りなんでしょう。
細かいですが、非常によい機能だと思っています。

:::message
今回の FizzBuzz のプログラム（ワンライナーも）ですが、実は今話題の AI（[ChatGPT](https://chat.openai.com/chat)） に書かせてみました。（これぐらい自分で書けるだろって話ですが...）
噂でどんなことができるかは何となく知っていたのですが、使ってみてやっぱりすごいなぁと感じます。
少しだけ画像認識の AI のプログラムを書いたことはあったのですが、ChatGPT のような自然言語処理はもっと難しい気がしています。
今後の発展に期待したいです！

![ChatGPT に FizzBuzz を解かせている様子](/images/docusaurus-is-good-here/fizzbuzz-ai.png)
*ChatGPT に FizzBuzz を解かせている様子*
:::

## ドキュメントに階層をつくる

FizzBuzz 問題のマークダウンファイルは何も考えずに `docs` 配下に作成しましたが、階層をつくってドキュメントを整理することができます。

では、`docs` 配下に `add-section` というフォルダを作成して、その下に先ほど作成した `fizzbuzz.md` を配置してみます。
すると、👇のように階層化されます。

![ドキュメントの階層化](/images/docusaurus-is-good-here/hierarchy.png)
*ドキュメントの階層化*

デフォルトでは、フォルダ名がそのままセクション名になりますが、カスタマイズすることができます。

`add-section` 配下に `_category_.json` というファイル名（固定）で新規にファイルを作成します。
ファイルの中身は次のようにします。

```json:docs/add-section/_category_.json
{
  "label": "追加セクション",
  "position": 4,
  "link": {
    "type": "generated-index",
    "description": "自分で追加したセクションです。"
  }
}
```

| キー名   | 値の説明                                                                                                                                                   |
| -------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------- |
| label    | サイドバーのセクション名                                                                                                                                   |
| position | サイドバーの上から数えた時の位置                                                                                                                           |
| link     | add-section のリンク先の設定<br>type に `generated-index` を設定すると、セクション配下の目次を作成してくれる<br>description は、タイトル下に表示される概要 |

すると、👇のようになります。

![追加セクション](/images/docusaurus-is-good-here/add-section.png)
*追加セクション*

各セクションでどんな内容を扱っているのかが見やすくてわかりやすいですね👍
ドキュメントを読む側も迷子になりづらくなるのかなって気がします。

## カスタム CSS の追加

デフォルトでも十分キレイな見た目ですが、カスタマイズしたい場合は CSS を適用することができます。

`add-section` 配下に `show-table.md` というファイル名で新規にファイルを作成します。
ファイルの中身は次のようにします。

```md:docs/add-section/show-table.md
# テーブル表示

テーブルを表示してみます。

| A          | B                                                                                                      | C                                                                                                      | D                                                                                                      | E                                                                                                      |
| ---------- | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| あいうえお | とーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーっても長い文章を含むテーブル | とーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーっても長い文章を含むテーブル | とーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーっても長い文章を含むテーブル | とーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーっても長い文章を含むテーブル |
| かきくけこ | とーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーっても長い文章を含むテーブル | とーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーっても長い文章を含むテーブル | とーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーっても長い文章を含むテーブル | とーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーーっても長い文章を含むテーブル |
```

![残念なテーブル表示](/images/docusaurus-is-good-here/show-table-not-good.png)
*残念なテーブル表示*

A 列がなんともおざなりな感じですね。。。
かわいそうなので、横スクロールできるように CSS を調整しましょう。

`src/css/custom.css` に次のスタイルを追記します。

```css:src/css/custom.css
/* 追記するスタイル */
table {
  white-space: nowrap;
  overflow-x: scroll;
}
```

👇のようになると思います。

![テーブル表示](/images/docusaurus-is-good-here/show-table-good.png)
*テーブル表示*

これで多少見た目が良くなりましたね！（横スクロールですけど💦）

# ドキュメントのバージョン切り替え機能を追加

新規にドキュメントを作成するのはよいのですが、プロダクトの運用を続けていくと、自ずとドキュメントもプロダクトに合わせてバージョンアップが必要になってきます。
プロダクトのアップデートが一斉に行われて、基本的に一つのバージョンしかサポートしないのであればとくに問題ないのですが、多くの場合、新旧のバージョンが混在します。

ここで問題になるのが、プロダクトのバージョンアップに伴ってドキュメントの Web サイトを更新してしまうと、古いバージョンを利用しているユーザが困ってしまいます。
とはいえ、公開してしまった URL の参照先を変更してもらうのもユーザへの周知が大変ですし、、、

でも安心してください！
なんと、Docusaurus ならバージョンの切り替えが簡単にできるんです！

それではやってみましょう。

次のコマンドで、現在のドキュメントのバージョン（`1.0.0` としました）をタグ付けします。

```
npm run docusaurus docs:version 1.0.0
```

:::message
👇のようなメッセージが出れば OK です。
```
> my-website@0.0.0 docusaurus
> docusaurus docs:version 1.0.0

[SUCCESS] [docs]: version 1.0.0 created!
```
:::

すると、次のフォルダおよびファイルが生成されます。

```
./my-website
├── versioned_docs
│   └── version-1.0.0  # バージョン 1.0.0 のドキュメントが配置されているフォルダ
├── versioned_sidebars # バージョンごとのサイドバー設定が配置されるフォルダ
└── versions.json      # バージョン管理ファイル
```

この状態で、現在の `docs` 配下のドキュメントは次期バージョン（Next）となり、現在の最新バージョンは `1.0.0` ということになります。
このままでも URL にバージョンを含めることでドキュメントを切り替えることができるのですが、コンフィグファイルを編集して、バージョンのドロップダウンを表示するようにしてみます。

```diff js:docusaurus.config.js
  themeConfig:
    /** @type {import('@docusaurus/preset-classic').ThemeConfig} */
    ({
      navbar: {
　　　　・・・省略・・・
        items: [
  　　　　・・・省略・・・
+         {
+           type: 'docsVersionDropdown',
+           position: 'right',
+         },
        ]
      },
  　　・・・省略・・・
    }),
```

これで、👇のようになります！（今は 1.0.0 と Next で内容が同じなので切り替えても変わり映えしませんが）

![バージョン切り替え](/images/docusaurus-is-good-here/add-version.png)
*バージョン切り替え*

また、`Next` バージョンや最新でないバージョンに切り替えると、最新バージョンを見るようにアナウンスしてくれます。

![最新バージョンを見るようにアナウンス](/images/docusaurus-is-good-here/next-version.png)
*最新バージョンを見るようにアナウンス*

めちゃくちゃ便利で簡単ですね！
次回以降のバージョンは、👇のように同様のコマンドを実行すれば自動で追加されます。

```
npm run docusaurus docs:version 2.0.0
```

# 最後に

まだまだ紹介しきれていないいいねポイントはありますが、ここからはぜひご自分で触ってみて、ドキュサウルスと一緒に発見してみてください！

えっ？、今回の編集した Docusaurus のソースファイルを公開してほしいって？
もちろん GitHub で公開しますよ！

でも、リンクを公開するのはこのアドベントカレンダーの勝手にシリーズ化の最後の投稿に貼り付けます。
それまで弊社のアドベントカレンダーをお楽しみください🙇
