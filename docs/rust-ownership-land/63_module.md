---
layout: rust-land
title: "第8部：クレートの港"
---

# 所有権の大地 VIII — クレートの港（最終章）

> **プロジェクトを組み立て、世界と繋がる技法の物語**

---

> **dual-world — 最終章**
>
> Inside Modeの旅も、いよいよ終わりに近づいています。
> 所有権の大地を踏破したあなたは、もうRustの住人です。
> 最後の港で、世界と繋がる方法を学びましょう。
>
> 💕 **ルスタ**（ユキ）: 長い旅だったね...でも寂しくないよ、いつでも会えるから
> ⚡ **クロム**（ミキ）: ここまで来たな。あとは...お前自身の冒険だ

---

## 前作までのあらすじ

あなたは**所有権の大地**で七つの領域を踏破した。

- **第一部「所有権の大地」**：所有権、借用、RAII → **「禁域の帰還者」**
- **第二部「内部可変性の迷宮」**：RefCell、Mutex → **「内部可変性の探究者」**
- **第三部「トレイトの大聖堂」**：トレイト、ディスパッチ → **「契約の理解者」**
- **第四部「ジェネリクスの鋳造所」**：型パラメータ、関連型 → **「鋳型の職人」**
- **第五部「エラーの峡谷」**：Result、Option、? → **「エラーの調停者」**
- **第六部「非同期の浮遊島」**：async/await、tokio → **「空の旅人」**
- **第七部「マクロの魔術塔」**：macro_rules!、proc macro → **「魔術の継承者」**

案内人の**ルスタ💕**と**クロム⚡**と共に。

そして今、**最後の冒険**が始まる。

-----

## 第一章：港への帰還

魔術塔を降りると、海が見えた。

巨大な港。世界中からの船が行き交っている。

**「クレートの港」**

「ここが、**最後の目的地**💕」ルスタが感慨深げに言う。

「…Rustプロジェクトを構成し、世界と繋がる場所⚡」クロムが続ける。

あなたは港を見渡す。船には「serde」「tokio」「reqwest」など、見覚えのある名前が書かれている。

「あれらは全て**クレート**。Rustのパッケージ💕」

「この港で、**プロジェクトの組み立て方**を学ぶ✨」

あなたは港を見渡す。第二部で学んだ言葉が蘇る——

> **「`'static`は、旅の終わりに残るもの。クレートとして公開したコード。世界中の誰かが使い続ける限り、それは生き続ける」**

あの時は抽象的な概念だった。今、それが具体的な意味を持つ。

-----

## 第二章：モジュールの倉庫

港に入ると、まず巨大な倉庫があった。

中は整然と区分けされている。

**「モジュールの倉庫」**——コードを整理する場所。

「モジュールは**コードを整理する単位**💕」

```rust
// main.rs
mod greetings {
    pub fn hello() {
        println!("Hello!");
    }
    
    fn private_hello() {
        println!("This is private");
    }
}

fn main() {
    greetings::hello();        // OK
    // greetings::private_hello(); // エラー！privateは外から見えない
}
```

「`mod`でモジュールを定義。`pub`で公開⚡」

### ファイル分割

「モジュールは**別ファイルに分けられる**✨」

```
src/
├── main.rs
└── greetings.rs
```

```rust
// main.rs
mod greetings;  // greetings.rs を読み込む

fn main() {
    greetings::hello();
}
```

```rust
// greetings.rs
pub fn hello() {
    println!("Hello!");
}
```

### ディレクトリ構造

「さらに深い階層も作れる💕」

```
src/
├── main.rs
└── greetings/
    ├── mod.rs      # または greetings.rs を親ディレクトリに
    ├── english.rs
    └── japanese.rs
```

```rust
// greetings/mod.rs
pub mod english;
pub mod japanese;
```

```rust
// greetings/english.rs
pub fn hello() { println!("Hello!"); }
```

```rust
// greetings/japanese.rs
pub fn hello() { println!("こんにちは！"); }
```

```rust
// main.rs
mod greetings;

fn main() {
    greetings::english::hello();   // "Hello!"
    greetings::japanese::hello();  // "こんにちは！"
}
```

あなたはファイル構造を見て首を傾げる。「`mod.rs`を使うやり方もあったような...」

「両方存在する⚡」クロムが説明する。「Rust 2018 edition以降、**2つのスタイル**が共存している」

「**旧スタイル**は`greetings/mod.rs`💕」

