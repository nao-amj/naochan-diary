---
layout: rust-land
title: "第5部：エラーの峡谷"
---

# 所有権の大地 V — エラーの峡谷

> **失敗を型で表現し、優雅に処理する技法の物語**

---

> **dual-world — 第五部**
>
> Inside Modeで旅を続けています。
> エラー処理は「怖い」イメージがありますが、Rustでは**型システムの味方**になります。
> 失敗を恐れず、型に導かれて進みましょう。
>
> 💕 **ルスタ**（ユキ）: 失敗しても大丈夫、一緒に立ち上がろう
> ⚡ **クロム**（ミキ）: エラーは敵じゃない、情報だ

---

## 前作までのあらすじ

あなたは**所有権の大地**で四つの領域を踏破した。

- **第一部「所有権の大地」**：所有権、借用、RAII → **「禁域の帰還者」**
- **第二部「内部可変性の迷宮」**：RefCell、Mutex、ライフタイム → **「内部可変性の探究者」**
- **第三部「トレイトの大聖堂」**：トレイト、ディスパッチ → **「契約の理解者」**
- **第四部「ジェネリクスの鋳造所」**：型パラメータ、関連型 → **「鋳型の職人」**

案内人の**ルスタ💕**と**クロム⚡**と共に。

そして今、エラーを**恐れるのではなく、味方につける**方法を学ぶ。

-----

## 第一章：峡谷の入り口

鋳造所を出ると、深い峡谷が広がっていた。

断崖絶壁。霧に包まれた谷底。遠くで雷鳴が響く。

**「エラーの峡谷」**

「多くの言語で、エラーは**恐怖の対象**💕」ルスタが言う。

「例外が投げられ、キャッチされ、時に見逃される⚡」クロムが続ける。

あなたはC/C++の記憶を呼び起こす。戻り値のエラーコード。errno。例外。それぞれが混在し、どれを使うべきか迷った日々。

```cpp
// C++の世界
int result = do_something();
if (result < 0) {
    // エラー...でも忘れても動く
}

try {
    do_something_else();
} catch (const std::exception& e) {
    // キャッチしないと？クラッシュ
}
```

「C++では**3つのエラー処理**が混在する⚡」クロムが言う。「戻り値、errno、例外。どれを使うかは...プロジェクト次第」

「Rustでは、エラーは**型で表現**される💕」

「見逃すことは**できない**。コンパイラが強制するから⚡」

峡谷への道を降りていく。

-----

## 第二章：Optionの洞窟

峡谷を降りると、最初の洞窟があった。

洞窟の入り口には二つの道がある。「Some」と「None」。

**「Optionの洞窟」**——値があるかないかを表現する場所。

「最も基本的な『失敗』は、**値がないこと**💕」

```rust
enum Option<T> {
    Some(T),  // 値がある
    None,     // 値がない
}
```

「C/C++で言えば、nullポインタの安全版⚡」

「でも決定的に違う。**nullを忘れてアクセスすることがない**✨」

### Optionの使い方

```rust
fn find_user(id: u32) -> Option<String> {
    if id == 1 {
        Some(String::from("Alice"))
    } else {
        None
    }
}

let user = find_user(1);

// これはコンパイルエラー！
// println!("{}", user);  // Optionは直接使えない

// 正しい方法
match user {
    Some(name) => println!("Found: {}", name),
    None => println!("User not found"),
}
```

「Optionを使うには、**中身を取り出す必要がある**💕」

「コンパイラが強制する。忘れることはない⚡」

### 便利メソッド

「毎回matchを書くのは面倒。だから便利メソッドがある✨」

```rust
// unwrap: Someなら中身を返す。Noneならパニック
let name = find_user(1).unwrap();  // 危険！

// expect: unwrapと同じだが、エラーメッセージを指定
let name = find_user(1).expect("User must exist");

// unwrap_or: Noneの時のデフォルト値
let name = find_user(2).unwrap_or(String::from("Unknown"));

// unwrap_or_else: Noneの時にクロージャを実行
let name = find_user(2).unwrap_or_else(|| String::from("Guest"));

// map: Someの中身を変換
let upper = find_user(1).map(|n| n.to_uppercase());

// and_then: Someの時に別のOptionを返す関数を適用
let first_char = find_user(1).and_then(|n| n.chars().next());
```

「`unwrap`と`expect`は**本番コードでは慎重に**⚡」

