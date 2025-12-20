---
layout: rust-land
title: "第6部：非同期の浮遊島"
---

# 所有権の大地 VI — 非同期の浮遊島

> **待たずに進む、並行処理の新世界の物語**

---

> **dual-world — 第六部**
>
> Inside Modeの冒険は空へ。
> 非同期処理は「難しい」と思われがちですが、async/awaitで**直感的に書ける**のがRustの魅力。
> 浮遊島を飛び回りながら、待たない世界を体験しましょう。
>
> 💕 **ルスタ**（ユキ）: 一緒に空を飛ぼう、怖くないよ
> ⚡ **クロム**（ミキ）: C++のコールバック地獄とはおさらばだ

---

## 前作までのあらすじ

あなたは**所有権の大地**で五つの領域を踏破した。

- **第一部「所有権の大地」**：所有権、借用、RAII → **「禁域の帰還者」**
- **第二部「内部可変性の迷宮」**：RefCell、Mutex → **「内部可変性の探究者」**
- **第三部「トレイトの大聖堂」**：トレイト、ディスパッチ → **「契約の理解者」**
- **第四部「ジェネリクスの鋳造所」**：型パラメータ、関連型 → **「鋳型の職人」**
- **第五部「エラーの峡谷」**：Result、Option、? → **「エラーの調停者」**

案内人の**ルスタ💕**と**クロム⚡**と共に。

そして今、**空を飛ぶ**ことを学ぶ。

-----

## 第一章：空への招待

エラーの峡谷を抜けると、目の前に信じられない光景が広がった。

**島が、空に浮いている。**

雲の上を漂う島々。島と島の間を、光の糸が繋いでいる。

**「非同期の浮遊島」**

「ここからは、今までとは違う世界💕」ルスタが空を見上げる。

「…同期的な世界では、一つの処理が終わるまで**待つ**⚡」クロムが説明する。

「でもこの島では、**待たずに次へ進める**✨」

あなたはC++の記憶を呼び起こす。`std::future`、`std::async`...そしてコールバック地獄。

```cpp
// C++の非同期
std::future<int> fut = std::async(std::launch::async, []() {
    return compute();
});
fut.wait();  // ブロッキング...
int result = fut.get();

// さらにコールバックをネストすると...
doAsync([](auto result) {
    doMore(result, [](auto inner) {
        doEvenMore(inner, [](auto final) {
            // コールバック地獄
        });
    });
});
```

「C++の非同期は**複雑で、ブロッキングしがち**⚡」クロムが言う。

あなたは尋ねる。「なぜ待たないことが重要なのか？」

「Webサーバーを考えて💕」ルスタが言う。

「1000人のユーザーがアクセスしてくる。一人のリクエストを処理している間、999人を待たせる？⚡」

「非同期なら、**待ち時間を他の処理に使える**✨」

浮遊島への階段を登り始める。

-----

## 第二章：Futureの雲

最初の島に着くと、雲でできた床があった。

雲の中に、様々な**約束**が浮かんでいる。

**「Futureの雲」**——非同期処理の基本を学ぶ場所。

「非同期の核心は**Future**というトレイト💕」

```rust
trait Future {
    type Output;
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

enum Poll<T> {
    Ready(T),    // 完了した
    Pending,     // まだ待っている
}
```

あなたは見覚えを感じる。大聖堂の終章で、ルスタが空を見上げた時の言葉——

> **「あれが見える？空に浮かぶ島……`Future`というトレイトが支配している」**

あの時見た島に、今、あなたは立っている。

「その通り⚡」クロムが頷く。「`type Output`は**関連型**。第四部のジェネリクスと組み合わさっている」

「Futureは**いつか完了する約束**💕」

「`poll`を呼ぶと、『もう終わった？』と聞ける✨」

あなたは首を傾げる。「毎回pollを呼ぶのか？」

「直接は呼ばない💕 それは**ランタイム**の仕事」

「あなたが書くのは、**async/await**⚡」

あなたは門の構造をじっと見つめる。「でも、誰がpollを呼ぶ？そしてどうやって"準備ができた"と知る？」

