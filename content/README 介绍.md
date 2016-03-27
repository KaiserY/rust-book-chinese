# Rust 编程语言

> [README.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/README.md)
> <br>
> commit 3a6dbb30a21be8d237055479af613e30415b0c56

欢迎阅读！这本书将教会你使用[Rust编程语言](http://www.rust-lang.org/)。Rust 是一个注重安全与速度的现代系统编程语言，通过在没有垃圾回收的情况下保证内存安全来实现它的目标，这使它成为一个在能够许多其它语言并不适合的用例中大展身手的语言：嵌入到其它语言中，在特定的时间和空间要求下编程，和编写例如设备驱动和操作系统这样的底层代码。它通过一系列的不产生运行时开销的编译时安全检查来提升目前语言所关注的领域，同时消除一切数据竞争。Rust同时也意在实现“零开销抽象”，即便在这些抽象看起来比较像一个高级语言的特性。即便如此，Rust也允许你像一个底层语言那样进行精确的控制。

《Rust编程语言》被分为数个部分。这个介绍是第一部分。之后是：

* [准备](./Getting Started 准备.md) - 为你的电脑安装 Rust 开发环境
* [学习Rust](./Learn Rust 学习Rust.md) - 通过一个小项目来学习 Rust 编程
* [语法和语义](./Syntax and Semantics 语法和语义.md) - Rust 各个部分，被拆分成小的部分讲解
* [高效Rust](./Effective Rust 高效Rust.md) - 编写优秀 Rust 代码的高级内容
* [Rust开发版](./Nightly Rust Rust开发版.md) - 还未出现在稳定版本中的最新功能
* [词汇表](./Glossary 词汇表.md) - 书中使用的术语的参考
* [参考文献](./bibliography 参考文献.md) - 影响过 Rust 的文献，关于 Rust 的论文

在阅读了介绍这部分之后，你可以根据喜好深入到“学习Rust”或“语法和语义”部分：如果你想通过项目深入了解，可以先选择“学习 Rust”；如果你想从头开始，并且学习一个完整的内容再学习另一个，你可以从“语法和语义”开始。丰富的交叉连接将这些部分联系到一起。

## 贡献
生成这本书（英文版）的源文件可以在 [GitHub](https://github.com/rust-lang/rust/tree/master/src/doc/book) 上找到。

> 以下内容在最新版中并未出现，暂时保留
## Rust 简介
Rust 是你会感兴趣的语言吗？让我们检查一些小的代码例子来展示它的部分威力。

使 Rust 显得独一无二的主要概念是“所有权”。考虑这个小例子：

```rust
fn main() {
    let mut x = vec!["Hello", "world"];
}
```

这个程序创建了一个叫做`x`的[变量绑定](./5.1.Variable Bindings 变量绑定.md)。这个绑定的值是一个`Vec<T>`，一个 vector，我们通过一个定义在标准库中的[宏](./5.34.Macros 宏.md)来创建它。这个宏叫做`vec`，并且我们通过一个`!`调用宏。这遵循了 Rust 的一般原则：让一切明了。宏可以做比函数调用复杂的多的多的工作，并且它们在视觉上也是有区别的。`!`也方便了解析，更容易编写工具，这也是很重要的。

我们使用了`mut`来使`x`可变：在 Rust 中绑定是默认是不可变的。在下面的例子中这个 vector 是可变的。

另外值得注意的是这里我们并不需要一个类型注释：因为 Rust 是静态类型的，我们并不需要显式的标明类型。Rust 拥有类型推断来平衡静态类型的能力和类型注释的冗余。

Rust 与堆分配相比倾向于栈分配：`x`被直接储存在栈上。然而，`Vec<T>`类型在堆上为 vector 的元素分配了空间。如果你并不熟悉这里的区别，目前你可以忽略它，或者看看[“栈和堆”](./4.1.The Stack and the Heap 栈和堆.md)。作为一个系统编程语言，Rust 给予你控制内存分配的能力，不过当我们上手后，这并不是什么大问题。

之前，我们提到“所有权”是 Rust 中的一个关键概念。在 Rust 用语中，`x`被认为“拥有”这个 vector。这意味着当`x`离开作用域，vector 的内存将被销毁。这由 Rust 编译器决定，而不是通过类似垃圾回收器这样的机制。换句话说，在 Rust 中，你并不需要自己调用像`malloc`和`free`这样的函数：编译器静态决定何时你需要分配和销毁内存，并自动调用这些函数。人非圣贤孰能无过,不过编译器永远也不会忘记。

让我们为例子再加一行：

```rust
fn main() {
    let mut x = vec!["Hello", "world"];

    let y = &x[0];
}
```

我们引入了另一个绑定，`y`。在这个例子中，`y`是对 vector 第一个元素的“引用”。Rust 的引用类似于其它语言中的指针，不过带有额外的编译时安全检查。引用用[“借用”](./5.8.References and Borrowing 引用和借用.md)它指向的内容，而不是拥有它，来与所有权系统交互。这里的区别是，当一个引用离开作用域，它不会释放之下的内存。如果它这么做了，我们会释放两次，这是很糟的！。

让我们增加第三行。这看起来并不会引起错误，不过实际上会造成一个编译错误：

```rust
fn main() {
    let mut x = vec!["Hello", "world"];

    let y = &x[0];

    x.push("foo");
}
```

`push`是 vector 的一个方法，它在 vector 的末尾附加另一个元素。当尝试编译这个程序时，我们得到一个错误：

```bash
error: cannot borrow `x` as mutable because it is also borrowed as immutable
    x.push(4);
    ^
note: previous borrow of `x` occurs here; the immutable borrow prevents
subsequent moves or mutable borrows of `x` until the borrow ends
    let y = &x[0];
             ^
note: previous borrow ends here
fn main() {

}
^
```

噢！Rust 编译器有时给出灰常详细的错误，而这就是其中之一。正如错误所解释的，即使让绑定可变，我们仍不能调用`push`。这是因为我们已经有了一个 vector 元素的引用，`y`。当有其它引用存在时改变值是危险的，因为我们可能使这个引用无效。在这个特定的例子中，当我们创建了 vector，我们可能只分配了 3 个元素的空间。增加一个元素意味着将分配一个新的能放下所有 4 个元素的空间，拷贝旧的值，并更新内部的指针指向这个内存。所有这些都木有问题。问题是`y`并没有被更新，很糟糕地，`y`成了一个“悬垂指针”(dangling pointer)。因此，在这个例子中任何对`y`的使用都会引起错误，而编译器会为我们捕获了这个错误。

该如何解决这个问题呢？这里我们可以采取两个方法。第一个方法是使用拷贝而非引用：

```rust
fn main() {
    let mut x = vec!["Hello", "world"];

    let y = x[0].clone();

    x.push("foo");
}
```

Rust 默认拥有[移动语义](./5.7.Ownership 所有权.md#移动语义)，所以如果想要拷贝一些数据，我们调用`clone()`方法。在这个例子中，`y`不再是一个储存在`x`中 vector 的一个引用，而是它第一个元素的拷贝，`"hello"`。现在我们并不拥有一个引用，所以`push()`就能正常工作。

如果真心需要一个引用，我们需要另一种方法：确保在尝试修改之前，让引用离开作用域。如下：

```rust
fn main() {
    let mut x = vec!["Hello", "world"];

    {
        let y = &x[0];
    }

    x.push("foo");
}
```

用一对大括号创建了一个内部作用域，`y`会在调用`push()`之前离开作用域，所以我们不会碰到问题。

所有权的概念并不仅仅善于防止悬垂指针，也解决了一整个系列的相关问题，比如迭代器无效，并发和其它问题。