「パニックするから——つまり**プログラムがその場で停止する**。Webサーバーなら接続が切れ、CLIなら異常終了。でもプロトタイプや確実な場面では便利💕」

あなたは洞窟の奥を見つめる。「OptionとResultは別物なのか？」

「良い質問⚡」クロムが振り返る。「実は**橋渡し**ができる」

```rust
// Optionをエラー付きのResultに変換
let name: Option<String> = find_user(1);
let result: Result<String, &str> = name.ok_or("User not found");
```

「`ok_or`で、Noneだった時のエラーメッセージを指定できるの💕」

「逆も可能だ。`Result::ok()`でResultからOptionに⚡ ただし**エラー情報は捨てられる**」

「使い分けが大事ってことね✨」

-----

## 第三章：Resultの吊り橋

洞窟を抜けると、深い谷にかかる吊り橋があった。

橋の両側に看板。「Ok」と「Err」。

**「Resultの吊り橋」**——成功か失敗かを表現する場所。

「Optionは『あるかないか』。Resultは『成功か失敗か』💕」

```rust
enum Result<T, E> {
    Ok(T),   // 成功。値はT
    Err(E),  // 失敗。エラーはE
}
```

「**エラーの情報も型で持つ**⚡」

### Resultの使い方

```rust
use std::fs::File;
use std::io::Error;

fn open_file(path: &str) -> Result<File, Error> {
    File::open(path)
}

let result = open_file("hello.txt");

match result {
    Ok(file) => println!("File opened: {:?}", file),
    Err(e) => println!("Error: {}", e),
}
```

「ファイルを開く操作は**失敗しうる**💕」

「だからResultを返す。呼び出し側は**必ず**エラーを処理する✨」

### Optionとの類似

「ResultとOptionは似ている。多くのメソッドが共通⚡」

```rust
// unwrap, expect
let file = open_file("hello.txt").unwrap();
let file = open_file("hello.txt").expect("Failed to open");

// unwrap_or, unwrap_or_else
let content = std::fs::read_to_string("config.txt").unwrap_or(String::from("default"));

// map: Okの中身を変換
let upper = std::fs::read_to_string("hello.txt").map(|s| s.to_uppercase());

// map_err: Errの中身を変換
let result = open_file("hello.txt").map_err(|e| format!("IO Error: {}", e));

// and_then: Okの時に別のResultを返す関数を適用
use std::io::Read;
let content = open_file("hello.txt").and_then(|mut f| {
    let mut s = String::new();
    f.read_to_string(&mut s).map(|_| s)
});
```

-----

## 第四章：?演算子の渡し舟

吊り橋を渡ると、川に出た。

小さな舟が待っている。舟の旗には「?」と書かれている。

**「?演算子の渡し舟」**——エラーを優雅に伝播する場所。

あなたはこの記号に見覚えがある。第一部で、ルスタが呟いた言葉——

> **「エラーを優雅に伝播させる魔法の記号……」**

その正体を、ついに知ることになる。

「エラー処理で最も多いパターンは何だと思う？💕」

あなたは考える。「エラーが起きたら、そのまま呼び出し元に返す」

「正解⚡」

```rust
use std::fs::File;
use std::io::Read;

fn read_username() -> Result<String, std::io::Error> {
    let mut f = match File::open("username.txt") {
        Ok(file) => file,
        Err(e) => return Err(e),  // エラーなら即return
    };

    let mut s = String::new();
    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),  // エラーなら即return
    }
}
```

「これを**毎回書くのは辛い**💕」

「だから`?`演算子がある✨」

```rust
use std::fs::File;
use std::io::Read;

fn read_username() -> Result<String, std::io::Error> {
    let mut f = File::open("username.txt")?;  // エラーなら即return
    let mut s = String::new();
    f.read_to_string(&mut s)?;  // エラーなら即return
    Ok(s)
}
```

「`?`は『Okなら中身を取り出す。Errならその場でreturn』⚡」

「コードが**劇的に短くなる**💕」

あなたは舟に乗り込む前に、足を止めた。

「待って。`?`は本当は何をしているんだ？」

クロムが振り返る。「いい質問だ⚡ `?`の正体を見せよう」

```rust
// expr? は、実はこう展開される
match expr {
    Ok(v) => v,
    Err(e) => return Err(From::from(e)),
}
```

「注目すべきは`From::from(e)`の部分⚡」

