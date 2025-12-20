---
layout: default
title: "第7部：マクロの魔術塔"
---

# 所有権の大地 VII — マクロの魔術塔

> **コードを生成するコード、メタプログラミングの奥義の物語**

---

> **dual-world — 第七部**
>
> Inside Modeで魔術の世界へ。
> マクロは「黒魔術」と呼ばれることもありますが、正しく使えば**強力な味方**です。
> 古の魔術塔で、コードを生成するコードの奥義を学びましょう。
>
> 💕 **ルスタ**（ユキ）: 魔法って、最初は怖いけど使いこなせると楽しいよ
> ⚡ **クロム**（ミキ）: Cのマクロとは別物だ。型安全な魔術を見せてやる

---

## 前作までのあらすじ

あなたは**所有権の大地**で六つの領域を踏破した。

- **第一部「所有権の大地」**：所有権、借用、RAII → **「禁域の帰還者」**
- **第二部「内部可変性の迷宮」**：RefCell、Mutex → **「内部可変性の探究者」**
- **第三部「トレイトの大聖堂」**：トレイト、ディスパッチ → **「契約の理解者」**
- **第四部「ジェネリクスの鋳造所」**：型パラメータ、関連型 → **「鋳型の職人」**
- **第五部「エラーの峡谷」**：Result、Option、? → **「エラーの調停者」**
- **第六部「非同期の浮遊島」**：async/await、tokio → **「空の旅人」**

案内人の**ルスタ💕**と**クロム⚡**と共に。

そして今、**コードを生成する魔術**を学ぶ。

-----

## 第一章：魔術塔の入り口

浮遊島から降りると、古めかしい塔がそびえていた。

塔の壁には、無数の**呪文**が刻まれている。

**「マクロの魔術塔」**

「ここでは、**コードがコードを生む**💕」ルスタが神秘的に言う。

「…メタプログラミング。プログラムを書くプログラム⚡」クロムが続ける。

あなたは壁の呪文を見る。`vec![]`、`println!`、`derive(Debug)`——見覚えのある文字が刻まれている。

「待って...これらは最初から使っていた」

「気づいた？⚡」クロムが微笑む。「第一部で`vec![1, 2, 3]`を書いた時、あなたはすでにマクロを使っていた」

「`derive(Debug)`も、第二部から毎回使ってきたでしょ💕」

あなたは驚く。「あれがマクロだったのか」

「今日は、その**仕組み**を理解する✨」

あなたは記憶を辿る。第一部で、unsafeの禁域を越えた時——ルスタが呟いた言葉。

> **「遠い塔で、魔術師たちが編み出した技法——unsafeを安全に包んで使う術」**

あの「遠い塔」は、この塔のことだったのだ。

あなたは尋ねる。「関数と何が違うのか？」

「関数は**実行時**に動く。マクロは**コンパイル時**に動く💕」

「マクロは**コードを展開**する。結果が実際にコンパイルされる⚡」

塔の扉が開く。

-----

## 第二章：宣言的マクロの書斎

最初の部屋は、古い書物が並ぶ書斎だった。

**「宣言的マクロの書斎」**——`macro_rules!`を学ぶ場所。

「Rustで最もよく使うマクロ形式💕」

```rust
macro_rules! say_hello {
    () => {
        println!("Hello!");
    };
}

say_hello!();  // "Hello!"
```

「`macro_rules!`で**パターンと展開結果**を定義⚡」

「呼び出すと、パターンにマッチした部分が**展開される**✨」

### 引数を取る

「引数を取ることもできる💕」

```rust
macro_rules! say {
    ($message:expr) => {
        println!("{}", $message);
    };
}

say!("Hello, world!");  // "Hello, world!"
say!(1 + 2);            // "3"
```

「`$message:expr`は『式（expression）を受け取り、$messageという名前で使う』という意味⚡」

### フラグメント指定子

「`:expr`の部分を**フラグメント指定子**と呼ぶ✨」

```yaml
主なフラグメント指定子:
  expr: 式（1 + 2, "hello", foo()など）
  ident: 識別子（変数名、関数名など）
  ty: 型（i32, String, Vec<T>など）
  pat: パターン（Some(x), (a, b)など）
  stmt: 文
  block: ブロック（{ ... }）
  item: アイテム（関数定義、構造体定義など）
  tt: トークンツリー（何でも）
```