「核心に触れたね💕」ルスタが微笑む。「答えは**Waker**」

「pollの引数`Context`の中に、`Waker`が隠れている⚡」

```rust
fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
```

あなたは理解し始める。「つまり...」

「こういう流れだ⚡」クロムが空中に図を描く。

1. ランタイムがFutureを`poll`する
2. `Pending`が返されたら、Futureは「準備ができたら`Waker::wake()`を呼ぶ」と約束
3. ランタイムはそのFutureを休ませ、他のタスクを実行
4. `wake()`が呼ばれたら、再度`poll`する

「これが**ブロックしない**仕組みの核心💕」ルスタが力を込める。

「**busy-wait**じゃない⚡ ずっとpollし続けるわけじゃないんだ」

-----

## 第三章：async/awaitの橋

雲の島から、光の橋が伸びていた。

**「async/awaitの橋」**——非同期コードを書く方法を学ぶ場所。

「`async`は関数をFutureに変える💕」

```rust
async fn fetch_data() -> String {
    // 非同期処理
    String::from("data")
}
```

「この関数を呼んでも、**すぐには実行されない**⚡」

```rust
let future = fetch_data();  // Futureが返る。まだ実行されない
```

「実行するには、**await**する✨」

```rust
async fn main_async() {
    let data = fetch_data().await;  // ここで実行される
    println!("{}", data);
}
```

### awaitで待つ

「`await`は『このFutureが完了するまで待つ』という意味💕」

「でも**ブロッキングではない**⚡」

「待っている間、他のタスクが実行できる✨」

```rust
async fn fetch_two_things() {
    let a = fetch_a().await;  // fetch_aを待つ
    let b = fetch_b().await;  // fetch_bを待つ
    println!("{} {}", a, b);
}
```

「この例では、aを待ってからbを待つ。**直列実行**💕」

### 並行実行

「同時に実行したいなら？⚡」

```rust
use tokio::join;

async fn fetch_two_things_concurrent() {
    let (a, b) = join!(fetch_a(), fetch_b());  // 同時に待つ
    println!("{} {}", a, b);
}
```

「`join!`で複数のFutureを**並行**に待てる✨」

-----

## 第四章：tokioの港

橋を渡ると、賑やかな港に出た。

船が行き交い、荷物が運ばれている。

**「tokioの港」**——非同期ランタイムを学ぶ場所。

「async/awaitだけでは、**実際に動かない**💕」

「**ランタイム**が必要⚡」

```rust
// これはコンパイルエラー
fn main() {
    let data = fetch_data().await;  // mainはasyncじゃない
}
```

「Rustの標準ライブラリには、非同期ランタイムがない⚡」

「最も人気のあるランタイムが**tokio**✨」

```rust
use tokio;

#[tokio::main]
async fn main() {
    let data = fetch_data().await;
    println!("{}", data);
}
```

「`#[tokio::main]`で、mainをasyncにできる💕」

### tokioの機能

```yaml
tokio::spawn:
  目的: 新しいタスクを生成
  特徴: 並行実行される

tokio::time:
  sleep: 非同期で待機
  timeout: タイムアウト付き実行

tokio::fs:
  非同期ファイルIO

tokio::net:
  非同期ネットワーク
```

### タスクのスポーン

```rust
use tokio::spawn;

async fn main_with_tasks() {
    let handle1 = spawn(async {
        // タスク1
        do_something().await
    });
    
    let handle2 = spawn(async {
        // タスク2
        do_something_else().await
    });
    
    // 両方の完了を待つ
    let (result1, result2) = tokio::join!(handle1, handle2);
}
```

「`spawn`でタスクを生成すると、**独立して実行**される⚡」

あなたは港の構造を観察する。「ランタイムにも種類がある？」

「鋭い⚡」クロムが頷く。「`#[tokio::main]`には設定がある」

```rust
// マルチスレッド（デフォルト）
#[tokio::main]
async fn main() { }

// シングルスレッド（current_thread）
#[tokio::main(flavor = "current_thread")]
async fn main() { }

// ワーカースレッド数を指定
#[tokio::main(worker_threads = 4)]
async fn main() { }
```