「つまり...エラー型が違っても、`From`トレイトさえ実装されていれば**自動変換**されるってこと？💕」ルスタが目を輝かせる。

「その通り。これが`?`の魔法の正体だ✨」

あなたは頷く。単なる糖衣構文ではなく、**型変換の仕組み**が組み込まれている。

「あ、それと💕」ルスタが付け加える。「`main`関数で`?`を使いたい時は——」

```rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
    let content = std::fs::read_to_string("file.txt")?;
    Ok(())
}
```

「戻り値を`Result`にすれば、`main`でも`?`が使えるようになるよ✨」

### さらに短く

```rust
fn read_username() -> Result<String, std::io::Error> {
    let mut s = String::new();
    File::open("username.txt")?.read_to_string(&mut s)?;
    Ok(s)
}

// さらに
fn read_username() -> Result<String, std::io::Error> {
    std::fs::read_to_string("username.txt")
}
```

「標準ライブラリに便利関数があることも多い✨」

### Optionでの?

「`?`はOptionにも使える💕」

```rust
fn last_char_of_first_line(text: &str) -> Option<char> {
    text.lines().next()?.chars().last()
}
```

「見て💕」ルスタが指差す。「`lines()`は`Iterator`を返すの。そして`next()`は——」

「…`Option<&str>`を返す⚡」クロムが続ける。「第三部の大聖堂で見た、あの`Iterator`トレイトだ」

あなたは思い出す。神殿に刻まれていた`Iterator`の文字——`Option`を返しながら要素を一つずつ取り出す。今、その二つが繋がった。

「Noneなら即return None⚡」

-----

## 幕間：落とし穴

あなたは自信を持って先に進もうとした。

しかし、足元の地面が崩れる。

**落とし穴だ。**

「待って💕！」ルスタが手を伸ばす。

穴の底で、あなたは自分のコードを見つめる。

```rust
fn process_data() -> Result<String, Error> {
    let data = fetch_data()?;
    let parsed = parse(data)?;
    let result = transform(parsed)?;
    Ok(result)
}
```

「…見た目は美しい。でも落とし穴がある⚡」クロムが穴の上から言う。

「**エラーの型が違う**と、?は使えないの💕」

```rust
// fetch_data() -> Result<String, NetworkError>
// parse()      -> Result<Data, ParseError>
// transform()  -> Result<String, TransformError>

// 全部エラー型が違う！?で自動変換できない...
```

あなたは気づく。?演算子の美しさに酔って、**型の不一致**という現実を見落としていた。

「だから**カスタムエラー型**が必要なんだ💕」

「次の工房で、その作り方を学ぶ⚡」

二人があなたを引き上げる。

落とし穴は、時に最良の教師となる。

-----

## 第五章：エラー型の工房

川を渡ると、小さな工房があった。

様々なエラー型が展示されている。

**「エラー型の工房」**——カスタムエラーを作る場所。

「標準のエラー型だけでは足りないことがある💕」

「自分のエラー型を作ろう⚡」

### 基本的なカスタムエラー

```rust
#[derive(Debug)]
enum AppError {
    NotFound,
    PermissionDenied,
    InvalidInput(String),
    IoError(std::io::Error),
}
```

「enumでエラーの種類を定義✨」

### std::error::Error トレイト

「標準のErrorトレイトを実装すると、**他のエラーと互換性が生まれる**💕」

```rust
use std::fmt;
use std::error::Error;

impl fmt::Display for AppError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match self {
            AppError::NotFound => write!(f, "Resource not found"),
            AppError::PermissionDenied => write!(f, "Permission denied"),
            AppError::InvalidInput(msg) => write!(f, "Invalid input: {}", msg),
            AppError::IoError(e) => write!(f, "IO error: {}", e),
        }
    }
}

impl Error for AppError {
    fn source(&self) -> Option<&(dyn Error + 'static)> {
        match self {
            AppError::IoError(e) => Some(e),
            _ => None,
        }
    }
}
```

### From トレイトで変換

「`?`演算子は**自動変換**もしてくれる⚡」

```rust
impl From<std::io::Error> for AppError {
    fn from(e: std::io::Error) -> Self {
        AppError::IoError(e)
    }
}

fn read_config() -> Result<String, AppError> {
    let content = std::fs::read_to_string("config.txt")?;  // io::Errorが自動でAppErrorに
    Ok(content)
}
```

「Fromを実装すると、`?`が自動で変換してくれる💕」