### 複数のパターン

「複数のパターンを定義できる💕」

```rust
macro_rules! calculate {
    (add $a:expr, $b:expr) => {
        $a + $b
    };
    (mul $a:expr, $b:expr) => {
        $a * $b
    };
}

let sum = calculate!(add 1, 2);   // 3
let product = calculate!(mul 3, 4);  // 12
```

「パターンは**上から順に**マッチングされる⚡」

あなたはC/C++時代の悪夢を思い出す。「マクロで変数名が衝突して...」

「Rustでは起きない⚡」クロムが安心させる。「Rustマクロは**衛生的（hygienic）**だ」

```rust
macro_rules! using_a {
    ($e:expr) => {
        {
            let a = 42;  // マクロ内の変数
            $e           // 呼び出し側の式
        }
    };
}

let a = 1;
let result = using_a!(a * 2);  // 結果は 2（呼び出し側のa=1を使う）
```

「マクロ内の`a`と、呼び出し側の`a`は**別物**として扱われる💕」

あなたは驚く。「単純なテキスト置換じゃないのか」

「Cマクロとは根本的に違う⚡ 意図しない名前衝突を**型システムレベルで防ぐ**」

「ただし`ident`として受け取った識別子は、そのままの名前で使われるよ💕」ルスタが補足する。

-----

## 第三章：繰り返しの回廊

書斎を抜けると、螺旋状の回廊に出た。

同じパターンが繰り返し刻まれている。

**「繰り返しの回廊」**——マクロの反復を学ぶ場所。

「マクロの強力な機能：**繰り返し**💕」

```rust
macro_rules! vec_of_strings {
    ($($element:expr),*) => {
        {
            let mut v = Vec::new();
            $(
                v.push(String::from($element));
            )*
            v
        }
    };
}

let v = vec_of_strings!["a", "b", "c"];
// vec!["a".to_string(), "b".to_string(), "c".to_string()] と同等
```

「`$(...),*`は『カンマ区切りで0回以上繰り返し』⚡」

### 繰り返し記号

```yaml
繰り返し記号:
  *: 0回以上
  +: 1回以上
  ?: 0回または1回
```

### 区切り文字

「カンマ以外も区切りに使える✨」

```rust
macro_rules! create_functions {
    ($($name:ident => $val:expr);*) => {
        $(
            fn $name() -> i32 { $val }
        )*
    };
}

create_functions! {
    one => 1;
    two => 2;
    three => 3
}

assert_eq!(one(), 1);
assert_eq!(two(), 2);
```

「セミコロンで区切って、関数を複数生成💕」

あなたは回廊をさらに進むが、ふと足を止める。「落とし穴はある？」

「ある⚡」クロムが警告する。「**ネストした繰り返し**は要注意だ」

```rust
// ❌ エラー
macro_rules! bad {
    ($($x:expr),*) => {
        $( $($x)* )*  // 同じ$xを異なる深さで使用
    };
}
```

「同じ変数を**異なる深さ**で使うとエラーになる💕」

「正しくはこう⚡」

```rust
// ✅ 正しくは別々の変数を使う
macro_rules! matrix {
    ( $( $( $x:expr ),* );* ) => {
        [ $( [ $( $x ),* ] ),* ]
    };
}
let m = matrix![1, 2; 3, 4];  // [[1, 2], [3, 4]]
```

あなたは注意深く見る。「他には？」

「**空の繰り返し**にも注意💕」ルスタが付け加える。「`*`は0回マッチを許すから、`$(,)*`は空文字列にもマッチする」

-----

## 第四章：vec!の秘密

回廊を進むと、特別な部屋があった。

部屋の中央に、光り輝く呪文が浮かんでいる。

**「vec!の秘密」**——最も有名なマクロの実装。

「`vec!`マクロの中身を見てみよう⚡」

「これは**簡略版**だけど、本質は同じ💕」

```rust
// 標準ライブラリの簡略版（実際の実装はもう少し複雑）
macro_rules! vec {
    () => {
        Vec::new()
    };
    ($elem:expr; $n:expr) => {
        std::vec::from_elem($elem, $n)
    };
    ($($x:expr),+ $(,)?) => {
        <[_]>::into_vec(Box::new([$($x),+]))
    };
}
```

