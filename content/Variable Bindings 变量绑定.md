# 变量绑定

> [variable-bindings.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/variable-bindings.md)
> <br>
> commit 20abf050e7f5698cd1012de00295ec805143735a

事实上每一个非“Hello World” Rust 程序都用了**变量绑定**。他们将一些值绑定到一个名字上，这样可以在之后使用他们。`let`被用来声明一个绑定，像这样：

```rust
fn main() {
    let x = 5;
}
```

在每个例子中都写上`fn main() {`有点冗长，所以之后我们将省略它。如果你是一路看过来的，确保你写了`main()`函数，而不是省略不写。否则，你将得到一个错误。

## 模式（Patterns）

在许多语言中，这叫做**变量**。不过 Rust 的变量绑定有一些不同的巧妙之处。例如`let`语句的左侧是一个“[模式](Patterns 模式.md)”，而不仅仅是一个变量。这意味着我们可以这样写：

```rust
let (x, y) = (1, 2);
```

在这个语句被计算后，`x`将会是1，而`y`将会是2。模式非常强大，并且本书中有[关于它的部分](Patterns 模式.md)。我们现在还不需要这些功能，所以接下来你只需记住有这个东西就行了。

## 类型注解（Type annotations）

Rust 是一个静态类型语言，这意味着我们需要先确定我们需要的类型。那为什么我们第一个例子能编译过呢？好的，Rust有一个叫做**类型推断**的功能。如果它能确认这是什么类型，Rust 不需要你明确地指出来。

若你愿意，我们也可以加上类型。类型写在一个冒号（`:`）后面：

```rust
let x: i32 = 5;
```

如果我叫你对着全班同学大声读出这一行，你应该大喊“`x`被绑定为`i32`类型，它的值是`5`”。

在这个例子中我们选择`x`代表一个 32 位的有符号整数。Rust 有许多不同的原生整数类型。以`i`开头的代表有符号整数而`u`开头的代表无符号整数。可能的整数大小是 8、16、32 和 64 位。

在之后的例子中，我们可能会在注释中注明变量类型。例子看起来像这样：

```rust
fn main() {
    let x = 5; // x: i32
}
```

注意注释和`let`表达式有类似的语法。理想的 Rust 代码中不应包含这类注释。不过我们偶尔会这么做来帮助你理解 Rust 推断的是什么类型。

## 可变性（Mutability）

绑定默认是**不可变的**（*immutable*）。下面的代码将不能编译：

```rust
let x = 5;
x = 10;
```

它会给你如下错误：

```text
error: re-assignment of immutable variable `x`
     x = 10;
     ^~~~~~~
```

如果你想一个绑定是可变的，使用`mut`：

```rust
let mut x = 5; // mut x: i32
x = 10;
```

不止一个理由使得绑定默认不可变的，不过我们可以通过一个 Rust 的主要目标来理解它：安全。如果你没有使用`mut`，编译器会捕获它，让你知道你改变了一个你可能并不打算让它改变的值。如果绑定默认是可变的，编译器将不可能告诉你这些。如果你确实想变量可变，解决办法也非常简单：加个`mut`。

尽量避免可变状态有一些其它好处，不过这不在这个教程的讨论范围内。大体上，你总是可以避免显式可变量，并且这也是 Rust 希望你做的。即便如此，有时，可变量是你需要的，所以这并不是被禁止的。

## 初始化绑定（Initializing bindings）

Rust 变量绑定有另一个不同于其它语言的方面：绑定要求在可以使用它之前必须初始化。

让我们尝试一下。将你的`src/main.rs`修改为为如下：

```rust
fn main() {
    let x: i32;

    println!("Hello world!");
}
```

你可以用`cargo build`命令去构建它。它依然会输出“Hello, world!”，不过你会得到一个警告：

```text
   Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
src/main.rs:2:9: 2:10 warning: unused variable: `x`, #[warn(unused_variables)] on by default
src/main.rs:2     let x: i32;
                      ^
```

Rust 警告我们从未使用过这个变量绑定，但是因为我们从未用过它，无害不罚。然而，如果你确实想使用`x`，事情就不一样了。让我们试一下。修改代码如下：

