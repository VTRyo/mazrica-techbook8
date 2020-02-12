# VIPERでiOSアプリのリファクタリング実践

 いわゆるAppleMVCで構築されたアプリケーションをリファクタリングする時、Redux、MVVM、Clean Architectureと悩んだ経験があるのではないでしょうか？
「VIPER」を導入すれば、既存実装を活かしつつ、責務が集中してしまったViewControllerを分割し、メンテナンス性の高いコードへ書き換えることが可能です。
今回は、実際にiOSアプリケーションを処理をVIPERでリファクタする方法を紹介します。

## はじめに
 著者のnao(@1wa46)です。マツリカでは、iOSアプリケーションの開発を担当しています。
私も「VIPER」を知ったのは、マツリカへ入社した半年前でした。iOSアプリケーション開発を長く対応している為、リファクタ、リアーキテクチャを経験しているなかで、「VIPER」がAppleMVCをリファクタするのに最適ではないかと感じました。今回は、その魅力をについてご紹介させて頂きます。

## VIPERとは
 VIPERは、MVCやMVVMなどに代表されるソフトウェアアーキテクチャのひとつです。
「Clean ArchitectureをiOSに応用したもの」です。VIPERは、Clean Architectureにイメージされる円形の図を使わずにビューを起点にし、ファットビューコントローラの低減を中心に据えているのが特徴的です。

