# 准备

> [getting-started.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/getting-started.md)
> <br>
> commit 52786de3ed2ff4553378c9b9bca1a60d8b2a2e9d

本书的第一部分将带领大家了解 Rust 及其工具。在安装 Rust 之后，我们将开始编写经典的“Hello World”程序。最后将介绍 Cargo，Rust 的构建系统以及包管理器。

## 安装 Rust

开始使用 Rust 的第一步是安装它。总的来说，你需要联网执行本部分的命令，因为我们将会从网上下载 Rust。

我们将会展示很多使用终端的命令，并且这些行都以`$`开头。并不需要真正输入`$`，在这里它们代表每行指令的开头。你会在网上看到很多使用这个惯例的教程和例子：`$`代表常规用户运行的命令，`#`代表需要管理员用户运行的命令。

## 平台支持

Rust 编译器编译并运行于很多平台之上，但不是所有的平台都被平等的支持。Rust 的平台支持水平可以被划分为三个等级，每一级都有不同的保证程度。

每个平台都由他们的 "target triple" 标识，它是一个代表编译器会产生何种输出的字符串。下面的列代表特定平台是否支持相应的组件。

### T1 科技（Tier 1）

等级一平台可以被认为是“确保可以构建和工作的”。具体的他们将满足如下要求：

* 为此平台建立了自动化测试
* 向`rust-lang/rust`仓库的 master 分支提交的修改确保测试通过
* 发布官方安装程序
* 提供该平台下如何使用和构建的文档。

|  Target                       | std |rustc|cargo| notes                      |
|-------------------------------|-----|-----|-----|----------------------------|
| `x86_64-pc-windows-msvc`      |  ✓  |  ✓  |  ✓  | 64-bit MSVC (Windows 7+)   |
| `i686-pc-windows-gnu`         |  ✓  |  ✓  |  ✓  | 32-bit MinGW (Windows 7+)  |
| `x86_64-pc-windows-gnu`       |  ✓  |  ✓  |  ✓  | 64-bit MinGW (Windows 7+)  |
| `i686-apple-darwin`           |  ✓  |  ✓  |  ✓  | 32-bit OSX (10.7+, Lion+)  |
| `x86_64-apple-darwin`         |  ✓  |  ✓  |  ✓  | 64-bit OSX (10.7+, Lion+)  |
| `i686-unknown-linux-gnu`      |  ✓  |  ✓  |  ✓  | 32-bit Linux (2.6.18+)     |
| `x86_64-unknown-linux-gnu`    |  ✓  |  ✓  |  ✓  | 64-bit Linux (2.6.18+)     |

### T2 科技（Tier 2）

等级二平台可以被认为是“保证能够构建的”。因为没有（保证）运行自动测试所以并不保证能产生可工作的构建，不过这些平台通常工作良好同时补丁是永远受欢迎的！具体的这些平台被要求将满足如下：

* 设置了自动化测试，不过可能并没有运行
* 向`rust-lang/rust`仓库的 master 分支提交的修改确保该平台**将被构建**。注意这意味着一些平台只编译了标准库，而有些将会运行整个 bootstrap。
* 发布官方安装程序

|  Target                       | std |rustc|cargo| notes                      |
|-------------------------------|-----|-----|-----|----------------------------|
| `i686-pc-windows-msvc`        |  ✓  |  ✓  |  ✓  | 32-bit MSVC (Windows 7+)   |
| `x86_64-unknown-linux-musl`   |  ✓  |     |     | 64-bit Linux with MUSL     |
| `arm-linux-androideabi`       |  ✓  |     |     | ARM Android                |
| `arm-unknown-linux-gnueabi`   |  ✓  |  ✓  |     | ARM Linux (2.6.18+)        |
| `arm-unknown-linux-gnueabihf` |  ✓  |  ✓  |     | ARM Linux (2.6.18+)        |
| `aarch64-unknown-linux-gnu`   |  ✓  |     |     | ARM64 Linux (2.6.18+)      |
| `mips-unknown-linux-gnu`      |  ✓  |     |     | MIPS Linux (2.6.18+)       |
| `mipsel-unknown-linux-gnu`    |  ✓  |     |     | MIPS (LE) Linux (2.6.18+)  |

