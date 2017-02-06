# 注释

> [comments.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/comments.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

现在我们写了一些函数，是时候学习一下注释了。注释是你帮助其他程序员理解你的代码的备注。编译器基本上会忽略它们。

Rust有两种需要你了解的注释格式：**行注释**（*line comments*）和**文档注释**（*doc comments*）。

```rust
// Line comments are anything after ‘//’ and extend to the end of the line.

let x = 5; // This is also a line comment.

// If you have a long explanation for something, you can put line comments next
// to each other. Put a space between the // and your comment so that it’s
// more readable.
```

另一种注释是文档注释。文档注释使用`///`而不是`//`，并且内建 Markdown 标记支持：

~~~rust
/// Adds one to the number given.
///
/// # Examples
///
/// ```
/// let five = 5;
///
/// assert_eq!(6, add_one(5));
/// # fn add_one(x: i32) -> i32 {
/// #     x + 1
/// # }
/// ```
fn add_one(x: i32) -> i32 {
    x + 1
}
~~~

有另外一种风格的文档注释，`//!`，用来注释包含它的项（也就是说，crate，模块或者函数），而不是位于它之后的项。它经常用在crate根文件（lib.rs）或者模块根文件（mod.rs）：

```rust
//! # The Rust Standard Library
//!
//! The Rust Standard Library provides the essential runtime
//! functionality for building portable Rust software.
```

当书写文档注释时，加上参数和返回值部分并提供一些用例将是非常，非常有帮助的。你会注意到我们在这里用了一个新的宏：`assert_eq!`。它比较两个值，并当它们不相等时`panic!`。这在文档中是非常有帮助的。还有一个宏，`assert!`，它在传递给它的值是`false`的时候`panic!`。

你可以使用[rustdoc](4.4.Documentation 文档.md)工具来将文档注释生成为HTML文档，也可以将代码示例作为测试运行！
