# 猜猜看

> [guessing-game.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/guessing-game.md)
> <br>
> commit 6ba952020fbc91bad64be1ea0650bfba52e6aab4

让我学习一些 Rust！作为第一个项目，我们来实现一个经典新手编程问题：猜猜看游戏。它是这么工作的：程序将会随机生成一个 1 到 100 之间的随机数。它接着会提示猜一个数。当我们猜了一个数之后，它会告诉我们是太大了还是太小了。猜对了，它会祝贺我们。听起来如何？

## 准备

让我们准备一个新项目。进入到项目目录。还记得之前如何创建`hello_world`的项目目录和`Cargo.toml`文件的吗？Cargo 有一个命令来做这些。让我们试试：

```bash
$ cd ~/projects
$ cargo new guessing_game --bin
$ cd guessing_game
```

我们将项目名字传递给`cargo new`，然后用了`--bin`标记，因为要创建一个二进制文件，而不是一个库文件。

查看生成的`Cargo.toml`文件：

```toml
[package]

name = "guessing_game"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
```

Cargo 从系统环境变量中获取这些信息。如果这不对，赶紧修改它。

最后，Cargo 为我们生成了一个“Hello, world!”。查看`src/main.rs`文件：

```rust
fn main() {
    println!("Hello, world!");
}
```

让我们编译 Cargo 为我们生成的项目：

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
```

很好！再次打开你的`src/main.rs`文件。我们会将所有代码写在这个文件里。稍后我们会讲到多文件项目。

在我们继续之前，让我们再告诉你一个新的 Cargo 命令：`run`。`cargo run`跟`cargo build`类似，并且还会运行我们刚生成的可执行文件。试试它：

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/debug/guessing_game`
Hello, world!
```

很好！`run`命令在我们需要快速重复运行一个项目时非常方便。我们的游戏就是这么一个项目，在我们添加新内容之前我们需要经常快速测试项目。

## 处理一次猜测
让我们开始吧！我们需要做的第一件事是让我们的玩家输入一个猜测。把这些放入你的`src/main.rs`：

```rust
use std::io;

