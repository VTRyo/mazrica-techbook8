# マツリカごった煮本

## 概要説明

マツリカ社員による技術本

## この本の目的


## 執筆・配布スケジュール

* 章目次確定：
* 本文初稿：1月18日
* レビュー＆追記：1月xx日
* 入稿:1月下旬 or 2月上旬
* 発行　技術書典（2月29日＠池袋サンシャインシティ）

## 著者紹介とあとがき

* 著者紹介：chap-contributors.md
* あとがき：chap-postscript.md

をそれぞれ記入してください。

# Dockerを使ってPDFをローカルで確認する（エンジニア向け）

* bash, zshでのコマンド。fish使いはうまいことやってくださいｗ

```
$ cd mazrica-techbook8
$ docker run -t --rm -v $(pwd):/book vvakame/review:3.1 /bin/bash -ci "cd /book && yarn && yarn build"
```

* PDFの場所
```
$ ls .review
```