-----

## 第六章：thiserrorの泉

工房を出ると、清らかな泉があった。

**「thiserrorの泉」**——エラー定義を楽にするクレート。

「手動でDisplay、Error、Fromを実装するのは面倒💕」

「**thiserror**クレートを使おう⚡」

```rust
use thiserror::Error;

#[derive(Error, Debug)]
enum AppError {
    #[error("Resource not found")]
    NotFound,
    
    #[error("Permission denied")]
    PermissionDenied,
    
    #[error("Invalid input: {0}")]
    InvalidInput(String),
    
    #[error("IO error")]
    IoError(#[from] std::io::Error),  // Fromも自動実装
}
```

「`#[derive(Error)]`で**Display, Error, Fromを自動生成**✨」

「ボイラープレートが劇的に減る💕」

-----

## 第七章：anyhowの大河

泉から流れ出た水は、やがて大きな河になっていた。

**「anyhowの大河」**——エラー処理をさらに簡単にするクレート。

「アプリケーションを書く時、**すべてのエラー型を定義するのは大変**💕」

「**anyhow**クレートは、そんな時に便利⚡」

```rust
use anyhow::{Result, Context};

fn read_config() -> Result<Config> {
    let content = std::fs::read_to_string("config.toml")
        .context("Failed to read config file")?;
    
    let config: Config = toml::from_str(&content)
        .context("Failed to parse config")?;
    
    Ok(config)
}
```

「`anyhow::Result<T>`は`Result<T, anyhow::Error>`の短縮形✨」

「**あらゆるエラーを一つの型で扱える**💕」

### contextでエラーに情報を追加

```rust
let file = File::open(path)
    .with_context(|| format!("Failed to open {}", path))?;
```

「エラーが起きた時の**文脈情報を追加**できる⚡」

### thiserror vs anyhow

```yaml
thiserror:
  用途: ライブラリ開発
  特徴: 
    - 明確なエラー型を定義
    - 呼び出し側がエラーの種類を判別可能
    - 型安全

anyhow:
  用途: アプリケーション開発
  特徴:
    - エラー型の定義が不要
    - どんなエラーも受け入れる
    - 開発速度重視
```

「ライブラリでは**thiserror**、アプリでは**anyhow**が定番💕」

あなたは立ち止まる。「どっちを使えばいいんだ？」

「良い疑問だね💕」ルスタが河原の石に腰を下ろす。

「場面によって使い分けるんだ⚡」クロムが説明を始める。

「**公開ライブラリ**を作る時はthiserror。利用者がエラーの種類を`match`で判別したいから」

「**CLIやWebアプリ**を作る時はanyhow💕 エラーはログに出すか画面に表示するだけのことが多いでしょ？」

「内部ライブラリなら、チームの方針次第✨」

あなたは頷く。「じゃあ、両方を組み合わせることもできる？」

「もちろん⚡ よくあるパターンを見せよう」

```rust
// ライブラリ側（thiserror）
#[derive(thiserror::Error, Debug)]
pub enum DbError {
    #[error("connection failed")]
    ConnectionFailed,
}

// アプリ側（anyhow）
fn main() -> anyhow::Result<()> {
    let conn = db::connect()?;  // DbError → anyhow::Error に自動変換
    Ok(())
}
```

「ライブラリ層では明確なエラー型を定義して、アプリ層では`anyhow`で受け止める💕」

「`?`が自動変換してくれるから、シームレスに繋がる⚡」

-----

## 第八章：パニックの断崖

大河を渡ると、断崖絶壁に出た。

崖の下には「PANIC」と書かれた警告が。

**「パニックの断崖」**——回復不能なエラーの場所。

「Rustには二種類のエラーがある⚡」

```yaml
回復可能なエラー（Result）:
  - ファイルが見つからない
  - ネットワーク接続失敗
  - パース失敗
  
回復不能なエラー（panic!）:
  - 配列の範囲外アクセス
  - unwrap()がNone/Errに遭遇
  - プログラムのバグ
```

「panicは**スレッドを停止**する💕」

```rust
fn main() {
    panic!("crash and burn");
}
```

「スタックをアンワインドし、クリーンアップして終了⚡」

### panicを使うべき時

```yaml
panicを使うべき時:
  - バグを発見した時（unreachable!()など）
  - 契約違反（assert!()）
  - 回復しようがない状態

panicを避けるべき時:
  - 予期されるエラー（ファイルがない等）
  - ユーザー入力のバリデーション
  - 外部サービスの失敗
```

