# 枚举

> [enums.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/enums.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

Rust 中的一个`enum`是一个代表数个可能变量的数据的类型。每个变量都可选是否关联数据：

```rust
enum Message {
    Quit,
    ChangeColor(i32, i32, i32),
    Move { x: i32, y: i32 },
    Write(String),
}
```

定义变量的语法与用来定义结构体的语法类似：你可以有不带数据的变量（像类单元结构体），带有命名数据的变量，和带有未命名数据的变量（像元组结构体）。然而，不像单独的结构体定义，一个`enum`是一个单独的类型。一个枚举的值可以匹配任何一个变量。因为这个原因，枚举有时被叫做“集合类型”：枚举可能值的集合是每一个变量可能值的集合的总和。

我们使用`::`语法来使用每个变量的名字：它们包含在`enum`名字自身中。这样的话，以下的情况都是可行的：

```rust
# enum Message {
#     Move { x: i32, y: i32 },
# }
let x: Message = Message::Move { x: 3, y: 4 };

enum BoardGameTurn {
    Move { squares: i32 },
    Pass,
}

let y: BoardGameTurn = BoardGameTurn::Move { squares: 1 };
```

这两个变量都叫做`Move`，不过他们包含在枚举名字中，他们可以无冲突的使用。

枚举类型的一个值包含它是哪个变量的信息，以及任何与变量相关的数据。这有时被作为一个“标记的联合”被提及。因为数据包括一个“标签”表明它的类型是什么。编译器使用这个信息来确保安全的访问枚举中的数据。例如，我们不能简单的尝试解构一个枚举值，就像它是其中一个可能的变体那样：

```rust,ignore
fn process_color_change(msg: Message) {
    let Message::ChangeColor(r, g, b) = msg; // This causes a compile-time error.
}
```

不支持这些操作（比较操作）可能看起来更像限制。不过这是一个我们可以克服的限制。有两种方法：我们自己实现相等（比较），或通过[`match` ](Match 匹配.md)表达式模式匹配变量，你会在下一部分学到它。我们还不够了解Rust如何实现相等，不过我们会在[特性](Traits.md)找到它们。

## 构造器作为函数（Constructors as functions）
一个枚举的构造器总是可以像函数一样使用。例如：

```rust
# enum Message {
# Write(String),
# }
let m = Message::Write("Hello, world".to_string());
```

与下面是一样的：

```rust
# enum Message {
# Write(String),
# }
fn foo(x: String) -> Message {
    Message::Write(x)
}

let x = foo("Hello, world".to_string());
```

这对我们没有什么直接的帮助，直到我们要用到[闭包](Closures 闭包.md)时，这时我们要考虑将函数作为参数传递给其他函数。例如，使用[迭代器](Iterators 迭代器.md)，我们可以这样把一个`String`的vector转换为一个`Message::Write`的vector：

```rust
# enum Message {
# Write(String),
# }

let v = vec!["Hello".to_string(), "World".to_string()];

let v1: Vec<Message> = v.into_iter().map(Message::Write).collect();
```
