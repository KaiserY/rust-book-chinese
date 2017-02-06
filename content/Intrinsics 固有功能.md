# 固有功能

> [intrinsics.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/intrinsics.md)
> <br>
> commit 024aa9a345e92aa1926517c4d9b16bd83e74c10d

> **注意**：固有功能将会永远是一个不稳定的接口，推荐使用稳定的 libcore 接口而不是直接使用编译器自带的功能。

可以像 FFI 函数那样导入它们，使用特殊的`rust-intrinsic`ABI。例如，如果在一个独立的上下文，但是想要能在类型间`transmute`，并想进行高效的指针计算，你可以声明函数：

```rust
#![feature(intrinsics)]
# fn main() {}

extern "rust-intrinsic" {
    fn transmute<T, U>(x: T) -> U;

    fn offset<T>(dst: *const T, offset: isize) -> *const T;
}
```

跟其它 FFI 函数一样，它们总是`unsafe`的。