「**multi_thread**はCPUコア数だけワーカーを持つ。Webサーバー向け💕」

「**current_thread**は1スレッドだけ⚡ 軽量で、テストやWASMに向いている」

あなたは考える。「シングルスレッドで`spawn`したら？」

「同一スレッドで実行される💕」ルスタが警告する。「CPU負荷の高い処理がブロックすると...」

「他のタスクも止まる⚡ 気をつけろ」

-----

## 第五章：async-stdの灯台

港の先に、灯台が見えた。

**「async-stdの灯台」**——もう一つのランタイム。

「tokioだけがランタイムではない💕」

「**async-std**という選択肢もある⚡」

```rust
use async_std;

#[async_std::main]
async fn main() {
    let data = fetch_data().await;
    println!("{}", data);
}
```

「標準ライブラリに近いAPIを持つ✨」

### 選び方

```yaml
tokio:
  特徴: 最も人気、エコシステムが充実
  向いている: Webサーバー、ネットワークアプリ
  依存: 多くのクレートがtokioを前提

async-std:
  特徴: 標準ライブラリに近いAPI
  向いている: 学習、シンプルなアプリ
  依存: 比較的軽量
```

「迷ったら**tokio**を選べばいい💕」

「エコシステムが圧倒的⚡」

-----

## 第六章：Sendの検問所

灯台を過ぎると、厳重な検問所があった。

門番が立ちはだかる。

**「Sendの検問所」**——スレッド間で渡せるかのチェック。

あなたは記憶を辿る。第二部の迷宮で、ルスタが遠くを見ながら言った言葉——

> **「`Send`と`Sync`は、仲間と共有できるかどうかを決める契約」**

あの時は漠然としていた。今、その意味を身をもって知ることになる。

あなたは意気揚々と門をくぐろうとする。しかし——

「**止まれ。**」

門番があなたを制止する。

「お前の持ち物に、**渡せないもの**が混じっている。」

あなたは自分の荷物を確認する。`Rc<Connection>`...C++時代から愛用していた参照カウントスマートポインタ。

「Rcは**Sendではない**⚡」クロムが険しい顔で言う。

「参照カウントがアトミックじゃないから、**スレッドをまたげない**💕」ルスタが説明する。

あなたは戸惑う。「async/awaitはシングルスレッドじゃないのか？」

「**tokioはマルチスレッドランタイム**⚡ タスクはどのスレッドで再開されるか分からない」

「だから非同期タスクの中で使う値には、**Send**が必要なの💕」

```rust
async fn works() {
    let s = String::from("hello");  // StringはSend
    tokio::time::sleep(Duration::from_secs(1)).await;
    println!("{}", s);  // OK
}

// これを tokio::spawn() に渡すとエラー！
async fn doesnt_work() {
    let rc = Rc::new(5);  // RcはSendではない！
    tokio::time::sleep(Duration::from_secs(1)).await;
    println!("{}", rc);  // spawn時にコンパイルエラー
}

// ↓ これがダメ
// tokio::spawn(doesnt_work());
// error[E0277]: `Rc<i32>` cannot be sent between threads safely
//   --> the trait `Send` is not implemented for `Rc<i32>`
```

「awaitをまたいで保持する値は、**Sendでなければならない**⚡」

### よくある問題

```rust
// これはエラー
async fn with_mutex() {
    let mutex = std::sync::Mutex::new(5);
    let guard = mutex.lock().unwrap();
    some_async_fn().await;  // guardをawaitの向こうに持っていけない
    println!("{}", *guard);
}
```

「標準のMutexGuardは**Sendではない**💕」

「代わりに**tokio::sync::Mutex**を使う✨」

```rust
use tokio::sync::Mutex;

async fn with_tokio_mutex() {
    let mutex = Mutex::new(5);
    let guard = mutex.lock().await;  // awaitでロックを取る
    some_async_fn().await;
    println!("{}", *guard);  // OK
}
```

