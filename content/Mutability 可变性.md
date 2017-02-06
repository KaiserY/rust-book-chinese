# 可变性

> [mutability.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/mutability.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

可变性，可以改变事物的能力，用在Rust中与其它语言有些许不同。可变性的第一方面是它并非默认状态：

```rust
let x = 5;
x = 6; // Error!
```

我们可以使用`mut`关键字来引入可变性：

```rust
let mut x = 5;

x = 6; // No problem!
```

这是一个可变的[变量绑定](Variable Bindings 变量绑定.md)。当一个绑定是可变的，它意味着你可以改变它指向的内容。所以在上面的例子中，`x`的值并没有多大的变化，不过这个绑定从一个`i32`变成了另外一个。

你也可以使用`&x`创建一个它的[引用](References and Borrowing 引用和借用.md)，不过如果你想要使用这个引用来改变它的值，你将会需要一个[可变引用](References and Borrowing 引用和借用.md)：

```rust
let mut x = 5;
let y = &mut x;
```

`y`是一个（指向）可变引用的不可变绑定，它意味着你不能把`y`与其它变量绑定（`y = &mut z`），不过`y`可以用来把`x`绑定到别的值上（`*y = 5`）。一个微妙的区别。

当然，如果你想它们都可变：

```rust
let mut x = 5;
let mut y = &mut x;
```

现在`y`可以绑定到另外一个值，并且它引用的值也可以改变。

很重要的一点是`mut`是[模式](Patterns 模式.md)的一部分，所以你可以这样做：

```rust
let (mut x, y) = (5, 6);

fn foo(mut x: i32) {
# }
```

注意这里`x`是可变的，`y`不是。

## 内部可变性 VS 外部可变性（Interior vs. Exterior Mutability）
然而，当我们谈到 Rust 中什么是“不可变”的时候，它并不意味着它不能被改变：这里是指它的“外部可变性”是不可变的。例如，考虑下[`Arc<T>`](http://doc.rust-lang.org/nightly/std/sync/struct.Arc.html)：

```rust
use std::sync::Arc;

let x = Arc::new(5);
let y = x.clone();
```

当我们调用`clone()`时，`Arc<T>`需要更新引用计数。然而你并未使用任何`mut`，`x`是一个不可变绑定，并且我们也没有取得`&mut 5`或者什么。那么发生了什么呢？

为了解释这些，我们不得不回到Rust指导哲学的核心，内存安全，和Rust用以保证它的机制，[所有权](Ownership 所有权.md)系统，和更具体的[借用](Borrow and AsRef Borrow 和 AsRef.md#borrow)：

> 你可以拥有这两种类型借用的其中一个，但不能同时拥有：

> * 拥有 1 个或多个不可变引用（&T）
> * 只有 1 个可变引用（&mut T）

因此，这就是“不可变性”的真正定义：当有两个引用指向同一事物是安全的吗？在`Arc<T>`的情况下，是安全的：改变完全包含在结构自身内部。它并不面向用户。为此，它用`clone()`分配`&T`。如果分配`&mut T`的话，那么，这将会是一个问题。

其它类型，像[std::cell](http://doc.rust-lang.org/nightly/std/cell/)模块中的这一个，则有相反的属性：内部可变性。例如：

```rust
use std::cell::RefCell;

let x = RefCell::new(42);

let y = x.borrow_mut();
```

`RefCell`使用`borrow_mut()`方法来分配它内部资源的`&mut`引用。这难道不危险吗？如果我们：

```rust
use std::cell::RefCell;

let x = RefCell::new(42);

let y = x.borrow_mut();
let z = x.borrow_mut();
# (y, z);
```

事实上这会在运行时引起恐慌。`RefCell`是这样工作的：它在运行时强制使用Rust的借用规则，并且如果有违反就会`panic!`。这让我们绕开了Rust可变性规则的另一方面。让我们先讨论一下它。

## 字段级别可变性（Field-level mutability）
可变性是借用（`&mut`）或者绑定（`let mut`）的属性之一。这意味着，例如，你不能让一个[结构体](Structs 结构体.md)的一些字段可变而另一些字段不可变：

```rust
struct Point {
    x: i32,
    mut y: i32, // Nope.
}
```

结构体的可变性位于它的绑定上：

```rust
struct Point {
    x: i32,
    y: i32,
}

let mut a = Point { x: 5, y: 6 };

a.x = 10;

let b = Point { x: 5, y: 6};

b.x = 10; // Error: cannot assign to immutable field `b.x`
```

然而，通过使用`Cell<T>`，你可以模拟字段级别的可变性：

```rust
use std::cell::Cell;

struct Point {
    x: i32,
    y: Cell<i32>,
}

let point = Point { x: 5, y: Cell::new(6) };

point.y.set(7);

println!("y: {:?}", point.y);
```

这会打印`y: Cell { value: 7 }`。我们成功的更新了`y`。