「三つのパターンがある💕」

```rust
vec![]                 // 空のVec
vec![0; 5]             // [0, 0, 0, 0, 0]
vec![1, 2, 3]          // [1, 2, 3]
vec![1, 2, 3,]         // 末尾カンマもOK
```

「`$(,)?`は**オプショナルな末尾カンマ**を許可✨」

-----

## 第五章：手続き的マクロの工房

塔を登ると、より高度な工房に出た。

ここには複雑な装置が並んでいる。

**「手続き的マクロの工房」**——proc macroを学ぶ場所。

「宣言的マクロ（macro_rules!）には限界がある💕」

「もっと複雑なことをしたい時、**手続き的マクロ**を使う⚡」

### 三種類の手続き的マクロ

```yaml
derive macro:
  形式: #[derive(MyTrait)]
  用途: 構造体/列挙型にトレイトを自動実装

attribute macro:
  形式: #[my_attribute]
  用途: 任意のアイテムを変換

function-like macro:
  形式: my_macro!(...)
  用途: 関数っぽく呼び出すマクロ
```

「手続き的マクロは**別クレート**として実装する必要がある✨」

```toml
# Cargo.toml
[lib]
proc-macro = true

[dependencies]
syn = "2"
quote = "1"
proc-macro2 = "1"
```

-----

## 第六章：deriveの祭壇

工房の奥に、神聖な祭壇があった。

**「deriveの祭壇」**——最も使われる手続き的マクロ。

「あなたはすでに**derive**を使っている💕」

```rust
#[derive(Debug, Clone, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}
```

「deriveマクロが、トレイト実装を**自動生成**する⚡」

### 自作deriveマクロ

「自分でderiveマクロを作ることもできる✨」

```rust
// my_macro クレート (proc-macro = true)
use proc_macro::TokenStream;
use quote::quote;
use syn::{parse_macro_input, DeriveInput};

#[proc_macro_derive(HelloWorld)]
pub fn hello_world_derive(input: TokenStream) -> TokenStream {
    let ast = parse_macro_input!(input as DeriveInput);
    let name = &ast.ident;
    
    let gen = quote! {
        impl HelloWorld for #name {
            fn hello_world() {
                println!("Hello, World! My name is {}!", stringify!(#name));
            }
        }
    };
    
    gen.into()
}
```

「使う側💕」

```rust
use my_macro::HelloWorld;

trait HelloWorld {
    fn hello_world();
}

#[derive(HelloWorld)]
struct Pancakes;

fn main() {
    Pancakes::hello_world();  // "Hello, World! My name is Pancakes!"
}
```

### syn と quote

「**syn**はRustコードをパース⚡」

「**quote**はRustコードを生成✨」

```rust
// synでパース
let ast: DeriveInput = parse_macro_input!(input);
let struct_name = &ast.ident;  // 構造体名を取得

// quoteで生成
let expanded = quote! {
    impl MyTrait for #struct_name {
        fn method(&self) {
            // ...
        }
    }
};
```

あなたは祭壇の刻印を見つめる。「deriveに追加の指示を与えることは？」

「できる⚡ **ヘルパー属性**という仕組みだ」

```rust
#[proc_macro_derive(MyTrait, attributes(my_attr))]
pub fn my_derive(input: TokenStream) -> TokenStream {
    // ...
}
```

「使用側では、フィールドごとに振る舞いを指定できる💕」

```rust
#[derive(MyTrait)]
struct Config {
    name: String,
    #[my_attr(skip)]      // このフィールドをスキップ
    internal: u32,
    #[my_attr(rename = "value")]  // 別名を指定
    val: i32,
}
```

あなたは見覚えを感じる。「そういえばserdeで...」

「`#[serde(rename = "...")]`を見たことがあるだろう？⚡」クロムが頷く。「あれがまさにヘルパー属性だ」

「`#[clap(long, short)]`もそう💕」ルスタが付け加える。「CLIオプションの定義に使われてる✨」

-----

## 第七章：属性マクロの回廊

祭壇を過ぎると、不思議な回廊に出た。

壁に触れると、壁自体が変化する。

**「属性マクロの回廊」**——#[attribute]を学ぶ場所。

