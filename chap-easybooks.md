# Markdownで合同誌執筆を支える技術

この章では、この合同誌や、親方Projectのワンストップシリーズ[^onestop-seriese]の執筆で使っている easybooks というソフトウェアについて説明します。

[^onestop-seriese]: Twitterハッシュタグ #ワンストップシリーズ

この合同誌を書いたメンバーの中ではもっとも近いタイミングで入社したえるきちです。大体、JavaScript/TypeScriptで何かしているエンジニアです。

easybooksは筆者がTypeScriptで作成しているソフトウェアで、MarkdownやRe:VIEW記法で書かれた原稿データを元にPDFを生成するソフトウェアです。執筆仲間でありライバルでもある大岡さん[^oukayuka-twitter]に「ePubいいぞー」と布教されているため、近いうちにePubを吐けるように改修すると思います[^easybooks-epub]。

[^oukayuka-twitter]: https://twitter.com/oukayuka
[^easybooks-epub]: この本が出ているタイミングではもう対応しているかもしれません。

## 技術書を作成する技術

技術書典や技書博などといったイベントが大盛り上がりしている技術同人誌界隈では、多くの人が、Re:VIEW[^Re-VIEW]（発音は**レ**ビューではなく**リ**ビューです）というソフトを使って執筆しています。

[^Re-VIEW]: https://github.com/kmuto/review/

Re:VIEWは、Rubyで書かれたソフトウェアで、Re:VIEW記法という、エンジニアにはなじみの薄い、独特な記法で書かれた原稿ファイルを、TeXなどのレイアウトエンジンを用いPDFなどを生成できるソフトウェアです。もともと大昔に、ASCII社が社内の書籍制作を電子化するために生み出したソフトウェアがあり、そこで使われていたものを参考としてRe:VIEW記法が生み出されました。もともと出版社が電子組版をする為に作られた仕組みの末裔なのです。

Re:VIEWが定番である理由はほとんどをテキストファイルで執筆できることと、TeXバックエンドによりそこそこ悪くない見た目の本を作れることにあります。

ところが、TeXはKnuth御大の生み出した偉大なソフトウェアですが、TeXの生まれた42年前とは異なり、ソフトウェア設計・開発の考え方も大きく進化した現代において、TeXは一般的な人類にはあまりにも難しすぎます。

そもそも我々にとってはあくまで本を書くのが目的であり、TeXを制することは二の次です。そこで筆者が生み出したのが、よりイージーに本を作成するための easybooks です。

## easybooks

easybooks は TypeScriptで書かれたソフトウェアで、Markdown/Re:VIEW記法からPDFを生成するソフトウェアです。内部的には、Markdownを、いったんツリー上のデータ構造であるAST[^AST]に変換し、ASTを加工し、Re:VIEW形式のデータを用意して、Re:VIEWのコマンドを叩いてPDFにしています。

[^AST]: Abstract Syntax Tree の略で、日本語では抽象構文木といいます。簡単にいうと構文解析の結果をツリーとして保持するものです。

現時点ではMarkdownを使えるようにするラッパーに過ぎません。既存のソフトウェアにはMarkdownをRe:VIEW記法に変換するソフトもありますが、easybooksではMarkdown拡張を用いて、Re:VIEW記法の大半を実装しています。

さらには、Re:VIEWの読みこみも実装済みであり、TeXの書き出し機能も実装を開発中であるため、将来的にはRe:VIEWには依存せずに済むようにする予定です。JavaScriptとRuby両方動かすとか面倒ですしね。

むしろTeX自体もなるべくなら使いたくありません。TeXはどうしても、インストールと設定が難解[^tex-difficult]ですし、依存するパッケージなどが多すぎるため、HTML+CSSや他の何かしらのレンダリングの仕組みを使えればそれが望ましいため、脱TeXを中期的目標として考えています。

[^tex-difficult]: 実際、Re:VIEWで執筆してる大半のサークル主はTeXに苦しめられた経験があることでしょう。もし苦労したことがない人は、宝くじを買えばきっと当たるくらいには幸運な方です。

