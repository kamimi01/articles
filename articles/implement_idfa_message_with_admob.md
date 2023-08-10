---
title: "SwiftUI と User Messaging Platform (UMP) で IDFA メッセージを表示する"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS", "Swift", "SwiftUI", "Admob", "AppTrackingTransparency"]
published: false
---

最近個人開発アプリに Admob を導入しました。

その際に Admob が提供する IDFA メッセージを使って、iOS の ATT ポップアップが表示される前にプレポップアップを表示するように実装してみました。💬

この記事では、SwiftUI ベースのアプリで IDFA メッセージに対応する方法を説明します。

## IDFA メッセージ とは

IDFA メッセージとは Google Admob が提供しているベータ機能で、**iOS の App Tracking Transparency（以下 ATT）のポップアップの前に表示されるプレポップアップ**です。

ベータ版と書いてあるのですが、少なくとも 2 年前から存在していると思われます。（ちゃんと調べてはいないですが 2 年前に書かれた記事を発見したため）

https://support.google.com/admob/answer/10115027?hl=ja

動画の方がわかりやすいと思うのでこちらをご覧ください。
「アプリでは広告を表示しています」のテキストと緑のボタンが表示されているポップアップが IDFA メッセージです。

![b026fdfea470db725f71203ade2ba422](https://github.com/kamimi01/articles/assets/47489629/d024c99f-f51b-4e98-93c3-1e6d0dbdf459 =300x)

この IDFA メッセージつまりプレポップアップを表示するメリットについて説明します。

プレポップアップを表示する目的は **ATT 対応をすると表示される ATT ポップアップでの許諾率を上げること**です。

iOS 14.5 から必須になった ATT 対応によって多くのアプリでポップアップが表示されるようになりましたが、この許諾率はあまり高くありません。
そのため、アプリによっては許諾率を上げるための工夫をしています。その代表的な方法の一つが、ATT ポップアップの前に情報を補足するための独自のポップアップを表示することになります。

そのほかにも方法はいくつかありますがここでは説明しません。詳しくは AppsFlyer が出している『[ATTオプトイン率を上げる5つの方法](https://www.appsflyer.com/ja/blog/trends-insights/increase-att-opt-in-rates/)』の記事がおすすめです。

Admob ではこのプレポップアップを用意してくれており、それが IDFA メッセージです。

UI はある程度決まっており、カスタマイズできることとできないことがあります。
以下がカスタマイズのための設定画面です。

![](https://storage.googleapis.com/zenn-user-upload/800a5716fd1c-20230808.png =700x)

主に以下のようなことをカスタマイズできます。

- ポップアップ全体のスタイル
  - 角丸、背景色など
- 表示するテキストの内容
  - 上と下。どちらかも必須で必要。
- 表示するテキストのスタイル
  - フォント、大きさ、色、スタイル、配置など
- ボタンのスタイル
  - フォント、大きさ、色、スタイル、配置など

### IDFA メッセージのメリット・デメリット

IDFA メッセージを表示するメリットとデメリットについて簡単に紹介します。

#### メリット

**プレポップアップのデザインと実装や ATT ポップアップ対応の実装の工数を減らせる**、ことが最も大きなメリットだと思っています。

プレポップアップのデザインは前述の画面で GUI で操作して簡単に作ることができるので、デザイナーじゃなくても簡単に準備できます。
また Admob が提供する関数を使用することで、 ATT フレームワークを使用した実装やプレポップアップの表示タイミングなども自分で SDK 側でよしなに行ってくれます。

詳しい実装に関しては、後述します。

#### デメリット

プレポップアップの画面は簡単に作ることができますが、できないこともあります。
例えばプレポップアップを表示するアプリでは、よく次に表示される ATT ポップアップのイメージ画像をプレポップアップに表示していることがあります。

ですが、IDFA メッセージでは表示できるのは**基本的にテキストのみ**です。
そのため画像や動画を含めるといった、リッチなプレポップアップを表示したい場合は、IDFA メッセージの使用は向いていません。

### 事前準備

Admob を使った実装を行う前にいくつかの準備が必要です。
以下のステップ１からステップ２までを事前に行なってください。

https://firebase.google.cn/docs/admob/ios/quick-start?hl=ja

## 実装方法

では早速実装に入ります。
主に UserMessagingPlatform（以下 UMP）フレームワークのメソッドを使用して実装します。

公式のドキュメントは以下です。

https://developers.google.com/admob/ios/privacy?hl=ja

ドキュメントではコールバックを使用した実装が紹介されていますが、それだとネストが深くなりがちで可読性が良くないという問題があります。
幸いにも今回使用するメソッドは全て Swift Concurrency に対応しているので、この記事ではそちらを使います。

### 1. SDK をプロジェクトに追加する

Swift Package Manager を使用して、SDK を追加します。
検索するときの URL は以下です。

```
https://github.com/googleads/swift-package-manager-google-mobile-ads.git
```

### 2. 同意情報をリクエストする

これから実装をしていくのですが、最終的には Admob の初期化メソッドである`GADMobileAds.sharedInstance().start()` を呼ぶことになります。

このメソッドは[こちら](https://firebase.google.com/docs/admob/ios/quick-start?hl=ja)のドキュメントによると、

> このメソッドの呼び出しはできるだけ早い段階で 1 度だけ行います。**できればアプリの起動時、かつ Firebase の初期化後に呼び出してください。**

ということなので、`AppDelegate`クラスを実装してその中で今回の実装をすることにしました。

まずは同意情報のリクエストを行います。

```swift
import UserMessagingPlatform

private func setupAdmobIfPossible() async throws {
    // UMPRequestParameters オブジェクトを作成
    let parameters = UMPRequestParameters()
    // 未成年の同意のためのタグを設定する。false ならユーザーが同意年齢を満たしていない。
    parameters.tagForUnderAgeOfConsent = false

    // 同意情報の更新をリクエストする
    try await UMPConsentInformation.sharedInstance.requestConsentInfoUpdate(with: parameters)

    let formStatus = UMPConsentInformation.sharedInstance.formStatus
    if formStatus != .available {
        throw NSError()  // 仮の実装なので実際は独自の Error を用意するなどして対応してください
    }
}
```

同意情報の更新のリクエスト用メソッドを呼び出します。これを呼び出すと、`formStatus`が`available`になります。念の為、メソッドの呼び出し後に`available`であるかどうかのチェックを行っています。

### 3. 同意フォームを読み込み、表示する

引き続き、`setupAdmobIfPossible()`メソッドを実装していきます。

```swift
private func setupAdmobIfPossible() async throws {
    // 2. の実装は省略

    try await loadAndPresentIfPossible()
}
```

以下のように新しく`loadAndPresentIfPossible()`という関数を実装し、呼び出します。

```swift
@MainActor
private func loadAndPresentIfPossible() async throws {
    guard let rootViewController = UIApplication.shared.rootViewController else {
        throw NSError()  // 仮の実装なので実際は独自の Error を用意するなどして対応してください
    }
    try await UMPConsentForm.loadAndPresentIfRequired(from: rootViewController)
}
```

ここでは rootViewController を取得して、UMP フレームワークが提供する`loadAndPresentIfRequired(from:)`のメソッドの引数に渡して呼び出しています。
ドキュメントコメントに記載がありますが、`loadAndPresentIfRequired(from:)`はメインスレッドで呼び出す必要があります。そのため`@MainActor`を付与してメインスレッドで実行するようにしました。

> Loads a consent form and immediately presents it from the provided viewController if UMPConsentInformation.sharedInstance.consentStatus is UMPConsentStatusRequired. Calls completionHandler after the user selects an option and the form is dismissed, or on the next run loop if no form is presented. **Must be called on the main queue.**

### 4. Admob の初期化

最後に一番やりたかった Admob の初期化を行います。
その前に、UMP フレームワークが提供する `canRequestAds`の値が`true`であることを確認してから呼び出します。

```swift
// Admob の初期化用メソッドを呼び出すために必要
import GoogleMobileAds

private func setupAdmobIfPossible() async throws {
    // 2 と 3 の実装は省略

    if UMPConsentInformation.sharedInstance.canRequestAds == false {
        throw NSError()  // 仮の実装なので実際は独自の Error を用意するなどして対応してください
    }

    await GADMobileAds.sharedInstance().start()
}
```

### 5. `AppDelegate`で呼び出す

最後に、前述したように`AppDelegate.swift`で実装した関数を呼び出します。
ドキュメントにも記載があるように、Firebase の初期化後に呼び出しを行っています。

```swift
class AppDelegate: NSObject, UIApplicationDelegate {
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {

        // Firebase の初期化設定
        FirebaseApp.configure()

        // Admob の初期化設定
        Task {
            do {
                await setupAdmobIfPossible()
            } catch {
                print(error.localizedDescription)
            }
        }

        return true
    }
}
```

### コード全体

最後に実際のアプリで使っているコード全体を載せておきます。
前述のコードは説明のために簡略化していたので、以下とは少し違う箇所があります。ご了承ください。🙏🏻

```swift
import GoogleMobileAds
import UserMessagingPlatform

struct AdmobManager {
    static func configure() {
        Task {
            await setupAdmobIfNeeded()
        }
    }

    private static func setupAdmobIfNeeded() async {
        do {
            try await presentFormIfPossible()
            await setupAdmob()
        } catch {
            print(error.localizedDescription)
        }
    }

    private static func presentFormIfPossible() async throws {
        let parameters = UMPRequestParameters()
        parameters.tagForUnderAgeOfConsent = false

        try await UMPConsentInformation.sharedInstance.requestConsentInfoUpdate(with: parameters)

        let formStatus = UMPConsentInformation.sharedInstance.formStatus
        if formStatus != .available {
            throw UMPError.formStatusIsNotAvailable(formStatus)
        }

        try await loadAndPresentIfPossible()

        if UMPConsentInformation.sharedInstance.canRequestAds == false {
            throw UMPError.cannotRequestAds
        }
    }

    @MainActor
    private static func loadAndPresentIfPossible() async throws {
        guard let rootViewController = UIApplication.shared.rootViewController else {
            throw UMPError.cannotGetRootViewController
        }
        try await UMPConsentForm.loadAndPresentIfRequired(from: rootViewController)
    }

    private static func setupAdmob() async {
        await GADMobileAds.sharedInstance().start()
    }

    private enum UMPError: Error {
        /// formStatus が available ではない
        case formStatusIsNotAvailable(_ formStatus: UMPFormStatus)
        /// ads をリクエストできない
        case cannotRequestAds
        /// rootViewController を取得できない
        case cannotGetRootViewController
    }
}
```

呼び出し元の`AppDelegate`は以下のようになっています。

```swift
class AppDelegate: NSObject, UIApplicationDelegate {
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil) -> Bool {

        // Firebase の初期化設定
        FirebaseApp.configure()

        // Admob の初期化設定
        AdmobManager.configure()

        return true
    }
}
```

GitHub は[こちら](https://github.com/kamimi01/MeetupReminder/blob/main/MeetupReminder/Advertisement/GoogleAdmob/AdmobManager.swift)（`AdmobManager.swift`） や [こちら](https://github.com/kamimi01/MeetupReminder/blob/ebfe8da437bc635567d1c9151cdbc0b72a363649/MeetupReminder/MeetupReminderApp.swift#L11-L22)(`MeetupReminderApp.swift`)です。

### 参考

公式ドキュメント
（サンプルコードが載っているドキュメントが複数ありました。どちらでもうまくいくと思うのですが、おそらく前者のスタートガイドの方がよりシンプルな書き方だと思います。）

https://developers.google.com/admob/ios/privacy?hl=ja

https://developers.google.com/interactive-media-ads/ump/ios/quick-start?hl=ja

以下は SwiftUI でするときに参考にしました。

https://stackoverflow.com/questions/65567571/how-to-integrate-tracking-transparency-and-eu-consent-in-google-admob-in-swi

https://zenn.dev/hituziando/articles/3938fac7588853

https://qiita.com/hyuga_amazia/items/bc8e393e10e6136c8753

## おわりに

SwiftUI を使用したアプリで、IDFA メッセージを実装してみました！

自分で直接 ATT フレームワークを使用しなくても、UMP フレームワークを使うことで、プレポップアップと ATT のポップアップの表示の両方を実装することができました。またプレポップアップの画面のカスタマイズもとても簡単でした。

この記事がどなたかのご参考になれば幸いです。😊