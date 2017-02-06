# 方法语法

> [method-syntax.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/method-syntax.md)
> <br>
> commit 6ba952020fbc91bad64be1ea0650bfba52e6aab4

函数是伟大的，不过如果你在一些数据上调用了一堆函数，这将是令人尴尬的。
考虑下面代码：

```rust
baz(bar(foo));
```

我们可以从左向右阅读，我们会看到“baz bar foo”。不过这不是函数被调用的顺序，调用应该是从内向外的：“foo bar baz”。如果能这么做不是更好吗？

```rust
foo.bar().baz();
```

幸运的是，正如对上面那个问题的猜测，你可以！Rust 通过`impl`关键字提供了使用**方法调用语法**（*method call syntax*）。

## 方法调用

这是它如何工作的：

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

fn main() {
    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
    println!("{}", c.area());
}
```

这会打印`12.566371`。

我们创建了一个代表圆的结构体。我们写了一个`impl`块，并且在里面定义了一个方法，`area`。

方法的第一参数比较特殊，`&self`。它有3种变体：`self`，`&self`和`&mut self`。你可以认为这第一个参数就是`x.foo()`中的`x`。这3种变体对应`x`可能的3种类型：`self`如果它只是栈上的一个值，`&self`如果它是一个引用，然后`&mut self`如果它是一个可变引用。因为我们的`area`以`&self`作为参数，我们就可以可以像其他参数那样使用它。因为我们知道是一个`Circle`，我们可以像任何其他结构体那样访问`radius`字段。

我们应该默认使用`&self`，就像相比获取所有权你应该更倾向于借用，同样相比获取可变引用更倾向于不可变引用一样。这是一个三种变体的例子：

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn reference(&self) {
       println!("taking self by reference!");
    }

    fn mutable_reference(&mut self) {
       println!("taking self by mutable reference!");
    }

    fn takes_ownership(self) {
       println!("taking ownership of self!");
    }
}
```

你可以有任意多个`impl`块。上面的例子也可以被写成这样：

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn reference(&self) {
       println!("taking self by reference!");
    }
}

impl Circle {
    fn mutable_reference(&mut self) {
       println!("taking self by mutable reference!");
    }
}

impl Circle {
    fn takes_ownership(self) {
       println!("taking ownership of self!");
    }
}
```

## 链式方法调用（Chaining method calls）
现在我们知道如何调用方法了，例如`foo.bar()`。那么我们最开始的那个例子呢，`foo.bar().baz()`？我们称这个为“方法链”，我们可以通过返回`self`来做到这点。

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

    fn grow(&self, increment: f64) -> Circle {
        Circle { x: self.x, y: self.y, radius: self.radius + increment }
    }
}

fn main() {
    let c = Circle { x: 0.0, y: 0.0, radius: 2.0 };
    println!("{}", c.area());

    let d = c.grow(2.0).area();
    println!("{}", d);
}
```

注意返回值：

```rust
# struct Circle;
# impl Circle {
fn grow(&self, increment: f64) -> Circle {
# Circle } }
```

我们看到我们返回了一个`Circle`。通过这个函数，我们可以增长一个圆的面积到任意大小。

## 关联函数（Associated functions）
我们也可以定义一个不带`self`参数的关联函数。这是一个Rust代码中非常常见的模式：

```rust
struct Circle {
    x: f64,
    y: f64,
    radius: f64,
}

impl Circle {
    fn new(x: f64, y: f64, radius: f64) -> Circle {
        Circle {
            x: x,
            y: y,
            radius: radius,
        }
    }
}

fn main() {
    let c = Circle::new(0.0, 0.0, 2.0);
}
```

这个**关联函数**（*associated function*）为我们构建了一个新的`Circle`。注意静态函数是通过`Struct::method()`语法调用的，而不是`ref.method()`语法。

## 创建者模式（Builder Pattern）
我们说我们需要我们的用户可以创建圆，不过我们只允许他们设置他们关心的属性。否则，`x`和`y`将是`0.0`，并且`radius`将是`1.0`。Rust 并没有方法重载，命名参数或者可变参数。我们利用创建者模式来代替。它看起像这样：

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

struct CircleBuilder {
    x: f64,
    y: f64,
    radius: f64,
}

impl CircleBuilder {
    fn new() -> CircleBuilder {
        CircleBuilder { x: 0.0, y: 0.0, radius: 1.0, }
    }

    fn x(&mut self, coordinate: f64) -> &mut CircleBuilder {
        self.x = coordinate;
        self
    }

    fn y(&mut self, coordinate: f64) -> &mut CircleBuilder {
        self.y = coordinate;
        self
    }

    fn radius(&mut self, radius: f64) -> &mut CircleBuilder {
        self.radius = radius;
        self
    }

    fn finalize(&self) -> Circle {
        Circle { x: self.x, y: self.y, radius: self.radius }
    }
}

fn main() {
    let c = CircleBuilder::new()
                .x(1.0)
                .y(2.0)
                .radius(2.0)
                .finalize();

    println!("area: {}", c.area());
    println!("x: {}", c.x);
    println!("y: {}", c.y);
}
```

我们在这里又声明了一个结构体，`CircleBuilder`。我们给它定义了一个创建者函数。我们也在`Circle`中定义了`area()`方法。我们还定义了另一个方法`CircleBuilder: finalize()`。这个方法从构造器中创建了我们最后的`Circle`。现在我们使用类型系统来强化我们的考虑：我们可以用`CircleBuilder`来强制生成我们需要的`Circle`。
