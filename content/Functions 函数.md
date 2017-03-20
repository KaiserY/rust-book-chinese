# 函数

> [functions.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/functions.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

到目前为止你应该见过一个函数，`main`函数：

```rust
fn main() {
}
```

这可能是最简单的函数声明。就像我们之前提到的，`fn`表示“这是一个函数”，后面跟着名字，一对括号因为这函数没有参数，然后是一对大括号代表函数体。下面是一个叫`foo`的函数：

```rust
fn foo() {
}
```

那么有参数是什么样的呢？下面这个函数打印一个数字：

```rust
fn print_number(x: i32) {
    println!("x is: {}", x);
}
```

下面是一个使用了`print_number`函数的完整的程序：

```rust
fn main() {
    print_number(5);
}

fn print_number(x: i32) {
    println!("x is: {}", x);
}
```

如你所见，函数参数与`let`声明非常相似：参数名加上冒号再加上参数类型。

下面是一个完整的程序，它将两个数相加并打印结果：

```rust
fn main() {
    print_sum(5, 6);
}

fn print_sum(x: i32, y: i32) {
    println!("sum is: {}", x + y);
}
```

在调用函数和声明函数时，你需要用逗号分隔多个参数。

与`let`不同，你**必须**为函数参数声明类型。下面代码将不能工作：

```rust
fn print_sum(x, y) {
    println!("sum is: {}", x + y);
}
```

你会获得如下错误：

```text
expected one of `!`, `:`, or `@`, found `)`
fn print_sum(x, y) {
```

这是一个有意为之的设计决定。即使像 Haskell 这样的能够全程序推断的语言，注明类型也经常作为一个最佳实践被建议。我们认为即使允许在在函数体中推断，也要强制函数声明参数类型。这是一个全推断与无推断的最佳平衡。

如果我们要一个返回值呢？下面这个函数给一个整数加一：

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
```

Rust 函数只能返回一个值，并且你需要在一个“箭头”后面声明类型，它是一个破折号（`-`）后跟一个大于号（`>`）。

注意这里并没有一个分号。如果你把它加上：

```rust
fn add_one(x: i32) -> i32 {
    x + 1;
}
```

你将会得到一个错误：

```text
error: not all control paths return a value
fn add_one(x: i32) -> i32 {
     x + 1;
}

help: consider removing this semicolon:
     x + 1;
          ^
```

这揭露了关于 Rust 两个有趣的地方：它是一个基于表达式的语言，并且分号与其它基于“大括号和分号”的语言不同。这两个方面是相关的。

## 表达式 VS 语句
Rust 主要是一个基于表达式的语言。只有两种语句，其它的一切都是表达式。

然而这又有什么区别呢？表达式返回一个值，而语句不是。这就是为什么这里我们以“不是所有控制路径都返回一个值”结束：`x + 1;`语句不返回一个值。Rust 中有两种类型的语句：“声明语句”和“表达式语句”。其余的一切是表达式。让我们先讨论下声明语句。

在一些语言中，变量绑定可以被写成一个表达式，不仅仅是语句。例如 Ruby：

```ruby
x = y = 5
```

然而，在 Rust 中，使用`let`引入一个绑定并**不是**一个表达式。下面的代码会产生一个编译时错误：

```rust
let x = (let y = 5); // expected identifier, found keyword `let`
```

编译器告诉我们这里它期望看到表达式的开头，而`let`只能开始一个语句，不是一个表达式。

注意赋值一个已经绑定过的变量（例如，`y = 5`）仍是一个表达式，即使它的（返回）值并不是特别有用。不像其它语言中赋值语句返回它赋的值（例如，前面例子中的`5`），在 Rust 中赋值的值是一个空的元组`()`：

```rust
let mut y = 5;

