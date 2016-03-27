# Vectors

> [vectors.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/vectors.md)
> <br>
> commit 5b9dd6a016adb5ed67e150643fb7e21dcc916845

“Vector”是一个动态或“可增长”的数组，被实现为标准库类型[`Vec<T>`](http://doc.rust-lang.org/std/vec/)（其中`<T>`是一个[泛型](Generics 泛型.md)语句）。vector总是在堆上分配数据。vector与切片就像`String`与`&str`一样。你可以使用`vec!`宏来创建它：

```rust
let v = vec![1, 2, 3, 4, 5]; // v: Vec<i32>
```

（与之前使用`println!`宏时不一样，我们用中括号`[]`配合`vec!`。为了方便，Rust 允许使用上述各种情况。）

对于重复初始值有另一种形式的`vec!`：

```rust
let v = vec![0; 10]; // ten zeroes
```

## 访问元素
为了vector特定索引的值，使用`[]`：

```rust
let v = vec![1, 2, 3, 4, 5];

println!("The third element of v is {}", v[2]);
```

索引从`0`开始，所以第3个元素是`v[2]`。

另外值得注意的是必须用`usize`类型的值来索引：

```rust
let v = vec![1, 2, 3, 4, 5];

let i: usize = 0;
let j: i32 = 0;

// works
v[i];

// doesn’t
v[j];
```

用非`usize`类型索引的话会给出类似如下的错误：

```text
error: the trait `core::ops::Index<i32>` is not implemented for the type
`collections::vec::Vec<_>` [E0277]
v[j];
^~~~
note: the type `collections::vec::Vec<_>` cannot be indexed by `i32`
error: aborting due to previous error
```

信息中有很多标点符号，不过关键是：你不能用`i32`来索引。

## 越界访问（Out-of-bounds Access）

如果尝试访问并不存在的索引：

```rust
let v = vec![1, 2, 3];
println!("Item 7 is {}", v[7]);
```

那么当前的线程会 [panic](Concurrency 并发.md#恐慌（panics）)并输出如下信息：

```text
thread '<main>' panicked at 'index out of bounds: the len is 3 but the index is 7'
```

如果你想处理越界错误而不是 panic，你可以使用像[`get`](http://doc.rust-lang.org/std/vec/struct.Vec.html#method.get)或[`get_mut`](http://doc.rust-lang.org/std/vec/struct.Vec.html#method.get)这样的方法，他们当给出一个无效的索引时返回`None`：

```rust
let v = vec![1, 2, 3];
match v.get(7) {
    Some(x) => println!("Item 7 is {}", x),
    None => println!("Sorry, this vector is too short.")
}
```

## 迭代
可以用`for`来迭代 vector 的元素。有3个版本：

```rust
let mut v = vec![1, 2, 3, 4, 5];

for i in &v {
    println!("A reference to {}", i);
}

for i in &mut v {
    println!("A mutable reference to {}", i);
}

for i in v {
    println!("Take ownership of the vector and its element {}", i);
}
```

vector还有很多有用的方法，可以看看[vector的API文档](http://doc.rust-lang.org/nightly/std/vec/)了解它们。