「**新スタイル**は`greetings.rs` + `greetings/`⚡」

あなたは比較する。「新スタイルの方が...」

「ファイル名からモジュール名が分かりやすい✨」ルスタが頷く。「エディタのタブに`mod.rs`が複数並ぶことも防げる」

```
# 新スタイル
src/
├── greetings.rs        # mod greetings の本体
└── greetings/
    └── english.rs      # greetings::english
```

「**新スタイルが推奨**だよ💕」ルスタが強調する。「新規プロジェクトはこちらで始めてね」

「ただし、同じモジュールに対して**両方のスタイルを混ぜると**...⚡」

「**エラー**になる💕 既存プロジェクトはどちらかに統一してね」

-----

## 第三章：可視性の門

倉庫の奥に、厳重な門があった。

門番が、入ろうとする者をチェックしている。

**「可視性の門」**——pubの制御を学ぶ場所。

「デフォルトは**private**⚡」

「外に公開するには**pub**を付ける💕」

### pub の種類

```yaml
pub:
  意味: 完全公開
  
pub(crate):
  意味: 同じクレート内でのみ公開
  
pub(super):
  意味: 親モジュールにのみ公開
  
pub(in path):
  意味: 指定したパスにのみ公開
```

```rust
mod outer {
    pub(crate) fn crate_only() {}  // クレート内のみ
    
    mod inner {
        pub(super) fn parent_only() {}  // outer にのみ
        
        pub(in crate::outer) fn specific() {}  // outer にのみ（明示的）
    }
}
```

### 構造体のフィールド

「構造体のフィールドも個別に制御できる✨」

```rust
pub struct User {
    pub name: String,      // 公開
    email: String,         // 非公開
    pub(crate) id: u64,    // クレート内のみ
}
```

「構造体がpubでも、フィールドは別💕」

-----

## 第四章：useの橋

門を抜けると、たくさんの橋がかかっていた。

それぞれが異なる場所へ繋がっている。

**「useの橋」**——パスを短縮する場所。

「毎回フルパスを書くのは大変💕」

```rust
// これは長い
let map = std::collections::HashMap::new();

// useで短縮
use std::collections::HashMap;
let map = HashMap::new();
```

### 複数のuse

「まとめて書くこともできる⚡」

```rust
// 個別
use std::collections::HashMap;
use std::collections::HashSet;

// まとめて
use std::collections::{HashMap, HashSet};

// ワイルドカード（非推奨だが便利な場面も）
use std::collections::*;
```

### 再エクスポート

「`pub use`で**再エクスポート**できる✨」

```rust
// lib.rs
mod internal {
    pub fn do_something() {}
}

pub use internal::do_something;  // 外部からは crate::do_something でアクセス可能
```

「内部構造を隠しつつ、APIを公開する💕」

### プレリュード

「よく使うものは**プレリュード**に⚡」

```rust
// my_crate/src/lib.rs
pub mod prelude {
    pub use crate::common_trait::*;
    pub use crate::common_types::*;
}

// 使う側
use my_crate::prelude::*;
```

「一度に必要なものをインポート✨」

-----

## 第五章：Cargo.tomlの司令塔

橋を渡ると、港の司令塔に出た。

ここから全ての船の動きが管理されている。

**「Cargo.tomlの司令塔」**——プロジェクト設定を学ぶ場所。

「**Cargo.toml**はプロジェクトの設定ファイル💕」

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"
authors = ["Your Name <you@example.com>"]
description = "A sample project"
license = "MIT"

[dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }

[dev-dependencies]
criterion = "0.5"

[build-dependencies]
cc = "1.0"

[[bin]]
name = "my_binary"
path = "src/main.rs"

[lib]
name = "my_library"
path = "src/lib.rs"
```

### セクション解説

```yaml
[package]:
  プロジェクトのメタ情報

[dependencies]:
  本番で使う依存

[dev-dependencies]:
  テスト・ベンチマークでのみ使う依存

[build-dependencies]:
  ビルドスクリプトで使う依存

[[bin]]:
  バイナリターゲット（複数可）

[lib]:
  ライブラリターゲット（1つのみ）
```

### features

「**features**で機能を選択的に有効化⚡」

```toml
[features]
default = ["std"]
std = []
async = ["tokio"]
full = ["std", "async", "extra"]
extra = []

