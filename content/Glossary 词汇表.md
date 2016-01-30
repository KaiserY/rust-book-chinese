# 词汇表

> [glossary.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/glossary.md)
> <br>
> commit 024aa9a345e92aa1926517c4d9b16bd83e74c10d

不是每位 Rustacean 都是系统编程或计算机科学背景的，所以我们加上了可能难以理解的词汇解释。

## 数量（Arity）
Arity代表函数或操作所需的参数数量。
```rust
let x = (2, 3);
let y = (4, 6);
let z = (8, 2, 6);
```
在上面的例子中`x`和`y`的Arity是`2`，`z`的Arity是`3`。

## 抽象语法树（Abstract Syntax Tree）
当一个编译器编译你程序的时候，它做了很多不同的事。其中之一就是将你程序中的文本转换为一个‘抽象语法树’，或者‘AST’。这个树是你程序结构的表现。例如，`2 + 3`可以转换为一个树：

```text
  +
 / \
2   3
```

而`2 + (3 * 4)`看起来像这样：

```text
  +
 / \
2   *
   / \
  3   4
```

## 参数数量（Arity）

Arity 代表一个函数或操作获取的参数的数量。

```rust
let x = (2, 3);
let y = (4, 6);
let z = (8, 2, 6);
```

在上面这个例子中`x`和`y`的arity是 2。`z`的arity是 3。

## 界限（Bounds）

界限是一个类型或[trait](Traits.md)的限制。例如，如果界限位于函数参数，那么传递给函数的参数类型必须遵守这个限制。

## 动态大小类型（DST (Dynamically Sized Type)）

一个没有静态大小或对齐的类型。（[更多信息](https://github.com/rust-lang/rust/blob/master/src/doc/nomicon/exotic-sizes.html#dynamically-sized-types-dsts)）

## 表达式（Expression）
在计算机编程中，一个表达式是一个值，常量，变量，运算符和函数的组合，它可以产生一个单一的值。例如，`2 + (3 * 4)`是一个返回14的表达式。值得注意的是表达式可以产生副作用。例如，一个表达式中的函数可能会执行一些不只是简单的返回一个值的操作。

## 面向表达式语言（Expression-Oriented Language）
在早期的编程语言中，[表达式](#表达式)和[语句](#语句)时两个不同句法范畴：表达式有一个值而语句做一件事。然而，之后的语言模糊了这个区别，允许表达式执行操作而让语句有一个值。在一个面向表达式的语言中，（几乎）所有语句都是一个表达式并因此返回一个值。由此，这些表达式自身也可以是更大表达式的一部分。

## 语句（Statement）
在计算机编程中，一个语句是一个编程语言能让计算执行操的最小的独立元素。
