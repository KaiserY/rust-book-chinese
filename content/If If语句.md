# If 语句

> [if.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/if.md)
> <br>
> commit 56ab2a1cdefec1ddee00ee99d3fb710906714d1b

Rust 的 `if` 并不是特别复杂，不过你会发现它更像动态类型语言而不是更传统的系统语言。所以让我来说说它，以便你能把握这些细节。

`if` 语句是**分支**这个更加宽泛的概念的一个特定形式。它的名字来源于树的树枝：一个选择点，根据选择的不同，将会使用不同的路径。

在 `if` 语句中，有一个引向两条路径的选择：

```rust
let x = 5;

if x == 5 {
    println!("x is five!");
}
```

如果在什么别的地方更改了`x`的值，这一行将不会输出。更具体一点，如果`if`后面的表达式的值为`true`，这个代码块将被执行。为`false`则不被执行。

如果你想当值为`false`时执行些什么，使用`else`：

```rust
let x = 5;

if x == 5 {
    println!("x is five!");
} else {
    println!("x is not five :(");
}
```

如果不止一种情况，使用`else if`：

```rust
let x = 5;

if x == 5 {
    println!("x is five!");
} else if x == 6 {
    println!("x is six!");
} else {
    println!("x is not five or six :(");
}
```

这些都是非常标准的情况。然而你也可以这么写：

```rust
let x = 5;

let y = if x == 5 {
    10
} else {
    15
}; // y: i32
```

你可以（或许也应该）这么写：

```rust
let x = 5;

let y = if x == 5 { 10 } else { 15 }; // y: i32
```

这代码可以被执行是因为`if`是一个表达式。表达式的值是任何被选择的分支的最后一个表达式的值。一个没有`else`的`if`总是返回`()`作为返回值。