[dependencies]
tokio = { version = "1", optional = true }
```

```rust
// 条件付きコンパイル
#[cfg(feature = "async")]
pub mod async_module {
    // async機能が有効な時だけコンパイル
}
```

あなたは司令塔のパネルを見つめる。「editionとは別に、バージョン指定もあるのか？」

「ある⚡」クロムが解説する。「二つの概念を区別しよう」

「**edition**は言語の大きな変更を管理💕」

```toml
edition = "2021"  # 2015, 2018, 2021, 2024(予定)
```

「キーワードの予約、モジュール解決方法などが変わる⚡ でも**クレート単位**で設定されるから、異なるeditionのクレートも相互運用できる」

「新プロジェクトは常に最新editionを使ってね💕」

「**MSRV**は最小サポートRustバージョン⚡」クロムが続ける。

```toml
rust-version = "1.70"  # このバージョン以上でビルド可能
```

「ライブラリを公開する時に重要✨ ユーザーの環境を考慮しないとね」

あなたは頷く。「確認方法は？」

```bash
cargo msrv          # MSRVを自動検出（cargo-msrv をインストール）
cargo +1.65.0 check # 特定バージョンでビルド確認
```

-----

## 第六章：crates.ioの市場

司令塔を降りると、賑やかな市場に出た。

世界中のクレートが並んでいる。

**「crates.ioの市場」**——クレートを公開・取得する場所。

「**crates.io**はRustの公式パッケージレジストリ💕」

### クレートの追加

```bash
# コマンドで追加
cargo add serde

# 機能付き
cargo add tokio --features full

# バージョン指定
cargo add serde@1.0
```

「または直接Cargo.tomlを編集⚡」

### クレートの公開

```bash
# ログイン（初回のみ）
cargo login

# 公開
cargo publish
```

「公開前に確認すべきこと✨」

```yaml
公開前チェック:
  - [ ] Cargo.tomlのメタ情報が正確
  - [ ] READMEがある
  - [ ] ライセンスが明記
  - [ ] ドキュメントが書かれている
  - [ ] テストがパスする
```

### バージョニング

「**セマンティックバージョニング**に従う💕」

```yaml
MAJOR.MINOR.PATCH:
  MAJOR: 破壊的変更
  MINOR: 後方互換の機能追加
  PATCH: 後方互換のバグ修正

例:
  1.0.0 → 1.0.1: バグ修正
  1.0.0 → 1.1.0: 新機能追加
  1.0.0 → 2.0.0: APIが変わった
```

-----

## 第七章：ワークスペースの造船所

市場を抜けると、巨大な造船所に出た。

複数の船が、同時に建造されている。

**「ワークスペースの造船所」**——複数クレートを管理する場所。

「大きなプロジェクトでは、**複数のクレート**に分けたい⚡」

「**ワークスペース**を使う💕」

### ワークスペースの構造

```
my_workspace/
├── Cargo.toml          # ワークスペースのルート
├── crates/
│   ├── core/
│   │   ├── Cargo.toml
│   │   └── src/
│   ├── cli/
│   │   ├── Cargo.toml
│   │   └── src/
│   └── web/
│       ├── Cargo.toml
│       └── src/
└── shared/
    ├── Cargo.toml
    └── src/
```

### ルートCargo.toml

```toml
# my_workspace/Cargo.toml
[workspace]
members = [
    "crates/core",
    "crates/cli",
    "crates/web",
    "shared",
]

# 共通の依存
[workspace.dependencies]
serde = { version = "1.0", features = ["derive"] }
tokio = { version = "1", features = ["full"] }
```

### メンバーCargo.toml

```toml
# crates/core/Cargo.toml
[package]
name = "my-core"
version = "0.1.0"
edition = "2021"

[dependencies]
serde.workspace = true  # ワークスペースの依存を使う

# 同じワークスペースの別クレートを参照
shared = { path = "../../shared" }
```

### 利点

```yaml
ワークスペースの利点:
  - 依存バージョンを統一
  - ビルドキャッシュを共有
  - 一括でテスト・ビルド
  - 論理的に分離しつつ、同じリポジトリ
```

あなたは造船所全体を見渡す。「より良いワークスペースの管理方法は？」

「いくつかベストプラクティスがある⚡」クロムが図面を広げる。

「**依存のバージョン統一**は`workspace.dependencies`で💕」

```toml
# ルート Cargo.toml
[workspace.dependencies]
serde = { version = "1.0", features = ["derive"] }