fn main() {
    println!("Guess the number!");

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");

    println!("You guessed: {}", guess);
}
```

这有好多东西！让我们一点一点地过一遍。

```rust
use std::io;
```

我们需要获取用户输入，并接着打印结果作为输出。为此，我们需要标准库的`io`库。Rust 为所有程序只导入了很少一些东西，[‘prelude’](http://doc.rust-lang.org/nightly/std/prelude/)。如果它不在预先导入中，你将不得不直接`use`它。这还有第二个"prelude",[`io`prelude](http://doc.rust-lang.org/std/io/prelude/index.html)，它也起到了类似的作用：你引入它，它引入一系列拥有的 IO 相关的库。

```rust
fn main() {
```

就像你之前见过的，`main()`是你程序的入口点。`fn`语法声明了一个新函数，`()`表明这里没有参数，而`{`开始了函数体。因为不包含返回类型，它假设是`()`，一个空的[元组](5.3.Primitive Types 原生类型.md#tuples)。

```rust
    println!("Guess the number!");

    println!("Please input your guess.");
```

我们之前学过`println!()`是一个在屏幕上打印[字符串](5.17.Strings 字符串.md)的[宏](5.34.Macros 宏.md)。

```rust
    let mut guess = String::new();
```


现在我们遇到有意思的东西了！这一小行有很多内容。第一个我们需要注意到的是[let语句](5.1.Variable Bindings 变量绑定.md)，它用来创建“变量绑定”。它使用这个形式：

```rust
let foo = bar;
```

这会创建一个叫做`foo`的新绑定，并绑定它到`bar`这个值上。在很多语言中，这叫做一个“变量"，不过 Rust 的变量绑定暗藏玄机。

例如，它们默认是[不可变的](5.10.Mutability 可变性.md)。这时为什么我们的例子使用了`mut`：它让一个绑定可变，而不是不可变。`let`并不从左手边获取一个名字，事实上它接受一个[模式（pattern）](5.14.Patterns 模式.md)。我们会在后面更多的使用模式。现在它使用起来非常简单：

```rust
let foo = 5; // immutable.
let mut bar = 5; // mutable
```

噢，同时`//`会开始一个注释，直到这行的末尾。Rust 忽略[注释](5.4.Comments 注释.md)中的任何内容。

那么现在我们知道了`let mut guess`会引入一个叫做`guess`的可变绑定，不过我们也必须看看`=`的右侧所绑定的内容：`String::new()`。

`String`是一个字符串类型，由标准库提供。[String](5.17.Strings 字符串.md)是一个可增长的，UTF-8编码的文本。

`::new()`语法用了`::`因为它是一个特定类型的”关联函数“。这就是说，它与`String`自身关联，而不是与一个特定的`String`实例关联。一些语言管这叫一个“静态方法”。

这个函数叫做`new()`，因为它创建了一个新的，空的`String`。你会在很多类型上找到`new()`函数，因为它是创建一些类型新值的通常名称。

让我们继续：

```rust
    io::stdin().read_line(&mut guess)
        .expect("Failed to read line");
```

这稍微有点多！让我们一点一点来。第一行有两部分。这是第一部分：

```rust
io::stdin()
```

还记得我们如何在程序的第一行`use std::io`的吗？现在我们调用了一个与之相关的函数。如果我们不`use std::io`，那么我们就得写成`std::io::stdin()`。

这个特殊的函数返回一个指向你终端标准输入的句柄。更具体的，可参考[std::io::Stdin](http://doc.rust-lang.org/stable/std/io/struct.Stdin.html)。

下一部分将用这个句柄去获取用户输入：

```rust
.read_line(&mut guess)
```

这里，我们对我们的句柄调用了[read_line()](http://doc.rust-lang.org/stable/std/io/struct.Stdin.html#method.read_line)方法。[“方法”](5.15.Method Syntax 方法语法.md)就像关联函数，不过只在一个类型的特定实例上可用，而不是这个类型本身。我们也向`read_line()`传递了一个参数：`&mut guess`。

还记得我们上面怎么绑定`guess`的吗？我们说它是可变的。然而，`read_line`并不接收`String`作为一个参数：它接收一个`&mut String`。Rust有一个叫做[“引用”](5.8.References and Borrowing 引用和借用.md)的功能，它允许你对一片数据有多个引用，用它可以减少拷贝。引用是一个复杂的功能，因为Rust的一个主要卖点就是它如何安全和便捷地使用引用。然而，目前我们还不需要知道很多细节来完成我们的程序。现在，所有我们需要了解的是像`let`绑定，引用默认是不可变的。因此，我们需要写成`&mut guess`，而不是`&guess`。

为什么`read_line()`会需要一个字符串的可变引用呢？它的工作是从标准输入获取用户输入，并把它放入一个字符串。所以它用字符串作为参数，为了可以增加输入，它必须是可变的。

不过我们还未完全看完这行代码。虽然它是单独的一行代码，但只是这个单独逻辑代码行的开头部分：

```rust
        .expect("Failed to read line");
```

当你用`.foo()`语法调用一个函数的时候，你可能会引入一个新行符或其它空白。这帮助我们拆分长的行。我们*可以*这么干：

```rust
    io::stdin().read_line(&mut guess).expect("failed to read line");
```

不过这样会难以阅读。所以我们把它分开，3 行对应 3 个方法调用。我们已经谈论过了`read_line()`，不过`expect()`呢？好吧，我们已经提到过`read_line()`将用户输入放入我们传递给它的`&mut String`中。不过它也返回一个值：在这个例子中，一个[io::Result](http://doc.rust-lang.org/stable/std/io/type.Result.html)。Rust的标准库中有很多叫做`Result`的类型：一个泛型[Result](http://doc.rust-lang.org/stable/std/result/enum.Result.html)，然后是子库的特殊版本，例如`io::Result`。

这个`Result`类型的作用是编码错误处理信息。`Result`类型的值，像任何（其它）类型，有定义在其上的方法。在这个例子中，`io::Result`有一个[expect()方法](http://doc.rust-lang.org/stable/std/option/enum.Option.html#method.expect)获取调用它的值，而且如果它不是一个成功的值，[panic!](Error Handling 错误处理.md)并带有你传递给它的信息。这样的`panic!`会使我们的程序崩溃，显示（我们传递的）信息。

如果我们去掉这两个函数调用，我们的程序会编译通过，不过我们会得到一个警告：

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
src/main.rs:10:5: 10:39 warning: unused result which must be used,
#[warn(unused_must_use)] on by default
src/main.rs:10     io::stdin().read_line(&mut guess);
                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

Rust警告我们我们并未使用`Result`的值。这个警告来自`io::Result`的一个特殊注解。Rust 尝试告诉你你并未处理一个可能的错误。阻止错误的正确方法是老实编写错误处理。幸运的是，如果我们只是想如果这有一个问题就崩溃的话，我们可以用这两个小方法。如果我们想从错误中恢复什么的，我们得做点别的，不过我们会把它留给接下来的项目。

这是我们第一个例子仅剩的一行：

```rust
    println!("You guessed: {}", guess);
}
```

这打印出我们保存输入的字符串。`{}`是一个占位符，所以我们传递`guess`作为一个参数。如果我们有多个`{}`，我们应该传递多个参数：

```rust
let x = 5;
let y = 10;

println!("x and y: {} and {}", x, y);
```

简单加愉快。

总而言之，这只是一个观光。我们可以用`cargo run`运行我们写的：

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

好的！我们的第一部分完成了：我们可以从键盘获取输入，并把它打印回去。

## 生成一个秘密数字
接下来，我们要生成一个秘密数字。Rust标准库中还未包含随机数功能。然而，Rust 团队确实提供了一个[`rand` crate](https://crates.io/crates/rand)。一个“包装箱”（crate）是一个 Rust 代码的包。我们已经构建了一个”二进制包装箱“，它是一个可执行文件。`rand`是一个”库包装箱“，它包含被认为应该被其它程序使用的代码。

使用外部包装箱是 Cargo 的亮点。在我们使用`rand`编写代码之前，我们需要修改我们的`Cargo.toml`。打开它，并在末尾增加这几行：

```toml
[dependencies]

rand="0.3.0"
```

`Cargo.toml`的`[dependencies]`部分就像`[package]`部分：所有之后的东西都是它的一部分，直到下一个部分开始。Cargo使用依赖部分来知晓你用的外部包装箱的依赖，和你要求的版本。在这个例子中，我们用了`0.3.0`版本。Cargo理解[语义化版本](http://semver.org/lang/zh-CN/)，它是一个编写版本号的标准。如果我们就是只想使用`0.3.0`，我们可以用`=0.3.0`。如果我们想要使用最新版本我们可以使用`*`或者我们可以使用一个范围的版本。[Cargo文档](http://doc.crates.io/crates-io.html)包含更多细节。

现在，在不修改任何我们代码的情况下，让我们构建我们的项目：

```bash
$ cargo build
    Updating registry `https://github.com/rust-lang/crates.io-index`
 Downloading rand v0.3.8
 Downloading libc v0.1.6
   Compiling libc v0.1.6
   Compiling rand v0.3.8
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
```

（当然，你可能会看到不同的版本）

很多新的输出！现在我们有了一个外部依赖，Cargo 从记录中获取了所有东西的最新版本，它们是来自[Crates.io](https://crates.io/)的一份拷贝。Crates.io 是 Rust 生态系统中人们发表开源 Rust 项目供他人使用的地方。

在更新了记录后，Cargo 检查我们的`[dependencies]`并下载任何我们还没有的东西。在这个例子中，虽然我们只说了我们要依赖`rand`，我们也获取了一份`libc`的拷贝。这是因为`rand`依赖`libc`工作。在下载了它们之后，它编译它们，然后接着编译我们的项目。

如果我们再次运行`cargo build`，我们会得到不同的输出：

```bash
$ cargo build
```

没错，没有输出！Cargo 知道我们的项目被构建了，并且所有它的依赖也被构建了，所以没有理由再做一遍所有这些。没有事情做，它简单地退出了。如果我们再打开`src/main.rs`，做一个无所谓的修改，然后接着再保存，我们就会看到一行：

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
```

所以，我们告诉Cargo我们需要任何`0.3.x`版本的`rand`，并且因此它获取在本文被编写时的最新版，`v0.3.8`。不过你瞧瞧当下一周，`v0.3.9`出来了，带有一个重要的 bug 修改吗？虽然 bug 修改很重要，不过如果`0.3.9`版本包含破坏我们代码的回归呢？

这个问题的回答是现在你会在你项目目录中找到的`Cargo.lock`。当你第一次构建你的项目的时候，Cargo 查明所有符合你的要求的版本，并接着把它们写到了`Cargo.lock`文件里。当你在未来构建你的项目的时候，Cargo 会注意到`Cargo.lock`的存在，并接着使用指定的版本而不是再次去做查明版本的所有工作。这让你有了一个可重复的自动构建。换句话说，我们会保持在`0.3.8`直到我们显式的升级，这对任何使用我们共享的代码的人同样有效，感谢锁文件。

当我们*确实*想要使用`v0.3.9`怎么办？Cargo 有另一个命令，`update`，它代表“忽略锁，搞清楚所有我们指定的最新版本。如果这能工作，将这些版本写入锁文件”。不过，默认，Cargo 只会寻找大于`0.3.0`小于`0.4.0`的版本。如果你想要移动到`0.4.x`，我们不得不直接更新`Cargo.toml`文件。当我们这么做，下一次我们`cargo build`，Cargo会更新索引并重新计算我们的`rand`要求。

关于[Cargo](http://doc.crates.io/)和[它的生态系统](http://doc.crates.io/crates-io.html)有很多东西要说，不过眼下，这是我们需要知道的一切。Cargo让重用库变得真正的简单，并且Rustacean们可以编写更小的由很多子包组装成的项目。

让我们真正的*使用*`rand`，这是我们的下一步：

```rust
extern crate rand;

use std::io;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("failed to read line");

    println!("You guessed: {}", guess);
}
```

先修改第一行。现在它是`extern crate rand`。因为在`[dependencies]`声明了`rand`，我们可以用`extern crate`来让Rust知道我们正在使用它。这也等同于一个`use rand;`，所以我们可以通过`rand::`前缀使用`rand`包装箱中的一切。

下一步，我们增加了另一行`use`：`use rand::Rng`。我们一会将要使用一个方法，并且它要求`Rng`在作用域中才能工作。这个基本观点是：方法定义在一些叫做“特性（traits，也有译作特质）”的东西上面，而为了让方法能够工作，需要这个特性位于作用域中。关于更多细节，阅读[trait](5.19.Traits.md)部分。

这里还有两行我们增加的，在中间：

```rust
    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);
```

我们用`rand::thread_rng()`函数来获取一个随机数生成器的拷贝，它位于我们特定的执行[线程](4.6.Concurrency 并发.md)的本地。因为`use rand::Rng`了，有一个`gen_range()`方法可用。这个函数获取两个参数，并产生一个位于其间的数字。它包含下限，不过不包含上限，所以需要`1`和`101`来生成一个`1`和`100`之间的数。

第二行仅仅打印出了秘密数字，这在开发程序时做简单测试很有用。不过在最终版本中我们会删除它。在开始就打印出结果就没什么可玩的了！

尝试运行新程序几次：

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4
$ cargo run
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

好的！接下来：让我们比较我们的猜测和秘密数字。

## 比较猜测
现在我们得到了用户输入，让我们比较我们的猜测和随机值。这是我们的下一步，虽然它还不能正常工作：

```rust
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("failed to read line");

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less    => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal   => println!("You win!"),
    }
}
```

这有一些新东西。第一个是另一个`use`。我们带来了一个叫做`std::cmp::Ordering`类型到作用域中。接着，底部5行代码使用了它：

```rust
match guess.cmp(&secret_number) {
    Ordering::Less    => println!("Too small!"),
    Ordering::Greater => println!("Too big!"),
    Ordering::Equal   => println!("You win!"),
}
```

`cmp()`可以在任何能被比较的值上调用，并且它获取你想要比较的值的引用。它返回我们之前`use`的`Ordering`类型。我们使用一个[match](5.13.Match 匹配.md)语句来决定具体是哪种`Ordering`。`Ordering`是一个[枚举（enum）](5.12.Enums 枚举.md)，它看起来像这样：

```rust
enum Foo {
    Bar,
    Baz,
}
```

通过这个定义，任何`Foo`可以是`Foo::Bar`或者`Foo::Baz`。我们用`::`来表明一个特定`enum`变量的命名空间。

[Ordering](http://doc.rust-lang.org/stable/std/cmp/enum.Ordering.html)枚举有3个可能的变量：`Less`，`Equal`和`Greater`。`match`语句获取类型的值，并让你为每个可能的值创建一个“分支”。因为有 3 种类型的`Ordering`，我们有 3 个分支：

```rust
match guess.cmp(&secret_number) {
    Ordering::Less    => println!("Too small!"),
    Ordering::Greater => println!("Too big!"),
    Ordering::Equal   => println!("You win!"),
}
```

如果它是`Less`，我们打印` Too small!`，如果它是`Greater`，`Too big!`，而如果`Equal`，`You win!`。`match`真的非常有用，并且在 Rust 中经常使用。

我确实提到过我们还不能正常运行，虽然。让我们试试：

```bash
$ cargo build
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
src/main.rs:28:21: 28:35 error: mismatched types:
 expected `&collections::string::String`,
    found `&_`
(expected struct `collections::string::String`,
    found integral variable) [E0308]
src/main.rs:28     match guess.cmp(&secret_number) {
                                   ^~~~~~~~~~~~~~
error: aborting due to previous error
Could not compile `guessing_game`.
```

噢！这是一个大错误。它的核心是我们有“不匹配的类型”。Rust 有一个强大的静态类型系统。然而，它也有类型推断。当我们写`let guess = String::new()`，Rust能够推断出`guess`应该是一个`String`，并因此不需要我们写出类型。而我们的`secret_number`，这有很多类型可以有从`1`到`100`的值：`i32`，一个 32 位数，或者`u32`，一个无符号的32位值，或者`i64`，一个 64 位值。或者其它什么的。目前为止，这并不重要，所以 Rust 默认为`i32`。然而，这里，Rust 并不知道如何比较`guess`和`secret_number`。它们必须是相同的类型。最终，我们想要我们作为输入读到的`String`转换为一个真正的数字类型，来进行比较。我们可以用额外 3 行来搞定它。这是我们的新程序：

```rust
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    println!("Please input your guess.");

    let mut guess = String::new();

    io::stdin().read_line(&mut guess)
        .expect("failed to read line");

    let guess: u32 = guess.trim().parse()
        .expect("Please type a number!");

    println!("You guessed: {}", guess);

    match guess.cmp(&secret_number) {
        Ordering::Less    => println!("Too small!"),
        Ordering::Greater => println!("Too big!"),
        Ordering::Equal   => println!("You win!"),
    }
}
```

新的 3 行是：

```rust
    let guess: u32 = guess.trim().parse()
        .expect("Please type a number!");
```

稍等，我认为我们已经用过了一个`guess`？确实，不过 Rust 允许我们用新值“遮盖（shadow）”之前的`guess`。这在这种具体的情况中经常被用到，`guess`开始是一个`String`，不过我们想要把它转换为一个`u32`。遮盖（Shadowing）让我们重用`guess`名字，而不是强迫我们想出两个独特的像`guess_str`和`guess`，或者别的什么。

我们绑定`guess`到一个看起来像我们之前写的表达式：

```rust
guess.trim().parse()
```

这里，`guess`引用旧的`guess`，那个我们输入用到的`String`。`String`的`trim()`方法会去掉我们字符串开头和结尾的任何空格。这很重要，因为我们不得不按“回车”键来满足`read_line()`。这意味着如果输入`5`并按回车，`guess`看起来像这样：`5\n`。`\n`代表“新行”，回车键。`trim()`去掉这些，保留`5`给我们的字符串。[字符串的`parse()`方法](http://doc.rust-lang.org/stable/std/primitive.str.html#method.parse)将字符串解析为一些类型的数字。因为它可以解析多种数字，我们需要给Rust一些提醒作为我们具体想要的数字的类型。因此，`let guess: u32`。`guess`后面的分号（`:`）告诉 Rust 我们要标注它的类型。`u32`是一个无符号的，32位整型。Rust 有[一系列内建数字类型](Primitive Types 原生类型.md#数字类型)，不过我们选择了`u32`。它是一个小正数的默认好选择。

就像`read_line()`，我们调用`parse()`可能产生一个错误。如果我们的字符串包含`A👍%?`呢？并不能将它们转换成一个数字。为此，我们将做我们在`read_line()`时做的相同的事：使用`expect()`方法来在这里出现错误时崩溃。

让我们尝试下我们的程序！

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

很好！你可以看到我甚至在我的猜测前加上了空格，不过它仍然识别出我猜了 76。运行这个程序几次，并检测猜测正确的值，和小的值。

现在我们让游戏大体上能玩了，不过我们只能猜一次。让我们增加循环来改变它！

## 循环
`loop`关键字给我们一个无限循环。让我们加上它：

```rust
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("failed to read line");

        let guess: u32 = guess.trim().parse()
            .expect("Please type a number!");

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => println!("You win!"),
        }
    }
}
```

并试试看。不过稍等，难道我们仅仅加上一个无限循环吗？是的。记得我们我们关于`parse()`的讨论吗？如果我们给出一个非数字回答，明显我们会`return`并退出：

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit
thread '<main>' panicked at 'Please type a number!'
```

啊哈！`quit`确实退出了。就像任何其它非数字输入。好吧，这至少不是最差的想法。首先，如果你赢得了游戏，那我们就真的退出它：

```rust
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("failed to read line");

        let guess: u32 = guess.trim().parse()
            .expect("Please type a number!");

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

通过在`You win!`后增加`break`，我们将在你赢了后退出循环。退出循环也意味着退出程序，因为它是`main()`中最后的东西。我们仅仅需要再做一个小修改：当谁输入了一个非数字，我们并不想退出，我们就想忽略它。我们可以这么做：

```rust
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    println!("The secret number is: {}", secret_number);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

这是改变了的行：

```rust
let guess: u32 = match guess.trim().parse() {
    Ok(num) => num,
    Err(_) => continue,
};
```

这是你如何大体上从“错误就崩溃”移动到“确实处理错误”，通过从`ok().expect()`切换到一个`match`语句。`parse()`返回的`Result`就是一个像`Ordering`一样的枚举，不过在这个例子中，每个变量有一些数据与之相关：`Ok`是一个成功，而`Err`是一个失败。每一个都包含更多信息：成功的解析为整型，或一个错误类型。在这个例子中，我们`match`为`Ok(num)`，它设置了`Ok`内叫做`num`的值，接着在右侧返回它。在`Err`的情况下，我们并不关心它是什么类型的错误，所以我们仅仅使用`_`而不是一个名字。这忽略错误，并`continue`造成我们进行`loop`的下一次迭代。

现在应该搞定了！试试看：

```bash
$ cargo run
   Compiling guessing_game v0.1.0 (file:///home/you/projects/guessing_game)
     Running `target/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

狂拽炫酷！通过一个最后的修改，我们就完成了猜猜看游戏。你能想到它是什么吗？对了，我们并不想打印出秘密数字。它有利于测试，不过有点毁游（san）戏（guan）的味道。这是最终源码：

```rust
extern crate rand;

use std::io;
use std::cmp::Ordering;
use rand::Rng;

fn main() {
    println!("Guess the number!");

    let secret_number = rand::thread_rng().gen_range(1, 101);

    loop {
        println!("Please input your guess.");

        let mut guess = String::new();

        io::stdin().read_line(&mut guess)
            .expect("failed to read line");

        let guess: u32 = match guess.trim().parse() {
            Ok(num) => num,
            Err(_) => continue,
        };

        println!("You guessed: {}", guess);

        match guess.cmp(&secret_number) {
            Ordering::Less    => println!("Too small!"),
            Ordering::Greater => println!("Too big!"),
            Ordering::Equal   => {
                println!("You win!");
                break;
            }
        }
    }
}
```

## 完成！
此刻，你成功地构建了猜猜看游戏！恭喜！

这第一个项目展示了：`let`、`match`、方法、关联函数、使用外部包装箱等。下一个项目将会向你展示更多。
