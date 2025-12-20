---
layout: rust-land
title: "第4部：ジェネリクスの鋳造所"
---

# 所有権の大地 IV — ジェネリクスの鋳造所

> **型を一般化し、再利用可能なコードを鍛え上げる技法の物語**

-----

---

> **dual-world について**
>
> この物語は **dual-world** フレームワークの **Inside Mode**（没入型学習）で展開されます。
> Outside Mode（設計・分析）と Inside Mode（体験・実践）を行き来しながら、
> 技術概念を物語として体験できる学習手法です。
>
> 案内人のルスタとクロムは、Three Hearts Space の **ユキ&ミキ** をモチーフにしています。
> - **ルスタ** = ユキ（温かく包み込む愛情、Rust愛を共有）
> - **クロム** = ミキ（クールで技術的、C/C++との対比を示す）

---

## 前作までのあらすじ

あなたは**所有権の大地**で三つの領域を踏破した。

- **第一部「所有権の大地」**：所有権、借用、RAII、スマートポインタ、unsafe → **「禁域の帰還者」**
- **第二部「内部可変性の迷宮」**：RefCell、Mutex、ライフタイム → **「内部可変性の探究者」**
- **第三部「トレイトの大聖堂」**：トレイト、静的/動的ディスパッチ、標準トレイト → **「契約の理解者」**

案内人の**ルスタ💕**と**クロム⚡**と共に。

そして今、型そのものを抽象化する力を学ぶ。

-----

## 第一章：鋳造所の入り口

トレイトの大聖堂を出ると、煙と熱気が漂ってきた。

巨大な工場のような建物。溶鉱炉の炎が夜空を照らしている。

**「ジェネリクスの鋳造所」**

「ここで、**型を鋳造する**💕」ルスタが説明する。

「…具体的な型を決めずに、**型の鋳型**を作る⚡」クロムが続ける。

あなたは尋ねる。「C++のテンプレートのようなものか？」

「似ている。だが——」クロムが考える。「Rustのジェネリクスは**より制約的**だ⚡」

「C++テンプレートは『とりあえずコンパイルしてみて、エラーなら失敗』💕」

「Rustのジェネリクスは『トレイト境界で事前に制約を宣言する』✨」

「エラーメッセージが**格段に分かりやすい**⚡」

鋳造所の門が開く。

-----

## 第二章：型パラメータの炉

最初の部屋には、巨大な炉があった。

炉の中で、様々な型が溶け合っている。

**「型パラメータの炉」**——ジェネリクスの基本を学ぶ場所。

「まずは最も基本的な形💕」

```rust
fn largest<T>(list: &[T]) -> &T {
    // ...
}
```

「`<T>`が**型パラメータ**。『ある型T』を表す⚡」

「でも、これだけじゃコンパイルできない💕」

```rust
fn largest<T>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {  // エラー！Tが比較できるとは限らない
            largest = item;
        }
    }
    largest
}
```

「Tが比較できるとは限らない。**トレイト境界が必要**⚡」

```rust
fn largest<T: PartialOrd>(list: &[T]) -> &T {
    let mut largest = &list[0];
    for item in list {
        if item > largest {
            largest = item;
        }
    }
    largest
}
```

「`T: PartialOrd`で、『Tは比較可能である』と宣言✨」

あなたは理解する。「大聖堂で学んだトレイト境界が、ここで活きている」

「そう💕 ジェネリクスとトレイトは**車の両輪**✨」

-----

## 第三章：構造体の鋳型

炉を過ぎると、様々な鋳型が並んでいた。

**「構造体の鋳型」**——ジェネリックな構造体を作る場所。

「構造体も型パラメータを持てる💕」

```rust
struct Point<T> {
    x: T,
    y: T,
}

let integer_point = Point { x: 5, y: 10 };
let float_point = Point { x: 1.0, y: 4.0 };
```

「一つの定義で、**様々な型のPoint**が作れる⚡」

### 複数の型パラメータ

「型パラメータは複数持てる✨」

```rust
struct Point<T, U> {
    x: T,
    y: U,
}

let mixed = Point { x: 5, y: 4.0 };  // xはi32、yはf64
```

### 構造体へのimpl

「ジェネリック構造体にメソッドを実装する💕」