let x = (y = 6);  // `x` has the value `()`, not `6`.
```

Rust中第二种语句是**表达式语句**。它的目的是把任何表达式变为语句。在实践环境中，Rust 语法期望语句后跟其它语句。这意味着你用分号来分隔各个表达式。这意味着Rust看起来很像大部分其它使用分号做为语句结尾的语言，并且你会看到分号出现在几乎每一行你看到的 Rust 代码。

那么我们说“几乎”的例外是神马呢？你已经见过它了，在这些代码中：

```rust
fn add_one(x: i32) -> i32 {
    x + 1
}
```

我们的函数声称它返回一个`i32`，但是带上个分号，它就会返回一个`()`。Rust意识到这大概不是我们想要的，并在之前我们看到的错误中建议去掉分号。

## 提早返回（Early returns）
不过提早返回怎么破？Rust确实有这么一个关键字，`return`：

```rust
fn foo(x: i32) -> i32 {
    return x;

    // we never run this code!
    x + 1
}
```

使用`return`作为函数的最后一行是可行的，不过被认为是一个糟糕的风格：

```rust
fn foo(x: i32) -> i32 {
    return x + 1;
}
```

如果你之前没有使用过基于表达式的语言，那么前面的没有`return`的定义可能看起来有点奇怪。不过它随着时间的推移它会变得直观。

## 发散函数（Diverging functions）
Rust有些特殊的语法叫“发散函数”，这些函数并不返回：

```rust
fn diverges() -> ! {
    panic!("This function never returns!");
}
```

`panic!`是一个宏，类似我们已经见过的`println!()`。与`println!()`不同的是，`panic!()`导致当前的执行线程崩溃并返回指定的信息。因为这个函数会崩溃，所以它不会返回，所以它拥有一个类型`!`，它代表“发散”。

如果你添加一个叫做`diverges()`的函数并运行，你将会得到一些像这样的输出：

```text
thread ‘main’ panicked at ‘This function never returns!’, hello.rs:2
```

如果你想要更多信息，你可以设定`RUST_BACKTRACE`环境变量来获取 backtrace ：

```bash
$ RUST_BACKTRACE=1 ./diverges
thread 'main' panicked at 'This function never returns!', hello.rs:2
stack backtrace:
   1:     0x7f402773a829 - sys::backtrace::write::h0942de78b6c02817K8r
   2:     0x7f402773d7fc - panicking::on_panic::h3f23f9d0b5f4c91bu9w
   3:     0x7f402773960e - rt::unwind::begin_unwind_inner::h2844b8c5e81e79558Bw
   4:     0x7f4027738893 - rt::unwind::begin_unwind::h4375279447423903650
   5:     0x7f4027738809 - diverges::h2266b4c4b850236beaa
   6:     0x7f40277389e5 - main::h19bb1149c2f00ecfBaa
   7:     0x7f402773f514 - rt::unwind::try::try_fn::h13186883479104382231
   8:     0x7f402773d1d8 - __rust_try
   9:     0x7f402773f201 - rt::lang_start::ha172a3ce74bb453aK5w
  10:     0x7f4027738a19 - main
  11:     0x7f402694ab44 - __libc_start_main
  12:     0x7f40277386c8 - <unknown>
  13:                0x0 - <unknown>
```

如果你需要覆盖一个已经设置的`RUST_BACKTRACE`的值，同时你又不能仅仅 unset 这个变量，这时把它设置成`0`来避免获得 backtrace。任何其它（非 0 ）值将打开 backtrace。

```text
$ export RUST_BACKTRACE=1
...
$ RUST_BACKTRACE=0 ./diverges
thread '<main>' panicked at 'This function never returns!', hello.rs:2
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```

`RUST_BACKTRACE`也可以用于 Cargo 的`run`命令：

```bash
$ RUST_BACKTRACE=1 cargo run
     Running `target/debug/diverges`
thread '<main>' panicked at 'This function never returns!', hello.rs:2
stack backtrace:
   1:     0x7f402773a829 - sys::backtrace::write::h0942de78b6c02817K8r
   2:     0x7f402773d7fc - panicking::on_panic::h3f23f9d0b5f4c91bu9w
   3:     0x7f402773960e - rt::unwind::begin_unwind_inner::h2844b8c5e81e79558Bw
   4:     0x7f4027738893 - rt::unwind::begin_unwind::h4375279447423903650
   5:     0x7f4027738809 - diverges::h2266b4c4b850236beaa
   6:     0x7f40277389e5 - main::h19bb1149c2f00ecfBaa
   7:     0x7f402773f514 - rt::unwind::try::try_fn::h13186883479104382231
   8:     0x7f402773d1d8 - __rust_try
   9:     0x7f402773f201 - rt::lang_start::ha172a3ce74bb453aK5w
  10:     0x7f4027738a19 - main
  11:     0x7f402694ab44 - __libc_start_main
  12:     0x7f40277386c8 - <unknown>
  13:                0x0 - <unknown>
```

发散函数可以被用作任何类型：

```rust
# fn diverges() -> ! {
#    panic!("This function never returns!");
# }
let x: i32 = diverges();
let x: String = diverges();
```

## 函数指针

我们也可以创建指向函数的变量绑定：

```rust
let f: fn(i32) -> i32;
```

`f`是一个指向一个获取`i32`作为参数并返回`i32`的函数的变量绑定。例如：

```rust
fn plus_one(i: i32) -> i32 {
    i + 1
}

// without type inference
let f: fn(i32) -> i32 = plus_one;

// with type inference
let f = plus_one;
```

你可以用`f`来调用这个函数：

```rust
# fn plus_one(i: i32) -> i32 { i + 1 }
# let f = plus_one;
let six = f(5);
```
