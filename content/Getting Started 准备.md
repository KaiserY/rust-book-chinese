# 准备

> [getting-started.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/getting-started.md)
> commit 0b8370c3978bb47de97ce754ea601fc1b654cd2b

本书的第一部分将带领大家了解 Rust 及其工具。在安装 Rust 之后，我们将开始编写经典的“Hello World”程序。最后我们还会介绍Cargo，Rust 的构建（编译）系统以及包管理器。

## 安装 Rust

开始使用 Rust 的第一步是安装它。总的来说，你需要联网执行本部分的命令，因为我们将会从网上下载 Rust。

我们将会展示很多使用终端的命令，并且这些行都以`$`开头。我们并不需要输入`$`，在这里它们代表每行指令的开头。你会在网上看到很多使用这个惯例的教程和例子：`$`代表常规用户运行的命令，`#`代表需要管理员用户运行的命令。

## 平台支持

Rust 编译器编译并运行于很多平台之上，但不是所有的平台都被平等的支持。Rust 的平台支持水平可以被划分为三个等级，每一级都有不同的保证程度。

每个平台都由他们的“目标三围”（"target triple" ？）标识，它是一个代表编译器会产生何种输出的字符串。下面的列代表特定平台是否支持相应的组件。

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

如果我们使用 Linux 或 Mac，所有我们需要做的就是打开一个终端并输入如下：

```bash
$ curl -sSf https://static.rust-lang.org/rustup.sh | sh
```

这会下载一个脚本，并开始安装。如果一切顺利，你将会看到这些：

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

如果你使用的是 Windows 安装包，我们可以再次运行`.msi`文件，它会给我们一个卸载选项。

## 疑难解答（Troubleshooting）

如果我们安装了 Rust，我们可以打开一个 shell，并输入：

```bash
$ rustc --version
```

你应该看到版本号，提交的 hash 值和提交时间。

如果你做到了，那么 Rust 已成功安装！恭喜你！（此处应有掌声）

如果这不能工作并且你在使用 Windows，检查 Rust 是否在你的`%PATH%`系统变量中。如果不是，再次运行安装程序，在“Change, repair, or remove installation”页面选择“Change”并确保“Add to PATH”指向本地硬盘。

如果还是搞不定，这里有几个地方你可以获取帮助。最简单的是通过[Mibbit](http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust)访问位于 irc.mozilla.org 的 #rust IRC频道 。点击上面的链接，你就可以与其它Rustaceans（简单理解为Ruster吧）聊天，我们会帮助你。其它给力的资源包括[用户论坛](https://users.rust-lang.org/)和[Stack Overflow](http://stackoverflow.com/questions/tagged/rust)。

安装程序（脚本）也会在本地安装一份文档拷贝，所以你可以离线阅读它们。在 UNIX 系统上，位置是`/usr/local/share/doc/rust`。在Windows，它位于你 Rust 安装位置的`share/doc`文件夹。