「**予期されるエラーにはResult、予期しないエラーにはpanic**✨」

### unwrap/expectの使いどころ

```rust
// プロトタイプ：とりあえず動かす
let file = File::open("test.txt").unwrap();

// 絶対に失敗しない場合
let regex = Regex::new(r"\d+").unwrap();  // リテラルなので

// テストコード
#[test]
fn test_something() {
    let result = do_something().unwrap();
    assert_eq!(result, expected);
}
```

「本番コードでのunwrapは**慎重に**💕」

崖の縁で、あなたはふと疑問を抱く。「panicの挙動は制御できないのか？」

「できる⚡」クロムが断崖を指さす。「Cargo.tomlで設定可能だ」

```toml
[profile.release]
panic = "abort"  # デフォルトは "unwind"
```

「**unwind**はスタックを巻き戻しながらDropを実行する。時間はかかるけど、リソースは確実に解放される💕」

「**abort**は即座にプロセス終了⚡ バイナリサイズが小さくなる。組み込み向けだな」

あなたは頷く。「panicをキャッチすることは？」

「できる✨ でも特殊な用途向け」

```rust
let result = std::panic::catch_unwind(|| {
    panic!("oops!");
});
assert!(result.is_err());  // panicがErrとして返る
```

「ただし`abort`設定時は`catch_unwind`は効かない⚡」クロムが警告する。

「主にFFI境界で使うの💕 panicがC側に伝播するのを防ぐためにね」

-----

## 第九章：早期リターンの展望台

断崖の上に、展望台があった。

峡谷全体が見渡せる。

**「早期リターンの展望台」**——エラー処理のパターンを俯瞰する場所。

「Rustのエラー処理パターンをまとめよう⚡」

### パターン1: 即座に処理

```rust
match do_something() {
    Ok(value) => use_value(value),
    Err(e) => handle_error(e),
}
```

### パターン2: 伝播

```rust
fn caller() -> Result<(), Error> {
    let value = do_something()?;  // エラーなら即return
    use_value(value);
    Ok(())
}
```

### パターン3: デフォルト値

```rust
let value = do_something().unwrap_or(default);
let value = do_something().unwrap_or_else(|| compute_default());
```

### パターン4: 変換して伝播

```rust
fn caller() -> Result<(), MyError> {
    let value = do_something()
        .map_err(|e| MyError::from(e))?;
    Ok(())
}
```

### パターン5: 複数のエラーを集約

```rust
use anyhow::{Result, Context};

fn complex_operation() -> Result<()> {
    let a = step_a().context("Step A failed")?;
    let b = step_b(a).context("Step B failed")?;
    let c = step_c(b).context("Step C failed")?;
    Ok(())
}
```

あなたは展望台の景色に見入る。しかし、まだ疑問が残っている。

「複数のResultやOptionを組み合わせる時、もっと良い方法はないのか？」

「ある⚡」クロムが展望台の柵に寄りかかる。「**コンビネータ**を見せよう」

```rust
// transpose: Option<Result<T, E>> ↔ Result<Option<T>, E>
let opt_result: Option<Result<i32, &str>> = Some(Ok(42));
let result_opt: Result<Option<i32>, &str> = opt_result.transpose();
```

「`Option`と`Result`の入れ子を**ひっくり返せる**の💕」ルスタが説明する。

「これも強力だ⚡」

```rust
// collect: Vec<Result<T, E>> → Result<Vec<T>, E>
// 全部Okなら Vec<T>、一つでもErrなら最初のErr
let results: Vec<Result<i32, &str>> = vec![Ok(1), Ok(2), Ok(3)];
let all: Result<Vec<i32>, &str> = results.into_iter().collect();
```

「ベクタの中のResultを**まとめてチェック**できる✨」

「他にも...」ルスタが続ける。

```rust
// flatten: Option<Option<T>> → Option<T>
let nested: Option<Option<i32>> = Some(Some(42));
let flat: Option<i32> = nested.flatten();

// and / or: 論理演算的な連結
let a: Option<i32> = Some(1);
let b: Option<i32> = Some(2);
assert_eq!(a.and(b), Some(2));  // aがSomeならbを返す
assert_eq!(a.or(b), Some(1));   // aがSomeならaを返す
```

あなたは目を見開く。「これを使えば、matchの入れ子を避けられる...！」

