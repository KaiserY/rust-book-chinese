# 语言项（Lang items）

> [lang-items.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/lang-items.md)
> <br>
> commit 893f42a83466cf02b6fd6d3c82d5419cdad47474

> **注意**：语言项通常由 Rust 发行版的 crate 提供，并且它自身有一个不稳定的接口。建议使用官方发布的 crate 而不是定义自己的版本。

`rustc`编译器有一些可插入的操作，也就是说，功能不是硬编码进语言的，而是在库中实现的，通过一个特殊的标记告诉编译器它存在。这个标记是`#[lang="..."]`属性并且有不同的值`...`,也就是不同的“语言项”。

例如，`Box`指针需要两个语言项，一个用于分配，一个用于释放。下面是一个独立的程序使用`Box`语法糖进行动态分配，通过`malloc`和`free`：

```rust,ignore
#![feature(lang_items, box_syntax, start, libc, core_intrinsics)]
#![no_std]
use core::intrinsics;

extern crate libc;

#[lang = "owned_box"]
pub struct Box<T>(*mut T);

#[lang = "exchange_malloc"]
unsafe fn allocate(size: usize, _align: usize) -> *mut u8 {
    let p = libc::malloc(size as libc::size_t) as *mut u8;

    // Check if `malloc` failed:
    if p as usize == 0 {
        intrinsics::abort();
    }

    p
}

#[lang = "exchange_free"]
unsafe fn deallocate(ptr: *mut u8, _size: usize, _align: usize) {
    libc::free(ptr as *mut libc::c_void)
}

#[lang = "box_free"]
unsafe fn box_free<T: ?Sized>(ptr: *mut T) {
    deallocate(ptr as *mut u8, ::core::mem::size_of_val(&*ptr), ::core::mem::align_of_val(&*ptr));
}

#[start]
fn main(argc: isize, argv: *const *const u8) -> isize {
    let x = box 1;

    0
}

#[lang = "eh_personality"] extern fn rust_eh_personality() {}
#[lang = "panic_fmt"] extern fn rust_begin_panic() -> ! { unsafe { intrinsics::abort() }
# #[lang = "eh_unwind_resume"] extern fn rust_eh_unwind_resume() {}
# #[no_mangle] pub extern fn rust_eh_register_frames () {}
# #[no_mangle] pub extern fn rust_eh_unregister_frames () {}
```

注意`abort`的使用：`exchange_malloc`语言项假设返回一个有效的指针，所以需要在内部进行检查。

其它语言项提供的功能包括：

* 通过特性重载运算符：`==`，`<`，解引用（`*`）和`+`等运算符对应的特性都有语言项标记；上面4个分别为`eq`，`ord`，`deref`和`add`
* 栈展开和一般故障：`eh_personality`，`fail`和`fail_bounds_checks`语言项
* `std::marker`中用来标明不同类型的特性：`send`，`sync`和`copy`。
* `std::marker`中的标记类型和变化指示器：`covariant_type`和`contravariant_lifetime`等

语言项由编译器延时加载；例如，如果你从未用过`Box`则就没有必要定义`exchange_malloc`和`exchange_free`的函数。`rustc`在一个项被需要而无法在当前包装箱或任何依赖中找到时生成一个错误。
