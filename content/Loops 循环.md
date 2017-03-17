# 循环

> [loops.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/loops.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

Rust 目前提供 3 种方法来进行一些迭代操作。他们是`loop`，`while`和`for`。每种方法都有自己的用途。

## loop

无限`loop`是 Rust 提供的最简单的循环。使用`loop`关键字，Rust 提供了一个直到一些终止语句被执行的循环方法。Rust 的无限`loop`看起来像这样：

```rust
loop {
    println!("Loop forever!");
}
```

## while

Rust 也有一个`while`循环。它看起来像：

```rust
let mut x = 5; // mut x: i32
let mut done = false; // mut done: bool

while !done {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 {
        done = true;
    }
}
```

`while`循环是当你不确定应该循环多少次时正确的选择。

如果你需要一个无限循环，你可能想要这么写：

```rust
while true {
```

然而，`loop`远比它适合处理这个情况：

```rust
loop {
```

Rust 的控制流分析会区别对待这个与`while true`，因为我们知道它会一直循环。现阶段理解这些细节**意味着**什么并不是非常重要，基本上，你给编译器越多的信息，越能确保安全和生成更好的代码，所以当你打算无限循环的时候应该总是倾向于使用`loop`。

## for

`for`用来循环一个特定的次数。然而，Rust的`for`循环与其它系统语言有些许不同。Rust的`for`循环看起来并不像这个“C语言样式”的`for`循环：

```c
for (x = 0; x < 10; x++) {
    printf( "%d\n", x );
}
```

相反，它看起来像这个样子：

```rust
for x in 0..10 {
    println!("{}", x); // x: i32
}
```

更抽象的形式：

```rust
for var in expression {
    code
}
```

这个表达式是一个[迭代器](Iterators 迭代器.md).迭代器返回一系列的元素。每个元素是循环中的一次重复。然后它的值与`var`绑定，它在循环体中有效。每当循环体执行完后，我们从迭代器中取出下一个值，然后我们再重复一遍。当迭代器中不再有值时，`for`循环结束。

在我们的例子中，`0..10`表达式取一个开始和结束的位置，然后给出一个含有这之间值得迭代器。当然它不包括上限值，所以我们的循环会打印`0`到`9`，而不是到`10`。

Rust 没有使用“C语言风格”的`for`循环是有意为之的。即使对于有经验的 C 语言开发者来说，要手动控制要循环的每个元素也都是复杂并且易于出错的。

## Enumerate方法
当你需要记录你已经循环了多少次了的时候，你可以使用`.enumerate()`函数。

### 对范围（On ranges）：

```rust
for (index, value) in (5..10).enumerate() {
    println!("index = {} and value = {}", index, value);
}
```

输出：

```text
index = 0 and value = 5
index = 1 and value = 6
index = 2 and value = 7
index = 3 and value = 8
index = 4 and value = 9
```

别忘了在范围外面加上括号。

### 对迭代器（On iterators）:

```rust
let lines = "hello\nworld".lines();

for (linenumber, line) in lines.enumerate() {
    println!("{}: {}", linenumber, line);
}
```

输出：

```text
0: hello
1: world
```
## 提早结束迭代（Ending iteration early）
让我们再看一眼之前的`while`循环：

```rust
let mut x = 5;
let mut done = false;

while !done {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 {
        done = true;
    }
}
```

我们必须使用一个`mut`布尔型变量绑定，`done`,来确定何时我们应该推出循环。Rust 有两个关键字帮助我们来修改迭代：`break`和`continue`。

这样，我们可以用`break`来写一个更好的循环：

```rust
let mut x = 5;

loop {
    x += x - 3;

    println!("{}", x);

    if x % 5 == 0 { break; }
}
```

现在我们用`loop`来无限循环，然后用`break`来提前退出循环。

`continue`比较类似，不过不是退出循环，它直接进行下一次迭代。下面的例子只会打印奇数：

```rust
for x in 0..10 {
    if x % 2 == 0 { continue; }

    println!("{}", x);
}
```

`break`和`continue`在`while`循环和[`for`循环](#for)中都有效。

## 循环标签（Loop labels）
你也许会遇到这样的情形，当你有嵌套的循环而希望指定你的哪一个`break`或`continue`该起作用。就像大多数语言，默认`break`或`continue`将会作用于最内层的循环。当你想要一个`break`或`continue`作用于一个外层循环，你可以使用标签来指定你的`break`或`continue`语句作用的循环。如下代码只会在`x`和`y`都为奇数时打印他们：

```rust
'outer: for x in 0..10 {
    'inner: for y in 0..10 {
        if x % 2 == 0 { continue 'outer; } // Continues the loop over `x`.
        if y % 2 == 0 { continue 'inner; } // Continues the loop over `y`.
        println!("x: {}, y: {}", x, y);
    }
}
```
