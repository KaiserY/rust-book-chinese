# 文档

> [documentation.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/documentation.md)
> <br>
> commit d8ee0745f734231edb16441776a60c33dae317c2

在任何软件项目中，文档都是重要的部分，其同样在Rust中是头等重要的。让我们讨论下Rust提供给我们的编写项目文档的工具。

## 关于`rustdoc`
Rust发行版中包含了一个工具，`rustdoc`，它可以生成文档。`rustdoc`也可以在Cargo中通过`cargo doc`使用。

文档可以使用两种方法生成：从源代码，或者从单独的Markdown文件。

### 文档化源代码
文档化Rust项目的主要方法是在源代码中添加注释。你可以使用文档注释以实现此目的：

~~~rust,ignore
/// Constructs a new `Rc<T>`.
///
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
pub fn new(value: T) -> Rc<T> {
    // Implementation goes here.
}
~~~

这段代码产生像[这样](http://doc.rust-lang.org/nightly/std/rc/struct.Rc.html#method.new)的文档。我忽略了函数的实现，而在此处留下了一个常规注释。

这个注释首先需要注意的地方是它使用了`///`，而不是`//`。三条斜线表明这是文档注释。

文档注释用Markdown语法编写。

Rust会跟踪这些注释，并在生成文档时使用它们。这在文档化像枚举这样的结构时很重要：

```rust
/// The `Option` type. See [the module level documentation](../) for more.
enum Option<T> {
    /// No value
    None,
    /// Some value `T`
    Some(T),
}
```

以上代码可以工作，但这个就不行了：

```rust
/// The `Option` type. See [the module level documentation](../) for more.
enum Option<T> {
    None, /// No value
    Some(T), /// Some value `T`
}
```

你会得到一个错误：

```text
hello.rs:4:1: 4:2 error: expected ident, found `}`
hello.rs:4 }
           ^
```

