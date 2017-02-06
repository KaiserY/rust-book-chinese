# 属性

> [attributes.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/attributes.md)
> <br>
> commit 024aa9a345e92aa1926517c4d9b16bd83e74c10d

在Rust中声明可以用“属性”标注，它们看起来像：

```rust
#[test]
# fn foo() {}
```

或像这样：

```rust
# mod foo {
#![test]
# }
```

这两者的区别是`!`，它改变了属性作用的对象：

```rust
#[foo]
struct Foo;

mod bar {
    #![bar]
}
```

`#[foo]`作用于下一个项，在这就是`struct`声明。`#![bar]`作用于包含它的项，在这是`mod`声明。否则，它们是一样的。它们都以某种方式改变它们附加到的项的意义。

例如，考虑一个像这样的函数：

```rust
#[test]
fn check() {
    assert_eq!(2, 1 + 1);
}
```

它被标记为`#[test]`。这意味着它是特殊的：当你运行[测试](Testing 测试.md)，这个函数将会执行。当你正常编译时，它甚至不会被包含进来。这个函数现在是一个测试函数。

属性也可以有附加数据：

```rust
#[inline(always)]
fn super_fast_fn() {
# }
```

或者甚至是键值：

```rust
#[cfg(target_os = "macos")]
mod macos_only {
# }
```

Rust属性被用在一系列不同的地方。在[参考手册](http://doc.rust-lang.org/nightly/reference.html#attributes)中有一个属性的全表。目前，你不能创建你自己的属性，Rust编译器定义了它们。
