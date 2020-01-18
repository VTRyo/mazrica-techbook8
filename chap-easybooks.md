# Markdownで合同誌執筆を支える技術

どうも、皆様ごきげんよう。この合同誌を書いたメンバーの中ではもっとも近いタイミングで入社したえるきちです。大体、JavaScript/TypeScriptで何かしているエンジニアです。

この章では、この合同誌や、親方Projectのワンストップシリーズ[^onestop-seriese]の執筆で使っている easybooks というソフトウェアについて説明します。

[^onestop-seriese]: Twitterでハッシュタグは #ワンストップシリーズ です。

easybooksは筆者がTypeScriptで作成しているソフトウェアで、MarkdownやRe:VIEW記法で書かれた原稿データを元にPDFを生成するソフトウェアです。執筆仲間でありライバルである大岡さんに「ePubいいぞー」と布教されているため、近いうちにePubを吐けるように改修すると思います[^easybooks-epub]。

[^easybooks-epub]: この本が出ているタイミングではもう対応しているかもしれません。

## 技術書を作成する技術

技術書典や技書博などといったイベントが大盛り上がりしている技術同人誌界隈では、多くの人が、Re:VIEW[^Re-VIEW]（発音はレビューではなくリビューです）というソフトを使って執筆しています。

[^Re-VIEW]: https://....

Re:VIEWは、Rubyで書かれたソフトウェアで、Re:VIEW記法という、エンジニアにはなじみが薄い独特な記法で書かれた原稿ファイルを、TeXようなレイアウトエンジンを用いPDFやePubを生成できるソフトウェアです。もともと大昔に、ASCII社が社内の書籍制作を電子化する過程で生まれたソフトウェアがあり、その記法をベースとして、Re:VIEW記法が生み出されました。元々出版社が電子組版をする為の仕組みの末裔なのです。

Re:VIEWが定番である理由は、Re:VIEW記法というテキストファイルで原稿データがほとんど完結できることと、TeXバックエンドによりそこそこ悪くない見た目の本を作れることにあります。

ただしTeXはKnuth御大の生み出した偉大なソフトウェアですが、TeXの生まれた42年前とは異なり、ソフトウェア設計・開発の考え方も大きく異なる現代において、TeXは人類には難しすぎます。

そもそも我々にとってはあくまで本を書くのが目的であり、TeXを制することは二の次です。

そこで筆者が生み出したのが、より easy に本を作成するための easybooks です。

## easybooks

easybooks は TypeScriptで書かれたソフトウェアで、Markdown/Re:VIEW記法からPDFを生成するソフトウェアです。内部的には、Markdownを、いったんツリー上のデータ構造であるASTに変換し、ASTを加工し、Re:VIEW形式のデータを用意して、Re:VIEWのコマンドを叩いてPDFにしています。

現時点ではMarkdownを使えるようにするラッパーに過ぎません。

ただし、ASTを使って加工する仕組みとして、Re:VIEWの読みこみも実装済みであり、TeXの書き出し機能も実装を開始していたり、仕組み上ePubの書き出しにも対応できるため、将来的にはRe:VIEWに非依存にする予定です[^ruby-and-javascript]。

[^ruby-and-javascript]: JavaScriptとRuby両方動かすとか面倒ですしね

むしろTeX自体もなるべくなら使いたくありません。TeXはどうしても、インストールと設定が難解ですし、依存するパッケージなどが多すぎるため、HTML+CSSや他の何かしらのレンダリングの仕組みを使えればそれが望ましいため、脱TeXを中期的目標として考えています。

実際、Re:VIEWで執筆してる大半のサークル主はTeXに苦しめられた経験があることでしょう！もし無ければ、とても幸せです。宝くじに当たるくらいのレアリティがあると思って間違い有りません。

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
![いちご](images/chap-easybooks/strawberries-4330211_640)
```

![いちご](images/chap-easybooks/strawberries-4330211_640)

* https://pixabay.com/ja/photos/%E3%82%A4%E3%83%81%E3%82%B4-%E3%83%95%E3%83%AB%E3%83%BC%E3%83%84-%E9%A3%9F%E5%93%81-%E9%A3%9F%E3%81%B9%E3%82%8B-4330211/

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

せっかくなので easybooks の中身についても解説していきます。easybooks は、Node.js という JavaScript 定番の処理系で動くアプリケーションです。

Markdown を処理するライブラリでオススメは unified と remark です。

unified はテキストの構文解析・加工・出力をプラグインで行う汎用的なライブラリであり、remark は unified の仕組みを使った Markdown を読み書き、加工するプラグインです。

remark では MDAST という Markdown 向け AST の型が定義されています。

```sh
$ npm i @types/mdast -D
$ yarn add @types/mdast -D
```

npmなら前者、yarnなら後者でMDASTをインストール可能です。

easybooks では Markdown に関してはほぼ remark に任せています。一部拡張を行っていたりもしますが大体は remark で事足ります。

また、Re:VIEW の読み書きをするプラグインや、TeXの書き出しをするプラグインを定義しています。