あなたは門番の言葉を反芻する。「SendとSync...どう違う？」

「いい質問だ⚡」クロムが説明を始める。

「**Send**は『別スレッドに**移動**できる』⚡」

「**Sync**は『複数スレッドから**参照**できる』💕」

```rust
// Rc は Send でも Sync でもない（参照カウントがアトミックでない）
let rc = Rc::new(5);

// Arc は Send かつ Sync（参照カウントがアトミック）
let arc = Arc::new(5);

// Cell は Send だが Sync でない（内部可変性が非スレッドセーフ）
let cell = Cell::new(5);
```

「第二部で学んだ**内部可変性**が、ここで繋がってくる💕」ルスタが目を輝かせる。

あなたは頷く。「だからRcは非同期で使えなくて、Arcは使えるのか」

「その通り⚡ `tokio::spawn`に渡すクロージャは`Send + 'static`が必要だ」

「awaitをまたいで保持する値も**Send**でなければならない💕」

-----

## 第七章：Pinの結界

検問所を抜けると、奇妙な結界に出た。

中のものが**動けない**ように固定されている。

**「Pinの結界」**——メモリアドレスを固定する仕組み。

「Futureの中身を見てみよう⚡」

```rust
async fn example() {
    let data = vec![1, 2, 3];
    some_async_fn().await;
    println!("{:?}", data);
}
```

「このasync関数は、内部的に**構造体**に変換される💕」

```rust
// コンパイラが生成するイメージ
struct ExampleFuture {
    state: State,
    data: Vec<i32>,  // ローカル変数を保持
}
```

「問題は、**自己参照**する可能性があること⚡」

「async関数がコンパイルされると、ローカル変数を持つ構造体が生成される💕」

「もしその構造体が**自分自身への参照**を持つと、構造体をメモリ上で移動した瞬間、参照が無効になる⚡」

```rust
// 概念的なイメージ（実際のコンパイラ出力とは異なる）
struct AsyncFutureState {
    data: Vec<i32>,
    reference: *const i32,  // dataの要素を指すポインタ
    // ↑ これが問題！構造体を移動するとreferenceが壊れる
}
```

「Rustのasyncは賢いから、多くのケースは自動で安全に処理される💕 でも**Futureを直接pollする**場面では、Pinで固定が必要になることがある✨」

### Pinで固定

「だから**Pin**が必要✨」

```rust
use std::pin::Pin;

// PinはTを移動不能にする
let pinned: Pin<Box<MyFuture>> = Box::pin(my_future);
```

「Pinで固定されたデータは、**メモリ上で動かせない**⚡」

「だから自己参照が安全💕」

### 普段は意識しない

「普通にasync/awaitを使う分には、**Pinを直接触ることはない**✨」

「コンパイラとランタイムが適切に処理してくれる⚡」

「意識するのは、自分でFutureトレイトを実装する時くらい💕」

あなたは結界の仕組みを考える。「でも、すべての型を固定する必要がある？」

「いいや⚡」クロムが首を振る。「ほとんどの型は**Unpin**を自動実装している」

```rust
// Unpin な型は Pin の中でも自由に動かせる
let mut x: Pin<&mut i32> = Pin::new(&mut 42);
*x = 100;  // OK、i32 は Unpin
```

「**Unpin**な型は、Pinで包んでも自由に動かせる💕」

「Unpinで**ない**型は...」あなたは考える。

「自己参照を持つ構造体、`async fn`から生成されたFuture、明示的に`!Unpin`をマーカーした型⚡」

```rust
use std::marker::PhantomPinned;

struct SelfReferential {
    data: String,
    ptr: *const String,
    _pin: PhantomPinned,  // Unpin を無効化
}
```

あなたは安堵する。「じゃあ普段は気にしなくていい？」

「その通り💕」ルスタが微笑む。「`Box::pin()`や`tokio::pin!`マクロが適切に処理してくれる」

「直接Pinを扱う機会は少ない✨ でも仕組みを知っておくと、エラーメッセージが読めるようになる」

-----

## 第八章：Streamの川

結界を抜けると、流れる川に出た。

