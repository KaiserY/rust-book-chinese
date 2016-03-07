# 不使用标准库

> [no-stdlib.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/no-stdlib.md)
> <br>
> commit 0394418752cd39c5da68e7e05d5a37bf5a30f0db

Rust 的标准库提供了很多有用的功能，不过它假设它的 host 系统的多种功能的支持：线程，网络，堆分配和其他功能。有些系统并没有这些功能，不过，Rust也能在这些系统上工作。为此，我们可以通过一个属性来告诉 Rust 我们不想使用标准库：`#![no_std]`。

> 注意：这个功能技术上是稳定的，不过有些附加条件。其一，你可以构建一个稳定的`#![no_std]`库，但二进制文件不行。关于没有标准库的库文件的细节，查看[关于`#![no_std]`的章节](https://github.com/rust-lang/rust/blob/master/src/doc/book/using-rust-without-the-standard-library.html)。

被标记为`#[start]`的函数传递的参数格式与 C 一致：

```rust
# #![feature(libc)]
#![feature(lang_items)]
#![feature(start)]
#![no_std]

// Pull in the system libc library for what crt0.o likely requires
extern crate libc;

// Entry point for this program
#[start]
fn start(_argc: isize, _argv: *const *const u8) -> isize {
    0
}

// These functions and traits are used by the compiler, but not
// for a bare-bones hello world. These are normally
// provided by libstd.
#[lang = "eh_personality"] extern fn eh_personality() {}
#[lang = "panic_fmt"] fn panic_fmt() -> ! { loop {} }
# #[lang = "eh_unwind_resume"] extern fn rust_eh_unwind_resume() {}
# #[no_mangle] pub extern fn rust_eh_register_frames () {}
# #[no_mangle] pub extern fn rust_eh_unregister_frames () {}
# // fn main() {} tricked you, rustdoc!
```

要覆盖编译器插入的`main`shim，你必须使用`#![no_main]`禁用它并通过正确的 ABI 和正确的名字来创建合适的函数，这也需要需要覆盖编译器的命名改编：

```rust
# #![feature(libc)]
#![feature(lang_items)]
#![feature(start)]
#![no_std]
#![no_main]

extern crate libc;

#[no_mangle] // ensure that this symbol is called `main` in the output
pub extern fn main(argc: i32, argv: *const *const u8) -> i32 {
    0
}

#[lang = "eh_personality"] extern fn eh_personality() {}
#[lang = "panic_fmt"] fn panic_fmt() -> ! { loop {} }
# #[lang = "eh_unwind_resume"] extern fn rust_eh_unwind_resume() {}
# #[no_mangle] pub extern fn rust_eh_register_frames () {}
# #[no_mangle] pub extern fn rust_eh_unregister_frames () {}
# // fn main() {} tricked you, rustdoc!
```

目前编译器对能够被可执行文件调用的符号做了一些假设。正常情况下，这些函数是由标准库提供的，不过没有它你就必须定义你自己的了。

这三个函数中的第一个`stack_exhausted`，当检测到栈溢出时被调用。这个函数对于如何被调用和应该干什么有一些限制，不顾如果栈限制寄存器没有被维护则一个线程可以有”无限的栈“，这种情况下这个函数不应该被触发。

第二个函数，`eh_personality`，被编译器的错误机制使用。它通常映射到 GCC 的特性函数上（查看[libstd实现](http://doc.rust-lang.org/std/rt/unwind/)来获取更多信息），不过对于不会触发恐慌的包装箱可以确定这个函数不会被调用。最后一个函数，`panic_fmt`，也被编译器的错误机制使用。

> 如下内容已被删除，暂时保留

## 使用 libcore
> **注意**：核心库的结构是不稳定的，建议在任何可能的情况下使用标准库。

通过上面的计数，我们构造了一个少见的运行Rust代码的可执行程序。标准库提供了很多功能，然而，这是Rust的生产力所需要的。如果标准库是不足的话，那么可以使用被设计为标准库替代的[libcore](http://doc.rust-lang.org/core/)。

核心库只有很少的依赖并且比标准库可移植性更强。另外，核心库包含编写符合习惯和高效Rust代码的大部分功能。

例如，下面是一个计算由C提供的两个向量的数量积的函数，使用常见的Rust实现。

```rust
# #![feature(libc)]
#![feature(lang_items)]
#![feature(start)]
#![feature(raw)]
#![no_std]

extern crate libc;

use core::mem;

#[no_mangle]
pub extern fn dot_product(a: *const u32, a_len: u32,
                          b: *const u32, b_len: u32) -> u32 {
    use core::raw::Slice;

    // Convert the provided arrays into Rust slices.
    // The core::raw module guarantees that the Slice
    // structure has the same memory layout as a &[T]
    // slice.
    //
    // This is an unsafe operation because the compiler
    // cannot tell the pointers are valid.
    let (a_slice, b_slice): (&[u32], &[u32]) = unsafe {
        mem::transmute((
            Slice { data: a, len: a_len as usize },
            Slice { data: b, len: b_len as usize },
        ))
    };

    // Iterate over the slices, collecting the result
    let mut ret = 0;
    for (i, j) in a_slice.iter().zip(b_slice.iter()) {
        ret += (*i) * (*j);
    }
    return ret;
}

#[lang = "panic_fmt"]
extern fn panic_fmt(args: &core::fmt::Arguments,
                    file: &str,
                    line: u32) -> ! {
    loop {}
}

#[lang = "eh_personality"] extern fn eh_personality() {}
# #[start] fn start(argc: isize, argv: *const *const u8) -> isize { 0 }
# #[lang = "eh_unwind_resume"] extern fn rust_eh_unwind_resume() {}
# #[no_mangle] pub extern fn rust_eh_register_frames () {}
# #[no_mangle] pub extern fn rust_eh_unregister_frames () {}
# fn main() {}
```
注意这里有一个额外的`lang`项与之前的例子不同，`panic_fmt`。它必须由libcore的调用者定义因为核心库声明了恐慌，但没有定义它。`panic_fmt`项是这个包装箱的恐慌定义，并且它必须确保不会返回。

正如你在例子中所看到的，核心库尝试在所有情况下提供Rust的功能，不管平台的要求如何。另外一些库，例如`liballoc`，为libcore增加了进行其它平台相关假设的功能，不过这依旧比标准库更有可移植性。
