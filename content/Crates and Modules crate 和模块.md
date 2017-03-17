# 包装箱和模块

> [crates-and-modules.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/crates-and-modules.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

当一个项目变大以后，良好的软件工程实践是把它分为一堆较小的部分，再把它们装配到一起。定义良好的接口也非常重要，以使有些功能是私有的，而有些是公有的。Rust 有一个模块系统来帮助我们处理这些工作。

# 基础术语：包装箱和模块
Rust 有两个不同的术语与模块系统有关：**包装箱**（*crate*）和**模块**（*module*）。包装箱是其它语言中**库**（*library*）或**包**（*package*）的同义词。因此，“Cargo”则是 Rust 包管理工具的名字：你通过 Cargo 把你的包装箱交付给别人。包装箱可以根据项目的不同，生成可执行文件或库文件。

每个包装箱有一个隐含的**根模块**（*root module*）包含了该包装箱的代码。你可以在根模块下定义一个子模块树。模块让你可以在包装箱内部为代码分区。

作为一个例子，让我们来创建一个**短语**（*phrases*）包装箱，它会给我们一些不同语言的短语。为了简单起见，仅有“你好”和“再见”这两种短语，并使用英语和日语作为这些短语的语言。我们采用如下模块布局：

```text
                                    +-----------+
                                +---| greetings |
                  +---------+   |   +-----------+
              +---| english |---+
              |   +---------+   |   +-----------+
              |                 +---| farewells |
+---------+   |                     +-----------+
| phrases |---+
+---------+   |                     +-----------+
              |                 +---| greetings |
              |   +----------+  |   +-----------+
              +---| japanese |--+
                  +----------+  |   +-----------+
                                +---| farewells |
                                    +-----------+
```

在这个例子中，`phrases`是我们包装箱的名字。剩下所有的都是模块。你可以看到它们组成了一个树，以包装箱为**根**（即`phrases`树的根）分叉出来。

现在我们想要在代码中定义这些模块。首先，用 Cargo 创建一个新包装箱：

```bash
$ cargo new phrases
$ cd phrases
```

如果你还记得，这会生成一个简单的项目：

```bash
$ tree .
.
├── Cargo.toml
└── src
    └── lib.rs

1 directory, 2 files
```

`src/lib.rs`是我们包装箱的根，与上面图表中的`phrases`对应。

# 定义模块

我们用`mod`关键字来定义我们的每一个模块。让我们把`src/lib.rs`写成这样：

```rust
mod english {
    mod greetings {
    }

    mod farewells {
    }
}

mod japanese {
    mod greetings {
    }

    mod farewells {
    }
}
```

在`mod`关键字之后是模块的名字。模块的命名采用Rust其它标识符的命名惯例：`lower_snake_case`。在大括号中（`{}`）是模块的内容。

在`mod`中，你可以定义子`mod`。我们可以用双冒号（`::`）标记访问子模块。我们的4个嵌套模块是`english::greetings`，`english::farewells`，`japanese::greetings`和`japanese::farewells`。因为子模块位于父模块的命名空间中，所以这些不会冲突：`english::greetings`和`japanese::greetings`是不同的，即便它们的名字都是`greetings`。

因为这个包装箱的根文件叫做`lib.rs`，且没有一个`main()`函数。Cargo会把这个包装箱构建为一个库：

```bash
$ cargo build
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
$ ls target/debug
build  deps  examples  libphrases-a7448e02a0468eaa.rlib  native
```

`libphrases-<hash>.rlib`是构建好的包装箱。在我们了解如何使用这个包装箱之前，先让我们把它拆分为多个文件。

# 多文件包装箱
如果每个包装箱只能有一个文件，这些文件将会变得非常庞大。把包装箱分散到多个文件也非常简单，Rust支持两种方法。

除了这样定义一个模块外：

```rust
mod english {
    // Contents of our module go here.
}
```

我们还可以这样定义：

```rust
mod english;
```

如果我们这么做的话，Rust会期望能找到一个包含我们模块内容的`english.rs`文件，或者包含我们模块内容的`english/mod.rs`文件：

注意在这些文件中，你不需要重新定义这些模块：它们已经由最开始的`mod`定义。

使用这两个技巧，我们可以将我们的包装箱拆分为两个目录和七个文件：

```bash
$ tree .
.
├── Cargo.lock
├── Cargo.toml
├── src
│   ├── english
│   │   ├── farewells.rs
│   │   ├── greetings.rs
│   │   └── mod.rs
│   ├── japanese
│   │   ├── farewells.rs
│   │   ├── greetings.rs
│   │   └── mod.rs
│   └── lib.rs
└── target
    └── debug
        ├── build
        ├── deps
        ├── examples
        ├── libphrases-a7448e02a0468eaa.rlib
        └── native
```

`src/lib.rs`是我们包装箱的根，它看起来像这样：

```rust
mod english;
mod japanese;
```

这两个定义告诉Rust去寻找`src/english.rs`和`src/japanese.rs`，或者`src/english/mod.rs`和`src/japanese/mod.rs`，具体根据你的偏好。在我们的例子中，因为我们的模块含有子模块，所以我们选择第二种方式。`src/english/mod.rs`和`src/japanese/mod.rs`都看起来像这样：

```rust
mod greetings;
mod farewells;
```

再一次，这些定义告诉Rust去寻找`src/english/greetings.rs`和`src/japanese/greetings.rs`，或者`src/english/farewells/mod.rs`和`src/japanese/farewells/mod.rs`。因为这些子模块没有自己的子模块，我们选择`src/english/greetings.rs`和`src/japanese/farewells.rs`。

现在`src/english/greetings.rs`和`src/japanese/farewells.rs`都是空的。让我们添加一些函数。

在`src/english/greetings.rs`添加如下：

```rust
fn hello() -> String {
    "Hello!".to_string()
}
```

在`src/english/farewells.rs`添加如下：

```rust
fn goodbye() -> String {
    "Goodbye.".to_string()
}
```

在`src/japanese/greetings.rs`添加如下：

```rust
fn hello() -> String {
    "こんにちは".to_string()
}
```

当然，你可以从本文复制粘贴这些内容，或者写点别的东西。事实上你写进去“konnichiwa”对我们学习模块系统并不重要。

在`src/japanese/farewells.rs`添加如下：

```rust
fn goodbye() -> String {
    "さようなら".to_string()
}
```

（这是“Sayōnara”，如果你很好奇的话。）

现在我们在包装箱中添加了一些函数，让我们尝试在别的包装箱中使用它。

# 导入外部的包装箱
我们有了一个库包装箱。让我们创建一个可执行的包装箱来导入和使用我们的库。

创建一个`src/main.rs`文件然后写入如下：（现在它还不能编译）

```rust
extern crate phrases;

fn main() {
    println!("Hello in English: {}", phrases::english::greetings::hello());
    println!("Goodbye in English: {}", phrases::english::farewells::goodbye());

    println!("Hello in Japanese: {}", phrases::japanese::greetings::hello());
    println!("Goodbye in Japanese: {}", phrases::japanese::farewells::goodbye());
}
```

`extern crate`声明告诉Rust我们需要编译和链接`phrases`包装箱。然后我们就可以在这里使用`phrases`的模块了。就想我们之前提到的，你可以用双冒号引用子模块和之中的函数。

（注意：当导入像“like-this”名字中包含连字符的 crate时，这样的名字并不是一个有效的 Rust 标识符，它可以通过将连字符变为下划线来转换，所以你应该写成`extern crate like_this;`）

另外，Cargo假设`src/main.rs`是二进制包装箱的根，而不是库包装箱的。现在我们的包中有两个包装箱：`src/lib.rs`和`src/main.rs`。这种模式在可执行包装箱中非常常见：大部分功能都在库包装箱中，而可执行包装箱使用这个库。这样，其它程序可以只使用我们的库，另外这也是各司其职的良好分离。

现在它还不能很好的工作。我们会得到4个错误，它们看起来像：

```bash
$ cargo build
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
src/main.rs:4:38: 4:72 error: function `hello` is private
src/main.rs:4     println!("Hello in English: {}", phrases::english::greetings::hello());
                                                   ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
note: in expansion of format_args!
<std macros>:2:25: 2:58 note: expansion site
<std macros>:1:1: 2:62 note: in expansion of print!
<std macros>:3:1: 3:54 note: expansion site
<std macros>:1:1: 3:58 note: in expansion of println!
phrases/src/main.rs:4:5: 4:76 note: expansion site
```

Rust 默认一切都是私有的。让我们深入了解一下这个。

# 导出公用接口
Rust允许你严格的控制你的接口哪部分是公有的，所以它们默认都是私有的。你需要使用`pub`关键字，来公开它。让我们先关注`english`模块，所以让我们像这样减少`src/main.rs`的内容：

```rust
extern crate phrases;

fn main() {
    println!("Hello in English: {}", phrases::english::greetings::hello());
    println!("Goodbye in English: {}", phrases::english::farewells::goodbye());
}
```

在我们的`src/lib.rs`，让我们给`english`模块声明添加一个`pub`：

```rust
pub mod english;
mod japanese;
```

然后在我们的`src/english/mod.rs`中，加上两个`pub`：

```rust
pub mod greetings;
pub mod farewells;
```

在我们的`src/english/greetings.rs`中，让我们在`fn`声明中加上`pub`：

```rust
pub fn hello() -> String {
    "Hello!".to_string()
}
```

然后在`src/english/farewells.rs`中：

```rust
pub fn goodbye() -> String {
    "Goodbye.".to_string()
}
```

这样，我们的包装箱就可以编译了，虽然会有警告说我们没有使用`japanese`的方法：

```bash
$ cargo run
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
src/japanese/greetings.rs:1:1: 3:2 warning: function is never used: `hello`, #[warn(dead_code)] on by default
src/japanese/greetings.rs:1 fn hello() -> String {
src/japanese/greetings.rs:2     "こんにちは".to_string()
src/japanese/greetings.rs:3 }
src/japanese/farewells.rs:1:1: 3:2 warning: function is never used: `goodbye`, #[warn(dead_code)] on by default
src/japanese/farewells.rs:1 fn goodbye() -> String {
src/japanese/farewells.rs:2     "さようなら".to_string()
src/japanese/farewells.rs:3 }
     Running `target/debug/phrases`
Hello in English: Hello!
Goodbye in English: Goodbye.
```

现在我们的函数是公有的了，我们可以使用它们。好的！然而，`phrases::english::greetings::hello()`非常长并且重复。Rust有另一个关键字用来导入名字到当前空间中，这样我们就可以用更短的名字来引用它们。让我们聊聊`use`。

# 用`use`导入模块
Rust有一个`use`关键字，它允许我们导入名字到我们本地的作用域中。让我们把`src/main.rs`改成这样：

```rust
extern crate phrases;

use phrases::english::greetings;
use phrases::english::farewells;

fn main() {
    println!("Hello in English: {}", greetings::hello());
    println!("Goodbye in English: {}", farewells::goodbye());
}
```

这两行`use`导入了两个模块到我们本地作用域中，这样我们就可以用一个短得多的名字来引用函数。作为一个传统，当导入函数时，导入模块而不是直接导入函数被认为是一个最佳实践。也就是说，你可以这么做：

```rust
extern crate phrases;

use phrases::english::greetings::hello;
use phrases::english::farewells::goodbye;

fn main() {
    println!("Hello in English: {}", hello());
    println!("Goodbye in English: {}", goodbye());
}
```

不过这并不理想。这意味着更加容易导致命名冲突。在我们的小程序中，这没什么大不了的，不过随着我们的程序增长，它将会成为一个问题。如果我们有命名冲突，Rust会给我们一个编译错误。举例来说，如果我们将`japanese`的函数设为公有，然后这样尝试：

```rust
extern crate phrases;

use phrases::english::greetings::hello;
use phrases::japanese::greetings::hello;

fn main() {
    println!("Hello in English: {}", hello());
    println!("Hello in Japanese: {}", hello());
}
```

Rust会给我们一个编译时错误：

```bash
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
src/main.rs:4:5: 4:40 error: a value named `hello` has already been imported in this module [E0252]
src/main.rs:4 use phrases::japanese::greetings::hello;
                  ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
error: aborting due to previous error
Could not compile `phrases`.
```

如果你从同样的模块中导入多个名字，我们不必写多遍。Rust有一个简便的语法：

```rust
use phrases::english::greetings;
use phrases::english::farewells;
```

我们可以使用这个简写：

```rust
use phrases::english::{greetings, farewells};
```

## 使用`pub use`重导出
你不仅可以用`use`来简化标识符。你也可以在包装箱内用它重导出函数到另一个模块中。这意味着你可以展示一个外部接口可能并不直接映射到内部代码结构。

让我们看个例子。修改`src/main.rs`让它看起来像这样：

```rust
extern crate phrases;

use phrases::english::{greetings,farewells};
use phrases::japanese;

fn main() {
    println!("Hello in English: {}", greetings::hello());
    println!("Goodbye in English: {}", farewells::goodbye());

    println!("Hello in Japanese: {}", japanese::hello());
    println!("Goodbye in Japanese: {}", japanese::goodbye());
}
```

然后修改`src/lib.rs`公开`japanese`模块：

```rust
pub mod english;
pub mod japanese;
```

接下来，把这两个函数声明为公有，先是`src/japanese/greetings.rs`：

```rust
pub fn hello() -> String {
    "こんにちは".to_string()
}
```

然后是`src/japanese/farewells.rs`：

```rust
pub fn goodbye() -> String {
    "さようなら".to_string()
}
```

最后，修改你的`src/japanese/mod.rs`为这样：

```rust
pub use self::greetings::hello;
pub use self::farewells::goodbye;

mod greetings;
mod farewells;
```

`pub use`声明将这些函数导入到了我们模块结构空间中。因为我们在`japanese`模块内使用了`pub use`，我们现在有了`phrases::japanese::hello()`和`phrases::japanese::goodbye()`函数，即使它们的代码在`phrases::japanese::greetings::hello()`和`phrases::japanese::farewells::goodbye()`函数中。内部结构并不反映外部接口。

这里我们对每个我们想导入到`japanese`空间的函数使用了`pub use`。我们也可以使用通配符来导入`greetings`的一切到当前空间中：`pub use self::greetings::*`。

那么`self`怎么办呢？好吧，默认，`use`声明是绝对路径，从你的包装箱根目录开始。`self`则使路径相对于你在结构中的当前位置。有一个更特殊的`use`形式：你可以使用`use super::`来到达你树中当前位置的上一级。一些同学喜欢把`self`看作`.`而把`super`看作`..`，它们在许多shell表示为当前目录和父目录。

除了`use之`外，路径是相对的：`foo::bar()`引用一个相对我们位置的`foo`中的函数。如果它带有`::`前缀，它引用了一个不同的`foo`，一个从你包装箱根开始的绝对路径。

另外，注意`pub use`出现在`mod`定义之前。Rust要求`use`位于最开始。

构建然后运行：

```bash
$ cargo run
   Compiling phrases v0.0.1 (file:///home/you/projects/phrases)
     Running `target/debug/phrases`
Hello in English: Hello!
Goodbye in English: Goodbye.
Hello in Japanese: こんにちは
Goodbye in Japanese: さようなら
```

## 复杂的导入

Rust 提供了多种高级选项来让你的`extern crate`和`use`语句变得简洁方便。这是一个例子：

```rust
extern crate phrases as sayings;

use sayings::japanese::greetings as ja_greetings;
use sayings::japanese::farewells::*;
use sayings::english::{self, greetings as en_greetings, farewells as en_farewells};

fn main() {
    println!("Hello in English; {}", en_greetings::hello());
    println!("And in Japanese: {}", ja_greetings::hello());
    println!("Goodbye in English: {}", english::farewells::goodbye());
    println!("Again: {}", en_farewells::goodbye());
    println!("And in Japanese: {}", goodbye());
}
```

这里发生了什么？

首先，`extern crate`和`use`都允许重命名导入的项。所以 crate 仍然叫“phrases”，不过这里我们以“sayings”来引用它。类似的，第一个`use`语句从 crate 中导入`japanese::greetings`，不过作为`ja_greetings`而不是简单的`greetings`。这可以帮助我们消除来自不同包中相似名字的项的歧义。

第二个`use`语句用了一个星号来引入`sayings::japanese::farewells`模块中的所有公有符号。如你所见之后我们可以不用模块标识来引用日语的`goodbye`函数。这类全局引用要保守使用。需要注意的是它只引入公有符号，哪怕在相同模块的代码中引入。


第三个`use`语句需要更多的解释。它使用了“大括号扩展（brace expansion）”来将三条`use`语句压缩成了一条（这类语法对曾经写过 Linux shell 脚本的人应该很熟悉）。语句的非压缩形式应该是：

```rust
use sayings::english;
use sayings::english::greetings as en_greetings;
use sayings::english::farewells as en_farewells;
```

如你所见，大括号压缩了位于同一位置的多个项的`use`语句，而且在这里`self`指向这个位置。注意：大括号不能与星号嵌套或混合。