```rust
fn main() {
    let x: i32;

    println!("The value of x is: {}", x);
}
```

然后尝试构建它。你会得到一个错误：

```bash
$ cargo build
   Compiling hello_world v0.0.1 (file:///home/you/projects/hello_world)
src/main.rs:4:39: 4:40 error: use of possibly uninitialized variable: `x`
src/main.rs:4     println!("The value of x is: {}", x);
                                                    ^
note: in expansion of format_args!
<std macros>:2:23: 2:77 note: expansion site
<std macros>:1:1: 3:2 note: in expansion of println!
src/main.rs:4:5: 4:42 note: expansion site
error: aborting due to previous error
Could not compile `hello_world`.
```

Rust 是不会让我们使用一个没有经过初始化的值的。

让我们讨论一下我们添加到`println!`中的内容。

如果你输出的字符串中包含一对大括号（`{}`，一些人称之为胡须。。（译注：moustaches，八字胡）），Rust将把它解释为插入值的请求。**字符串插值**（*String interpolation*）是一个计算机科学术语，代表“在字符串中插入值”。我们加上一个逗号，然后是一个`x`，来表示我们想插入`x`的值。逗号用来分隔我们传递给函数和宏的参数，如果你想传递多个参数的话。

当你只写了大括号的时候，Rust 会尝试检查值的类型来显示一个有意义的值。如果你想指定详细的语法，有[很多选项可供选择](http://doc.rust-lang.org/std/fmt/)。现在，让我们保持默认格式，整数并不难打印。

## 作用域和隐藏（Scope and shadowing）

让我们回到绑定的话题上。变量绑定有一个作用域 - 他们被限制只能在他们被定义的块中存在。一个块是一个被`{`和`}`包围的语句集合。函数定义也是块！在下面的例子中我们定义了两个变量绑定，`x`和`y`，他们位于不同的作用域中。`x`可以在`fn main() {}`块中被访问，而`y`只能在内部块内访问：

```rust
fn main() {
    let x: i32 = 17;
    {
        let y: i32 = 3;
        println!("The value of x is {} and value of y is {}", x, y);
    }
    println!("The value of x is {} and value of y is {}", x, y); // This won't work.
}
```

第一个`println!`将会打印“The value of x is 17 and the value of y is 3”，不过这个并不能编译成功，因为第二个`println!`并不能访问`y`的值，因为它已不在作用域中。相反我们得到如下错误：

```bash
$ cargo build
   Compiling hello v0.1.0 (file:///home/you/projects/hello_world)
main.rs:7:62: 7:63 error: unresolved name `y`. Did you mean `x`? [E0425]
main.rs:7     println!("The value of x is {} and value of y is {}", x, y); // This won't work.
                                                                       ^
note: in expansion of format_args!
<std macros>:2:25: 2:56 note: expansion site
<std macros>:1:1: 2:62 note: in expansion of print!
<std macros>:3:1: 3:54 note: expansion site
<std macros>:1:1: 3:58 note: in expansion of println!
main.rs:7:5: 7:65 note: expansion site
main.rs:7:62: 7:63 help: run `rustc --explain E0425` to see a detailed explanation
error: aborting due to previous error
Could not compile `hello`.

To learn more, run the command again with --verbose.
```

另外，变量可以被隐藏。这意味着一个后声明的并位于同一作用域的相同名字的变量绑定将会覆盖前一个变量绑定：

```rust
let x: i32 = 8;
{
    println!("{}", x); // Prints "8".
    let x = 12;
    println!("{}", x); // Prints "12".
}
println!("{}", x); // Prints "8".
let x =  42;
println!("{}", x); // Prints "42".
```

隐藏和可变绑定可能表现为同一枚硬币的两面，他们是两个不同的概念，不能互换使用。举个例子，隐藏允许我们将一个名字重绑定为不同的类型。它也可以改变一个绑定的可变性。注意隐藏并不改变和销毁被绑定的值，这个值会在离开作用域之前继续存在，即便无法通过任何手段访问到它。

```rust
let mut x: i32 = 1;
x = 7;
let x = x; // `x` is now immutable and is bound to `7`

let y = 4;
let y = "I can also be bound to text!"; // `y` is now of a different type
```
