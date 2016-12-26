# 不使用标准库开发 Rust

> [no-stdlib.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/using-rust-without-the-standard-library.md)
> <br>
> commit 658253d30c124b67c964904400c4dc58a1b557b2

Rust 的标准库提供了很多有用的功能，不过它假设它的 host 系统的多种功能的支持：线程，网络，堆分配和其他功能。有些系统并没有这些功能，不过，Rust也能在这些系统上工作。为此，我们可以通过一个属性来告诉 Rust 我们不想使用标准库：`#![no_std]`。

> 注意：这个功能技术上是稳定的，不过有些附加条件。其一，你可以构建一个稳定的`#![no_std]`库，但二进制文件不行。关于没有标准库的库文件的细节，查看[关于`#![no_std]`的章节](https://github.com/rust-lang/rust/blob/master/src/doc/book/using-rust-without-the-standard-library.html)。

为了使用`#![no_std]`，在 crate 的根文件上加入：

```rust
#![no_std]

fn plus_one(x: i32) -> i32 {
    x + 1
}
```

很多暴露于标准库中的功能通过[`core` crate](https://github.com/rust-lang/rust/blob/stable/src/doc/core/index.html)也同样可用。当我们使用标准库时，Rust 自动将`std`引入到作用域中，允许我们不用显示导入就能使用相关功能。相似的，当使用`#![no_std]`，Rust 会将`core`引入作用域中，以及[它的 prelude](https://github.com/rust-lang/rust/blob/stable/src/doc/core/prelude/v1)。这意味着很多代码也是能正常运行的：

```rust
#![no_std]

fn may_fail(failure: bool) -> Result<(), &'static str> {
    if failure {
        Err("this didn’t work!")
    } else {
        Ok(())
    }
}
```