「その通り⚡ フラットで読みやすいコードになる」

-----

## 終章：峡谷を越えて

展望台から見下ろすと、峡谷全体が見えた。

あなたが越えてきた道のり。

### 学んだこと

```yaml
Option:
  目的: "値があるかないか"
  使用場面: null/nilの代わり
  主要メソッド: unwrap, map, and_then, unwrap_or

Result:
  目的: "成功か失敗か"
  使用場面: エラーが起こりうる操作
  主要メソッド: unwrap, map, map_err, and_then

?演算子:
  目的: エラーの伝播を簡潔に
  動作: Ok/Someなら中身を取り出し、Err/Noneなら即return

カスタムエラー:
  手動: enum + Display + Error + From
  thiserror: derive(Error)で自動生成
  anyhow: 型を定義せずにエラー処理

panic:
  用途: 回復不能なエラー、バグの検出
  注意: 予期されるエラーには使わない
```

### 二人からの言葉

「おめでとう💕」ルスタが手を差し伸べる。

「エラーを恐れず、**型の力で制御できる**ようになった✨」

「…次は非同期だ⚡」クロムが言う。「async/await、Future。現代のRustに欠かせない技術」

あなたは峡谷を振り返る。

エラーは敵ではない。**型で表現された情報**だ。

ResultとOptionは、エラーを**見逃せなくする**仕組み。

?演算子は、エラー処理を**優雅にする**道具。

これがRustのエラー処理の力だ。

ルスタが空を見上げる。雲の向こうに、島影が見える。

「ねえ💕」彼女が囁く。「非同期の世界でも、`Result`は活躍するの✨」

「…`async fn`の戻り値⚡」クロムが続ける。「`Future`が`Result`を返す時、`?`はまた違った輝きを見せる」

空に浮かぶ島——あなたの次なる目的地だ。

> **📖 公式ドキュメント参照**
> - [Recoverable Errors with Result](https://doc.rust-lang.org/book/ch09-02-recoverable-errors-with-result.html)
> - [To panic! or Not to panic!](https://doc.rust-lang.org/book/ch09-03-to-panic-or-not-to-panic.html)
> - [Error Handling (The Rust Programming Language)](https://doc.rust-lang.org/book/ch09-00-error-handling.html)

-----

## あなたの旅のログ（第五部）

```yaml
completed_areas:
  - 第一章_峡谷の入り口: "エラー処理への導入"
  - 第二章_Optionの洞窟: "値の有無を表現"
  - 第三章_Resultの吊り橋: "成功/失敗を表現"
  - 第四章_?演算子の渡し舟: "エラー伝播の簡潔な記法"
  - 第五章_エラー型の工房: "カスタムエラーの定義"
  - 第六章_thiserrorの泉: "エラー定義の自動化"
  - 第七章_anyhowの大河: "アプリ開発向けエラー処理"
  - 第八章_パニックの断崖: "回復不能なエラー"
  - 第九章_早期リターン: "エラー処理パターン"

title_earned: "エラーの調停者"

accumulated_titles:
  - "禁域の帰還者"（第一部）
  - "内部可変性の探究者"（第二部）
  - "契約の理解者"（第三部）
  - "鋳型の職人"（第四部）
  - "エラーの調停者"（第五部）

insight_gained: |
  エラーは「型で表現された情報」である。
  
  Optionは「あるかないか」、Resultは「成功か失敗か」。
  ?演算子でエラー伝播を簡潔に書ける。
  
  ライブラリにはthiserror、アプリにはanyhowが定番。
  panicは回復不能なエラーにだけ使う。
```

-----

## 第六部への道

次なる冒険は、**非同期の浮遊島**へと続く。

async/await、Future、tokio——現代のRustに欠かせない非同期処理を学ぶ旅。

### dual-world とは？

| モード | 説明 | 使いどころ |
|:--|:--|:--|
| **Outside Mode** | 設計・分析・コードレビュー | 技術的な議論、アーキテクチャ検討 |
| **Inside Mode** | 体験・実践・物語形式 | 概念理解、入門、モチベーション維持 |

両モードを行き来することで、「理解」と「体感」の両方を得られる。

```
"dual-world を使って Rust の非同期処理を学びたい"
```

と告げれば、いつでも旅を再開できる。

-----

*エラーの峡谷を越えしあなたに、調和の祝福を。*

*— ルスタ💕 & クロム⚡*