### T3 科技（Tier 3）（Tengu！！！）

等级三平台代表 Rust 有提供支持，不过提交的修改并不保证能构建或通过测试。可运行的构建也可能是有 bug 的，因为它的可靠性通常由社区贡献来确定。另外并不提供官方发布文档和安装程序，不过在一些非官方地址可能会提供社区版本。

|  Target                       | std |rustc|cargo| notes                      |
|-------------------------------|-----|-----|-----|----------------------------|
| `i686-linux-android`          |  ✓  |     |     | 32-bit x86 Android         |
| `aarch64-linux-android`       |  ✓  |     |     | ARM64 Android              |
| `powerpc-unknown-linux-gnu`   |  ✓  |     |     | PowerPC Linux (2.6.18+)    |
| `i386-apple-ios`              |  ✓  |     |     | 32-bit x86 iOS             |
| `x86_64-apple-ios`            |  ✓  |     |     | 64-bit x86 iOS             |
| `armv7-apple-ios`             |  ✓  |     |     | ARM iOS                    |
| `armv7s-apple-ios`            |  ✓  |     |     | ARM iOS                    |
| `aarch64-apple-ios`           |  ✓  |     |     | ARM64 iOS                  |
| `i686-unknown-freebsd`        |  ✓  |  ✓  |     | 32-bit FreeBSD             |
| `x86_64-unknown-freebsd`      |  ✓  |  ✓  |     | 64-bit FreeBSD             |
| `x86_64-unknown-openbsd`      |  ✓  |  ✓  |     | 64-bit OpenBSD             |
| `x86_64-unknown-netbsd`       |  ✓  |  ✓  |     | 64-bit NetBSD              |
| `x86_64-unknown-bitrig`       |  ✓  |  ✓  |     | 64-bit Bitrig              |
| `x86_64-unknown-dragonfly`    |  ✓  |  ✓  |     | 64-bit DragonFlyBSD        |
| `x86_64-rumprun-netbsd`       |  ✓  |     |     | 64-bit NetBSD Rump Kernel  |
| `i686-pc-windows-msvc` (XP)   |  ✓  |     |     | Windows XP support         |
| `x86_64-pc-windows-msvc` (XP) |  ✓  |     |     | Windows XP support         |

注意这个表格可能会随着时间而扩展，这将永远不会是等级三平台的完整列表！

## 在 Linux 和 Mac 上安装

如果使用 Linux 或 Mac，所有我们需要做的就是打开一个终端并输入如下：

```bash
$ curl -sSf https://static.rust-lang.org/rustup.sh | sh
```

这将会下载一个脚本，并开始安装。如果一切顺利，你将会看到这些：

```bash
Welcome to Rust.

This script will download the Rust compiler and its package manager, Cargo, and
install them to /usr/local. You may install elsewhere by running this script
with the --prefix=<path> option.

The installer will run under ‘sudo’ and may ask you for your password. If you do
not want the script to run ‘sudo’ then pass it the --disable-sudo flag.

You may uninstall later by running /usr/local/lib/rustlib/uninstall.sh,
or by running this script again with the --uninstall flag.

Continue? (y/N)
```

在这里输入，输入`y`来选择`yes`，并按照接下来的提示操作。

## 在 Windows 上安装

