# 引用和借用

> [references-and-borrowing.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/references-and-borrowing.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

这篇教程是现行 3 个 Rust 所有权系统章节的第二部分。所有权系统是 Rust 最独特且最引人入胜的特性之一，也是作为 Rust 开发者应该熟悉的。Rust 所追求最大的目标 -- 内存安全，关键在于所有权。所有权系统有一些不同的概念，每个概念独自成章：

* [所有权](5.8.Ownership 所有权.md)，关键章节
* 借用，你正在阅读的这个章节
* [生命周期](5.10.Lifetimes 生命周期.md)，关于借用的高级概念

这 3 章依次互相关联，你需要完整地阅读全部 3 章来对 Rust 的所有权系统进行全面的了解。

## 原则（Meta）
在我们开始详细讲解之前，这有两点关于所有权系统重要的注意事项。

Rust 注重安全和速度。它通过很多**零开销抽象**（*zero-cost abstractions*）来实现这些目标，也就是说在 Rust 中，实现抽象的开销尽可能的小。所有权系统是一个典型的零开销抽象的例子。本文提到所有的分析都是**在编译时完成的**。你不需要在运行时为这些功能付出任何开销。

然而，这个系统确实有一个开销：学习曲线。很多 Rust 初学者会经历我们所谓的“与借用检查器作斗争”的过程，也就是指 Rust 编译器拒绝编译一个作者认为合理的程序。这种“斗争”会因为程序员关于所有权系统如何工作的基本模型与 Rust 实现的实际规则不匹配而经常发生。当你刚开始尝试 Rust 的时候，你很可能会有相似的经历。然而有一个好消息：更有经验的 Rust 开发者反映，一旦他们适应所有权系统一段时间之后，与借用检查器的冲突会越来越少。

记住这些之后，让我们来学习关于借用的内容。

## 借用
在[所有权](Ownership 所有权.md)章节的最后，我们有一个看起来像这样的糟糕的函数：

```rust
fn foo(v1: Vec<i32>, v2: Vec<i32>) -> (Vec<i32>, Vec<i32>, i32) {
    // Do stuff with `v1` and `v2`.

    // Hand back ownership, and the result of our function.
    (v1, v2, 42)
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let (v1, v2, answer) = foo(v1, v2);
```

然而这并不是理想的 Rust 代码，因为它没有利用'借用'这个编程语言的特点。这是它的第一步：

```rust
fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
    // Do stuff with `v1` and `v2`.

    //  Return the answer.
    42
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let answer = foo(&v1, &v2);

// We can use `v1` and `v2` here!
```

一个更具体的例子：

```rust
fn main() {
    // Don't worry if you don't understand how `fold` works, the point here is that an immutable reference is borrowed.
    fn sum_vec(v: &Vec<i32>) -> i32 {
        return v.iter().fold(0, |a, &b| a + b);
    }
    // Borrow two vectors and sum them.
    // This kind of borrowing does not allow mutation through the borrowed reference.
    fn foo(v1: &Vec<i32>, v2: &Vec<i32>) -> i32 {
        // Do stuff with `v1` and `v2`.
        let s1 = sum_vec(v1);
        let s2 = sum_vec(v2);
        // Return the answer.
        s1 + s2
    }

    let v1 = vec![1, 2, 3];
    let v2 = vec![4, 5, 6];

    let answer = foo(&v1, &v2);
    println!("{}", answer);
}
```

与其获取`Vec<i32>`作为我们的参数，我们获取一个引用：`&Vec<i32>`。并与其直接传递`v1`和`v2`，我们传递`&v1`和`&v2`。我们称`&T`类型为一个”引用“，而与其拥有这个资源，它借用了所有权。一个借用变量的绑定在它离开作用域时并不释放资源。这意味着`foo()`调用之后，我们可以再次使用原始的绑定。

引用是不可变的，就像绑定一样。这意味着在`foo()`中，向量完全不能被改变：

```rust
fn foo(v: &Vec<i32>) {
     v.push(5);
}

let v = vec![];

foo(&v);
```

有如下错误：

```text
error: cannot borrow immutable borrowed content `*v` as mutable
v.push(5);
^
```

放入一个值改变了向量，所以我们不允许这样做

## `&mut`引用
这有第二种类型的引用：`&mut T`。一个“可变引用”允许你改变你借用的资源。例如：

```rust
let mut x = 5;
{
    let y = &mut x;
    *y += 1;
}
println!("{}", x);
```

这会打印`6`。我们让`y`是一个`x`的可变引用，接着把`y`指向的值加一。你会注意到`x`也必须被标记为`mut`，如果它不是，我们不能获取一个不可变值的可变引用。

你也会发现我们在`y`前面加了一个星号（`*`），成了`*y`，这是因为`y`是一个`&mut`引用。你也需要使用他们（星号）来访问引用的内容。

否则，`&mut`引用就像一个普通引用。这两者之间,以及它们是如何交互的**有**巨大的区别。你会发现在上面的例子有些不太靠谱，因为我们需要额外的作用域，包围在`{`和`}`之间。如果我们移除它们，我们得到一个错误：

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
note: previous borrow of `x` occurs here; the mutable borrow prevents
subsequent moves, borrows, or modification of `x` until the borrow ends
        let y = &mut x;
                     ^
note: previous borrow ends here
fn main() {

}
^
```

正如这个例子表现的那样，有一些规则是你必须要掌握的。

## 规则

Rust 中的借用有一些规则：

第一，任何借用必须位于比拥有者更小的作用域。第二，对于同一个资源（resource）的借用，以下情况不能同时出现在同一个作用域下：

* 1 个或多个不可变引用（&T）
* 唯一 1 个可变引用（&mut T）

`译者注：即同一个作用域下，要么只有一个对资源A的可变引用（&mut T），要么有N个不可变引用（&T），但不能同时存在可变和不可变的引用`

你可能注意到这些看起来很眼熟，虽然并不完全一样，它类似于数据竞争的定义：

> 当 2 个或更多个指针同时访问同一内存位置，当它们中至少有 1 个在写，同时操作并不是同步的时候存在一个“数据竞争”

通过引用，你可以拥有你想拥有的任意多的引用，因为它们没有一个在写。如果你在写，并且你需要2个或更多相同内存的指针，则你只能一次拥有一个`&mut`。这就是Rust如何在编译时避免数据竞争：如果打破规则的话，我们会得到错误。

在记住这些之后，让我们再次考虑我们的例子。

## 理解作用域（Thinking in scopes）
这是代码：

```rust
fn main() {
    let mut x = 5;
    let y = &mut x;

    *y += 1;

    println!("{}", x);
}
```

这些代码给我们如下错误：

```text
error: cannot borrow `x` as immutable because it is also borrowed as mutable
    println!("{}", x);
                   ^
```

这是因为我们违反了规则：我们有一个指向`x`的`&mut T`，所以我们不允许创建任何`&T`。一个或另一个。错误记录提示了我们应该如何理解这个错误：

```text
note: previous borrow ends here
fn main() {

}
^
```

换句话说，可变借用在剩下的例子中一直存在。我们需要的是可变借用在我们尝试调用`println!`**之前**结束并生成一个不可变借用。在 Rust 中，借用绑定在借用有效的作用域上。而我们的作用域看起来像这样：

```rust
fn main() {
    let mut x = 5;

    let y = &mut x;    // -+ &mut borrow of `x` starts here.
                       //  |
    *y += 1;           //  |
                       //  |
    println!("{}", x); // -+ - Try to borrow `x` here.
}                      // -+ &mut borrow of `x` ends here.
```

这些作用域冲突了：我们不能在`y`在作用域中时生成一个`&x`。

所以我们增加了一个大括号：

```rust
let mut x = 5;

{
    let y = &mut x; // -+ &mut borrow starts here.
    *y += 1;        //  |
}                   // -+ ... and ends here.

println!("{}", x);  // <- Try to borrow `x` here.
```

这就没有问题了。我们的可变借用在我们创建一个不可变引用之前离开了作用域。不过作用域是看清一个借用持续多久的关键。

## 借用避免的问题（Issues borrowing prevents）
为什么要有这些限制性规则？好吧，正如我们记录的，这些规则避免了数据竞争。数据竞争能造成何种问题呢？这里有一些。

### 迭代器失效（Iterator invalidation）
一个例子是“迭代器失效”，它在当你尝试改变你正在迭代的集合时发生。Rust 的借用检查器阻止了这些发生：

```rust
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
}
```

这会打印出 1 到 3.因为我们在向量上迭代，我们只得到了元素的引用。同时`v`本身作为不可变借用，它意味着我们在迭代时不能改变它：

```rust
let mut v = vec![1, 2, 3];

