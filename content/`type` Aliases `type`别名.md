# type 别名

> [type-aliases.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/type-aliases.md)
> <br>
> commit 1eb36b80a43154107729da3e496d0b3fb9e57259

`type`关键字让你定义另一个类型的别名：

```rust
type Name = String;
```

你可以像一个真正类型那样使用这个类型：

```rust
type Name = String;

let x: Name = "Hello".to_string();
```

然而要注意的是，这一个**别名**，完全不是一个新的类型。换句话说，因为Rust是强类型的，你可以预期两个不同类型的比较会失败：

```rust
let x: i32 = 5;
let y: i64 = 5;

if x == y {
   // ...
}
```

这给出

```text
error: mismatched types:
 expected `i32`,
    found `i64`
(expected i32,
    found i64) [E0308]
     if x == y {
             ^
```

不过，如果我们有一个别名：

```rust
type Num = i32;

let x: i32 = 5;
let y: Num = 5;

if x == y {
   // ...
}
```

这会无错误的编译。从任何角度来说，`Num`类型的值与`i32`类型的值都是一样的。

你也可以在泛型中使用类型别名：

```rust
use std::result;

enum ConcreteError {
    Foo,
    Bar,
}

type Result<T> = result::Result<T, ConcreteError>;
```

这创建了一个特定版本的`Result`类型，它总是有一个`ConcreteError`作为`Result<T, E>`的`E`那部分。这通常用于标准库中创建每个子部分的自定义错误。例如，[`io::Result`](http://doc.rust-lang.org/nightly/std/io/type.Result.html)。