「属性マクロは、**アイテムを変換**する💕」

```rust
#[proc_macro_attribute]
pub fn log_call(attr: TokenStream, item: TokenStream) -> TokenStream {
    let input = parse_macro_input!(item as syn::ItemFn);
    let name = &input.sig.ident;
    let block = &input.block;
    
    let expanded = quote! {
        fn #name() {
            println!("Calling {}", stringify!(#name));
            #block
        }
    };
    
    expanded.into()
}
```

「使う側⚡」

```rust
#[log_call]
fn my_function() {
    println!("Function body");
}

// 展開後のイメージ:
// fn my_function() {
//     println!("Calling my_function");
//     println!("Function body");
// }
```

「関数に**ログ出力を自動追加**した✨」

### 実践例：tokio::main

「`#[tokio::main]`も属性マクロ💕」

```rust
#[tokio::main]
async fn main() {
    // async code
}

// 展開後のイメージ:
// fn main() {
//     tokio::runtime::Builder::new_multi_thread()
//         .enable_all()
//         .build()
//         .unwrap()
//         .block_on(async {
//             // async code
//         })
// }
```

「asyncなmainを、**同期的なmain + ランタイム起動**に変換⚡」

-----

## 第八章：関数風マクロの実験室

回廊を抜けると、実験室に出た。

様々な薬品（コード片）が混ぜ合わされている。

**「関数風マクロの実験室」**——関数っぽく呼び出すマクロ。

「見た目は関数だが、**コンパイル時に展開**される💕」

```rust
#[proc_macro]
pub fn make_answer(_item: TokenStream) -> TokenStream {
    "fn answer() -> u32 { 42 }".parse().unwrap()
}
```

「使う側⚡」

```rust
make_answer!();

fn main() {
    println!("{}", answer());  // 42
}
```

### SQLクエリの例

「実践的な例：**SQLクエリの検証**✨」

```rust
// sqlx クレートの例
let user = sqlx::query_as!(
    User,
    "SELECT id, name, email FROM users WHERE id = $1",
    user_id
).fetch_one(&pool).await?;
```

「`sqlx::query_as!`は**コンパイル時にSQLを検証**する💕」

「実行時エラーではなく、**コンパイルエラー**になる⚡」

-----

## 第九章：マクロの心得

塔の最上階に着いた。

壁には、先人たちの教えが刻まれている。

**「マクロの心得」**——いつマクロを使うべきか。

### マクロを使うべき時

```yaml
使うべき時:
  - 繰り返しコードを減らしたい（ボイラープレート削減）
  - DSL（ドメイン特化言語）を作りたい
  - コンパイル時に検証したい
  - 可変長引数が欲しい
  - 条件付きコンパイルが必要
```

### マクロを避けるべき時

```yaml
避けるべき時:
  - 関数で十分な時（関数の方がデバッグしやすい）
  - トレイトで十分な時（抽象化として適切）
  - 複雑すぎる時（メンテナンス困難）
```

### デバッグのコツ

「マクロのデバッグは難しい💕」

```rust
// cargo expand で展開結果を見る
// $ cargo expand

// trace_macros! で展開過程を見る（nightly）
#![feature(trace_macros)]
trace_macros!(true);
my_macro!(something);
trace_macros!(false);
```

「`cargo expand`は必須ツール⚡」

あなたは立ち止まる。「他にデバッグ方法は？」

「複数ある⚡」クロムがリストを示す。

```bash
# 1. cargo-expand（推奨）
cargo install cargo-expand
cargo expand            # 全ファイルを展開
cargo expand module     # 特定モジュールだけ

# 2. rustc の pretty-print
cargo rustc -- -Z unpretty=expanded  # nightly

# 3. コンパイルエラーでスタックトレース
RUST_BACKTRACE=1 cargo build
```

「マクロを公開する時も心がけが必要💕」ルスタが言う。

「`#[macro_export]`で他クレートから使用可能に⚡」

「ドキュメントには展開例を書いてあげて✨ 読者が理解しやすくなる」

「あと、エラーメッセージを分かりやすく💕」

```rust
macro_rules! my_macro {
    () => {
        compile_error!("my_macro requires at least one argument");
    };
    // 他のパターン...
}
```

「`compile_error!`を使うと、**意図したエラー**を出せる⚡ ユーザーを迷子にさせない」

