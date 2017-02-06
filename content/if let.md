# if let

> [if-let.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/if-let.md)
> <br>
> commit 797a0bd1c13175398aa0e2e45f6dbb61bcb8c329

`if let`允许你合并`if`和`let`来减少特定类型模式匹配的开销。

例如，让我们假设我们有一些`Option<T>`。我们想让它是`Some<T>`时在其上调用一个函数，而它是`None`时什么也不做。这看起来像：

```rust
# let option = Some(5);
# fn foo(x: i32) { }
match option {
    Some(x) => { foo(x) },
    None => {},
}
```

我们并不一定要在这使用`match`，例如，我们可以使用`if`：

```rust
# let option = Some(5);
# fn foo(x: i32) { }
if option.is_some() {
    let x = option.unwrap();
    foo(x);
}
```

这两种选项都不是特别吸引人。我们可以使用`if let`来优雅地完成相同的功能：

```rust
# let option = Some(5);
# fn foo(x: i32) { }
if let Some(x) = option {
    foo(x);
}
```

如果一个[模式](Patterns 模式.md)匹配成功，它绑定任何值的合适的部分到模式的标识符中，并计算这个表达式。如果模式不匹配，啥也不会发生。

如果你想在模式不匹配时做点其他的，你可以使用`else`：

```rust
# let option = Some(5);
# fn foo(x: i32) { }
# fn bar() { }
if let Some(x) = option {
    foo(x);
} else {
    bar();
}
```

## `while let`
类似的，当你想一直循环，直到一个值匹配到特定的模式的时候，你可以选择使用`while let`。使用`while let`可以把类似这样的代码：

```rust
let mut v = vec![1, 3, 5, 7, 11];
loop {
    match v.pop() {
        Some(x) =>  println!("{}", x),
        None => break,
    }
}
```

变成这样的代码：

```rust
let mut v = vec![1, 3, 5, 7, 11];
while let Some(x) = v.pop() {
    println!("{}", x);
}
```
