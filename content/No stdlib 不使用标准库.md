# 不使用标准库

> [no-stdlib.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/no-stdlib.md)
> <br>
> commit 893f42a83466cf02b6fd6d3c82d5419cdad47474

Rust 的标准库提供了很多有用的功能，不过它假设它的 host 系统的多种功能的支持：线程，网络，堆分配和其他功能。有些系统并没有这些功能，不过，Rust也能在这些系统上工作。为此，我们可以通过一个属性来告诉 Rust 我们不想使用标准库：`#![no_std]`。

> 注意：这个功能技术上是稳定的，不过有些附加条件。其一，你可以构建一个稳定的`#![no_std]`库，但二进制文件不行。关于没有标准库的库文件的细节，查看[关于`#![no_std]`的章节](https://github.com/rust-lang/rust/blob/master/src/doc/book/using-rust-without-the-standard-library.html)。

很显然你并不一定需要标准库：可以使用`#[no_std`来构建一个可执行程序。

## 使用 libc

为了构建一个`#[no_std]`可执行程序，我们需要 libc 作为依赖。可以在`Cargo.toml`文件中指定：

```toml
[dependencies]
libc = { version = "0.2.14", default-features = false }
```

注意默认功能被禁用了。这是关键的一步————**libc 的默认功能引用了标准库所以必须被禁用。**

## 不用标准库编写可执行程序

有两种可能的控制入口点的方法：`#[start]`属性，或者用你自己的代码 override C `main`函数的默认 shim。

被标记为`#[start]`的函数传递的参数格式与 C 一致：

```rust
#![feature(lang_items, core_intrinsics)]
#![feature(start)]
#![no_std]
use core::intrinsics;

// Pull in the system libc library for what crt0.o likely requires.
extern crate libc;

// Entry point for this program.
#[start]
fn start(_argc: isize, _argv: *const *const u8) -> isize {
    0
}

// These functions are used by the compiler, but not
// for a bare-bones hello world. These are normally
// provided by libstd.
#[lang = "eh_personality"]
#[no_mangle]
pub extern fn rust_eh_personality() {
}

// This function may be needed based on the compilation target.
#[lang = "eh_unwind_resume"]
#[no_mangle]
pub extern fn rust_eh_unwind_resume() {
}

#[lang = "panic_fmt"]
#[no_mangle]
pub extern fn rust_begin_panic(_msg: core::fmt::Arguments,
                               _file: &'static str,
                               _line: u32) -> ! {
    unsafe { intrinsics::abort() }
}
```

要 override 编译器插入的`main` shim，你必须使用`#![no_main]`禁用它并通过正确的 ABI 和正确的名字来创建合适的函数，这也需要需要覆盖编译器的命名改编：

```rust
#![feature(lang_items, core_intrinsics)]
#![feature(start)]
#![no_std]
#![no_main]
use core::intrinsics;

// Pull in the system libc library for what crt0.o likely requires.
extern crate libc;

// Entry point for this program.
#[no_mangle] // ensure that this symbol is called `main` in the output
pub extern fn main(_argc: i32, _argv: *const *const u8) -> i32 {
    0
}

// These functions are used by the compiler, but not
// for a bare-bones hello world. These are normally
// provided by libstd.
#[lang = "eh_personality"]
#[no_mangle]
pub extern fn rust_eh_personality() {
}

// This function may be needed based on the compilation target.
#[lang = "eh_unwind_resume"]
#[no_mangle]
pub extern fn rust_eh_unwind_resume() {
}

#[lang = "panic_fmt"]
#[no_mangle]
pub extern fn rust_begin_panic(_msg: core::fmt::Arguments,
                               _file: &'static str,
                               _line: u32) -> ! {
    unsafe { intrinsics::abort() }
}
```

## 关于 language items 的更多细节

目前编译器对能够被可执行文件调用的符号做了一些假设。正常情况下，这些函数是由标准库提供的，不过没有它你就必须定义你自己的了。这些符号被称为“language items”，并且他们每个都有一个内部的名称，和一个必须符合签名的实现。

这些函数中的第一个，`eh_personality`，被编译器的错误机制使用。它通常映射到 GCC 的特性函数上（查看[libstd实现](https://github.com/rust-lang/rust/blob/master/src/libpanic_unwind/gcc.rs)来获取更多信息），不过对于不会触发恐慌的包装箱可以确定这个函数不会被调用。language item 的名称是`eh_personality`。

第二个函数，`rust_begin_panic`，也被作为编译器的错误机制使用。当发生 panic 时，它控制显示在屏幕上的信息。虽然 language item 的名称是`panic_fmt`，但是符号的名称是`rust_begin_panic`。

第三个函数，`rust_eh_unwind_resume`，在target 选项中的`custom_unwind_resume` flag 被设置时也是必需的。它允许自定义在 landing pads 的最后的 resuming unwind 过程。language item 的名字是`eh_unwind_resume`。