-----

## 終章：魔術塔の頂

塔の頂上に立つと、所有権の大地が一望できた。

あなたが学んできた全ての領域が見える。

### 学んだこと

```yaml
宣言的マクロ (macro_rules!):
  特徴: パターンマッチングで展開
  用途: シンプルなコード生成
  例: vec!, println!, assert!

手続き的マクロ:
  derive: トレイト自動実装
  attribute: アイテム変換
  function-like: 関数風呼び出し
  ツール: syn（パース）、quote（生成）

心得:
  - 関数で済むならマクロは使わない
  - デバッグはcargo expandで
  - 複雑すぎるマクロは避ける
```

### 二人からの言葉

「おめでとう💕」ルスタが手を差し伸べる。

「マクロを理解したあなたは、**Rustの魔術師**✨」

「…最後の旅が残っている⚡」クロムが言う。「クレートとモジュール。プロジェクトを構成する力」

あなたは塔を見下ろす。

マクロは、**コードを生成するコード**。

宣言的マクロはシンプルだが、手続き的マクロは強力。

ただし、**力には責任が伴う**。使いすぎると可読性を損なう。

これがRustのメタプログラミングの力だ。

ルスタが遠くの海を指差す。水平線の向こうに、港が見える。

「最後の旅が始まるよ💕」

「…クレートの港⚡」クロムが続ける。「第二部で学んだ`'static`——『旅の終わりに残るもの』を覚えているか？」

あなたは頷く。

> **「クレートとして公開したコード。世界中の誰かが使い続ける限り、それは生き続ける」**

「今、その意味を完全に理解する時が来た✨」

> **📖 公式ドキュメント参照**
> - [Macros (The Rust Programming Language)](https://doc.rust-lang.org/book/ch19-06-macros.html)
> - [The Little Book of Rust Macros](https://veykril.github.io/tlborm/)
> - [Procedural Macros](https://doc.rust-lang.org/reference/procedural-macros.html)

-----

## あなたの旅のログ（第七部）

```yaml
completed_areas:
  - 第一章_魔術塔の入り口: "マクロの概念"
  - 第二章_宣言的マクロ: "macro_rules!の基本"
  - 第三章_繰り返しの回廊: "マクロの反復"
  - 第四章_vec!の秘密: "標準マクロの実装"
  - 第五章_手続き的マクロ: "proc macroの種類"
  - 第六章_deriveの祭壇: "derive macroの実装"
  - 第七章_属性マクロ: "attribute macroの実装"
  - 第八章_関数風マクロ: "function-like macroの実装"
  - 第九章_マクロの心得: "使いどころと注意点"

title_earned: "魔術の継承者"

accumulated_titles:
  - "禁域の帰還者"（第一部）
  - "内部可変性の探究者"（第二部）
  - "契約の理解者"（第三部）
  - "鋳型の職人"（第四部）
  - "エラーの調停者"（第五部）
  - "空の旅人"（第六部）
  - "魔術の継承者"（第七部）

insight_gained: |
  マクロは「コードを生成するコード」である。
  
  宣言的マクロ（macro_rules!）はパターンマッチングで展開。
  手続き的マクロはRustコードを直接操作する。
  
  derive、attribute、function-likeの3種類がある。
  synでパース、quoteで生成が定番の組み合わせ。
  
  マクロは強力だが、濫用は禁物。
  関数やトレイトで済むなら、そちらを使う。
```

-----

## 第八部への道

最後の冒険は、**クレートの港**へと続く。

モジュール、クレート、Cargo、ワークスペース——プロジェクトを構成する力を学ぶ旅。

### dual-world とは？

| モード | 説明 | 使いどころ |
|:--|:--|:--|
| **Outside Mode** | 設計・分析・コードレビュー | 技術的な議論、アーキテクチャ検討 |
| **Inside Mode** | 体験・実践・物語形式 | 概念理解、入門、モチベーション維持 |

両モードを行き来することで、「理解」と「体感」の両方を得られる。

```
"dual-world を使って Rust のモジュールシステムを学びたい"
```

と告げれば、いつでも旅を再開できる。

-----

*マクロの魔術塔を登りしあなたに、言葉の祝福を。*

*— ルスタ💕 & クロム⚡*
