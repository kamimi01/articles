# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a Japanese technical writing repository containing articles published on Zenn and other platforms. The codebase focuses on content creation and quality assurance for technical articles written in Japanese.

## Common Commands

### Content Management
- `npx zenn new:article` - Create a new Zenn article
- `npx zenn new:book` - Create a new Zenn book
- `npx zenn preview` - Start local preview server for Zenn content

### Quality Assurance
- `npx textlint articles/` - Lint all articles in the articles directory
- `npx textlint articles-not-zenn/` - Lint articles in the non-Zenn directory
- `npx textlint --fix articles/` - Auto-fix textlint issues where possible

## Repository Structure

The repository is organized into content categories:

- `articles/` - Articles published on Zenn platform
- `articles-not-zenn/` - Articles for other platforms or unpublished content
- `books/` - Zenn books (currently empty)

## Configuration

### Textlint Configuration
The repository uses comprehensive Japanese technical writing rules via `.textlintrc`:
- `preset-ja-technical-writing` - Japanese technical writing standards with customizations for punctuation and doubled particles
- `preset-ja-spacing` - Spacing rules between half-width and full-width characters, with special handling around code blocks
- Custom rules allow full-width question/exclamation marks and colons as period marks

### Content Guidelines
Articles are written in Japanese and follow technical writing conventions. The textlint configuration enforces consistent spacing, punctuation, and style across all content.

## 記事の特徴

既存記事の分析に基づく特徴:

### 技術的な焦点
- 主要トピック: iOS/Swift開発、プライバシー/広告関連（App Tracking Transparency、Admob）、UI/UX実装
- 実践的な実装例を含む深い技術的内容
- アプリ開発者向けの実世界のソリューションに焦点

### 執筆スタイル
- 包括的で詳細な説明（記事は通常250-300行）
- 明確な問題提起と解決策を持つ教育的アプローチ
- 頻繁に使用される要素:
  - シンタックスハイライト付きのコード例
  - コンセプトを説明するビジュアル（スクリーンショット、図表）
  - 明確な構成のためのセクションヘッダー
  - 重要な情報のための箇条書きと番号付きリスト
  - 外部参考資料とドキュメントリンク

### コンテンツ構造
- 問題や動機を説明する明確な導入
- 複雑なトピックのための背景/コンテキストセクション
- ステップバイステップの実装ガイド
- 実践的な考慮事項とエッジケース
- 追加リソースを含むまとめ/結論

### 日本語の文体特徴

#### 文末表現
- 一貫して**です・ます調**（丁寧語）を使用
- よく使われる文末パターン:
  - 「～と思います」「～と思っています」（I think...）
  - 「～ことにしました」（I decided to...）
  - 「～してみました」（I tried...）
  - 「～かなと思いました」（I thought it might be...）
  - 「～できるようになりました」（became able to...）

#### 特徴的な表現
- 「ということで」（therefore/so） - 場面転換によく使用
- 「残念ながら」（unfortunately）
- 「正直に言うと」（honestly speaking）
- 「～について」「～に関して」（about/regarding）- トピック導入時
- 括弧を使った補足表現：「（と思います。。）」

#### 助動詞の使用
- 「～てみる」（try doing）: 「作ってみました」「してみました」
- 「～ていく」（ongoing action）: 「実装していきます」「説明していきます」
- 「～ておく」（preparatory action）: 「書き残そうと思います」

### トーン
- プロフェッショナルでありながら親しみやすい（親しみやすい敬語）
- 時に自己批判的（「面倒だったからです。。」）
- 戦略的な絵文字の使用（🙏🏻、😊、🎉）
- 技術的正確性と親しみやすさのバランス