## リファクタリングする対象
今回、リファクタリングするのは、RSSをパースしてUITableViewへ表示する簡単なアプリケーションです。Swift5で約100行で実装されています。下記のGitHubを確認してください。(https://github.com/honda-n/swift-rss-sample)

## 開発に必要なもの
 Xcodeはもちろんインストールされているとおもいますので、その他で、必要はツールをインストールしていきます。
### Cocoapodsをインストールする
 今回のサンプルで、RSSをパースする目的でFeedKit(https://github.com/nmdias/FeedKit)を利用している為、Cocoapodsを追加します。
```
$ [sudo] gem install cocoapods
```

### ファイル自動生成のGemをインストールする
 VIPERのテンプレートを生成する目的で、generambaを導入します。
Generamba(https://github.com/strongself/Generamba)は、Xcodeを操作するために作成されたコードジェネレーターです。 主にVIPERモジュールを生成するように設計されていますが、他のクラスの生成用にカスタマイズするのは非常に簡単です（Objective-CとSwiftの両対応）。
```
$ [sudo] gem install generamba
```

### generambaをセットアップする
 まずリファクタリングするアプリケーションをクローンして、セットアップを行います。
Q&A形式で回答していくだけで、テンプレートを生成に必要なRambafileが生成されます。
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

### テンプレートをインストールする
 直下に生成されたRambafileを元にテンプレートをインストールします。
```
$ generamba template install
Updating shared generamba-catalog specs...
Installing swifty_viper...
```

## リファクタリング
 VIPERへ書き換える準備が整いましたので、ここからは、VIPERアーキテクチャを導入してリファクタリングを行います。
### コードを生成
 今回は、generambaでインストールした、swifty_viperというテンプレートを元に、Mainという名前でVIPERのコードを生成します。
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
![ツリー](images/chap-nao/スクリーンショット_nao_01)

### UITableViewControllerを継承するように書き換える
 初めに、生成された至ってシンプルなViewControllerを、UITableViewControllerを継承するように、書き換えます。
```
//  MainMainViewController.swift
class MainViewController: UITableViewController {
...
```

### StoryBoardからモジュールを初期化できるようにする
 標準で導入されるVIPERのテンプレートには、StoryBoardから初期化を行えるようになる為、設定を追加する必要があります。

 Main.storyboardを開き、RSS ScenceのCustom ClassをMainViewControllerへ変更します。
![MainViewControllerを追加](images/chap-nao/スクリーンショット_nao_02)

 RSS Scence へ NSObjectを追加して、Custom ClassへMainModuleInitializerを追加します。
 ModuleInitializer は、テンプレートから生成されたクラスで StoryBoard 上へ設定することで、ModuleConfiguratorを呼び出し、VIPERの各クラスを初期化することが出来ます。
![MainModuleInitializerを追加](images/chap-nao/スクリーンショット_nao_03)

 最期に@IBOutletをMainViewControllerへ繋げばコントローラーの置き換えは完了です。
![ツリー](images/chap-nao/スクリーンショット_nao_04)

### Extensionクラスは、別ファイルへ
 ここからはコードを置き換えてリファクタリングしていきます。
初めにextensionで定義されたUIKitのクラス拡張を別ファイルへ定義します。
今回は、UIImage+Extenstion.swiftとしていますが、命名は自由です。
```
//  UIImage+Extenstion.swift
import UIKit

extension UIImage {
    // UIImageを指定したCGSizeでリサイズする
    func resize(size _size: CGSize) -> UIImage? {
        let widthRatio = _size.width / size.width
        let heightRatio = _size.height / size.height
        let ratio = widthRatio < heightRatio ? widthRatio : heightRatio

        let resizedSize = CGSize(
           width: size.width * ratio,
           height: size.height * ratio
        )
        // 引数のCGSizeを元に、描画を変更する
        UIGraphicsBeginImageContextWithOptions(resizedSize, false, 0.0)
        draw(in: CGRect(origin: .zero, size: resizedSize))
        let resizedImage = UIGraphicsGetImageFromCurrentImageContext()
        UIGraphicsEndImageContext()

        return resizedImage
    }

    // ネットワークからUIImageを取得する
    static func imageWithUrl(_ URL: URL) -> UIImage? {
        do {
            let data = try Data(contentsOf: URL)
            return UIImage(data: data)
        }catch let error {
            print("Error : \(error.localizedDescription)")
            return nil
        }
    }
}
```

### RemoteDataManagerを定義する
 サンプルのRSSを取得する処理は、DateManagerとして書き出します。
VIPERの場合、テストを作成する際にモックできるようにProtocolを定義し、クラスで継承して作成しています。

```
//  RemoteDateManager.swift
import Foundation
import FeedKit

protocol RemoteDataManagerProtocol {
    func get(completion: ((Feed) -> Void)?, fail: ((String) -> Void)?)
}

class RemoteDataManager: RemoteDataManagerProtocol {
    func get(completion: ((Feed) -> Void)?, fail: ((String) -> Void)?) {
        guard let url = URL(
          string: "https://feeds.soundcloud.com
            /users/soundcloud:users:466377696/sounds.rss"
          ) else {
            return
        }
        let parser = FeedParser(URL: url)
        parser.parseAsync { (result) in
            switch result {
            case .success(let feed):
                completion?(feed)
            case .failure(let error):
                fail?(error.localizedDescription)
            }
        }
    }
}
```
### Interactor
 Interactor は、データに関わる処理(取得等)を配置します。
#### InteractorInput protocolを定義
 Interactorへ先程作成したDataManagerクラスを呼び出すプロトコルを定義します。
```
//  MainMainInteractorInput.swift
import Foundation

protocol MainInteractorInput {
    func fetchRSS()
}

```

#### InteractorOutput protocolを定義
 呼び出された結果を呼び出し先へ返す為、RSSGeted関数とエラーを返すfaild関数をプロトコルを定義します。
```
//  MainMainInteractorOutput.swift
import Foundation
import FeedKit

protocol MainInteractorOutput: class {
    func RSSGeted(_ feed: Feed)
    func faild(_ error: String)
}
```

#### Interactorへ初期化処理を追加
 InteractorInput のプロトコルを継承したクラスへ処理を移植します。
Interactorのinitrizerを追加して、DataManagerを受け取るようにします。
この形にすることでUnitTestを定義する際に、モックした処理を定義しやすくなります。

```
//  MainMainInteractor.swift
class MainInteractor: MainInteractorInput {
    weak var output: MainInteractorOutput!
    var remoteDataManager: RemoteDataManagerProtocol!
    // 初期化する際にDataManagerを受け取る
    init(remoteDataManager: RemoteDataManagerProtocol) {
        self.remoteDataManager = remoteDataManager
    }

    func fetchRSS() {
        remoteDataManager.get(completion: { (feed) in
            self.output.RSSGeted(feed)
        }) { (error) in
            self.output.faild(error)
        }
    }
```
### Router
 Routerは、ナビゲーション遷移や、モーダル遷移など、画面遷移を担当します。
#### RouterInput protocolを定義
 Routerは別画面へ遷移などの処理のプロトコルを定義します。
```
//  MainMainRouterInput.swift
import Foundation

protocol MainRouterInput {
    func presentWebView(URL: URL)
}
```

#### MainRouterへ遷移処理を移植
 RouterInputのプロトコルを継承したクラスへ遷移処理を定義します。

```
//  MainMainRouter.swift
import SafariServices

class MainRouter: MainRouterInput {
    private weak var viewController: UIViewController?
    private weak var presenter: MainPresenter?

    init(_ viewController: UIViewController?, presenter: MainPresenter) {
        self.viewController = viewController
        self.presenter = presenter
    }
    // インアップブラウザを表示する処理
    func presentWebView(URL: URL) {
        let safariViewController = SFSafariViewController(url: URL)
        viewController?.present(safariViewController,
          animated: true, completion: nil)
    }
}
```
### Configurator
 Configuratorは、VIPERの各クラスを初期化することが出来ます。
#### ModuleConfigurator configure関数を修正
 Configuratorは、StoryBoardから初期化される際の処理で、今回はRouterとInteractorへ初期化処理を追加した為、対応したクラスの初期化を変更します。

```
//  MainMainConfigurator.swift
import UIKit

class MainModuleConfigurator {
    ...
    private func configure(viewController: MainViewController) {
        let presenter = MainPresenter()
        let router = MainRouter(viewController, presenter: presenter)
        presenter.view = viewController
        presenter.router = router

        let interactor = MainInteractor(
            remoteDataManager: RemoteDataManager())
        interactor.output = presenter

        presenter.interactor = interactor
        viewController.output = presenter
    }
    ...
}
```

### Presenter
 Presenterは、Viewから受け取った依頼を各クラスへ依頼する司令塔のような役割を担当します。

#### 機能を集約する Presenterを定義する
 ViewはUIKitに関わる処理に限定する為、それ以外の処理をPresenterへ移植します。
```
//  MainMainPresenter.swift
import FeedKit

class MainPresenter: MainModuleInput, MainViewOutput {
    weak var view: MainViewInput!
    var interactor: MainInteractorInput!
    var router: MainRouterInput!
    var editingFeed: Feed?
    // ViewController から呼び出され、起動時にRSSを取得しに行く
    func viewIsReady() {
        interactor.fetchRSS()
    }
    // Viewへ処理を返す箇所はこの関数へ集約しています
    func updateView() {
        guard let feed = editingFeed else {
            return
        }
        var image: UIImage? = nil
        if let urlString = feed.rssFeed?.image?.url {
            guard let URL = URL(string: urlString) else {
                return
            }
            image = UIImage.imageWithUrl(URL)
        }
        view.showRSS(items: feed.rssFeed?.items, image: image)
    }
    // ViewのCellをクリックした際の処理
    func showLink(URL: URL) {
        router.presentWebView(URL: URL)
    }
}

extension MainPresenter: MainInteractorOutput {
    // Interactorへ依頼された処理の結果を受けています
    func RSSGeted(_ feed: Feed) {
        editingFeed = feed
        updateView()
    }

    func faild(_ error: String) {
        print(error)
    }
}
```

### View
 Viewは、画面更新など純粋なUIKitの処理とPresenterへ処理を依頼する事のみを担当します。
#### MainViewInput protocol
 ViewInputには、Presenterから描画したいRSSを受け取る関数を追加します。
```
//  MainMainViewInput.swift
import FeedKit

protocol MainViewInput: class {
    func setupInitialState()
    func showRSS(items: [RSSFeedItem]?, image: UIImage?)
}
```

#### MainViewOutput protocol
 ViewOutputには、セルがクリックされた場合にインアップブラウザを表示する処理を追加します。
```
//  MainMainViewOutput.swift
import UIKit

protocol MainViewOutput {
    func viewIsReady()
    func showLink(URL: URL)
}

```

#### 最後にViewControllerからPresenterへ依頼する
 最後に、ViewContollerから、Presenterの各処理を呼び出しを追加します。
こうすることでViewControllerは、UIKit関連の処理をシンプルに扱うだけとなります。
```
//  MainMainViewController.swift
import UIKit
import FeedKit
import SafariServices

class MainViewController: UITableViewController {
    var items: [RSSFeedItem]? = nil
    var image: UIImage? = nil
    var output: MainViewOutput!

    // MARK: Life cycle
    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.register(UITableViewCell.self
          , forCellReuseIdentifier: "cell")
        tableView.tableFooterView = UIView()
        output.viewIsReady()
    }
}

extension MainViewController: MainViewInput {
    // MARK: MainViewInput
    func setupInitialState() {}

    func showRSS(items: [RSSFeedItem]?, image: UIImage?) {
        self.items = items
        self.image = image
        DispatchQueue.main.async {
            self.tableView.reloadData()
        }
    }
}

extension MainViewController {
    ...

    override func tableView(_ tableView: UITableView,
       didSelectRowAt indexPath: IndexPath) {

        guard let link = items?[indexPath.row].link else {
            return
        }

        guard let URL = URL(string: link) else {
            return
        }

        output.showLink(URL: URL)
    }
}
```

### Entity
 Entityは、データ構造のStructやクラスを担当します。今回は、FeedKitのFeedクラスを利用している為、省略しています。

## テストを作成
 generambaのセットアップした時にあるように、生成されたVIPERのファイルへ対応したテストファイルが生成されています。
今回は、InteractorTestを修正して、だだしくRSSを取得する関数が実行されるかのテスト追加します。
```
//  MainMainInteractorTests.swift
import XCTest
import FeedKit
@testable import swift_rss_sample

class MainInteractorTests: XCTestCase {
    var interactor: MainInteractor!
    var presenter: MockPresenter!
    var remoteDataManager: MockRemoteDataManager!

    override func setUp() {
        super.setUp()
        remoteDataManager = MockRemoteDataManager()
        presenter = MockPresenter()
        interactor = MainInteractor(remoteDataManager: remoteDataManager)
        interactor.output = presenter
    }
    ...
    func test_getCall() {
        interactor.fetchRSS()
        XCTAssert(presenter.RSSGetedCalled)
    }

    class MockPresenter: MainInteractorOutput {
        var RSSGetedCalled = false
        var failedCalled = false

        func RSSGeted(_ feed: Feed) {
            RSSGetedCalled = true
        }

        func faild(_ error: String) {
            failedCalled = true
        }
    }
    // 前出した通り、DataManagerProtocolを検証してモックを作成する
    class MockRemoteDataManager: RemoteDataManagerProtocol {
        func get(completion: ((Feed) -> Void)?,
          fail: ((String) -> Void)?) {
            completion?(Feed.rss(RSSFeed()))
        }
    }
}
```

## まとめ
今回は、ざっくりとした解説となりますが、いかがでしょうか？
約100行のコードのサンプルを題材とした為、リファクタリングでコード量は増加しています。ですが、可読性はかなり向上し、いつでも拡張できる状態に出来ています。
また、VIPERはライブラリへ依存しないピュアなアーキテクチャです。ライブラリの更新に悩むこともないでしょう。
ぜひ、リファクタリング、リアーキテクチャに悩んでいる方は、選択肢へ入れてみてはどうでしょうか？
実装をした結果はGitHubを確認できます。(https://github.com/honda-n/swift-viper-rss)
長くなりましたが、ありがとうございました！
