# 模式

> [patterns.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/patterns.md)
> <br>
> commit 6ba952020fbc91bad64be1ea0650bfba52e6aab4

模式在Rust中十分常见。我们在[变量绑定](http://doc.rust-lang.org/nightly/book/variable-bindings.html)，[匹配语句](http://doc.rust-lang.org/nightly/book/match.html)和其它一些地方使用它们。让我们开始一个快速的关于模式可以干什么的教程！

快速回顾：你可以直接匹配常量，并且`_`作为“任何”类型：

```rust
let x = 1;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

这会打印出`one`。

这里有一个模式的陷阱：就像任何引入一个新绑定的语句，他们会引入隐藏。例如：

```rust
let x = 1;
let c = 'c';

match c {
    x => println!("x: {} c: {}", x, c),
}

println!("x: {}", x)
```

这会打印：

```text
x: c c: c
x: 1
```

换句话说，`x =>`匹配到了模式并引入了一个叫做`x`的新绑定。这个新绑定的作用域是匹配分支并拥有`c`的值。注意匹配作用域外的`x`的值对内部的`x`的值并无影响。因为我们已经有了一个`x`，新的`x`隐藏了它。

## 多重模式（Multiple patterns）

你可以使用`|`匹配多个模式：

```rust
let x = 1;

match x {
    1 | 2 => println!("one or two"),
    3 => println!("three"),
    _ => println!("anything"),
}
```

这会输出`one or two`。

## 解构（Destructuring）
如果你有一个复合数据类型，例如一个[结构体](http://doc.rust-lang.org/stable/book/structs.html)，你可以在模式中解构它：

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { x, y } => println!("({},{})", x, y),
}
```

我们可以用`:`来给出一个不同的名字：

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { x: x1, y: y1 } => println!("({},{})", x1, y1),
}
```

如果你只关心部分值，我们不需要给它们都命名：

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { x, .. } => println!("x is {}", x),
}
```

这会输出`x is 0`。

你可以对任何成员进行这样的匹配，不仅仅是第一个：

```rust
struct Point {
    x: i32,
    y: i32,
}

let origin = Point { x: 0, y: 0 };

match origin {
    Point { y, .. } => println!("y is {}", y),
}
```

这会输出`y is 0`。

这种“解构”行为可以用在任何复合数据类型上，例如[元组](https://doc.rust-lang.org/stable/book/primitive-types.html#tuples)和[枚举](https://doc.rust-lang.org/stable/book/enums.html)

## 忽略绑定（Ignoring bindings）

你可以在模式中使用`_`来忽视它的类型和值。例如，这是一个`Result<T, E>`的`match`：

```rust
# let some_value: Result<i32, &'static str> = Err("There was an error");
match some_value {
    Ok(value) => println!("got a value: {}", value),
    Err(_) => println!("an error occurred"),
}
```

在第一个分支，我们绑定了`Ok`变量中的值为`value`，不过在`Err`分支，我们用`_`来忽视特定的错误，而只是打印了一个通用的错误信息。

`_`在任何创建绑定的模式中都有效。这在忽略一个大大结构体的部分字段时很有用：

```rust
fn coordinate() -> (i32, i32, i32) {
    // generate and return some sort of triple tuple
# (1, 2, 3)
}

let (x, _, z) = coordinate();
```

这里，我们绑定元组第一个和最后一个元素为`x`和`z`，不过省略了中间的元素。

相似的，你可以在模式中用`..`来忽略多个值。

```rust
enum OptionalTuple {
    Value(i32, i32, i32),
    Missing,
}

let x = OptionalTuple::Value(5, -2, 3);

match x {
    OptionalTuple::Value(..) => println!("Got a tuple!"),
    OptionalTuple::Missing => println!("No such luck."),
}
```

这会打印`Got a tuple!`。

## `ref`和`ref mut`
如果你想要一个引用，使用`ref`关键字：

```rust
let x = 5;

match x {
    ref r => println!("Got a reference to {}", r),
}
```

这会输出`Got a reference to 5`。

这里，`match`中的`r`是`&i32`类型的。换句话说，`ref`关键字创建了一个在模式中使用的引用。如果你需要一个可变引用，`ref mut`同样可以做到：

```rust
let mut x = 5;

match x {
    ref mut mr => println!("Got a mutable reference to {}", mr),
}
```

## 范围（Ranges）

你可以用`...`匹配一个范围的值：

```rust
let x = 1;

match x {
    1 ... 5 => println!("one through five"),
    _ => println!("anything"),
}
```

这会输出`one through five`。

范围经常用在整数和`char`上。

```rust
let x = '💅';

match x {
    'a' ... 'j' => println!("early letter"),
    'k' ... 'z' => println!("late letter"),
    _ => println!("something else"),
}
```

这会输出`something else`。

## 绑定

你可以使用`@`把值绑定到名字上：

```rust
let x = 1;

match x {
    e @ 1 ... 5 => println!("got a range element {}", e),
    _ => println!("anything"),
}
```

这会输出`got a range element 1`。在你想对一个复杂数据结构进行部分匹配的时候，这个特性十分有用：

```rust
#[derive(Debug)]
struct Person {
    name: Option<String>,
}

let name = "Steve".to_string();
let mut x: Option<Person> = Some(Person { name: Some(name) });
match x {
    Some(Person { name: ref a @ Some(_), .. }) => println!("{:?}", a),
    _ => {}
}
```

这会输出 `Some("Steve")`，因为我们把Person里面的`name`绑定到`a`。

如果你在使用`|`的同时也使用了`@`，你需要确保名字在每个模式的每一部分都绑定名字：

```rust
let x = 5;

match x {
    e @ 1 ... 5 | e @ 8 ... 10 => println!("got a range element {}", e),
    _ => println!("anything"),
}
```

## 守卫（Guards）
你可以用`if`来引入*匹配守卫*（*match guards*）：

```rust
enum OptionalInt {
    Value(i32),
    Missing,
}

let x = OptionalInt::Value(5);

match x {
    OptionalInt::Value(i) if i > 5 => println!("Got an int bigger than five!"),
    OptionalInt::Value(..) => println!("Got an int!"),
    OptionalInt::Missing => println!("No such luck."),
}
```

这会输出`Got an int!`。

如果你在`if`中使用多重模式，`if`条件将适用于所有模式：

```rust
let x = 4;
let y = false;

match x {
    4 | 5 if y => println!("yes"),
    _ => println!("no"),
}
```

这会打印`no`，因为`if`适用于整个` 4 | 5`，而不仅仅是`5`，换句话说，`if`语句的优先级是这样的：

```text
(4 | 5) if y => ...
```

而不是这样：

```text
4 | (5 if y) => ...
```

## 混合与匹配（Mix and Match）
(口哨)！根据你的需求，你可以对上面的多种匹配方法进行组合：

```rust,ignore
match x {
    Foo { x: Some(ref name), y: None } => ...
}
```

模式十分强大。好好使用它们。
