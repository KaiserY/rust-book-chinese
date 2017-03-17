# 匹配

> [match.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/match.md)
> <br>
> commit e7d60c00c68e72fa954bcff311bc92e1c445823c

一个简单的[`if`](If If语句.md)/`else`往往是不够的，因为你可能有两个或更多个选项。这样`else`也会变得异常复杂。Rust 有一个`match`关键字，它可以让你有效的取代复杂的`if`/`else`组。看看下面的代码：

```rust
let x = 5;

match x {
    1 => println!("one"),
    2 => println!("two"),
    3 => println!("three"),
    4 => println!("four"),
    5 => println!("five"),
    _ => println!("something else"),
}
```

`match`使用一个表达式然后基于它的值分支。每个分支都是`val => expression`这种形式。当匹配到一个分支，它的表达式将被执行。`match`属于“模式匹配”的范畴，`match`是它的一个实现。有[一个整个关于模式的部分](Patterns 模式.md)讲到了所有可能的模式。

那么这有什么巨大的优势呢？这确实有优势。第一，`match`强制**穷尽性检查**（*exhaustiveness checking*）。你看到了最后那个下划线开头的分支了吗？如果去掉它，Rust 将会给我们一个错误：

```text
error: non-exhaustive patterns: `_` not covered
```

Rust 试图告诉我们忘记了一个值。编译器从`x`推断它可以是任何 32 位整型值；例如从 -2,147,483,648 到 2,147,483,647。`_`就像一个**匹配所有**的分支，它会捕获所有没有被`match`分支捕获的所有可能值。如你所见，在上个例子中，我们提供了 1 到 5 的`mtach`分支，如果`x`是 6 或者其他值，那么它会被`_`捕获。

`match`也是一个表达式，也就是说它可以用在`let`绑定的右侧或者其它直接用到表达式的地方：

```rust
let x = 5;

let number = match x {
    1 => "one",
    2 => "two",
    3 => "three",
    4 => "four",
    5 => "five",
    _ => "something else",
};
```

有时，这是一个把一种类型的数据转换为另一个类型的好方法。

## 匹配枚举（Matching on enums）
`match`的另一个重要的作用是处理枚举的可能变量：

```rust
enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
}

fn quit() { /* ... */ }
fn change_color(r: i32, g: i32, b: i32) { /* ... */ }
fn move_cursor(x: i32, y: i32) { /* ... */ }

fn process_message(msg: Message) {
    match msg {
        Message::Quit => quit(),
        Message::ChangeColor(r, g, b) => change_color(r, g, b),
        Message::Move { x, y: new_name_for_y } => move_cursor(x, new_name_for_y),
        Message::Write(s) => println!("{}", s),
    };
}
```

再一次，Rust编译器检查穷尽性，所以它要求对每一个枚举的变量都有一个匹配分支。如果你忽略了一个，除非你用`_`否则它会给你一个编译时错误。

与之前的`match`的作用不同，你不能用常规的`if`语句来做这些。你可以使用[if let](if let.md)语句，它可以被看作是一个`match`的简略形式。