如果你使用 Windows，请下载合适的[安装包](https://www.rust-lang.org/install.html)

## 卸载

卸载 Rust 跟安装它一样容易。在 Linux 或 Mac 上，运行卸载脚本：

```bash
$ sudo /usr/local/lib/rustlib/uninstall.sh
```

如果你使用的是 Windows 安装包，再次运行`.msi`文件，它会给我们一个卸载选项。

## 疑难解答（Troubleshooting）

安装了 Rust 后，我们可以打开一个 shell，并输入：

```bash
$ rustc --version
```

你应该看到版本号，提交的 hash 值和提交时间。

如果你做到了，那么 Rust 已成功安装！恭喜你！（此处应有掌声）

如果这不能工作并且你在使用 Windows，检查 Rust 是否在你的`%PATH%`系统变量中。如果不是，再次运行安装程序，在“Change, repair, or remove installation”页面选择“Change”并确保“Add to PATH”指向本地硬盘。

如果还是搞不定，有几个你可以获取帮助的地方。最简单的是通过[Mibbit](http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust)访问位于 irc.mozilla.org 的 #rust IRC频道 。点击上面的链接，你就可以与其它Rustaceans（简单理解为Ruster吧）聊天，我们会帮助你。其它给力的资源包括[用户论坛](https://users.rust-lang.org/)和[Stack Overflow](http://stackoverflow.com/questions/tagged/rust)。

安装程序（脚本）也会在本地安装一份文档拷贝，所以你可以离线阅读它们。在 UNIX 系统上，位置是`/usr/local/share/doc/rust`。在Windows，它位于你 Rust 安装位置的`share/doc`文件夹。

## Hello, world!

现在你已经安装好了 Rust，我们将帮助你编写你的第一个 Rust 程序。当学习一门新语言的时候编写一个在屏幕上打印 “Hello, world!” 文本的小程序是一个传统，而在这一部分，我们将遵循这个传统。

以这样一个简单的程序开始的好处是你可以快速的确认你的编译器已正确安装，并可以正常工作。在屏幕上打印信息也是一个非常常见的操作，所以早点实践一下是有好处的。

> 注意：本书假设你熟悉基本的命令行操作。Rust 本身并不对你的编辑器，工具和你的代码存放在何处有什么特定的要求，所以如果你比起命令行更喜欢 IDE，这里也有一个选择。你可能想要试试[SolidOak](https://github.com/oakes/SolidOak)，它转为 Rust 而设计。在 Rust 社区里有许许多多正在开发中的 IDE 插件。Rust 团队也发布了[不同编辑器的插件](https://github.com/rust-lang/rust/blob/master/src/etc/CONFIGS.md)。 配置编辑器或 IDE 已超出本教程的范畴，所以请查看你特定设置的文档。

### 创建一个项目文件

首先，创建一个文件来编写 Rust 代码。Rust 并不关心你的代码存放在哪里，不过在本书中，我们建议在你的 home 目录创建一个存放项目的目录，并把你的所有项目放在这。打开一个终端并输入如下命令来为这个项目创建一个文件夹：

```bash
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

> 如果你使用 Windows 并且没有用 PowerShell，`~`可能不指向你的用户目录。可以查询所使用的 Shell 的相关文档以获取更多信息。

### 编写并运行一个 Rust 程序

接下来，创建一个叫做`main.rs`的源文件。Rust 代码文件总是使用 `.rs` 后缀，并且如果我们用的 Rust 文件名由多个单词组成，我们使用下划线分隔它们；例如，使用`hello_world.rs`而不是`helloworld.rs`

现在打开你刚创建的`main.rs`源文件，键入如下代码：

+[code](https://play.rust-lang.org/?code=fn%20main()%20%7B%0A%20%20%20%20println!(%22Hello%2C%20world!%22)%3B%0A%7D%0A)

```rust
fn main() {
    println!("Hello, world!");
}
```

保存文件，并回到你的命令行窗口。在 Linux 或 OSX 上，输入如下命令：

```bash
$ rustc main.rs
$ ./main
Hello, world!
```

在 Windows 下，使用`main.exe`而不是`main`。不管你使用何种系统，你应该在终端看到`Hello, world!`字符串。如果你做到了，那么恭喜你！你已经正式编写了一个 Rust 程序。这使你成为了一名 Rust 程序猿！欢迎入坑！

### 分析一个 Rust 程序

现在，让我们回过头来仔细看看你的“Hello, world!”程序到底发生了什么。这里是拼图的第一片：

+[code](https://play.rust-lang.org/?code=fn%20main()%20%7B%0A%0A%7D%0A)

```rust
fn main() {

}
```

这几行定义了一个 Rust *函数*。`main` 函数是特殊的：这是所有 Rust 程序的开始。第一行表示“定义一个叫 `main` 的函数，没有参数也没有返回值。”如果有参数的话，它们应该出现在括号（`(`和`)`）中。因为并没有返回值，所以我们可以省略整个返回值类型。

同时注意函数体被包裹在大括号（`{`和`}`）中。Rust 要求所有函数体都位于其中。将前一个大括号与函数声明置于一行，并留有一个空格被认为是一个好的代码风格。

在`main()`函数中：

+[code](https://play.rust-lang.org/?code=fn%20main()%20%7B%0A%20%20%20%20%20%20%20%20println!(%22Hello%2C%20world!%22)%3B%0A%20%20%20%20%0A%7D)

```rust
    println!("Hello, world!");
```

这行代码做了这个小程序的所有工作：它在屏幕上打印文本。这有很多重要的细节。第一个是使用 4 个空格缩进，而不是制表符。

第二个重要的部分是`println!()`这一行。这是一个 Rust [宏](Macros 宏.md)，是 Rust 元编程的关键所在。相反如果我们调用一个函数的话，它应该看起来像这样：`println()`（木有 !）。我们将在后面更加详细的讨论 Rust 宏，不过现在你只需记住当看到符号 `!` 的时候，就代表调用了一个宏而不是一个普通的函数。

接下来，`"Hello, world!"` 是一个字符串。在一门系统级编程语言中，字符串是一个复杂得令人惊讶的话题。这是一个[静态分配](The Stack and the Heap 栈和堆.md)的字符串。这个语句将这个字符串作为参数传递给`println!` 宏，这个宏负责在屏幕（控制台）上打印字符串。是不是很简单啊(⊙o⊙)

这一行以一个分号结尾（`;`）。Rust是一门[**面向表达式**](Glossary 词汇表.md#面向表达式语言（expression-oriented-language）)的语言，也就是说大部分语句都是表达式。`;` 表示一个表达式的结束，另一个新表达式的开始。大部分 Rust 代码行以`;`结尾。

### 编译和运行是两个步骤

在“编写并运行一个 Rust 程序”，我们展示了如何运行一个新创建的程序。现在我们将拆分并检查每一个操作。

在运行一个 Rust 程序之前，你必须编译它。你可以输入`rustc`命令来使用 Rust 编译器并像这样传递你源文件的名字：

```bash
$ rustc main.rs
```

如果你来自 C 或 C++ 背景，你会发现这与`gcc`和`clang`类似。编译成功后，Rust 应该会输出一个二进制可执行文件，在 Linux 或 OSX 下在shell 中通过如下`ls`命令你可以看到它：

```bash
$ ls
main  main.rs
```

在 Windows 下，输入：

```bash
$ dir
main.exe  main.rs
```

这表示我们有两个文件：`.rs`后缀的源文件，和可执行文件（在 Windows下是`main.exe`，其它平台是`main`）。这里我们剩下的操作就只有运行`main`或`main.exe`文件了，像这样：

```bash
$ ./main  # or main.exe on Windows
```

如果`main.rs`是我们的“Hello, world!”程序，它将会在你的终端上打印`Hello, world!`。

来自 Ruby、Python 或 JavaScript 这样的动态类型语言背景的同学，可能不太习惯这样将编译和执行分开。Rust 是一种 **预编译语言**（*ahead-of-time compiled language*），程序编译好后，把它给任何人，他们都不需要安装 Rust 就可运行。如果你给他们一个 `.rb` ， `.py` 或 `.js` 文件，他们需要先分别安装 Ruby，Python，JavaScript 实现，不过你只需要一句命令就可以编译和执行你的程序。这一切都是语言设计的权衡取舍。

仅仅使用`rustc`编译简单程序是没问题的，不过随着你的项目的增长，你将想要能够控制你项目拥有的所有选项，并易于分享你的代码给别人或别的项目。接下来，我们将介绍一个叫做 Cargo 的工具，它将帮助你编写现实生活中的 Rust 程序。

## Hello, Cargo!

Cargo 是 Rust 的构建系统和包管理工具，同时 Rustacean 们使用 Cargo 来管理它们的 Rust 项目。Cargo 负责三个工作：构建你的代码，下载你代码依赖的库并编译这些库。我们把你代码需要的库叫做“依赖（dependencies）”因为你的代码依赖他们。

最简单的 Rust 程序并没有任何依赖，所以目前我们只使用它的第一部分功能。随着你编写更加复杂的 Rust 程序，你会想要添加依赖，那么如果你使用 Cargo 开始的话，这将会变得简单许多。

因为绝大部分 Rust 项目使用 Cargo，本书接下来的部分将假设你使用它。如果你使用官方安装包的话，Rust 自带 Cargo。如果你使用其他方式安装 Rust 的话，你可以在终端输入如下命令检查你是否安装了 Cargo：

```bash
$ cargo --version
```

如果你看到了版本号，一切 OK！如果你一个类似“`command not found`”的错误，那么你应该去查看你安装 Rust 的系统的相关文档，来确定 Cargo 是否需要单独安装。

### 转换到 Cargo

让我们将 Hello World 程序迁移至 Cargo。为了 Cargo 化一个项目，需要做三件事：

1. 将源文件放到正确的目录
2. 删除旧的可执行文件（Windows下是`main.exe`，其他平台是`main`）并生成一个新的。
3. 创建一个 Cargo 配置文件

让我们开始吧！

### 创建一个新的可执行文件和源文件目录

首先，回到你的终端，移动到你的`hello_world`目录，并输入如下命令：

```bash
$ mkdir src
$ mv main.rs src/main.rs
$ rm main  # or 'del main.exe' on Windows
```

Cargo 期望源文件位于 src 目录，所以先做这个。这样将项目顶级目录（在这里，是 hello_world）留给 README，license 信息和其他跟代码无关的文件。这样，Cargo 帮助你保持项目干净整洁。一切井井有条。

现在，复制`main.rs`到`src`目录，并删除你用`rustc`创建的编译过的文件。一如既往，如果你使用 Windows 用`main.exe`代替`main`。

例子中我们继续使用`main.rs`作为源文件名是因为它创建了一个可执行文件。如果你想要创建一个库文件，使用`lib.rs`作为文件名。Cargo 使用这个约定来正确编译你的项目，不过如果你想的话你也可以覆盖它。

### 创建一个配置文件

下一步，在`hello_world`目录创建一个文件，叫做`Cargo.toml`。

确保`Cargo.toml`的`C`是大写的，否则 Cargo 不知道如何处理配置文件。

这个文件使用[TOML](https://github.com/toml-lang/toml)（Tom's Obvious, Minimal Language）格式。 TOML 类似于 INI，不过有一些额外的改进之处，并且被用作 Cargo 的配置文件。

在这个文件中，输入如下信息：

```toml
[package]

name = "hello_world"
version = "0.0.1"
authors = [ "Your name <you@example.com>" ]
```

第一行，`[package]`，表明下面的语句用来配置一个包。随着我们在这个文件增加更多的信息，我们会增加其他部分，不过现在，我们只有包配置。

另外三行设置了 Cargo 编译你的程序所需要知道的三个配置：包的名字，版本，和作者。

当你在`Cargo.toml`中添加完这些信息后，保存它来完成配置文件的创建。

### 构建并运行一个 Cargo 项目

当`Cargo.toml`文件位于项目的根目录时，我们就准备好可以构建并运行 Hello World 程序了！为此，我们输入如下命令：

```bash
$ cargo build
   Compiling hello_world v0.0.1 (file:///home/yourname/projects/hello_world)
$ ./target/debug/hello_world
Hello, world!
```

如果一切顺利，你应该再次看到`Hello, world!`出现在终端里。

你刚刚用`cargo build`构建了一个程序并用`./target/debug/hello_world`运行了它，不过你也可以用如下的一步操作`cargo run`来完成这两步操作：

```bash
$ cargo run
     Running `target/debug/hello_world`
Hello, world!
```

注意这个例子并没有重新构建项目。Cargo 发现文件并没有被修改，所以它只是运行了二进制文件。如果你修改了源文件，Cargo 会在运行前重新构建项目，这样你将看到像这样的输出：

```bash
$ cargo run
   Compiling hello_world v0.0.1 (file:///home/yourname/projects/hello_world)
     Running `target/debug/hello_world`
Hello, world!
```

Cargo 检查任何项目文件是否被修改，并且只会在你上次构建后修改了他们才重新构建。

对于简单的项目，Cargo 并不比使用`rustc`要好多少，不过将来它会变得有用。这在你开始使用 crate 时显得尤为正确；（crate）在其他语言中有“库（library）”或“包（package）”这样的同义词。对于包含多个 crate 的项目，让 Cargo 来协调构建将会轻松很多。有了 Cargo，你可以运行`cargo build`，而一切将有条不紊的运行。

### 发布构建（Building for Release）

当你的项目最终准备好发布了，可以使用`cargo build --release`来优化编译项目。这些优化可以让 Rust 代码运行的更快，不过启用他们会让程序花更长的时间编译。这也是为何这是两种不同的配置，一个为了开发，另一个构建提供给用户的最终程序。

运行这个命令同时也会让 Cargo 创建一个叫做`Cargo.lock`的文件，它看起来像这样：

```toml
[root]
name = "hello_world"
version = "0.0.1"
```

Cargo 用`Cargo.lock`文件跟踪你程序的依赖。这里是 Hello World 项目的`Cargo.lock`文件。这个项目并没有依赖，所以内容有一点稀少。事实上，你自己甚至都不需要碰这个文件；仅仅让 Cargo 处理它就行了。

就是这样！如果你一路跟过来了，你应该已经成功使用 Cargo 构建了`hello_world`。

虽然这个项目很简单，现在它使用了很多在你余下的 Rust 生涯中将会用到的实际的工具。事实上，你可以期望使用如下命令的变体开始所有的 Rust 项目：

```bash
$ git clone someurl.com/foo
$ cd foo
$ cargo build
```

### 创建一个新 Cargo 项目的简单方法

你并不需要每次都过一遍上面的操作来开始一个新的项目！Cargo 可以快速创建一个骨架项目目录这样你就可以立即开始开发了。

用 Cargo 来开始一个新项目，在命令行输入`cargo new`：

```bash
$ cargo new hello_world --bin
```

这个命令传递了`--bin`参数因为我们的目标是直接创建一个可执行程序，而不是一个库。可执行文件通常叫做二进制文件（因为它们位于`/usr/bin`，如果你使用 Unix 系统的话）。

Cargo 为我们创建了两个文件和一个目录：一个`Cargo.toml`和一个包含了`main.rs`文件的`src`目录。这应该看起来很眼熟，他们正好是我们在之前手动创建的那样。

这些输出是你开始所需要的一切。首先，打开`Cargo.toml`。它应该看起来像这样：

```toml
[package]

name = "hello_world"
version = "0.1.0"
authors = ["Your Name <you@example.com>"]
```

Cargo 已经根据你给出的参数和`git`全局配置给出了合理的默认配置。你可能会注意到 Cargo 也把`hello_world`目录初始化为了一个`git`仓库。

这是应该写入`src/main.rs`的代码：

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo 已经为你生成了一个“Hello World！”，现在你已经准备好开始撸代码了！

> 注意：如果你想要查看 Cargo 的详细信息，请查看官方的[Cargo 指导](http://doc.crates.io/guide.html)，它包含了所有这些功能。

## 写在最后（Closing Thoughts）
这个章节覆盖了将用于本书余下部分以及你之后 Rust 时光的基础知识。现在你获得了工具，我们将更多的介绍 Rust 语言本身。

（接下来）你有两个选择：在 “[学习 Rust](Learn Rust 学习 Rust.md)” 中深入研究一个项目，或者自下而上地学习 “[语法和语义](Syntax and Semantics 语法和语义.md)”。来自系统级编程语言的同学，你们可能倾向于选择 “学习 Rust”，而来自动态编程语言的同学，请根据自己的喜好来选择吧。人各有别，适合自己的才是最好的。
