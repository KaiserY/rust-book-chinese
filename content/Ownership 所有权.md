# 所有权

> [ownership.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/ownership.md)
> <br>
> commit fcc356373bba8c20a18d26bc81242c77c4153089

这篇教程是现行3个Rust所有权系统之一。所有权系统是Rust最独特且最引人入胜的特性之一，也是作为Rust开发者应该熟悉的。Rust所追求最大的目标 -- 内存安全，关键在于所有权。所有权系统有一些不同的概念，每个概念独自成章：

* 所有权，你正在阅读的这个章节
* [借用](5.9.References and Borrowing 引用和借用.md)，以及它关联的特性: "引用" (references)
* [生命周期](5.10.Lifetimes 生命周期.md)，关于借用的高级概念

这3章依次互相关联，你需要完整地阅读全部3章来对Rust的所有权系统进行全面的了解。

## 原则（Meta）
在我们开始详细讲解之前，这有两点关于所有权系统重要的注意事项。

Rust 注重安全和速度。它通过很多*零开销抽象*（*zero-cost abstractions*）来实现这些目标，也就是说在 Rust 中，实现抽象的开销尽可能的小。所有权系统是一个典型的零开销抽象的例子。本文提到所有的分析都是**在编译时完成的**。你不需要在运行时为这些功能付出任何开销。

然而，这个系统确实有一个开销：学习曲线。很多 Rust 初学者会经历我们所谓的“与借用检查器作斗争”的过程，也就是指 Rust 编译器拒绝编译一个作者认为合理的程序。这种“斗争”会因为程序员关于所有权系统如何工作的基本模型与 Rust 实现的实际规则不匹配而经常发生。当你刚开始尝试 Rust 的时候，你很可能会有相似的经历。然而有一个好消息：更有经验的 Rust 开发者反映，一旦他们适应所有权系统一段时间之后，与借用检查器的冲突会越来越少。

记住这些之后，让我们来学习关于所有权的内容。

## 所有权（Ownership）
Rust 中的[变量绑定](Variable Bindings 变量绑定.md)有一个属性：它们有它们所绑定的的值的*所有权*。这意味着当一个绑定离开作用域，它们绑定的资源就会被释放。例如：

```rust
fn foo() {
    let v = vec![1, 2, 3];
}
```

当`v`进入作用域，一个新的[Vec<T>](http://doc.rust-lang.org/stable/std/vec/struct.Vec.html)被创建，向量（vector）也在[堆](The Stack and the Heap 栈和堆.md)上为它的3个元素分配了空间。当`v`在`foo()`的末尾离开作用域，Rust将会清理掉与向量（vector）相关的一切，甚至是堆上分配的内存。这在作用域的结尾是一定（deterministically）会发生的。

我们将会在本章的后面详细介绍[vector](Vectors.md)；这里我们只是用它来作为一个在运行时在堆上分配内存的类型的例子。他们表现起来像[数组](Primitive Types 原生类型.md#数组)，除了通过`push()`更多元素他们的大小会改变。

vector 有一个[泛型类型](Generics 泛型.md)`Vec<T>`，所以在这个例子中`v`是`Vec<i32>`类型的。我们将会在本章的后面详细介绍泛型。

## 移动语义（Move semantics）
然而这里有更巧妙的地方：Rust 确保了对于任何给定的资源都*正好（只）有一个*绑定与之对应。例如，如果我们有一个 vector，我们可以把它赋予另外一个绑定：

```rust
let v = vec![1, 2, 3];

let v2 = v;
```

不过，如果之后我们尝试使用`v`，我们得到一个错误：

```rust
let v = vec![1, 2, 3];

let v2 = v;

println!("v[0] is: {}", v[0]);
```

它看起来像这样：

```text
error: use of moved value: `v`
println!("v[0] is: {}", v[0]);
                        ^
```

当我们定义了一个取得所有权的函数，并尝试在我们把变量作为参数传递给函数之后使用这个变量时，会发生相似的事情：

```rust
fn take(v: Vec<i32>) {
    // what happens here isn’t important.
}

let v = vec![1, 2, 3];

take(v);

println!("v[0] is: {}", v[0]);
```

一样的错误：“use of moved value”。当我们把所有权转移给别的别的绑定时，我们说我们“移动”了我们引用的值。这里你并不需要什么类型的特殊注解，这是 Rust 的默认行为。

## 细节
在移动了绑定后我们不能使用它的原因是微妙的，也是重要的。当我们写了这样的代码：

```rust
let v = vec![1, 2, 3];

let v2 = v;
```

第一行为 vector 对象`v`和它包含的数据分配了内存。向量对象储存在[栈](The Stack and the Heap 栈和堆.md)上并包含一个指向[堆](The Stack and the Heap 栈和堆.md)上`[1, 2, 3]`内容的指针。当我们从`v`移动到`v2`，它为`v2`创建了一个那个指针的拷贝。这意味着这将会有两个指向向量内容的指针。这将会因为引入了一个数据竞争而违反 Rust 的安全保证。因此，Rust 禁止我们在移动后使用`v`。

注意到优化可能会根据情况移除栈上字节（例如上面的向量）的实际拷贝也是很重要的。所以它也许并不像它开始看起来那样没有效率。

## `Copy`类型
我们已经知道了当所有权被转移给另一个绑定以后，你不能再使用原始绑定。然而，这里有一个[trait](Traits.md)会改变这个行为，它叫做`Copy`。我们还没有讨论到 trait，不过目前，你可以理解为一个为特定类型增加额外行为的标记。例如：

```rust
let v = 1;

let v2 = v;

println!("v is: {}", v);
```

在这个情况，`v`是一个`i32`，它实现了`Copy`。这意味着，就像一个移动，当我们把`v`赋值给`v2`,产生了一个数据的拷贝。不过，不像一个移动，我们仍可以在之后使用`v`。这是因为`i32`并没有指向其它数据的指针，对它的拷贝是一个完整的拷贝。

所有基本类型都实现了`Copy` trait，因此他们的所有权并不像你想象的那样遵循“所有权规则”被移动。作为一个例子，如下两段代码能够编译时因为`i32`和`bool`类型实现了`Copy` trait。

```rust
fn main() {
    let a = 5;

    let _y = double(a);
    println!("{}", a);
}

fn double(x: i32) -> i32 {
    x * 2
}
```

```rust
fn main() {
    let a = true;

    let _y = change_truth(a);
    println!("{}", a);
}

fn change_truth(x: bool) -> bool {
    !x
}
```

如果我们使用了没有实现`Copy`trait的类型，我们会得到一个编译错误，因为我们尝试使用一个移动了的值。

```text
error: use of moved value: `a`
println!("{}", a);
               ^
```

我们会在[trait](Traits.md)部分讨论如何编写你自己类型的`Copy`。

## 所有权之外（More than ownership）
当然，如果我们不得不在每个我们写的函数中交还所有权：

```rust
fn foo(v: Vec<i32>) -> Vec<i32> {
    // do stuff with v

    // hand back ownership
    v
}
```

这将会变得烦人。它在我们获取更多变量的所有权时变得更糟：

```rust
fn foo(v1: Vec<i32>, v2: Vec<i32>) -> (Vec<i32>, Vec<i32>, i32) {
    // do stuff with v1 and v2

    // hand back ownership, and the result of our function
    (v1, v2, 42)
}

let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 3];

let (v1, v2, answer) = foo(v1, v2);
```

额！返回值，返回的代码行（上面的最后一行），和函数调用都变得更复杂了。

幸运的是，Rust 提供了一个 trait，借用，它帮助我们解决这个问题。这个主题将在下一个部分讨论！
