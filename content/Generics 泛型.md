# 泛型

> [generics.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/generics.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

有时，当你编写函数或数据类型时，我们可能会希望它能处理多种类型的参数。幸运的是，Rust有一个能给我们更好选择的功能：泛型。泛型在类型理论中叫做**参数多态**（*parametric polymorphism*），它意味着它们是对于给定参数（parametric）能够有多种形式（`poly`是多，`morph`是形态）的函数或类型。

不管怎么样，类型理论就说这么多，现在我们来看些泛型代码。Rust 标准库提供了一个范型的类型——`Option<T>`：

```rust
enum Option<T> {
    Some(T),
    None,
}
```

之前你已见过几次的`<T>`部分代表它是一个泛型数据类型。在上面的枚举声明中，每当我们看到`T`，我们用这个类型代替我们泛型中使用的类型。下面是一个使用`Option<T>`的例子，它带有额外的类型标注：

```rust
let x: Option<i32> = Some(5);
```

在类型声明中，我们看到`Option<i32>`。注意它与`Option<T>`的相似之处。在这个特定的`Option`中，`T`的值为`i32`。在绑定的右侧，我们用了`Some(T)`，其中`T`是`5`。因为它是`i32`型的，两边类型相符，所以皆大欢喜。如果不相符，我们会得到一个错误：

```rust
let x: Option<f64> = Some(5);
// error: mismatched types: expected `core::option::Option<f64>`,
// found `core::option::Option<_>` (expected f64 but found integral variable)
```

这并不意味着我们不能写用`f64`的`Option<T>`！只是类型必须相符：

```rust
let x: Option<i32> = Some(5);
let y: Option<f64> = Some(5.0f64);
```

这样就好了。一处定义，到处使用。

不一定只有一个类型是泛型的。想想Rust标准库中另一个类似的`Result<T, E>`类型：

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

这里有两个泛型类型：`T`和`E`。另外，大写字母可以是任何你喜欢的（大写）字母。我们可以定义`Result<T, E>`为：

```rust
enum Result<A, Z> {
    Ok(A),
    Err(Z),
}
```

如果你想这么做的话。惯例告诉我们第一个泛型参数应该是`T`，代表`type`，然后我们用`E`来代表`error`。然而，Rust并不管这些。

`Result<T, E>`意图作为计算的返回值，并为了能够在不能工作时返回一个错误。

## 泛型函数
我们可以用熟悉的语法编写一个获取泛型参数的函数：

```rust
fn takes_anything<T>(x: T) {
    // Do something with `x`.
}
```

语法有两部分：`<T>`代表“这个函数带有一个泛型类型”，而`x: T`代表“`x`是`T`类型的”。

多个参数可以有相同的泛型类型：

```rust
fn takes_two_of_the_same_things<T>(x: T, y: T) {
    // ...
}
```

我们可以写一个获取多个（泛型）类型的版本：

```rust
fn takes_two_things<T, U>(x: T, y: U) {
    // ...
}
```

## 泛型结构体（Generic structs）
你也可以在一个`struct`中储存泛型类型：

```rust
struct Point<T> {
    x: T,
    y: T,
}

let int_origin = Point { x: 0, y: 0 };
let float_origin = Point { x: 0.0, y: 0.0 };
```

与函数类似，`<T>`是我们声明的泛型参数，而我们也接着在类型定义中使用`x: T`。

当你想要给泛型`struct`增加一个实现时，你可以在`impl`声明类型参数：

```rust
# struct Point<T> {
#     x: T,
#     y: T,
# }
#
impl<T> Point<T> {
    fn swap(&mut self) {
        std::mem::swap(&mut self.x, &mut self.y);
    }
}
```

目前为止你已经见过了支持几乎任何类型的泛型。他们在很多地方都是有用的：你已经见过了`Option<T>`，接下来你还将见到像[`Vec<T>`](http://doc.rust-lang.org/std/vec/struct.Vec.html)这样的通用容器类型。另一方面，通常你想要用灵活性去换取更强的表现力。阅读[trait bound](Traits.md#泛型函数的-trait-bound（trait-bounds-on-generic-functions）)章节来了解为什么和如何做。