## easybookの使い方

まずは easybooks の使い方について解説します。

### 動作条件

* RubyおよびRe:VIEWをインストールしていること
* TeXをインストールしていること
* Node.js LTSをインストールしていること

### 設定ファイルの書き方

```json
{
  "bookname": "example-book",
  "booktitle": "Example Book",
  "aut": ["なまえ"],
  "language": "ja",
  "toc": true,
  "rights": "copyrights",
  "colophon": false,
  "history": [["発行日"]],
  "prt": "印刷所",
  "pbl": "サークル名",
  "secnolevel": 3,
  "titlepage": false,
  "review_version": 3.0,
  "texstyle": ["reviewmacro"],
  "texdocumentclass": [
    "review-jsbook",
    "media=ebook,paper=a5,serial_pagination=true,hiddenfolio=nikko-pc,open
any,fontsize=10pt,baselineskip=15pt,line_length=40zw,number_of_lines=33,he
ad_space=14mm,headsep=3mm,headheight=5mm,footskip=10mm"
  ],
  "sty_templates": {
    "url": "https://github.com/TechBooster/ReVIEW-Template/archive/2cde584
d33e8a6f5e6cf647e62fb6b3123ce4dfa.zip",
    "dir": "ReVIEW-Template-2cde584d33e8a6f5e6cf647e62fb6b3123ce4dfa/artic
les/sty/"
  },
  "templates": ["images"],
  "catalog": {
    "CHAPS": ["about-easybooks.md", "sample.re"]
  }
}
```

このJSONで設定する項目は、Re:VIEWのconfig.ymlとcatalog.ymlを合わせたような内容 + easybooks 専用の設定です。

|設定項目|設定内容|
|--------|--------|
|sty_templates|`url`からzipをダウンロードして、`dir`以下をTeX styとして利用します。|
|templates|画像や自分のカスタムstyファイルなどのあるディレクトリを指定します|
|catalog|Re:VIEWの`catalog.yml`に設定する項目をそのまま設定します|

### 動かす方法

```sh
$ npx easybooks example-book.json
```

npxコマンドを使う場合、インストールしていなくても実行は可能です。インストールしているとバージョン固定ができる、ネットワークが死んでても動くなどの利点はありますがが、お手軽さをとってインストールせずに動かしてもいいでしょう。

インストールするときは、`npm i easybooks -S` や `yarn add easybooks` コマンドで行います。

## Markdownの書き方

### セクションとか

```md
### セクションとか

#### [column] コラム
#### [/column] コラム閉じる

## [notoc] ToCに出力しない（前書きとか）
```

### リスト

* ほげ
* ふが

```md
* ほげ
* ふが
```

### 表

|ひょう|ひょー|
|------|------|
|hoge|ほげ|
|fuga|ふが|

```md {caption="GFM table"}
|ひょう|ひょー|
|------|------|
|hoge|ほげ|
|fuga|ふが|
```

| Left-aligned | Center-aligned | Right-aligned |
| :---         |     :---:      |          ---: |
| git status   | git status     | git status    |
| git diff     | git diff       | git diff      |

```md caption={"left/center/right align"}
| Left-aligned | Center-aligned | Right-aligned |
| :---         |     :---:      |          ---: |
| git status   | git status     | git status    |
| git diff     | git diff       | git diff      |
```


### コード

```sh
$ yarn add easybooks
```

````md
```sh
$ yarn add easybooks
```
````

```js {caption="JavaScriptなコード"}
console.log('hoge')
```

````md
```js {caption="JavaScriptなコード"}
console.log('hoge')
```
````

```ts {id="typescript" caption="TypeScriptなコード"}
console.log('hoge')
```

本文からの参照は[list:typescript]で。

````md
```ts {id="typescript" caption="TypeScriptなコード"}
console.log('hoge')
```

本文からの参照は[list:typescript]で。
````