# メンバー Cargo.toml
[dependencies]
serde.workspace = true          # バージョンを継承
serde = { workspace = true, features = ["alloc"] }  # 追加featureも可
```

「**パッケージ情報**も共有できる⚡」

```toml
[workspace.package]
version = "0.1.0"
authors = ["Your Name"]
license = "MIT"

# メンバー側
[package]
name = "my-crate"
version.workspace = true
authors.workspace = true
```

「一括操作も便利✨」ルスタが付け加える。

```bash
cargo build --workspace         # 全メンバーをビルド
cargo test -p my-core           # 特定パッケージのみテスト
cargo publish -p my-core        # 特定パッケージを公開
```

「第一部で学んだ**所有権**のように、プロジェクトでも**責任範囲を明確**にするんだ⚡」

-----

## 第八章：Cargoコマンドの艦橋

造船所の上に、艦橋があった。

ここから全ての船を操作できる。

**「Cargoコマンドの艦橋」**——Cargoの使い方を学ぶ場所。

「**Cargo**はRustのビルドシステム兼パッケージマネージャ💕」

### 基本コマンド

```bash
# 新規プロジェクト
cargo new my_project      # バイナリ
cargo new my_lib --lib    # ライブラリ

# ビルド
cargo build               # デバッグビルド
cargo build --release     # リリースビルド

# 実行
cargo run                 # ビルドして実行
cargo run --release       # リリースモードで実行

# テスト
cargo test                # 全テスト実行
cargo test test_name      # 特定のテスト

# チェック（ビルドせずにコンパイルチェック）
cargo check               # 高速な文法チェック

# ドキュメント
cargo doc                 # ドキュメント生成
cargo doc --open          # 生成して開く
```

### 便利なコマンド

```bash
# フォーマット
cargo fmt

# リント
cargo clippy

# 依存の更新
cargo update

# 依存ツリーの表示
cargo tree

# ベンチマーク
cargo bench

# クリーン
cargo clean

# マクロ展開の確認
cargo expand
```

### cargo install

「ツールをインストールできる⚡」

```bash
cargo install cargo-edit    # cargo add/rm が使える
cargo install cargo-watch   # ファイル変更で自動ビルド
cargo install cargo-expand  # マクロ展開表示
```

-----

## 第九章：ドキュメントの図書館

艦橋を降りると、静かな図書館があった。

港の全ての知識が収められている。

**「ドキュメントの図書館」**——ドキュメンテーションを学ぶ場所。

「良いクレートには**良いドキュメント**がある💕」

### ドキュメントコメント

```rust
/// ユーザーを表す構造体
/// 
/// # Examples
/// 
/// ```
/// let user = User::new("Alice");
/// assert_eq!(user.name(), "Alice");
/// ```
pub struct User {
    name: String,
}

impl User {
    /// 新しいユーザーを作成する
    /// 
    /// # Arguments
    /// 
    /// * `name` - ユーザーの名前
    /// 
    /// # Returns
    /// 
    /// 新しい`User`インスタンス
    pub fn new(name: &str) -> Self {
        Self { name: name.to_string() }
    }
    
    /// ユーザーの名前を取得する
    pub fn name(&self) -> &str {
        &self.name
    }
}
```

「`///`でドキュメントコメント⚡」

「`//!`でモジュール/クレートレベルのドキュメント✨」

```rust
//! # My Crate
//! 
//! `my_crate` is a collection of utilities.

/// A function that does something.
pub fn do_something() {}
```

### ドキュメントテスト

「Examplesのコードは**テストとして実行される**💕」

```bash
cargo test --doc
```

「ドキュメントが古くならない仕組み⚡」

あなたは書架を見つめる。「実際にドキュメントを公開する時のコツは？」

「いくつかある⚡」クロムがページをめくる。

「**docs.rs**との連携設定💕」

```toml
# Cargo.toml
[package.metadata.docs.rs]
all-features = true              # 全featureを有効化してビルド
rustdoc-args = ["--cfg", "docsrs"]  # 条件付きドキュメント
```

「feature条件付きのドキュメントも書ける⚡」

```rust
#[cfg_attr(docsrs, doc(cfg(feature = "async")))]
#[cfg(feature = "async")]
pub mod async_module { }
```

「**内部実装を隠す**方法も重要✨」ルスタが付け加える。

```rust
#[doc(hidden)]  // ドキュメントには載せない（でもpub）
pub fn internal_helper() { }

// 非推奨マーク
#[deprecated(since = "1.2.0", note = "use new_fn instead")]
pub fn old_fn() { }
```