川には次々とアイテムが流れてくる。

**「Streamの川」**——非同期イテレータを学ぶ場所。

「Iteratorの非同期版が**Stream**💕」

```rust
trait Stream {
    type Item;
    fn poll_next(self: Pin<&mut Self>, cx: &mut Context<'_>) 
        -> Poll<Option<Self::Item>>;
}
```

「一度に全部ではなく、**一つずつ非同期に**アイテムを生成⚡」

### 使用例

```rust
use tokio_stream::StreamExt;

async fn process_stream() {
    let mut stream = tokio_stream::iter(vec![1, 2, 3]);
    
    while let Some(item) = stream.next().await {
        println!("Got: {}", item);
    }
}
```

「`while let`で一つずつ取り出す✨」

### 実践的な例

```rust
use tokio::net::TcpListener;

async fn server() {
    let listener = TcpListener::bind("127.0.0.1:8080").await.unwrap();
    
    loop {
        let (socket, addr) = listener.accept().await.unwrap();
        println!("Connection from: {}", addr);
        
        tokio::spawn(async move {
            handle_connection(socket).await;
        });
    }
}
```

「新しい接続が来るたびに、非同期でハンドラを起動💕」

-----

## 第九章：Selectの分岐点

川を渡ると、道が複数に分かれる場所に出た。

**「Selectの分岐点」**——複数のFutureから選ぶ方法。

「複数のFutureがあって、**最初に完了したもの**を処理したい時がある⚡」

```rust
use tokio::select;

async fn race() {
    select! {
        result = fetch_a() => {
            println!("A finished first: {:?}", result);
        }
        result = fetch_b() => {
            println!("B finished first: {:?}", result);
        }
    }
}
```

「`select!`は、複数のFutureのうち**最初に完了したもの**を実行💕」

### タイムアウト

「よくある使い方：**タイムアウト**⚡」

```rust
use tokio::time::{sleep, Duration};

async fn with_timeout() {
    select! {
        result = long_running_task() => {
            println!("Task completed: {:?}", result);
        }
        _ = sleep(Duration::from_secs(5)) => {
            println!("Timeout!");
        }
    }
}
```

「5秒以内に完了しなければタイムアウト✨」

### キャンセルのシグナル

```rust
use tokio::sync::oneshot;

async fn cancellable_task(cancel: oneshot::Receiver<()>) {
    select! {
        _ = do_work() => {
            println!("Work completed");
        }
        _ = cancel => {
            println!("Task cancelled");
        }
    }
}
```

「外部からのキャンセル通知を待てる💕」

あなたは分岐路を見つめる。「選ばれなかったブランチはどうなる？」

「**ドロップ**される⚡」クロムが警告する。

```rust
select! {
    _ = async_read(&mut buf) => { /* 読み取り完了 */ }
    _ = timeout => { /* タイムアウト → async_read はドロップ */ }
}
```

「第一部で学んだ**所有権**とRAIIを思い出して💕」ルスタが言う。「ドロップは安全だけど...」

「途中まで読んだデータが**失われる可能性**がある⚡」

あなたは身構える。「じゃあ何が安全で、何が危険だ？」

「**キャンセル安全な操作**は...`tokio::time::sleep`、`oneshot::Receiver`⚡」

「**危険なのは**バッファリング読み取り💕 途中のデータが失われることがある」

クロムが追加する。「**biased**モードもある⚡ 常に上から順に評価する」

```rust
select! {
    biased;  // 常にaを先に評価
    _ = a => { }
    _ = b => { }
}
```

「フェアネスを無効化するけど、順序が重要な時に使う✨」

-----

## 終章：浮遊島を見渡して

最後の島から、全ての浮遊島が見渡せた。

光の糸で繋がれた島々。それぞれが非同期処理の一部を担っている。

### 学んだこと