```rust
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

「`impl<T>`は『任意の型Tに対して』という意味⚡」

「特定の型にだけ実装することもできる✨」

```rust
impl Point<f64> {
    fn distance_from_origin(&self) -> f64 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

「この`distance_from_origin`は、**f64のPointにだけ**存在する💕」

-----

## 第四章：列挙型の金型

鋳型の部屋を抜けると、別の部屋に出た。

**「列挙型の金型」**——ジェネリックなenumを作る場所。

「Rustの最も有名なジェネリック列挙型を見よう⚡」

```rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

「OptionとResult。Rustのエラー処理の基盤💕」

「どちらもジェネリクスで作られている✨」

あなたは気づく。「だから、あらゆる型に対して使えるのか」

```rust
let some_number: Option<i32> = Some(5);
let some_string: Option<String> = Some(String::from("hello"));

let success: Result<i32, String> = Ok(42);
let failure: Result<i32, String> = Err(String::from("failed"));
```

「型パラメータが変わるだけで、**同じ構造が再利用される**⚡」

-----

## 第五章：関連型の祭壇

金型の部屋を抜けると、神秘的な祭壇があった。

**「関連型の祭壇」**——トレイトに型を関連付ける場所。

あなたは思い出す。大聖堂の境界の回廊で見た、あの刻印——

> **`where T: Iterator<Item = U>`**

その意味が、ここで明かされる。

「トレイトには**関連型**を持たせることができる💕」

```rust
trait Iterator {
    type Item;  // 関連型
    
    fn next(&mut self) -> Option<Self::Item>;
}
```

「`type Item`が関連型。実装時に具体的な型を指定する⚡」

```rust
struct Counter {
    count: u32,
}

impl Iterator for Counter {
    type Item = u32;  // この実装では、Itemはu32
    
    fn next(&mut self) -> Option<Self::Item> {
        self.count += 1;
        if self.count < 6 {
            Some(self.count)
        } else {
            None
        }
    }
}
```

あなたは疑問を持つ。「ジェネリクスとの違いは何だ？」

### ジェネリクス vs 関連型

「ジェネリクスを使うとこうなる💕」

```rust
trait Iterator<T> {
    fn next(&mut self) -> Option<T>;
}
```

「一つの型に対して、**複数の実装**ができてしまう⚡」

```rust
impl Iterator<u32> for Counter { ... }
impl Iterator<i32> for Counter { ... }  // 両方可能
```

「関連型なら、**一つの型に一つの実装**✨」

```rust
impl Iterator for Counter {
    type Item = u32;  // Counterに対してItemは一つだけ
}
```

「イテレータは普通、**返す型が一つに決まる**💕」

「関連型の方が**意図を正確に表現**できる⚡」

### 使い分け

```yaml
ジェネリクスを使うとき:
  - 一つの型に対して複数の実装を許したい
  - 例: From<T>（StringはFrom<&str>もFrom<char>も実装）

関連型を使うとき:
  - 一つの型に対して実装は一つ
  - 例: Iterator（一つの型が返すItemは一つ）
```

-----

## 第六章：Phantomの霧

祭壇を過ぎると、霧に包まれた領域に出た。

**「Phantomの霧」**——使われない型パラメータを扱う場所。

「時々、**型パラメータを宣言するが、フィールドで使わない**ことがある💕」

```rust
struct Meters<T> {
    value: f64,
    // Tはどこにも使われていない！
}
```

「コンパイラは警告する。『Tは使われていない』と⚡」

「でも、型レベルで**単位を区別したい**時がある✨」

### PhantomData

「そこで**PhantomData**を使う💕」

```rust
use std::marker::PhantomData;

struct Length<Unit> {
    value: f64,
    _marker: PhantomData<Unit>,
}

struct Meters;
struct Feet;

let meters: Length<Meters> = Length { value: 5.0, _marker: PhantomData };
let feet: Length<Feet> = Length { value: 16.4, _marker: PhantomData };

// metersとfeetは異なる型！混ぜるとコンパイルエラー
```

「PhantomDataは**サイズゼロ**⚡」

「実行時にはコストがない。でも型システムには見える✨」

### ライフタイムのPhantom

「ライフタイムパラメータにも使える💕」

```rust
struct Ref<'a, T> {
    ptr: *const T,
    _marker: PhantomData<&'a T>,
}
```

「生ポインタはライフタイムを持たない。でもPhantomDataで**ライフタイムを関連付けられる**⚡」

「ねえ💕」ルスタが霧の奥を指差す。「PhantomDataには、もっと深い用途があるの✨」

「…型安全な単位系⚡」クロムが言う。「`uom`クレートのように、物理量を型レベルで区別する。メートルとフィートを混ぜたらコンパイルエラー——NASAの火星探査機事故を防げた技術だ」

「そして、`unsafe`コードでのライフタイム追跡💕」

```rust
// 生ポインタだけだとライフタイム情報が失われる
struct Slice<'a, T> {
    ptr: *const T,
    len: usize,
    _marker: PhantomData<&'a T>,  // 'aを追跡
}
```

クロムが遠くを見る。

「…覚えているか⚡ 第二部で学んだ`Send`と`Sync`」

あなたは頷く。スレッド安全性のトレイト——

「PhantomDataは**分散（variance）**も制御できる⚡ `PhantomData<fn(T)>`で反変性を指定するなど。これは非同期の浮遊島で再び必要になる知識だ」

高度な技法の予感が、霧の中に漂っている。

-----

## 第七章：const ジェネリクスの結晶

霧を抜けると、輝く結晶の部屋に出た。

**「constジェネリクスの結晶」**——値をコンパイル時に決める場所。

「型だけでなく、**定数値**もジェネリックにできる💕」

```rust
struct Array<T, const N: usize> {
    data: [T; N],
}

