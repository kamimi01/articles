---
title: "ipatool で IPA ファイルを App Store からダウンロードする"
emoji: "🔧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["iOS", "Swift"]
published: true # trueを指定する
published_at: 2023-07-31 18:30 # 未来の日時を指定する
---

IPA ファイルを App Store からダウンロードしたいこと、あると思います。

自分が開発に関わっているアプリであれば自分で IPA ファイルを作成できるので問題ないですが、 **関わっていないまたは関わっているけどなんらかの理由で IPA ファイルを作成できない** 場合もあります。

今回は [ipatool](https://github.com/majd/ipatool) というコマンドラインツールを使って自分が開発に関わっていないアプリの IPA ファイルの取得を行ってみます。

このツールを初めて使う人のため、ツールの導入方法から実際にダウンロードする方法について説明します。

## ipatool とは

MIT ライセンスで公開されているコマンドラインツールです。App Store にある iOS アプリの検索と IPA ファイル形式でアプリのコピーをダウンロードすることができます。

https://github.com/majd/ipatool

## 導入

方法は２つあります。

1. [GitHub のリリースノート](https://github.com/majd/ipatool/releases)から取得する
2. homebrew を使用してインストールする

ここでは後者について説明します。

と言っても、以下のコマンドを実行するだけです。

```shell
brew tap majd/repo
brew install ipatool
```

## 使い方

### App Store の認証

まず App Store の認証を行う必要があります。

以下を実行します。

```shell
ipatool auth --email "mail@example.com" --password "password"
```

すると2段階認証のためのコードが求められますので、入力して進むと認証完了です。

![](https://storage.googleapis.com/zenn-user-upload/248644cfe569-20230731.png)

### App Store 内のアプリ検索

以下のコマンドで検索することができます。

```shell
ipatool search --limit 1 "検索したいワード"
```

取得できる情報は以下のとおりです。

- Bundle ID
- Apple ID（アプリの公開リンクについているやつ e.g. `1673161138`）
- アプリ名
- 価格
- バージョン

試しに今話題の Twitter こと X を調べてみました。（つい最近アプリ名が X に変わったので）

![](https://storage.googleapis.com/zenn-user-upload/09793b7de911-20230731.png)

アプリ名はちゃんと「X」であることがわかりました。

### IPA ファイルのダウンロード

では本題の IPA ファイルのダウンロードです。
コマンドは以下のとおりです。

:::message
コマンドの引数として `--bundle-identifier` が必要なのでダウンロードしたいアプリの Bundle ID がわかっている必要があります。
わからない場合、前述の「App Store 内のアプリ検索」に書いた方法で知ることができます。
:::

```shell
ipatool download --bundle-identifier <ダウンロードしたいアプリの Bundle ID> --output <出力したいファイル名>.ipa
```

Twitter アプリの場合、以下のコマンドで IPA ファイルをダウンロードすることができました。

![](https://storage.googleapis.com/zenn-user-upload/f83615493c17-20230731.png)

ちなみに `--bundle-identifier` の代わりに `-b`、
`--output` の代わりに `-o` を使っても同じです。今回は長い方を使っていますが、普段は短い方を使った方が便利だと思います。

念の為、ダウンロードした IPA ファイルを Zip ファイルに直して中身を見てみました。

![](https://storage.googleapis.com/zenn-user-upload/aab3a2f1471b-20230731.png)

見覚えのある階層になっていたので iOS エンジニアのよく知る IPA ファイルが取得できていると考えて良さそうです。

## 参考

https://github.com/majd/ipatool

https://onejailbreak.com/blog/ipatool/

## おわりに

今回は紹介を省きましたが、ipatool の README には`purchase`というアプリのライセンスを取得するコマンドも用意されていましたので、気になる方は見てみてください。

自分で開発していないアプリの IPA ファイルを App Store からダウンロードできることを知らなかったので、何か困ったときの選択肢が一つ増えました。

この記事がお役に立てば幸いです。