这个[不幸的错误](https://github.com/rust-lang/rust/issues/22547)是有道理的；文档注释适用于它后面的内容，而在最后的注释后面没有任何内容。

### 编写文档注释

不管怎样，让我们来详细了解一下注释的每一部分：

```rust
/// Constructs a new `Rc<T>`.
# fn foo() {}
```

文档注释的第一行应该是其功能的一个简短摘要。一句话。唯基础。高层次。

```rust
///
/// Other details about constructing `Rc<T>`s, maybe describing complicated
/// semantics, maybe additional options, all kinds of stuff.
///
# fn foo() {}
```

我们原始的例子只有一行摘要，不过如果有更多东西要写，我们可以新开一个新的段落增加更多解释。

### 特殊部分

下面，是特殊部分。它由一个标头指示，`#`。有四种经常使用的标头。到目前为止，它们不是特殊语法，只是传统。

```rust
/// # Panics
# fn foo() {}
```

Rust中不可恢复的函数滥用（如程序错误）通常用恐慌（panics）表示，它至少也要杀死整个当前线程。如果你的函数有这样一个非凡的契约，需要识别或者强制引发恐慌，记录文档是非常重要的。

```rust
/// # Errors
# fn foo() {}
```

如果你的函数或方法返回`Result<T, E>`，那么描述何种情况下它会返回`Err(E)`是件值得做的好事。这并不如`Panics`重要，因为失败被编码进了类型系统，不过仍旧值得一做。

```rust
/// # Safety
# fn foo() {}
```

如果你的函数是`unsafe`的，你应该解释调用者需负责维护哪些不可变量。

~~~rust
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
# fn foo() {}
~~~

第四个，`Examples`。包含一个或多个使用你的函数或方法的例子，这样你的用户会为此感谢你的。这些例子写在代码块注释中（我们稍后会讨论到），并且可以有不止一个段落：

~~~rust
/// # Examples
///
/// Simple `&str` patterns:
///
/// ```
/// let v: Vec<&str> = "Mary had a little lamb".split(' ').collect();
/// assert_eq!(v, vec!["Mary", "had", "a", "little", "lamb"]);
/// ```
///
/// More complex patterns with a lambda:
///
/// ```
/// let v: Vec<&str> = "abc1def2ghi".split(|c: char| c.is_numeric()).collect();
/// assert_eq!(v, vec!["abc", "def", "ghi"]);
/// ```
# fn foo() {}
~~~

让我们聊聊这些代码块的细节。

### 代码块注释

要在注释中写Rust代码，使用三个重音号：

~~~rust
/// ```
/// println!("Hello, world");
/// ```
# fn foo() {}
~~~

如果你要写一些非Rust的代码，你可以加上注解：

~~~rust
/// ```c
/// printf("Hello, world\n");
/// ```
# fn foo() {}
~~~

这会根据你选择的语言高亮代码。如果你只是想展示普通文本，选择`text`。

选择正确的注释是重要的，因为`rustdoc`用一种有意思的方法使用它：在库crate中它可以用来实际测试你的示例代码，使之不至于过时。如果你写了C代码但是忽略了注解，`rustdoc`会认为它是Rust代码，会在你生成文档时提示。

## 用作测试的文档

讨论下我们的样本示例文档：

~~~rust
/// ```
/// println!("Hello, world");
/// ```
# fn foo() {}
~~~

你会注意到这里并不需要`fn main()`或者别的东西。`rustdoc`会自动加一个`main()`来包裹你的代码，用试探法来把它放到正确的位置。例如：

~~~rust
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
# fn foo() {}
~~~

这是最终的测试：

```rust
fn main() {
    use std::rc::Rc;
    let five = Rc::new(5);
}
```

下面是`rustdoc`预处理示例的完整算法：

1. 任何`#![foo]`开头的属性会被完整的作为包装箱属性
2. 一些通用的`allow`属性被插入，包括`unused_variables`、`unused_assignments`、`unused_mut`、`unused_attributes`和`dead_code`。小的例子经常触发这些lint检查
3. 如果例子并未包含`extern crate`，那么`extern crate <mycrate>;`被插入（注意缺失了`#[macro_use]`）
4. 最后，如果例子不包含`fn main`，剩下的文本将被包装到`fn main() { your_code }`中

有时，这是不够的。例如，我们已经考虑到了所有`///`开头的代码样例了吗？普通文本：

```text
/// Some documentation.
# fn foo() {}
```

与它的输出看起来有些不同：

```rust
/// Some documentation.
# fn foo() {}
```

是的，你猜对了：你写的以`#`开头的行会在输出中被隐藏，不过会在编译你的代码时被使用。你可以利用这一点。在这个例子中，文档注释需要适用于一些函数，所以我只想向你展示文档注释，我需要在下面增加一些函数定义。同时，这只是用来满足编译器的，所以省略它会使得例子看起来更清楚。你可以使用这个技巧来详细的解释较长的例子，同时保留你文档的可测试行。

例如，想象一下我们想要为如下代码写文档：

```rust
let x = 5;
let y = 6;
println!("{}", x + y);
```

最终我们可能想要文档变成这样：

> 首先，我们把`x`设置为`5`：
>
> ```rust
> let x = 5;
> # let y = 6;
> # println!("{}", x + y);
> ```
>
> 接着，我们把`y`设置为`6`：
>
> ```rust
> # let x = 5;
> let y = 6;
> # println!("{}", x + y);
> ```
>
> 最后，我们打印`x`和`y`的和：
>
> ```rust
> # let x = 5;
> # let y = 6;
> println!("{}", x + y);
> ```

为了让每个代码块可以执行，我们想要每个代码块都有整个程序，不过我们并不想读者每回都看到所有的行。这是我们的源代码：

~~~text
    首先，我们把`x`设置为`5`：

    ```text
    let x = 5;
    # let y = 6;
    # println!("{}", x + y);
    ```

    接着，我们把`y`设置为`6`：

    ```text
    # let x = 5;
    let y = 6;
    # println!("{}", x + y);
    ```

    最后，我们打印`x`和`y`的和：

    ```text
    # let x = 5;
    # let y = 6;
    println!("{}", x + y);
    ```
~~~

通过重复例子的所有部分，你可以确保你的例子仍能编译，同时只显示与你解释相关的部分。

### 文档化宏

下面是一个宏的文档例子：

~~~rust
/// Panic with a given message unless an expression evaluates to true.
///
/// # Examples
///
/// ```
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(1 + 1 == 2, “Math is broken.”);
/// # }
/// ```
///
/// ```should_panic
/// # #[macro_use] extern crate foo;
/// # fn main() {
/// panic_unless!(true == false, “I’m broken.”);
/// # }
/// ```
#[macro_export]
macro_rules! panic_unless {
    ($condition:expr, $($rest:expr),+) => ({ if ! $condition { panic!($($rest),+); } });
}
# fn main() {}
~~~

你会注意到3个地方：我们需要添加我们自己的`extern crate`行，这样我们可以添加`#[macro_use]`属性。第二，我们也需要添加我们自己的`main()`（为了上面讨论过的原因）。最后，用`#`机智的注释掉这两个代码，这样它们不会出现在输出中。

另一个`#`好用的情况是当你想要忽略错误处理的时候。例如你想要如下情况。

```rust
/// use std::io;
/// let mut input = String::new();
/// try!(io::stdin().read_line(&mut input));
```

问题是`try!`返回一个`Result<T, E>`而测试函数并不返回任何值所以这会产生一个类型不匹配错误。

~~~rust
/// A doc test using try!
///
/// ```
/// use std::io;
/// # fn foo() -> io::Result<()> {
/// let mut input = String::new();
/// try!(io::stdin().read_line(&mut input));
/// # Ok(())
/// # }
/// ```
# fn foo() {}
~~~

你可以将代码放进函数里来解决这个问题。在运行文档测试时它捕获并返回`Result<T, E>`。这种模式不时出现在标准库中。

### 运行文档测试

要运行测试，要么

```bash
$ rustdoc --test path/to/my/crate/root.rs
# or（或者）
$ cargo test
```

对了，`cargo test`也会测试内嵌的文档。**然而，`cargo test`将不会测试二进制 crate，只测试库 crate**。这是由于`rustdoc`的运行机制：它链接要测试的库，不过对于一个二进制文件，木有什么好链接的。

这还有一些注释有利于帮助`rustdoc`在测试你的代码时正常工作：

~~~rust
/// ```ignore
/// fn foo() {
/// ```
# fn foo() {}
~~~

`ignore`指令告诉Rust忽略你的代码。这几乎不会是你想要的，因为这是最不受支持的。相反，如果不是代码的话考虑注释为`text`，或者使用`#`来形成一个可运行但只显示你关心部分的例子。

~~~rust
/// ```should_panic
/// assert!(false);
/// ```
# fn foo() {}
~~~

`should_panic`告诉`rustdoc`这段代码应该正确编译，但是作为一个测试则不能通过。

~~~rust
/// ```no_run
/// loop {
///     println!("Hello, world");
/// }
/// ```
# fn foo() {}
~~~

`no_run`属性会编译你的代码，但是不运行它。这对像如“如何开始一个网络服务”这样的例子很重要，你会希望确保它能够编译，不过它可能会在一个没有网络连接的测试环境中运行。

### 文档化模块
Rust有另一种文档注释，`//!`。这种注释并不文档化接下来的内容，而是包围它的内容。换句话说：

```rust
mod foo {
    //! This is documentation for the `foo` module.
    //!
    //! # Examples

    // ...
}
```

这是你会看到`//!`最常见的用法：作为模块文档。如果你在`foo.rs`中有一个模块，打开它你常常会看到这些：

```rust
//! A module for using `foo`s.
//!
//! The `foo` module contains a lot of useful functionality blah blah blah...
```

### Crate 文档

Crate 文档可以通过在 crate 根文件，也就是`lib.rs`，的开头放置文档内注释（`//!`）来编写：

```rust
//! This is documentation for the `foo` crate.
//!
//! The foo crate is meant to be used for bar.
```

### 文档注释风格

查看[RFC 505](https://github.com/rust-lang/rfcs/blob/master/text/0505-api-comment-conventions.md)以了解文档风格和格式的惯例。

### 其它文档

所有这些行为都能在非 Rust 代码文件中工作。因为注释是用 Markdown 编写的，它们通常是`.md`文件。

当你在 Markdown 文件中写文档时，你并不需要加上注释前缀。例如：

~~~rust
/// # Examples
///
/// ```
/// use std::rc::Rc;
///
/// let five = Rc::new(5);
/// ```
# fn foo() {}
~~~

在一个Markdown文件中，就是：

~~~markdown
# Examples

```
use std::rc::Rc;

let five = Rc::new(5);
```
~~~

不过文档写在 Markdown 文件中要加一点：Markdown文件需要有一个像这样的标题：

```markdown
% The title

This is the example documentation.
```

`%`行需要放在文件的第一行。

## `doc`属性
在更底层，文档注释是文档属性的语法糖：

```rust
/// this
# fn foo() {}

#[doc="this"]
# fn bar() {}
```

跟下面这个是类似的：

```rust
//! this

#![doc="this"]
```

写文档时你不会经常看见这些属性，不过当你要改变一些选项，或者写一个宏的时候比较有用。

## 重导出（Re-exports）

`rustdoc`对公有重导出部分会在两个地方都显示文档：

```rust
extern crate foo;

pub use foo::bar;
```

这既会为`bar`在`foo`包装箱中生成文档，也会在你的包装箱中生成文档。它会在两个地方使用相同的内容。

这种行为可以通过`no_inline`来阻止：

```rust
extern crate foo;

#[doc(no_inline)]
pub use foo::bar;
```

## 缺失文档

有时你想要确保你项目中每一个公开的项都有文档，特别是当你编写一个库的时候。Rust 允许你为此产生警告或错误，当一个项缺少文档时。为了生成警告，你需要使用`warn`：

```rust
#![warn(missing_docs)]
```

而为了生成错误你需要使用`deny`：

```rust
#![deny(missing_docs)]
```

有时你想要显式的禁用警告/错误来让一些项没有文档。这可以使用`allow`来搞定：

```rust
#[allow(missing_docs)]
struct Undocumented;
```

你可能甚至希望在文档中完全隐藏某些项：

```rust
#[doc(hidden)]
struct Hidden;
```

## 控制 HTML
你可以通过`#![doc]`属性控制`rustdoc`生成的THML文档的几个地方：

```rust
#![doc(html_logo_url = "https://www.rust-lang.org/logos/rust-logo-128x128-blk-v2.png",
       html_favicon_url = "https://www.rust-lang.org/favicon.ico",
       html_root_url = "https://doc.rust-lang.org/")]
```

这里设置了一些不同的选项，带有一个logo，一个网站图标，和一个根URL。

## 配置文档测试

你也可以通过`#![doc(test(..))]`属性来配置`rustdoc`测试你文档示例的方式。

```rust
#![doc(test(attr(allow(unused_variables), deny(warnings))))]
```

这允许示例中存在未使用的变量，但其他 lint 警告抛出仍会使测试失败。

## 生成选项
`rustdoc`也提供了一些其他命令行选项，以便进一步定制：

* `--html-in-header FILE`：在`<head>...</head>`部分的末尾加上`FILE`内容
* `--html-before-content FILE`：在`<body>`之后，在渲染内容之前加上`FILE`内容
* `--html-after-content FILE`：在所有渲染内容之后加上`FILE`内容

## 安全事项
文档注释中的Markdown会被不加以处理地放置于最终的网页中。注意原始的HTML文本：

```rust
/// <script>alert(document.cookie)</script>
# fn foo() {}
```
