# 装箱语法和模式

> [box-syntax-and-patterns.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/box-syntax-and-patterns.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

目前唯一稳定的创建`Box`的方法是通过`Box::new`方法。并且不可能在一个模式匹配中稳定的析构一个`Box`。不稳定的`box`关键字可以用来创建和析构`Box`。下面是一个用例：

```rust
#![feature(box_syntax, box_patterns)]

fn main() {
    let b = Some(box 5);
    match b {
        Some(box n) if n < 0 => {
            println!("Box contains negative number {}", n);
        },
        Some(box n) if n >= 0 => {
            println!("Box contains non-negative number {}", n);
        },
        None => {
            println!("No box");
        },
        _ => unreachable!()
    }
}
```

注意这些功能目前隐藏在`box_syntax`（装箱创建）和`box_patterns`（析构和模式匹配）gate 之后因为它的语法在未来可能会改变。

## 返回指针

在很多有指针的语言中，你的函数可以返回一个指针来避免拷贝大的数据结构。例如：

```rust
struct BigStruct {
    one: i32,
    two: i32,
    // Etc.
    one_hundred: i32,
}

fn foo(x: Box<BigStruct>) -> Box<BigStruct> {
    Box::new(*x)
}

fn main() {
    let x = Box::new(BigStruct {
        one: 1,
        two: 2,
        one_hundred: 100,
    });

    let y = foo(x);
}
```

要点是通过传递一个装箱，你只需拷贝了一个指针，而不是那构成了`BigStruct`的一百个`int`值。

上面是 Rust 中的一个反模式。相反，这样写：

```rust
#![feature(box_syntax)]

struct BigStruct {
    one: i32,
    two: i32,
    // Etc.
    one_hundred: i32,
}

fn foo(x: Box<BigStruct>) -> BigStruct {
    *x
}

fn main() {
    let x = Box::new(BigStruct {
        one: 1,
        two: 2,
        one_hundred: 100,
    });

    let y: Box<BigStruct> = box foo(x);
}
```

这在不牺牲性能的前提下获得了灵活性。

你可能会认为这会给我们带来很差的性能：返回一个值然后马上把它装箱？难道这在哪里不都是最糟的吗？Rust 显得更聪明。这里并没有拷贝。`main`为装箱分配了足够的空间，向`foo`传递一个指向他内存的`x`，然后`foo`直接向`Box<T>`中写入数据。

因为这很重要所以要说两遍：返回指针会阻止编译器优化你的代码。允许调用函数选择它们需要如何使用你的输出。