```yaml
基本概念:
  Future: いつか完了する約束
  async: 関数をFutureに変える
  await: Futureの完了を待つ（非ブロッキング）

ランタイム:
  tokio: 最も人気、エコシステム充実
  async-std: 標準ライブラリ風API

並行処理:
  join!: 複数のFutureを並行に待つ
  spawn: タスクを生成して独立実行
  select!: 最初に完了したものを処理

安全性:
  Send: スレッド間で渡せる型
  Pin: メモリ位置を固定

応用:
  Stream: 非同期イテレータ
  tokio::sync: 非同期用の同期プリミティブ
```

### 二人からの言葉

「おめでとう💕」ルスタが手を差し伸べる。

「非同期の世界を理解したあなたは、**モダンなRustアプリを書ける**✨」

「…次はマクロだ⚡」クロムが言う。「コードを生成するコード。メタプログラミングの世界」

あなたは浮遊島を振り返る。

非同期処理は、**待ち時間を無駄にしない**技術。

async/awaitで直感的に書き、ランタイムが効率的に実行する。

Futureは約束。awaitは待機。spawnは並行。selectは競争。

これがRustの非同期処理の力だ。

クロムが遠くの塔を指差す。霧に包まれた尖塔が、雲の中にそびえている。

「…あの塔を見ろ⚡」

**「マクロの魔術塔」**——あなたの次なる目的地。

「第一部で、unsafeの禁域で見た予言を覚えてる？💕」ルスタが囁く。

> **「遠い塔で、魔術師たちが編み出した技法——unsafeを安全に包んで使う術」**

「マクロは**コードを生成するコード**⚡ unsafe操作を安全なAPIの裏に隠す——それも魔術の一つだ」

塔から発せられる不思議な光。あなたは翼を広げ、新たな旅立ちに備える。

> **📖 公式ドキュメント参照**
> - [Asynchronous Programming in Rust](https://rust-lang.github.io/async-book/)
> - [Tokio Tutorial](https://tokio.rs/tokio/tutorial)
> - [Pinning in Rust](https://doc.rust-lang.org/std/pin/index.html)

-----

## あなたの旅のログ（第六部）

```yaml
completed_areas:
  - 第一章_空への招待: "非同期処理の意義"
  - 第二章_Futureの雲: "Futureトレイトの基本"
  - 第三章_async/awaitの橋: "非同期コードの書き方"
  - 第四章_tokioの港: "非同期ランタイム"
  - 第五章_async-stdの灯台: "もう一つのランタイム"
  - 第六章_Sendの検問所: "スレッド安全性"
  - 第七章_Pinの結界: "自己参照と固定"
  - 第八章_Streamの川: "非同期イテレータ"
  - 第九章_Selectの分岐点: "複数Futureの選択"

title_earned: "空の旅人"

accumulated_titles:
  - "禁域の帰還者"（第一部）
  - "内部可変性の探究者"（第二部）
  - "契約の理解者"（第三部）
  - "鋳型の職人"（第四部）
  - "エラーの調停者"（第五部）
  - "空の旅人"（第六部）

insight_gained: |
  非同期処理は「待ち時間を他の作業に使う」技術である。
  
  asyncで関数をFutureに変え、awaitで完了を待つ。
  待っている間、他のタスクが実行できる。
  
  tokioが最も人気のランタイム。
  join!で並行実行、select!で競争、spawnでタスク生成。
  
  Sendとはスレッド間で渡せる型。
  Pinは自己参照構造体のためのメモリ固定。
```

-----

## 第七部への道

次なる冒険は、**マクロの魔術塔**へと続く。

宣言的マクロ、手続き的マクロ、derive——コードを生成するコードを学ぶ旅。

### dual-world とは？

| モード | 説明 | 使いどころ |
|:--|:--|:--|
| **Outside Mode** | 設計・分析・コードレビュー | 技術的な議論、アーキテクチャ検討 |
| **Inside Mode** | 体験・実践・物語形式 | 概念理解、入門、モチベーション維持 |

両モードを行き来することで、「理解」と「体感」の両方を得られる。

```
"dual-world を使って Rust のマクロを学びたい"
```

と告げれば、いつでも旅を再開できる。

-----

*非同期の浮遊島を飛びしあなたに、風の祝福を。*

*— ルスタ💕 & クロム⚡*