for i in &v {
    println!("{}", i);
    v.push(34);
}
```

这里是错误：

```text
error: cannot borrow `v` as mutable because it is also borrowed as immutable
    v.push(34);
    ^
note: previous borrow of `v` occurs here; the immutable borrow prevents
subsequent moves or mutable borrows of `v` until the borrow ends
for i in &v {
          ^
note: previous borrow ends here
for i in &v {
    println!(“{}”, i);
    v.push(34);
}
^
```

我们不能修改`v`因为它被循环借用。

### 释放后使用
引用必须与它引用的值存活得一样长。Rust 会检查你的引用的作用域来保证这是正确的。

如果 Rust 并没有检查这个属性，我们可能意外的使用了一个无效的引用。例如：

```rust
let y: &i32;
{
    let x = 5;
    y = &x;
}

println!("{}", y);
```

我们得到这个错误：

```text
error: `x` does not live long enough
    y = &x;
         ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
let y: &i32;
{
    let x = 5;
    y = &x;
}

note: ...but borrowed value is only valid for the block suffix following
statement 0 at 4:18
    let x = 5;
    y = &x;
}
```

换句话说，`y`只在`x`存在的作用域中有效。一旦`x`消失，它变成无效的引用。为此，这个错误说借用“并没有存活得足够久”因为它在应该有效的时候是无效的。

当引用在它引用的变量**之前**声明会导致类似的问题：

```rust
let y: &i32;
let x = 5;
y = &x;

println!("{}", y);
```

我们得到这个错误：

```text
error: `x` does not live long enough
y = &x;
     ^
note: reference must be valid for the block suffix following statement 0 at
2:16...
    let y: &i32;
    let x = 5;
    y = &x;

    println!("{}", y);
}

note: ...but borrowed value is only valid for the block suffix following
statement 1 at 3:14
    let x = 5;
    y = &x;

    println!("{}", y);
}
```

在上面的例子中，`y`在`x`之前被声明，意味着`y`比`x`生命周期更长，这是不允许的。