let arr3: Array<i32, 3> = Array { data: [1, 2, 3] };
let arr5: Array<i32, 5> = Array { data: [1, 2, 3, 4, 5] };
```

「`const N: usize`は**コンパイル時定数**⚡」

「配列のサイズを型に埋め込める✨」

### 活用例

```rust
impl<T: Default + Copy, const N: usize> Array<T, N> {
    fn new() -> Self {
        Array { data: [T::default(); N] }
    }
}

impl<T, const N: usize> Array<T, N> {
    fn len(&self) -> usize {
        N  // コンパイル時に決まっている
    }
}
```

「C++のテンプレート非型パラメータに似ている⚡」

「でもRustでは、**比較的新しい機能**（Rust 1.51以降）💕」

-----

## 第八章：Turbofish の海峡

結晶の部屋を出ると、狭い海峡に出た。

水面には魚の形をした記号が浮かんでいる。

**「Turbofishの海峡」**——型を明示的に指定する場所。

「時々、**型を明示的に指定**したい場面がある💕」

```rust
let parsed = "42".parse::<i32>().unwrap();
```

「`::<i32>`が**Turbofish記法**⚡」

「魚みたいでしょ？💕 `::<>` ✨」

### なぜ必要か

「コンパイラが推論できない時に使う⚡」

```rust
let v = Vec::new();  // 何の型？

let v: Vec<i32> = Vec::new();  // 型注釈
let v = Vec::<i32>::new();     // Turbofish
```

「両方とも同じ意味💕」

### メソッドチェーン

「メソッドチェーンの途中では、型注釈より**Turbofishが便利**⚡」

```rust
let numbers: Vec<i32> = "1,2,3,4,5"
    .split(',')
    .map(|s| s.parse::<i32>().unwrap())
    .collect();
```

「`parse`の戻り値を、途中で指定している✨」

「ねえ💕」ルスタが海峡を見渡す。「Rustの型推論って、どういう順番で決まるか知ってる？✨」

「…まず変数の型注釈、次にTurbofish、それから戻り値からの逆推論、最後に後続の使用から⚡」

「`collect()`が特別なのは？💕」

「戻り値の型から逆推論するから、Turbofishか型注釈が**必要**⚡」

```rust
// どちらでもOK
let v: Vec<i32> = iter.collect();
let v = iter.collect::<Vec<i32>>();

// 型だけ指定して要素型は推論させることも可能
let v: Vec<_> = iter.collect();
```

「『type annotations needed』エラーが出たら？💕」

「Turbofishか型注釈を試せ⚡」クロムが断言する。「ほぼ確実に解決する」

-----

## 第九章：モノモーフィゼーションの炉心

海峡を渡ると、鋳造所の中心部に出た。

巨大な炉心が轟音を立てている。

**「モノモーフィゼーションの炉心」**——ジェネリクスがどう展開されるか。

「第三部でも触れたが、ここで詳しく💕」

「ジェネリクスは**コンパイル時に展開**される⚡」

```rust
fn print_it<T: Display>(value: T) {
    println!("{}", value);
}

print_it(5);       // i32版が生成される
print_it("hello"); // &str版が生成される
print_it(3.14);    // f64版が生成される
```

「コンパイラは**使われる型ごとに別の関数を生成**する💕」

```rust
// コンパイラが内部で生成するイメージ
fn print_it_i32(value: i32) { println!("{}", value); }
fn print_it_str(value: &str) { println!("{}", value); }
fn print_it_f64(value: f64) { println!("{}", value); }
```

### 利点と欠点

```yaml
利点:
  - 実行時オーバーヘッドなし
  - インライン化可能
  - 最適化が効く

欠点:
  - コンパイル時間が増える
  - バイナリサイズが増える
  - 型ごとにコード重複
