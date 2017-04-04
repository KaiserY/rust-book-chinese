# 原生类型

> [primitive-types.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/primitive-types.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

Rust有一系列被认为是“原生”的类型。这意味着它们是内建在语言中的。Rust被构建为在标准库中也提供了一些建立在这些类型之上的有用的类型，不过它们也大部分是原生的。

## 布尔型
Rust 有一个内建的布尔类型，叫做`bool`。它有两个值，`true`和`false`：

```rust
let x = true;

let y: bool = false;
```

布尔型通常用在[if语句](If If语句.md)中。

你可以在[标准库文档](http://doc.rust-lang.org/nightly/std/primitive.bool.html)中找到更多关于`bool`的文档。

## `char`
`char`类型代表一个单独的 Unicode 字符的值。你可以用单引号（`'`）创建`char`：

```rust
let x = 'x';
let two_hearts = '💕';
```

不像其它语言，这意味着Rust的`char`并不是 1 个字节，而是 4 个。

你可以在[标准库文档](http://doc.rust-lang.org/nightly/std/primitive.char.html)中找到更多关于`char`的文档。

## 数字类型
Rust有一些分类的大量数字类型：有符号和无符号，定长和变长，浮点和整型。

这些类型包含两部分：分类，和大小。例如，`u16`是一个拥有16位大小的无符号类型。更多字节让你拥有更大的数字。

如果一个数字常量没有推断它类型的条件，它采用默认类型：

```rust
let x = 42; // `x` has type `i32`.

let y = 1.0; // `y` has type `f64`.
```

这里有一个不同数字类型的列表，以及它们在标注库中的文档：

* [i8](http://doc.rust-lang.org/nightly/std/primitive.i8.html)
* [i16](http://doc.rust-lang.org/nightly/std/primitive.i16.html)
* [i32](http://doc.rust-lang.org/nightly/std/primitive.i32.html)
* [i64](http://doc.rust-lang.org/nightly/std/primitive.i64.html)
* [u8](http://doc.rust-lang.org/nightly/std/primitive.u8.html)
* [u16](http://doc.rust-lang.org/nightly/std/primitive.u16.html)
* [u32](http://doc.rust-lang.org/nightly/std/primitive.u32.html)
* [u64](http://doc.rust-lang.org/nightly/std/primitive.u64.html)
* [isize](http://doc.rust-lang.org/nightly/std/primitive.isize.html)
* [usize](http://doc.rust-lang.org/nightly/std/primitive.usize.html)
* [f32](http://doc.rust-lang.org/nightly/std/primitive.f32.html)
* [f64](http://doc.rust-lang.org/nightly/std/primitive.f64.html)

让我们按分类重温一遍：

### 有符号和无符号
整型有两种变体：有符号和无符号。为了理解它们的区别，让我们考虑一个 4 比特大小的数字。一个有符号，4 比特数字你可以储存`-8`到`+7`的数字。有符号数采用“补码”表示。一个无符号 4 比特的数字，因为它不需要储存负数，可以出储存`0`到`+15`的数字。

### 固定大小类型
固定大小类型在其表现中有特定数量的位。有效的位大小是`8`，`16`，`32`和`64`。那么，`u32`是无符号的，32 位整型，而`i64`是有符号，64 位整型。

### 可变大小类型
Rust 也提供了依赖底层机器指针大小的类型。这些类型拥有“size”分类，并有有符号和无符号变体。它有两个类型：`isize`和`usize`。

### 浮点类型
Rust 也有两个浮点类型：`f32`和`f64`。它们对应 IEEE-754 单精度和双精度浮点数。

## 数组
像很多编程语言一样，Rust有用来表示数据序列的列表类型。最基本的是**数组**，一个定长相同类型的元素列表。数组默认是不可变的。

```rust
let a = [1, 2, 3]; // a: [i32; 3]
let mut m = [1, 2, 3]; // m: [i32; 3]
```

数组的类型是`[T; N]`。我们会在[泛型部分](Generics 泛型.md)的时候讨论这个`T`标记。`N`是一个编译时常量，代表数组的长度。

有一个可以将数组中每一个元素初始化为相同值的简写。在这个例子中，`a`的每个元素都被初始化为`0`：

```rust
let a = [0; 20]; // a: [i32; 20]
```

你可以用`a.len()`来获取数组`a`的元素数量：

```rust
let a = [1, 2, 3];

println!("a has {} elements", a.len());
```

你可以用**下标**（*subscript notation*）来访问特定的元素：

```rust
let names = ["Graydon", "Brian", "Niko"]; // names: [&str; 3]

println!("The second name is: {}", names[1]);
```

就跟大部分编程语言一个样，下标从0开始，所以第一个元素是`names[0]`，第二个是`names[1]`。上面的例子会打印出`The second name is: Brian`。如果你尝试使用一个不在数组中的下标，你会得到一个错误：数组访问会在运行时进行边界检查。这种不适当的访问是其它系统编程语言中很多bug的根源。

你可以在[标准库文档](http://doc.rust-lang.org/stable/std/primitive.array.html)中找到更多关于`array`的文档。

## 切片（Slices）
一个**切片**（*slice*）是一个数组的引用（或者“视图”）。它有利于安全，有效的访问数组的一部分而不用进行拷贝。比如，你可能只想要引用读入到内存的文件中的一行。原理上，片段并不是直接创建的，而是引用一个已经存在的变量。片段有预定义的长度，可以是可变也可以是不可变的。

在底层，slice 代表一个指向数据开始的指针和一个长度。

### 切片语法（Slicing syntax）

你可以用一个`&`和`[]`的组合从多种数据类型创建一个切片。`&`表明切片类似于[引用](References and Borrowing 引用和借用.md)，这个我们会在本部分的后面详细介绍。带有一个范围的`[]`，允许你定义切片的长度：

```rust
let a = [0, 1, 2, 3, 4];
let complete = &a[..]; // A slice containing all of the elements in `a`.
let middle = &a[1..4]; // A slice of `a`: only the elements `1`, `2`, and `3`.
```

片段拥有`&[T]`类型。当我们涉及到[泛型](Generics 泛型.md)时会讨论这个`T`。

你可以在[标准库文档](http://doc.rust-lang.org/stable/std/primitive.slice.html)中找到更多关于`slices`的文档。


## `str`
Rust的`str`类型是最原始的字符串类型。作为一个[不定长类型](Unsized Types 不定长类型.md)，它本身并不是非常有用，不过当它用在引用后是就有用了，例如[&str](Strings 字符串.md)。如你所见，我们到时候再讲。

你可以在[标准库文档](http://doc.rust-lang.org/stable/std/primitive.str.html)中找到更多关于`str`的文档。

## 元组（Tuples）
元组（tuples）是固定大小的有序列表。如下：

```rust
let x = (1, "hello");
```

这是一个长度为2的元组，有括号和逗号组成。下面也是同样的元组，不过注明了数据类型：

```rust
let x: (i32, &str) = (1, "hello");
```

如你所见，元组的类型跟元组看起来很像，只不过类型取代的值的位置。细心的读者可能会注意到元组是异质的：这个元组中有一个`i32`和一个`&str`。在系统编程语言中，字符串要比其它语言中来的复杂。现在，可以认为`&str`是一个**字符串片段**（*string slice*），我们马上会讲到它。

你可以把一个元组赋值给另一个，如果它们包含相同的类型和[数量](Glossary 词汇表.md#参数数量（arity）)。当元组有相同的长度时它们有相同的数量。

```rust
let mut x = (1, 2); // x: (i32, i32)
let y = (2, 3); // y: (i32, i32)

x = y;
```

你可以通过一个**解构let**（*destructuring let*）访问元组中的字段。下面是一个例子：

```rust
let (x, y, z) = (1, 2, 3);

println!("x is {}", x);
```

还记得[之前](Variable Bindings 变量绑定.md)我曾经说过`let`语句的左侧远比一个赋值绑定强大吗？这就是证据。我们可以在`let`左侧写一个模式，如果它能匹配右侧的话，我们可以一次写多个绑定。这种情况下，`let`“解构”或“拆开”了元组，并分成了三个绑定。

这个模式是很强大的，我们后面会经常看到它。

你可以一个逗号来消除一个单元素元组和一个括号中的值的歧义：

```rust
(0,); // single-element tuple
(0); // zero in parentheses
```

### 元组索引（Tuple Indexing）
你也可以用索引语法访问一个元组的字段：

```rust
let tuple = (1, 2, 3);

let x = tuple.0;
let y = tuple.1;
let z = tuple.2;

println!("x is {}", x);
```

就像数组索引，它从`0`开始，不过也不像数组索引，它使用`.`，而不是`[]`。

你可以在[标准库文档](http://doc.rust-lang.org/stable/std/primitive.tuple.html)中找到更多关于`tuple`的文档。

## 函数
函数也有一个类型！它们看起来像这样：

```rust
fn foo(x: i32) -> i32 { x }

let x: fn(i32) -> i32 = foo;
```

在这个例子中，`x`是一个“函数指针”，指向一个获取一个`i32`参数并返回一个`i32`值的函数。
