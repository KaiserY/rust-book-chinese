# Traits

> [traits.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/traits.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

trait 是一个告诉 Rust 编译器一个类型必须提供哪些功能语言特性。

你还记得`impl`关键字吗，曾用[方法语法](Method Syntax 方法语法.md)调用方法的那个？

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

trait 也很类似，除了我们用函数标记来定义一个 trait，然后为结构体实现 trait。例如，我们为`Circle`实现`HasArea` trait：

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

trait HasArea {
    fn area(&self) -> f64;
}

impl HasArea for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}
```

如你所见，`trait`块与`impl`看起来很像，不过我们没有定义一个函数体，只是函数标记。当我们`impl`一个trait时，我们使用`impl Trait for Item`，而不是仅仅`impl Item`。

`Self`可以被用在类型标记中表示被作为参数传递的实现了这个 trait 的类型的一个实例。`Self`，`&Self`和`&mut Self`可以根据所需不同级别的所有权来使用。

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

trait HasArea {
    fn area(&self) -> f64;

    fn is_larger(&self, &Self) -> bool;
}

impl HasArea for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }

    fn is_larger(&self, other: &Self) -> bool {
        self.area() > other.area()
    }
}
```

## 泛型函数的 trait bound（Trait bounds on generic functions）

trait 很有用是因为他们允许一个类型对它的行为提供特定的承诺。泛型函数可以显式的限制（或者叫 [bound](Glossary 词汇表.md#界限（bounds）)）它接受的类型。考虑这个函数，它并不能编译：

```rust
fn print_area<T>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}
```

Rust抱怨道：

```text
error: no method named `area` found for type `T` in the current scope
```


因为`T`可以是任何类型，我们不能确定它实现了`area`方法。不过我们可以在泛型`T`添加一个 trait bound，来确保它实现了对应方法：

```rust
# trait HasArea {
#     fn area(&self) -> f64;
# }
fn print_area<T: HasArea>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}
```

`<T: HasArea>`语法是指`any type that implements the HasArea trait`（任何实现了`HasArea`trait的类型）。因为 trait 定义了函数类型标记，我们可以确定任何实现`HasArea`将会拥有一个`.area()`方法。

这是一个扩展的例子演示它如何工作：

```rust
trait HasArea {
    fn area(&self) -> f64;
}

struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl HasArea for Circle {
    fn area(&self) -> f64 {
        std::f64::consts::PI * (self.radius * self.radius)
    }
}

struct Square {
    x: f64,
    y: f64,
    side: f64,
}

impl HasArea for Square {
    fn area(&self) -> f64 {
        self.side * self.side
    }
}

fn print_area<T: HasArea>(shape: T) {
    println!("This shape has an area of {}", shape.area());
}

fn main() {
    let c = Circle {
        x: 0.0f64,
        y: 0.0f64,
        radius: 1.0f64,
    };

    let s = Square {
        x: 0.0f64,
        y: 0.0f64,
        side: 1.0f64,
    };

    print_area(c);
    print_area(s);
}
```

这个程序会输出：

```text
This shape has an area of 3.141593
This shape has an area of 1
```

如你所见，`print_area`现在是泛型的了，并且确保我们传递了正确的类型。如果我们传递了错误的类型：

```rust
print_area(5);
```

我们会得到一个编译时错误：

```text
error: the trait bound `_ : HasArea` is not satisfied [E0277]
```

## 泛型结构体的 trait bound（Trait bounds on generic structs）

泛型结构体也从 trait bound 中获益。所有你需要做的就是在你声明类型参数时附加上 bound。这里有一个新类型`Rectangle<T>`和它的操作`is_square()`：

```rust
struct Rectangle<T> {
    x: T,
    y: T,
    width: T,
    height: T,
}

impl<T: PartialEq> Rectangle<T> {
    fn is_square(&self) -> bool {
        self.width == self.height
    }
}

fn main() {
    let mut r = Rectangle {
        x: 0,
        y: 0,
        width: 47,
        height: 47,
    };

    assert!(r.is_square());

    r.height = 42;
    assert!(!r.is_square());
}
```

`is_square()`需要检查边是相等的，所以边必须是一个实现了[`core::cmp::PartialEq`](http://doc.rust-lang.org/cmp/trait.PartialEq.html) trait 的类型：

```rust
impl<T: PartialEq> Rectangle<T> { ... }
```

现在，一个长方形可以用任何可以比较相等的类型定义了。

这里我们定义了一个新的接受任何精度数字的`Rectangle`结构体——讲道理，很多类型——只要他们能够比较大小。我们可以对`HasArea`结构体，`Square`和`Circle`做同样的事吗？可以，不过他们需要乘法，而要处理它我们需要了解[运算符 trait](Operators and Overloading 运算符和重载.md)更多。

## 实现 trait 的规则（Rules for implementing traits）

目前为止，我们只在结构体上添加 trait 实现，不过你可以为任何类型实现一个 trait，比如`f32`。

```rust
trait ApproxEqual {
    fn approx_equal(&self, other: &Self) -> bool;
}
impl ApproxEqual for f32 {
    fn approx_equal(&self, other: &Self) -> bool {
        // Appropriate for `self` and `other` being close to 1.0.
        (self - other).abs() <= ::std::f32::EPSILON
    }
}

println!("{}", 1.0.approx_equal(&1.00000001));
```

这看起来有点像狂野西部（Wild West），不过这还有两个限制来避免情况失去控制。第一是如果 trait 并不定义在你的作用域，它并不能实现。这是个例子：为了进行文件I/O，标准库提供了一个[`Write`](http://doc.rust-lang.org/nightly/std/io/trait.Write.html)trait来为`File`增加额外的功能。默认，`File`并不会有这个方法：

```rust
let mut f = std::fs::File::create("foo.txt").ok().expect("Couldn’t create foo.txt");
let buf = b"whatever"; // buf: &[u8; 8], a byte string literal.
let result = f.write(buf);
# result.unwrap(); // Ignore the error.
```

这里是错误：

```text
error: type `std::fs::File` does not implement any method in scope named `write`
let result = f.write(buf);
               ^~~~~~~~~~
```

我们需要先`use`这个`Write` trait：

```rust,no_run
use std::io::Write;

let mut f = std::fs::File::create("foo.txt").expect("Couldn’t create foo.txt");
let buf = b"whatever";
let result = f.write(buf);
# result.unwrap(); // Ignore the error.
```

这样就能无错误的编译了。

这意味着即使有人做了像给`i32`增加函数这样的坏事，它也不会影响你，除非你`use`了那个 trait。

这还有一个实现 trait 的限制。要么是 trait 要么是你写实现的类型必须由你定义。更准确的说，它们中的一个必须定义于你编写`impl`的相同的 crate 中。关于 Rust 的模块和包系统，详见[crate 和模块](Crates and Modules crate 和模块.md)。

所以，我们可以为`i32`实现`HasArea`trait，因为`HasArea`在我们的包装箱中。不过如果我们想为`i32`实现`Float`trait，它是由 Rust 提供的，则无法做到，因为这个 trait 和类型都不在我们的包装箱中。



There’s one more restriction on implementing traits: either the trait or the type you’re implementing it for must be defined by you. Or more precisely, one of them must be defined in the same crate as the impl you're writing. For more on Rust's module and package system, see the chapter on crates and modules.

关于trait的最后一点：带有trait限制的泛型函数是**单态**（*monomorphization*）（mono：单一，morph：形式）的，所以它是**静态分发**（*statically dispatched*）的。这是什么意思？查看[trait对象](Trait Objects trait 对象.md)来了解更多细节。

## 多 trait bound（Multiple trait bounds）

你已经见过你可以用一个trait限定一个泛型类型参数：

```rust
fn foo<T: Clone>(x: T) {
    x.clone();
}
```

如果你需要多于1个限定，可以使用`+`：

```rust
use std::fmt::Debug;

fn foo<T: Clone + Debug>(x: T) {
    x.clone();
    println!("{:?}", x);
}
```

`T`现在需要实现`Clone`和`Debug`。

## where 从句（Where clause）

编写只有少量泛型和trait的函数并不算太糟，不过当它们的数量增加，这个语法就看起来比较诡异了：

```rust
use std::fmt::Debug;

fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
}
```

函数的名字在最左边，而参数列表在最右边。限制写在中间。

Rust有一个解决方案，它叫“where 从句”：

```rust
use std::fmt::Debug;

fn foo<T: Clone, K: Clone + Debug>(x: T, y: K) {
    x.clone();
    y.clone();
    println!("{:?}", y);
}

fn bar<T, K>(x: T, y: K) where T: Clone, K: Clone + Debug {
    x.clone();
    y.clone();
    println!("{:?}", y);
}

fn main() {
    foo("Hello", "world");
    bar("Hello", "world");
}
```

`foo()`使用我们刚才的语法，而`bar()`使用`where`从句。所有你所需要做的就是在定义参数时省略限制，然后在参数列表后加上一个`where`。对于很长的列表，你也可以加上空格：

```rust
use std::fmt::Debug;

fn bar<T, K>(x: T, y: K)
    where T: Clone,
          K: Clone + Debug {

    x.clone();
    y.clone();
    println!("{:?}", y);
}
```

这种灵活性可以使复杂情况变得简洁。

`where`也比基本语法更强大。例如：

```rust
trait ConvertTo<Output> {
    fn convert(&self) -> Output;
}

impl ConvertTo<i64> for i32 {
    fn convert(&self) -> i64 { *self as i64 }
}

// Can be called with T == i32.
fn normal<T: ConvertTo<i64>>(x: &T) -> i64 {
    x.convert()
}

// Can be called with T == i64.
fn inverse<T>(x: i32) -> T
        // This is using ConvertTo as if it were "ConvertTo<i64>".
        where i32: ConvertTo<T> {
    x.convert()
}
```

这突显出了`where`从句的额外的功能：它允许限制的左侧可以是任意类型（在这里是`i32`），而不仅仅是一个类型参数（比如`T`）。

## 默认方法（Default methods）

关于trait还有最后一个我们需要讲到的功能。它简单到只需我们展示一个例子：

```rust
trait Foo {
    fn is_valid(&self) -> bool;

    fn is_invalid(&self) -> bool { !self.is_valid() }
}
```

`Foo`trait的实现者需要实现`is_valid()`，不过并不需要实现`is_invalid()`。它会使用默认的行为。你也可以选择覆盖默认行为：

```rust
# trait Foo {
#     fn is_valid(&self) -> bool;
#
#     fn is_invalid(&self) -> bool { !self.is_valid() }
# }
struct UseDefault;

impl Foo for UseDefault {
    fn is_valid(&self) -> bool {
        println!("Called UseDefault.is_valid.");
        true
    }
}

struct OverrideDefault;

impl Foo for OverrideDefault {
    fn is_valid(&self) -> bool {
        println!("Called OverrideDefault.is_valid.");
        true
    }

    fn is_invalid(&self) -> bool {
        println!("Called OverrideDefault.is_invalid!");
        true // Overrides the expected value of is_invalid()
    }
}

let default = UseDefault;
assert!(!default.is_invalid()); // Prints "Called UseDefault.is_valid."

let over = OverrideDefault;
assert!(over.is_invalid()); // Prints "Called OverrideDefault.is_valid."
```

## 继承（Inheritance）
有时，实现一个trait要求实现另一个trait：

```rust
trait Foo {
    fn foo(&self);
}

trait FooBar : Foo {
    fn foobar(&self);
}
```

`FooBar`的实现也必须实现`Foo`，像这样：

```rust
# trait Foo {
#     fn foo(&self);
# }
# trait FooBar : Foo {
#     fn foobar(&self);
# }
struct Baz;

impl Foo for Baz {
    fn foo(&self) { println!("foo"); }
}

impl FooBar for Baz {
    fn foobar(&self) { println!("foobar"); }
}
```

如果我们忘了实现`Foo`，Rust会告诉我们：

```text
error: the trait bound `main::Baz : main::Foo` is not satisfied [E0277]
```

## Deriving

重复的实现像`Debug`和`Default`这样的 trait 会变得很无趣。为此，Rust 提供了一个[属性](Attributes 属性.md)来允许我们让 Rust 为我们自动实现 trait：

```rust
#[derive(Debug)]
struct Foo;

fn main() {
    println!("{:?}", Foo);
}
```

然而，deriving 限制为一些特定的 trait：

* [Clone](http://doc.rust-lang.org/core/clone/trait.Clone.html)
* [Copy](http://doc.rust-lang.org/core/marker/trait.Copy.html)
* [Debug](http://doc.rust-lang.org/core/fmt/trait.Debug.html)
* [Default](http://doc.rust-lang.org/core/default/trait.Default.html)
* [Eq](http://doc.rust-lang.org/core/cmp/trait.Eq.html)
* [Hash](http://doc.rust-lang.org/core/hash/trait.Hash.html)
* [Ord](http://doc.rust-lang.org/core/cmp/trait.Ord.html)
* [PartialEq](http://doc.rust-lang.org/core/cmp/trait.PartialEq.html)
* [PartialOrd](http://doc.rust-lang.org/core/cmp/trait.PartialOrd.html)
