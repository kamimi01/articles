---
title: "SwiftUI で絵文字だけ１文字だけが入力できるテキストフィールドを作る"
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

![Simulator Screen Recording - iPhone 14 Pro - 2023-08-13 at 00 21 54](https://github.com/kamimi01/articles/assets/47489629/4ed543a2-962b-4cdf-ab33-fd39f315ce7c =300x)

## 実装

### 1. 絵文字キーボードが開くようにする

絵文字専用のテキストフィールドなので、開くキーボードは絵文字のキーボードになるようにします。
`textInputMode` をオーバーライドして、`primaryLanguage` が `emoji`　の場合の `UITextInputMode` が返るようにします。

```swift
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

```swift
extension Character {
    var isEmoji: Bool {
        guard let scalar = unicodeScalars.first else { return false }
        return scalar.properties.isEmoji && (scalar.value > 0x238C || unicodeScalars.count > 1)
    }
}
```

次に `String` を拡張して、先ほど実装した `isEmoji` を使って絵文字の文字列だけを返す関数を実装します。

```swift
extension String {
    func onlyEmoji() -> String {
        return self.filter({ $0.isEmoji })
    }
}
```

### 3. `UIViewRepresentable`で`UITextField`をつくる

最後に、`UIViewRepresentable` を使って、`UITextField` を実装していきます。

ポイントは、`UITextFieldDelegate` の `textFieldDidChangeSelection(_:)` を使用し、テキストの変更を検知して絵文字だけがテキストフィールドに表示されるようにしていることです。

```swift
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

## 参考

https://stackoverflow.com/questions/66397745/how-to-make-sure-that-only-emoji-can-be-entered-in-the-textfield-swiftui

https://dev.classmethod.jp/articles/swiftui-textfield-dame-zettai/

https://dev.classmethod.jp/articles/swift-uitextfield-dame-zettai/

https://qiita.com/hcrane/items/ca5b1d6cbff57fe8fc9b

https://blog.studysapuri.jp/entry/2022/03/28/using-uikit-in-swiftui

## まとめ

絵文字だけ１文字だけを入力できるテキストフィールドを作ってみました。

絵文字キーボードを表示するという要件がなければ、SwiftUI の TextField を使って UITextField を取り出すことで実装できそうと考えたのですが、`textInputMode` がゲッターしかなくセッターがなくそれが難しそうに思えたので、UIKit の UITextField を使い、オーバーライドすることで実装しました。

他にももっとスマートな書き方があれば、ご教授いただけますとうれしいです。😊

この記事がどなたかの参考になれば幸いです。🧜🏻