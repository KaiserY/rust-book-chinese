# 运算符与重载

> [operators-and-overloading.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/operators-and-overloading.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

Rust 允许有限形式的运算符重载。特定的运算符可以被重载。要支持一个类型间特定的运算符，你可以实现一个的特定的重载运算符的trait。

例如，`+`运算符可以通过`Add`特性重载：

```rust
use std::ops::Add;

#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;

    fn add(self, other: Point) -> Point {
        Point { x: self.x + other.x, y: self.y + other.y }
    }
}

fn main() {
    let p1 = Point { x: 1, y: 0 };
    let p2 = Point { x: 2, y: 3 };

    let p3 = p1 + p2;

    println!("{:?}", p3);
}
```

在`main`中，我们可以对我们的两个`Point`用`+`号，因为我们已经为`Point`实现了`Add<Output=Point>`。

有一系列可以这样被重载的运算符，并且所有与之相关的trait都在[`std::ops`](http://doc.rust-lang.org/stable/std/ops/)模块中。查看它的文档来获取完整的列表。

实现这些特性要遵循一个模式。让我们仔细看看[`Add`](http://doc.rust-lang.org/stable/std/ops/trait.Add.html)：

```rust
# mod foo {
pub trait Add<RHS = Self> {
    type Output;

    fn add(self, rhs: RHS) -> Self::Output;
}
# }
```

这里总共涉及到3个类型：你`impl Add`的类型，`RHS`，它默认是`Self`，和`Output`。对于一个表达式`let z = x + y`，`x`是`Self`类型的，`y`是`RHS`，而`z`是`Self::Output`类型。

```rust
# struct Point;
# use std::ops::Add;
impl Add<i32> for Point {
    type Output = f64;

    fn add(self, rhs: i32) -> f64 {
        // Add an i32 to a Point and get an f64.
# 1.0
    }
}
```

将允许你这样做：

```rust
let p: Point = // ...
let x: f64 = p + 2i32;
```

## 在泛型结构体中使用运算符 trait

现在我们知道了运算符 trait 是如何定义的了，我们可以更通用的定义来自[trait 章节]()的`HasArea` trait 和`Square`结构体：

```rust
use std::ops::Mul;

trait HasArea<T> {
    fn area(&self) -> T;
}

struct Square<T> {
    x: T,
    y: T,
    side: T,
}

impl<T> HasArea<T> for Square<T>
        where T: Mul<Output=T> + Copy {
    fn area(&self) -> T {
        self.side * self.side
    }
}

fn main() {
    let s = Square {
        x: 0.0f64,
        y: 0.0f64,
        side: 12.0f64,
    };

    println!("Area of s: {}", s.area());
}
```

对于`HasArea`和`Square`，我们声明了一个类型参数`T`并取代`f64`。`impl`则需要更深入的修改：

```rust
impl<T> HasArea<T> for Square<T>
        where T: Mul<Output=T> + Copy { ... }
```

`area`方法要求我们可以进行边的乘法，所以我们声明的`T`类型必须实现`std::ops::Mul`。比如上面提到的`Add`，`Mul`自身获取一个`Output`参数：因为我们知道相乘时数字并不会改变类型，我也设定它为`T`。`T`也必须支持拷贝，所以 Rust 并不尝试将`self.side`移动进返回值。
