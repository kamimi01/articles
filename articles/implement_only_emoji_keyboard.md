---
title: "SwiftUI と UITextField でなるべく自然な絵文字入力専用ビューを実装する"
emoji: "🧜🏻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Swift", "iOS", "SwiftUI", "UITextField"]
published: true
published_at: 2023-08-13 11:30 # 未来の日時を指定する
---

SwiftUI で個人開発アプリを作っている中で、**絵文字だけ１文字だけを入力できるテキストフィールド**が欲しいと思ったので作ってみました。

SwiftUI の TextField で実装したかったですが難しそうだとわかったので、UIKit の UITextField を使いました。

## できること

以下の３つができるようにしました。

- テキストフィールドを押下すると、絵文字のキーボードが開く
- テキストフィールドに入力できるのは、絵文字だけ
- テキストフィールドに入力できるのは、１文字だけ

完成形はこちらです。

![Simulator Screen Recording - iPhone 14 Pro - 2023-08-13 at 05 21 55](https://github.com/kamimi01/articles/assets/47489629/47293ec2-38f4-4d14-824e-a559f1242779 =300x)


## 実装

### 1. 絵文字キーボードが開くようにする

絵文字専用のテキストフィールドなので、開くキーボードは絵文字のキーボードになるようにします。
`textInputMode` をオーバーライドして、`primaryLanguage` が `emoji`　の場合の `UITextInputMode` が返るようにします。

```swift:EmojiTextField.swift
class EmojiTextField: UITextField {
    override var textInputContextIdentifier: String? { "" }

    override var textInputMode: UITextInputMode? {
        for mode in UITextInputMode.activeInputModes where mode.primaryLanguage == "emoji" {
            return mode
        }
        return nil
    }
}
```

### 2. 絵文字だけを入力可能にする

次に絵文字だけを入力可能にする実装です。

`Character` を拡張して、絵文字かどうかを判定する変数 `isEmoji` を実装します。

```swift:Character+Extension.swift
extension Character {
    var isEmoji: Bool {
        guard let scalar = unicodeScalars.first else { return false }
        return scalar.properties.isEmoji && (scalar.value > 0x238C || unicodeScalars.count > 1)
    }
}
```

次に `String` を拡張して、先ほど実装した `isEmoji` を使って絵文字の文字列だけを返す関数を実装します。

```swift:String+Extension.swift
extension String {
    func onlyEmoji() -> String {
        return self.filter({ $0.isEmoji })
    }
}
```

### 3. `UIViewRepresentable`で`UITextField`をつくる

最後に `UIViewRepresentable` を使って、`UITextField` を実装していきます。

ポイントは、`UITextFieldDelegate` の `textFieldDidChangeSelection(_:)` を使用し、テキストの変更を検知して絵文字だけがテキストフィールドに表示されるようにしていることです。

```swift:OneEmojiTextField.swift
struct OneEmojiTextField: UIViewRepresentable {
    @Binding var inputText: String
    let fontSize: CGFloat

    func makeUIView(context: Context) -> UITextField {
        let textField = EmojiTextField()
        textField.font = .systemFont(ofSize: fontSize)
        textField.delegate = context.coordinator
        return textField
    }

    func updateUIView(_ uiView: UITextField, context: Context) {}
}

extension OneEmojiTextField {
    func makeCoordinator() -> Coordinator {
        Coordinator(self)
    }

    class Coordinator: NSObject, UITextFieldDelegate {
        var parent: OneEmojiTextField

        init(_ control: OneEmojiTextField) {
            self.parent = control
            super.init()
        }

        func textFieldDidChangeSelection(_ textField: UITextField) {
            guard let profileImageEmoji = textField.text else { return }

            // 絵文字だけ、１文字だけ表示されるように入力制限する
            let emojiText = String(profileImageEmoji.onlyEmoji().prefix(1))
            textField.text = emojiText
            parent.inputText = emojiText
        }
    }
}
```

---

以下のように SwiftUI の View から呼び出して使います。

```swift
OneEmojiTextField(inputText: $profileImageEmoji, fontSize: 80)
```

### おまけ

より自然な見た目にするための工夫やエッジケースの対応方法について考えてみました。

#### 一般的なテキストフィールド感をなくす

絵文字１文字を表示するためにテキストフィールドを使っていることに少し違和感がありました。
かといって自分で絵文字の選択肢を用意して独自の View を準備するのは色々大変です。（何か良いライブラリなどはあるのかもしれませんが。。）

そこで２つ改善をしてみました。

正直テキストフィールド感を完全に無くせているわけではないので微妙な改善もあるかもしれないですが、あくまで絵文字キーボードを使いたいという前提のもと対応してみました。

##### 見た目の改善

そこでキャレットや範囲選択など、一般的なテキストフィールドの見た目や操作を制限してみました。
具体的には[こちら](https://qiita.com/Simmon/items/f9d60ab51cc6b0b4b3bc)を参考に、`EmojiTextField` クラスに以下の実装をしてみました。

```swift:EmojiTextField.swift
class EmojiTextField: UITextField {
    // 省略

    // 入力カーソル非表示
    override func caretRect(for position: UITextPosition) -> CGRect {
        return .zero
    }

    // 範囲選択カーソル非表示
    override func selectionRects(for range: UITextRange) -> [UITextSelectionRect] {
        return []
    }

    // コピー・ペースト・選択等のメニュー非表示
    override func canPerformAction(_ action: Selector, withSender sender: Any?) -> Bool {
        return false
    }
}
```

こうすることで、普通のテキストフィールド感をなくすことができた気がします。

##### 挙動の改善

すでに絵文字が入力されている状態から別の絵文字に変更したいときはキーボードの x ボタンで削除しないと変えることができません。
これはテキストフィールドでできているので当たり前のことですが、絵文字設定用 View と考えるとこれは少し不自然な挙動かなと思いました。

別の絵文字ボタンが押されたらその絵文字に切り替わって欲しいです。

ということで `textFieldDidChangeSelection(_:)` の実装を以下のように修正しました。

```swift
func textFieldDidChangeSelection(_ textField: UITextField) {
    guard let profileImageEmoji = textField.text else { return }

    var emojiText: String {
        let tmpEmojiText = profileImageEmoji.onlyEmoji()
        // 1 文字以上入力された場合は最後に入力された絵文字を表示する
        if tmpEmojiText.count > 1 {
            return String(tmpEmojiText.suffix(1))
        }
        return String(tmpEmojiText.prefix(1))
    }
    textField.text = emojiText
    parent.inputText = emojiText
}
```

これで絵文字の変更も一般的なテキストフィールドよりも簡単に行うことができるようになりました。

ちなみにデフォルトの絵文字キーボードを使っていれば、絵文字の検索もできます。私は絵文字をテキストで検索することがけっこうあるので、これが使えるのは大きいです。

#### キーボードのモードに絵文字が存在しない場合の対応

キーボードの選択肢に絵文字がないケース、つまり絵文字キーボードが表示できないケースを考えてみます。
デフォルトでは絵文字キーボードは表示できるようになっているので多くないケースだと思いますが、設定アプリから表示できないようにすることができます。

![](https://storage.googleapis.com/zenn-user-upload/01321a98933a-20230813.png =300x)

ここから削除されている場合、前述の実装だと絵文字キーボードは表示されません。

試しに `EmojiTextField` クラスの `textInputMode` の実装を以下のように書き換えて、`activeInputMode` の中身を見てみます。

```swift:EmojiTextField.swift
class EmojiTextField: UITextField {
    override var textInputContextIdentifier: String? { "" }

    override var textInputMode: UITextInputMode? {
        for mode in UITextInputMode.activeInputModes {
            print("activeInputModes:", mode.primaryLanguage)
            if mode.primaryLanguage == "emoji" {
                return mode
            }
        }
        return nil
    }
}
```

絵文字キーボードがある場合の出力。

```
activeInputModes: Optional("ja-JP")
activeInputModes: Optional("ja-JP")
activeInputModes: Optional("en-JP")
```

絵文字キーボードがない場合の出力。

```
activeInputModes: Optional("ja-JP")
activeInputModes: Optional("ja-JP")
activeInputModes: Optional("en-JP")
activeInputModes: Optional("emoji")
```

ということで`emoji` のモードがないので、絵文字キーボードが表示できません。

私のアプリではこの場合にはテキストフィールドではなく、SwiftUI のただの Text を表示することにしました。しかし画像を変更しようとタップするユーザーがいそうなので、そういったユーザー向けに絵文字キーボードの表示ができるように設定を変えることを促すのもよさそうです。

## 参考

https://stackoverflow.com/questions/66397745/how-to-make-sure-that-only-emoji-can-be-entered-in-the-textfield-swiftui

https://dev.classmethod.jp/articles/swiftui-textfield-dame-zettai/

https://dev.classmethod.jp/articles/swift-uitextfield-dame-zettai/

https://qiita.com/hcrane/items/ca5b1d6cbff57fe8fc9b

https://blog.studysapuri.jp/entry/2022/03/28/using-uikit-in-swiftui

https://qiita.com/Simmon/items/f9d60ab51cc6b0b4b3bc

## まとめ

絵文字だけ１文字だけを入力できるテキストフィールドを作ってみました。

絵文字キーボードを表示するという要件がなければ、SwiftUI の TextField を使って UITextField を取り出すことで実装できそうと考えたのですが、`textInputMode` がゲッターしかなくセッターがなく初期化にオーバーライドするしかなさそうだったので、UIKit の UITextField を使い、オーバーライドすることで実装しました。

他にももっとスマートな書き方があれば、ご教授いただけますとうれしいです。😊

この記事がどなたかの参考になれば幸いです。🧜🏻

追記：今回はライブラリなどは調べずに独自に方法を考えてみましたが、EmojiPicker というライブラリがとても使いやすかったです。非常に簡単に使えたのでおすすめです。

https://github.com/Kelvas09/EmojiPicker