### 脚注

```md
ほげほげします[^hoge]。

[^hoge]: hogeはほげです。
```

ほげほげします[^hoge]。

[^hoge]: hogeはほげです。


### 画像

```md
![いちご](images/chap-easybooks/strawberries-4330211_640/jpg)
```

![いちご](images/chap-easybooks/strawberries-4330211_640.jpg)

* https://pixabay.com/ja/photos/%E3%82%A4%E3%83%81%E3%82%B4-%E3%83%95%E3%83%AB%E3%83%BC%E3%83%84-%E9%A3%9F%E5%93%81-%E9%A3%9F%E3%81%B9%E3%82%8B-4330211/

```md
![いちご](images/chap-easybooks/strawberries-4330211_640/jpg?scale=0.5)
```

![いちご](images/chap-easybooks/strawberries-4330211_640.jpg?scale=0.5)


### コメント

```md
<!--
コメント！
-->
```

<!--
コメント！
-->

## easybooks の中身について解説

せっかくなので easybooks の中身についても解説していきます。easybooks は、Node.js という JavaScript 定番の処理系（ランタイム）で動くアプリケーションです。

Markdown を処理するライブラリでオススメは unified[^unified-url] と remark[^remark-url] です。

[^unified-url]: https://github.com/unifiedjs/unified
[^remark-url]: https://github.com/remarkjs/remark

unified はテキストの構文解析・加工・出力をプラグインで行う汎用的なライブラリであり、remark は unified の仕組みを使った Markdown を読み書きする定番のプラグインです。同様のものに rehype というHTMLを処理するものもあります。

remark も rehype も利用実績が多いため、自作したりマイナーなライブラリを使うよりは脆弱性やバグが少ないことを期待できます。

remark では、Markdownのオリジナル文法だけではなく、CommonMark や Github Fravored Markdown など、拡張文法にも対応しています。また MDAST という Markdown 向け AST の型が定義されています。

```sh
$ npm i @types/mdast -D
$ yarn add @types/mdast -D
```

npmなら前者、yarnなら後者でMDASTをインストール可能です。

easybooks では Markdown に関してはほぼ remark に任せています。一部拡張を行っていたりもしますが大体は remark で事足ります。

また、Re:VIEW の読み書きをするプラグインや、TeXの書き出しをするプラグインを定義しています。

### unifiedの使い方

まず `unified().use(plugin)` のように、`use`メソッドでプラグインを登録します。

あとは、`parse` メソッドでテキストの構文解析を行い、`process` メソッドで加工をし、`stringify`メソッドで何かしらのテキスト出力を行います。


### プラグインの作り方

[list:parse-review]は easybooks の Re:VIEWのパーサープラグインのコードの一部です。

```ts {id=parse-review}
export default function parseReview() {
  //@ts-ignore
  this.Parser = (doc: string, vfile: any): EBAST.Root => {
    return {
      type: 'root',
      children: parse(doc),
    }
  }
}
```

thisを操作するときに面倒なので `//@ts-ignore` で TypeScript の警告を抑制しています。

プラグインは、`this` に `Parser` や `Compiler` などの関数を追加するしておくと、プラグインとして起動できます。

`EBAST` は、EasyBooks AST で、Markdown の AST である MDAST をベースとしつつ、easybooks 用に拡張した型です。

`parse` では、Re:VIEW記法を正規表現などを使い構文解析を行っています。

[list:ebast-to-review]は EBAST を Re:VIEWソースに変換するためのプラグインの一部です。

```ts {id=ebast-to-review}
export default function ebastToReview() {
  // @ts-ignore
  this.Compiler = (tree: EBAST.Root, vfile: any) => {
    return compiler(tree, {
      list: 0,
      id: 0,
      chapter: typeof vfile.data === 'string' ? vfile.data : '',
    })
  }
}
```

`Compiler` は AST を受け取り、実際の出力文字列に変換する関数です。

`compiler` 関数は、ASTを再帰的に処理する関数です。

