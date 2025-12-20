# 所有権の大地 — Rust学習物語

> C/C++経験者のためのRust没入型学習コンテンツ（全8部）

**Live Site**: https://nao-amj.github.io/naochan-diary/rust-ownership-land/

---

## 運用フロー

```
master/*.md (正本)
    ↓ コピー + フロントマター追加
docs/rust-ownership-land/*.md (GitHub Pages用)
    ↓ Jekyll ビルド
https://nao-amj.github.io/naochan-diary/rust-ownership-land/

    ↓ gh issue edit --body-file
GitHub Issues #55-#63 (外部公開・議論用)
```

### Single Source of Truth

正本は `three_hearts_space/data/ctx/projects/rust_ownership_land/master/*.md`

### 同期コマンド

```bash
# 1. master/*.md を編集（正本）

# 2. docs同期（フロントマター付きでコピー）
cp master/*.md /path/to/naochan-diary/docs/rust-ownership-land/

# 3. Issue同期
gh issue edit 55 --repo nao-amj/naochan-diary --body-file master/55_intro.md
gh issue edit 56 --repo nao-amj/naochan-diary --body-file master/56_ownership.md
gh issue edit 57 --repo nao-amj/naochan-diary --body-file master/57_overview.md
gh issue edit 58 --repo nao-amj/naochan-diary --body-file master/58_borrowing.md
gh issue edit 59 --repo nao-amj/naochan-diary --body-file master/59_lifetime.md
gh issue edit 60 --repo nao-amj/naochan-diary --body-file master/60_error_handling.md
gh issue edit 61 --repo nao-amj/naochan-diary --body-file master/61_async.md
gh issue edit 62 --repo nao-amj/naochan-diary --body-file master/62_macro.md
gh issue edit 63 --repo nao-amj/naochan-diary --body-file master/63_module.md

# 4. コミット & プッシュ
git add -A && git commit -m "docs: 記事更新" && git push
```

---

## カスタムテーマ: Claude Code風

### 特徴

| 要素 | 実装 |
|------|------|
| フォント | SF Mono / Fira Code / JetBrains Mono |
| Dark背景 | #1a1a1a |
| Darkアクセント | #f97316 (オレンジ) |
| Light背景 | #ffffff |
| Lightアクセント | #d73a49 (赤) |
| テーマ切替 | 右上ボタン + localStorage保持 |

### ファイル構成

```
docs/
├── _layouts/
│   └── rust-land.html    # カスタムレイアウト（テーマ切替対応）
└── rust-ownership-land/
    ├── index.html        # 目次ページ（静的HTML）
    ├── _config.yml       # Jekyll設定
    ├── README.md         # このファイル
    ├── 55_intro.md       # 第1部
    ├── 56_ownership.md   # 第2部
    ├── 57_overview.md    # シリーズ概要
    ├── 58_borrowing.md   # 第3部
    ├── 59_lifetime.md    # 第4部
    ├── 60_error_handling.md  # 第5部
    ├── 61_async.md       # 第6部
    ├── 62_macro.md       # 第7部
    └── 63_module.md      # 第8部
```

### フロントマター

各.mdファイルには以下のフロントマターが必要：

```yaml
---
layout: rust-land
title: "第X部：タイトル"
---
```

---

## 章構成

| # | ファイル | タイトル | 称号 |
|:-:|:---------|:---------|:-----|
| 1 | 55_intro.md | 所有権の大地 | 禁域の帰還者 |
| 2 | 56_ownership.md | 内部可変性の迷宮 | 内部可変性の探究者 |
| 3 | 58_borrowing.md | トレイトの大聖堂 | 契約の理解者 |
| 4 | 59_lifetime.md | ジェネリクスの鋳造所 | 鋳型の職人 |
| 5 | 60_error_handling.md | エラーの峡谷 | エラーの調停者 |
| 6 | 61_async.md | 非同期の浮遊島 | 空の旅人 |
| 7 | 62_macro.md | マクロの魔術塔 | 魔術の継承者 |
| 8 | 63_module.md | クレートの港 | 港の航海士 |

**最終称号**: 所有権の大地の継承者

---

## 開発プロセス

このプロジェクトで実証された「dual-world開発プロセス」：

1. **構造設計** - 全体構成、伏線マップ設計
2. **初期実装** - 各部ドラフト作成
3. **Devil批評** - 厳しい視点からのレビュー
4. **体系的改善** - 優先度分類→順次対応
5. **統合確認** - 全体の繋がり確認

---

*dual-world framework による没入型Rust学習コンテンツ*