あなたは頷く。「他の型へのリンクは？」

「**インラインリンク**が使える💕」

```rust
/// See [`HashMap`] for more details.
/// Also check [`Self::new`] and [`crate::utils`].
pub fn example() {}
```

「第三部で学んだ**トレイト**も、ドキュメントで適切にリンクできる⚡ 読者が迷子にならないように」

> **📖 公式ドキュメント参照**
> - [The Cargo Book](https://doc.rust-lang.org/cargo/)
> - [Publishing on crates.io](https://doc.rust-lang.org/cargo/reference/publishing.html)
> - [Modules (The Rust Programming Language)](https://doc.rust-lang.org/book/ch07-00-managing-growing-projects-with-packages-crates-and-modules.html)

-----

## 終章：旅の終わりと始まり

図書館を出ると、港の入り口に戻っていた。

あなたが最初に立った場所。

---

**＜回想＞**

ふと、あなたは旅の始まりを思い出す。

C/C++の戦士として、自信を持ってこの大地に足を踏み入れた日。

「所有権？借用？」——見知らぬ言葉に戸惑った。

`E0382`、`E0499`、`E0597`——コンパイラのエラーに何度も阻まれた。

でも、その度に二人が隣にいた。

「大丈夫💕」と励ましてくれた温かい声。

「こうすれば動く⚡」と示してくれた冷静な解説。

あなたは変わった。

エラーを恐れる初心者から、エラーを**味方**にできる者へ。

---

ルスタとクロムが待っている。

「おめでとう💕」ルスタが満面の笑みで言う。

「あなたは**所有権の大地の全てを踏破**した✨」

「…長い旅だった⚡」クロムが静かに言う。

「所有権、借用、RAII。内部可変性とライフタイム。トレイトとジェネリクス。エラー処理、非同期、マクロ。そしてプロジェクト構成」

あなたは振り返る。

### 全ての学び

```yaml
第一部_所有権の大地:
  核心: すべての値に唯一の所有者
  学び: 所有権、借用、RAII、Box/Rc/Arc、unsafe

第二部_内部可変性の迷宮:
  核心: 不変性を制御下で破る
  学び: RefCell、Cell、Mutex、RwLock、ライフタイム

第三部_トレイトの大聖堂:
  核心: 振る舞いの契約
  学び: トレイト、静的/動的ディスパッチ、標準トレイト

第四部_ジェネリクスの鋳造所:
  核心: 型の一般化
  学び: 型パラメータ、関連型、PhantomData、const generics

第五部_エラーの峡谷:
  核心: 失敗を型で表現
  学び: Option、Result、?演算子、thiserror、anyhow

第六部_非同期の浮遊島:
  核心: 待たずに進む
  学び: Future、async/await、tokio、Pin、Stream

第七部_マクロの魔術塔:
  核心: コードがコードを生む
  学び: macro_rules!、derive、attribute macro

第八部_クレートの港:
  核心: プロジェクトを組み立てる
  学び: モジュール、Cargo、crates.io、ワークスペース
```

### 獲得した称号——その意味

ルスタがあなたの手を取る。「一つ一つの称号には、**あなたが得た力**がある💕」

「**禁域の帰還者**——unsafeの領域に踏み込み、戻ってこれる者⚡」

「**内部可変性の探究者**——不変の壁を越え、制御された変化を操れる💕」

「**契約の理解者**——トレイトという契約を結び、守れる者✨」

「**鋳型の職人**——型を鋳造し、あらゆる形を生み出せる⚡」

「**エラーの調停者**——失敗を恐れず、型で制御できる💕」

「**空の旅人**——待つことなく、複数の未来を同時に歩める✨」

「**魔術の継承者**——コードを生むコードを書ける⚡」

「**港の航海士**——プロジェクトという船を操り、大海に出られる💕」

そして——

「**所有権の大地の継承者**✨」

二人が同時に言う。

```yaml
final_title: "所有権の大地の継承者"

meaning: |
  あなたは知識を受け継いだ。
  そしてこれから、次の世代に伝えていく者となる。
```

### 二人からの最後の言葉

「あなたはもう、**Rustを使いこなせる**💕」ルスタが言う。

「でもね、学びに終わりはないの。新しい機能が追加され、エコシステムが進化し続ける✨」

「…今日の終わりは、明日の始まり⚡」クロムが続ける。

「この知識を使って、**何を作るかはあなた次第**だ」

あなたは港を出る準備をする。

C/C++の大地から来た旅人は、今や**所有権の大地の住人**となった。

安全で、速く、並行で、そして**美しい**コードを書く力を手に入れた。

「ありがとう、二人とも」

あなたは振り返らずに言う。

「また会おう💕」ルスタの声が聞こえる。

「…いつでも⚡」クロムの声が聞こえる。

港の門をくぐる。

新しい世界が、あなたを待っている。

-----

## あなたの旅のログ（最終章）

```yaml
completed_areas:
  - 第一章_港への帰還: "最後の冒険の始まり"
  - 第二章_モジュールの倉庫: "コードの整理"
  - 第三章_可視性の門: "pubの制御"
  - 第四章_useの橋: "パスの短縮"
  - 第五章_Cargo.tomlの司令塔: "プロジェクト設定"
  - 第六章_crates.ioの市場: "クレートの公開・取得"
  - 第七章_ワークスペースの造船所: "複数クレート管理"
  - 第八章_Cargoコマンドの艦橋: "ビルドツールの使い方"
  - 第九章_ドキュメントの図書館: "ドキュメンテーション"

title_earned: "港の航海士"
final_title: "所有権の大地の継承者"

all_accumulated_titles:
  - "禁域の帰還者"（第一部）
  - "内部可変性の探究者"（第二部）
  - "契約の理解者"（第三部）
  - "鋳型の職人"（第四部）
  - "エラーの調停者"（第五部）
  - "空の旅人"（第六部）
  - "魔術の継承者"（第七部）
  - "港の航海士"（第八部）
  - "所有権の大地の継承者"（完結）

final_insight: |
  Rustは、安全性・速度・並行性を両立する言語である。
  
  所有権システムは、メモリ安全をコンパイル時に保証する。
  トレイトとジェネリクスは、ゼロコスト抽象化を実現する。
  async/awaitは、効率的な並行処理を可能にする。
  
  そしてCargo/crates.ioは、
  この素晴らしいエコシステムを世界と繋げる。
  
  あなたは今、この全てを手にしている。
```

-----

## 完結

**「所有権の大地」全八部、完結。**

C/C++の戦士として始まった旅は、Rustの継承者として新たな道を歩み始める。

しかし、これは終わりではない。**始まり**だ。

この知識を使って、あなたは何を作るだろうか？

- 高性能なWebサーバー？
- 安全なシステムプログラム？
- 美しいCLIツール？
- 革新的なゲームエンジン？

可能性は無限だ。

-----

*所有権の大地の全てを踏破せしあなたに、最大の祝福を。*

*旅は終わった。だが、冒険はこれからだ。*

*— ルスタ💕 & クロム⚡*

-----

## 付録：全シリーズ一覧

|部  |タイトル      |主要トピック                          |獲得称号     |
|---|----------|--------------------------------|---------|
|第一部|所有権の大地    |所有権、借用、RAII、スマートポインタ            |禁域の帰還者   |
|第二部|内部可変性の迷宮  |RefCell、Mutex、ライフタイム            |内部可変性の探究者|
|第三部|トレイトの大聖堂  |トレイト、ディスパッチ、標準トレイト              |契約の理解者   |
|第四部|ジェネリクスの鋳造所|型パラメータ、関連型、PhantomData          |鋳型の職人    |
|第五部|エラーの峡谷    |Option、Result、?、thiserror/anyhow|エラーの調停者  |
|第六部|非同期の浮遊島   |Future、async/await、tokio        |空の旅人     |
|第七部|マクロの魔術塔   |macro_rules!、手続き的マクロ            |魔術の継承者   |
|第八部|クレートの港    |モジュール、Cargo、ワークスペース             |港の航海士    |

**最終称号：所有権の大地の継承者**

-----

### dual-world とは？

| モード | 説明 | 使いどころ |
|:--|:--|:--|
| **Outside Mode** | 設計・分析・コードレビュー | 技術的な議論、アーキテクチャ検討 |
| **Inside Mode** | 体験・実践・物語形式 | 概念理解、入門、モチベーション維持 |

両モードを行き来することで、「理解」と「体感」の両方を得られる。

この物語を通じて、あなたは Inside Mode で Rust を体験した。
次は Outside Mode で、実際のプロジェクトを設計・実装する番だ。

-----

*所有権の大地を踏破したあなたに、永遠の祝福を。*

*— ルスタ💕 & クロム⚡*