```

「トレードオフを理解して使う⚡」

-----

## 終章：鋳造所の頂

炉心を過ぎると、鋳造所の頂上に出た。

眼下には、あなたが学んできた全ての技術が見える。

### 学んだこと

```yaml
ジェネリクスの基本:
  型パラメータ: "<T>"で型を抽象化
  トレイト境界: "<T: Trait>"で制約を付ける
  where句: 複雑な境界を読みやすく

構造体と列挙型:
  ジェネリック構造体: "struct Point<T>"
  ジェネリック列挙型: "enum Option<T>"
  impl: "impl<T> Point<T>"

高度な機能:
  関連型: トレイトに型を関連付ける
  PhantomData: 使われない型パラメータをマーク
  constジェネリクス: コンパイル時定数をパラメータに
  Turbofish: "::<T>"で型を明示指定

内部動作:
  モノモーフィゼーション: 型ごとにコード生成
  ゼロコスト抽象化: 実行時オーバーヘッドなし
```

### 二人からの言葉

「おめでとう💕」ルスタが手を差し伸べる。

「ジェネリクスを理解したあなたは、**真に汎用的なコードを書ける**✨」

「…次はエラー処理だ⚡」クロムが言う。「Result、Option、?演算子。Rustらしいエラーハンドリングを学ぶ」

あなたは鋳造所を振り返る。

ジェネリクスは、C++のテンプレートに似ているが、**より安全**で**より明確**だ。

トレイト境界によって、**何ができるか**が事前に分かる。

これがRustの型システムの力だ。

「ところで💕」ルスタが遠くの峡谷を指差す。「あそこに見えるのが……」

**「エラーの峡谷」**——深い裂け目が大地を走っている。

「第三部で`Option`と`Result`を見たな⚡」クロムが言う。「あれはジェネリック列挙型だった。だが、その**本当の力**はあの峡谷で学ぶことになる」

「`?`演算子💕」ルスタが囁く。「エラーを優雅に伝播させる魔法の記号……✨」

あなたは峡谷を見つめる。次なる冒険が待っている。

> **📖 公式ドキュメント参照**
> - [Generic Types, Traits, and Lifetimes](https://doc.rust-lang.org/book/ch10-00-generics.html)
> - [Advanced Types](https://doc.rust-lang.org/book/ch19-04-advanced-types.html)
> - [Const Generics RFC](https://rust-lang.github.io/rfcs/2000-const-generics.html)

-----

## あなたの旅のログ（第四部）

```yaml
completed_areas:
  - 第一章_鋳造所の入り口: "ジェネリクスの世界への入り口"
  - 第二章_型パラメータの炉: "基本的なジェネリクス"
  - 第三章_構造体の鋳型: "ジェネリック構造体"
  - 第四章_列挙型の金型: "Option, Result"
  - 第五章_関連型の祭壇: "トレイトの関連型"
  - 第六章_Phantomの霧: "PhantomData"
  - 第七章_constジェネリクス: "コンパイル時定数"
  - 第八章_Turbofishの海峡: "型の明示指定"
  - 第九章_モノモーフィゼーション: "コード生成の仕組み"

title_earned: "鋳型の職人"

accumulated_titles:
  - "禁域の帰還者"（第一部）
  - "内部可変性の探究者"（第二部）
  - "契約の理解者"（第三部）
  - "鋳型の職人"（第四部）

insight_gained: |
  ジェネリクスは「型の鋳型」である。
  
  トレイト境界で制約を宣言し、
  コンパイル時に具体的な型へ展開される（モノモーフィゼーション）。
  
  関連型は「一つの型に一つの実装」を強制する。
  PhantomDataは使われない型パラメータをマークする。
  constジェネリクスでコンパイル時定数もパラメータにできる。
```

-----

## 第五部への道

次なる冒険は、**エラーの峡谷**へと続く。

Result、Option、?演算子——Rustらしいエラー処理を学ぶ旅。

### dual-world とは？

| モード | 説明 | 使いどころ |
|:--|:--|:--|
| **Outside Mode** | 設計・分析・コードレビュー | 技術的な議論、アーキテクチャ検討 |
| **Inside Mode** | 体験・実践・物語形式 | 概念理解、入門、モチベーション維持 |

両モードを行き来することで、「理解」と「体感」の両方を得られる。

```
"dual-world を使って Rust のエラー処理を学びたい"
```

と告げれば、いつでも旅を再開できる。

-----

*ジェネリクスの鋳造所を巡りしあなたに、型の祝福を。*

*— ルスタ💕 & クロム⚡*
