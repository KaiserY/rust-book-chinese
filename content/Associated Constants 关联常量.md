# 关联常量

> [associated-constants.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/associated-constants.md)
> <br>
> commit 024aa9a345e92aa1926517c4d9b16bd83e74c10d

通过`associated_consts`功能，你像这样可以定义常量：

```rust
#![feature(associated_consts)]

trait Foo {
    const ID: i32;
}

impl Foo for i32 {
    const ID: i32 = 1;
}

fn main() {
    assert_eq!(1, i32::ID);
}
```

任何`Foo`的定义都必须定义`ID`，不定义的话：

```rust
#![feature(associated_consts)]

trait Foo {
    const ID: i32;
}

impl Foo for i32 {
}
```

会给出

```text
error: not all trait items implemented, missing: `ID` [E0046]
     impl Foo for i32 {
     }
```

也可以实现一个默认值：

```rust
#![feature(associated_consts)]

trait Foo {
    const ID: i32 = 1;
}

impl Foo for i32 {
}

impl Foo for i64 {
    const ID: i32 = 5;
}

fn main() {
    assert_eq!(1, i32::ID);
    assert_eq!(5, i64::ID);
}
```

如你所见，当实现`Foo`时，你可以不实现它（关联常量），当作为`i32`时。接着它将会使用默认值。不过，作为`i64`时，我们可以增加我们自己的定义。

关联常量并不一定要关联在一个 trait 上。一个`struct`的`impl`块或`enum`也行：

```rust
#![feature(associated_consts)]

struct Foo;

impl Foo {
    const FOO: u32 = 3;
}
```
