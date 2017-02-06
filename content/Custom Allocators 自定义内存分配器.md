# 自定义内存分配器

> [custom-allocators.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/custom-allocators.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

分配内存并不总是最简单的事情，同时通常 Rust 默认会负责它，不过经常自定义内存分配会变得必要。编译器和标准库目前允许在编译时切换目前默认使用的全局分配器。设计目前称作[RFC 1183](https://github.com/rust-lang/rfcs/blob/master/text/1183-swap-out-jemalloc.md)不过这里我们会教你如何获取你自己的分配器并运行起来。

## 默认分配器

编译器目前自带两个默认分配器：`alloc_system`和`alloc_jemalloc`（然而一些目标平台并没有 jemalloc）。这些分配器是正常的 Rust crate 并包含分配和释放内存的 routine 的实现。标准库并不假设使用任何一个编译，而且编译器会在编译时根据被产生的输出类型决定使用哪个分配器。

编译器产生的二进制文件默认会使用`alloc_jemalloc`（如果可用的话）。在这种情况下编译器“控制了一切”，从它超过了最终链接的权利的角度来看。大体上这意味着分配器选择可以被交给编译器。

动态和静态库，然而，默认使用`alloc_system`。这里 Rust 通常是其他程序的“客人”或者处于并没有权决定应使用的分配器的世界。为此它求助于标准 API（例如，`malloc`和`free`）来获取和释放内存。

## 切换分配器

虽然编译器默认的选择大部分情况工作良好，也经常需要定制特定的方面。覆盖编译器关于使用哪个分配器的选择可以简单的通过链接到期望的分配器实现：

```rust
#![feature(alloc_system)]

extern crate alloc_system;

fn main() {
    let a = Box::new(4); // Allocates from the system allocator.
    println!("{}", a);
}
```

在这个例子中生成的二进制文件并不会默认链接到 jemalloc 而是使用了系统分配器。同理生成一个默认使用 jemalloc 的动态库可以写成：

```rust
#![feature(alloc_jemalloc)]
#![crate_type = "dylib"]

extern crate alloc_jemalloc;

pub fn foo() {
    let a = Box::new(4); // Allocates from jemalloc.
    println!("{}", a);
}
# fn main() {}
```

### 编写一个自定义分配器

有时甚至 jemalloc 与系统分配器之间的选择都是不够的并需要一个新的自定义的分配器。这种情况你要编写你自己实现了分配器 API（例如与`alloc_system`和`alloc_jemallo`相同）的 crate。作为一个例子，让我们看看一个简单的和声明化的`alloc_system`版本：

```rust
# // Only needed for rustdoc --test down below.
# #![feature(lang_items)]
// The compiler needs to be instructed that this crate is an allocator in order
// to realize that when this is linked in another allocator like jemalloc should
// not be linked in.
#![feature(allocator)]
#![allocator]

// Allocators are not allowed to depend on the standard library which in turn
// requires an allocator in order to avoid circular dependencies. This crate,
// however, can use all of libcore.
#![no_std]

// Let's give a unique name to our custom allocator:
#![crate_name = "my_allocator"]
#![crate_type = "rlib"]

// Our system allocator will use the in-tree libc crate for FFI bindings. Note
// that currently the external (crates.io) libc cannot be used because it links
// to the standard library (e.g. `#![no_std]` isn't stable yet), so that's why
// this specifically requires the in-tree version.
#![feature(libc)]
extern crate libc;

// Listed below are the five allocation functions currently required by custom
// allocators. Their signatures and symbol names are not currently typechecked
// by the compiler, but this is a future extension and are required to match
// what is found below.
//
// Note that the standard `malloc` and `realloc` functions do not provide a way
// to communicate alignment so this implementation would need to be improved
// with respect to alignment in that aspect.

#[no_mangle]
pub extern fn __rust_allocate(size: usize, _align: usize) -> *mut u8 {
    unsafe { libc::malloc(size as libc::size_t) as *mut u8 }
}

#[no_mangle]
pub extern fn __rust_deallocate(ptr: *mut u8, _old_size: usize, _align: usize) {
    unsafe { libc::free(ptr as *mut libc::c_void) }
}

#[no_mangle]
pub extern fn __rust_reallocate(ptr: *mut u8, _old_size: usize, size: usize,
                                _align: usize) -> *mut u8 {
    unsafe {
        libc::realloc(ptr as *mut libc::c_void, size as libc::size_t) as *mut u8
    }
}

#[no_mangle]
pub extern fn __rust_reallocate_inplace(_ptr: *mut u8, old_size: usize,
                                        _size: usize, _align: usize) -> usize {
    old_size // This api is not supported by libc.
}

#[no_mangle]
pub extern fn __rust_usable_size(size: usize, _align: usize) -> usize {
    size
}

# // Only needed to get rustdoc to test this:
# fn main() {}
# #[lang = "panic_fmt"] fn panic_fmt() {}
# #[lang = "eh_personality"] fn eh_personality() {}
# #[lang = "eh_unwind_resume"] extern fn eh_unwind_resume() {}
# #[no_mangle] pub extern fn rust_eh_register_frames () {}
# #[no_mangle] pub extern fn rust_eh_unregister_frames () {}
```

在我们编译了这个 crate 之后，他可以被如下使用：

```rust
extern crate my_allocator;

fn main() {
    let a = Box::new(8); // Allocates memory via our custom allocator crate.
    println!("{}", a);
}
```

### 自定义分配器的限制

使用自定义分配器时要满足一些限制，否则可能导致编译器错误：

* 任何一个程序只能链接到一个分配器。二进制、动态库和静态库必须正好链接到一个分配器上，并且，如果没有显式选择，编译器会选择一个。另一方面，rlib 并不需要链接到一个分配器（不过仍然可以）。

* 一个标记为`#![needs_allocator]`（例如，目前的`liballoc`）的分配器使用者和一个`#[allocator]` crate 不能直接依赖一个需要分配器的 crate（例如，循环引用是不允许的）。这基本上意味着目前分配器必须依赖 libcore。
