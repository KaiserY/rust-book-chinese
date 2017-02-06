# 不安全代码

> [unsafe.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/unsafe.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

Rust主要魅力是它强大的静态行为保障。不过安全检查天性保守：有些程序实际上是安全的，不过编译器不能验证它是否是真的。为了写这种类型的程序，我们需要告诉编译器稍微放松它的限制。为此，Rust有一个关键字，`unsafe`。使用`unsafe`的代码比正常代码有更少的限制。

让我们过一遍语法，接着我们讨论语义。`unsafe`用在两个上下文中。第一个标记一个函数为不安全的：

```rust
unsafe fn danger_will_robinson() {
    // Scary stuff...
}
```

例如所有从[FFI](Foreign Function Interface 外部函数接口.md)调用的函数都必须标记为`unsafe`。第二个`unsafe`的用途是一个不安全块。

```rust
unsafe {
    // Scary stuff...
}
```

第三个是不安全trait：

```rust
unsafe trait Scary { }
```

而第四个是`impl`这些trait：

```rust
# unsafe trait Scary { }
unsafe impl Scary for i32 {}
```

显式勾勒出那些可能会有bug并造成大问题的代码是很重要的。如果一个Rust程序段错误了，你可以确认它位于标记为`unsafe`部分的什么地方。

## “安全”指什么？（What does ‘safe’ mean?）
安全，在Rust的上下文中，意味着“不做任何不安全的事”。不过也要明白，有一些特定的行为在你的代码中可能并不合意，但很明显**并不是**不安全的：

* 死锁
* 内存或其他资源的泄露
* 退出但未调用析构函数
* 整型溢出

Rust不能避免所有类型的软件错误。有bug的代码可能并将会出现在Rust中。这些事并不很光彩，不过它们并不特别的定义为`unsafe`。

另外，如下列表全是 Rust 中的未定义行为，并且必须被避免，即便在编写`unsafe`代码时：

* 数据竞争
* 解引用一个空/悬垂裸指针
* 读[`undef`](http://llvm.org/docs/LangRef.html#undefined-values)（未初始化）内存
* 使用裸指针打破[指针重叠规则](http://llvm.org/docs/LangRef.html#pointer-aliasing-rules)（pointer aliasing rules）
* `&mut T`和`&T`遵循LLVM范围的[`noalias`](http://llvm.org/docs/LangRef.html#noalias)模型，除了如果`&T`包含一个`UnsafeCell<U>`的话。不安全代码必须不能违反这些重叠（aliasing）保证
* 不使用`UnsafeCell<U>`改变一个不可变值/引用
* 通过编译器固有功能调用未定义行为：
  * 使用`std::ptr::offset`（`offset`功能）来索引超过对象边界的值，除了允许的末位超出一个字节
  * 在重叠（overlapping）缓冲区上使用`std::ptr::copy_nonoverlapping_memory`（`memcpy32/memcpy64`功能）
* 原生类型的无效值，即使是在私有字段/本地变量中：
  * 空/悬垂引用或装箱
  * `bool`中一个不是`false`（`0`）或`true`（`1`）的值
  * `enum`中一个并不包含在类型定义中判别式
  * `char`中一个代理字（surrogate）或超过`char::MAX`的值
  * `str`中非UTF-8字节序列
* 在外部代码中使用Rust或在Rust中使用外部语言

## 不安全的超级力量（Unsafe Superpowers）
在不安全函数和不安全块，Rust将会让你做3件通常你不能做的事：只有3件。它们是：

1. 访问和更新一个[静态可变变量](`const` and `static`.md#static)
2. 解引用一个裸指针
3. 调用不安全函数。这是最NB的能力

这就是全部。注意到`unsafe`不能（例如）“关闭借用检查”是很重要的。为随机的Rust代码加上`unsafe`并不会改变它的语义，它并不会开始接受任何东西。

不过**确实**它会让你写的东西打破一些规则。让我们按顺序过一遍这3个能力。

### 访问和更新一个`static mut`
Rust有一个叫`static mut`的功能，它允许改变全局状态。这么做可能造成一个数据竞争，所以它天生是不安全的。关于更多细节，查看[静态量](`const` and `static`.md#static)部分。

### 解引用一个裸指针
裸指针让你做任意的指针算数，并会产生一系列不同的内存安全（safety & security）问题。在某种意义上，解引用一个任意指针的能力是你可以做的最危险的事之一。更多关于裸指针，查看[它的部分](Raw Pointers 裸指针.md)。

### 调用不安全函数
最后的能力能用于`unsafe`的两个方面：你只能在一个不安全块中调用被标记为`unsafe`的函数。

这个能力是强力和多变的。Rust暴露了一些作为不安全函数的[编译器固有功能](Intrinsics 固有功能.md)，并且一些不安全函数绕开了安全检查，用安全换速度。

我在重复一遍：即便你**可以**在一个不安全块和函数中做任何事并不意味你应该这么做。编译器会表现得像你在保持它不变一样（The compiler will act as though you’re upholding its invariants），所以请小心。
