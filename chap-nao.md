# VIPER でiOSアプリのリファクタリング実践(仮)

 いわゆるAppleMVCで構築されたアプリケーションをリファクタリングする時、Redux、MVVM、Clean Architectureと悩んだ経験があるのではないでしょうか？
「VIPER」を導入すれば、既存実装を活かしつつ、責務が集中してしまったViewControllerを分割し、メンテナンス性の高いコードへ書き換えることが可能です。
 今回は、実際にiOSアプリケーションを処理をVIPERでリファクタする方法を紹介します。

## はじめに
 著者のnao(@1wa46)です。マツリカでは、iOSアプリケーションの開発を担当しています。
私も「VIPER」を知ったのは、マツリカへ入社した半年前でした。iOSアプリケーション開発を長く対応している為、リファクタ、リアーキテクチャを経験しているなかで、「VIPER」がAppleMVCをリファクタするのに最適ではないかと感じました。今回は、その魅力をについてご紹介させて頂きます。

## VIPERとは
 VIPERは、MVCやMVVMなどに代表されるソフトウェアアーキテクチャのひとつです。
「Clean ArchitectureをiOSに応用したもの」です。VIPERは、Clean Architectureにイメージされる円形の図を使わずにビューを起点にし、ファットビューコントローラの低減を中心に据えているのが特徴的です。

## ファクタリングするiOSアプリケーション
今回、リファクタリングするのは、RSSをパースしてUITableViewへ表示する簡単なアプリケーションです。Swift5で100行程度で実装されています。下記のGitHubを確認してください。(https://github.com/honda-n/swift-rss-sample)

## 開発に必要なもの
 Xcodeはもちろんインストールされているとおもいますので、その他で、必要はツールをインストールしていきます。
#### 手順１ Cocoapodsをインストールする
```
$ [sudo] gem install cocoapods
```
 今回のサンプルで、RSSをパースする目的でFeedKit(https://github.com/nmdias/FeedKit)を利用している為、Cocoapodsを追加しています。

#### 手順２ ファイル自動生成のGemをインストールする

```
$ [sudo] gem install generamba
```

#### 手順３ generamba の setup
```
$ git clone https://github.com/honda-n/swift-rss-sample.git
 swift-viper-rss 👈改行せずに入力してください

# Cocoapodsをインストール
$ cd swift-viper-rss
$ pod install

# 任意: git管理する場合は、一旦 .git/を消して再度設定
$ rm -rf .git/
$ git init
$ git remote add origin [git URL]

# テンプレートを準備
$ generamba setup
The company name which will be used in the headers: sample
The name of your project is swift-viper-rss. Do you want to use it?
 (yes/no) yes
The project prefix (if any):
The path to a .xcodeproj file of the project is
 'swift-rss-sample.xcodeproj'. Do you want to use it? (yes/no) yes
Select the appropriate target for adding your MODULES (type the index):
0. swift-rss-sample
1. swift-rss-sampleTests
 0
Are you using unit-tests in this project? (yes/no) yes
Select the appropriate target for adding your TESTS (type the index):
0. swift-rss-sample
1. swift-rss-sampleTests
 1
Do you want to add all your modules by one path? (yes/no)
 yes
Do you want to use the same paths for your files both in Xcode and
 the filesystem? (yes/no) yes
The default path for creating new modules:
 swift-rss-sample/Classes/Modules
The default path for creating tests:
 swift-rss-sampleTests/Classes/Modules
Are you using Cocoapods? (yes/no)
 yes
The path to a Podfile is 'Podfile'. Do you want to use it? (yes/no)
 yes
Are you using Carthage? (yes/no) no
Do you want to add some well known templates to the Rambafile? (yes/no)
 yes
...

Rambafile successfully created! Now run `generamba template install`.
```
 直下に Rambafile が生成されているはずです。

#### 手順4 テンプレートをインストール
```
$ generamba template install
Updating shared generamba-catalog specs...
Installing swifty_viper...
```
 これで準備は完了です。

## リファクタリング

#### コードを生成
 Mainという名前で、VIPERのコードを生成します。
```
$ generamba gen Main swifty_viper

...

Creating code files...
Creating test files...
Module successfully created!
Name: Main
Project file path: swift-rss-sample/Classes/Modules/Main
Project group path: swift-rss-sample/Classes/Modules/Main
Test file path: swift-rss-sampleTests/Classes/Modules/Main
Test group path: swift-rss-sampleTests/Classes/Modules/Main
```

このようにコードが生成されます。
![ツリー](images/スクリーンショット_nao_01)

#### UITableViewControllerを継承するように書き換える

```
# MainViewController
class MainViewController: UIViewController, MainViewInput {
...
```

#### StoryBoardからモジュールを初期化できるようにする

Main.storyboardを開き、RSS Scenceのcustom classを MainViewControllerへ変更します。

![MainViewControllerを追加](images/スクリーンショット_nao_02)

RSS Scence へ NSObjectを追加して、custom classへMainModuleInitializerを追加します。

![ainModuleInitializerを追加](images/スクリーンショット_nao_03)

最期に@IBOutletをMainViewControllerへ繋げばコントローラーの置き換えは完了です。

![ツリー](images/スクリーンショット_nao_04)

#### ViewContoler クラスから処理を分割する




## まとめ
