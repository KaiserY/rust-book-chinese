# `Borrow` 和 `AsRef`

> [borrow-and-asref.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/borrow-and-asref.md)
> <br>
> commit 6976991569977e8097da5f7660a31a42d11e48d2

[`Borrow`](http://doc.rust-lang.org/std/borrow/trait.Borrow.html)和[`AsRef`](http://doc.rust-lang.org/std/convert/trait.AsRef.html)特性非常相似。这是一个快速的关于这两个特性意义的复习。

## `Borrow`

`Borrow`特性用于当你处于某种目的写了一个数据结构，并且你想要使用一个要么拥有要么借用的类型作为它的同义词。

例如，[`HashMap`](http://doc.rust-lang.org/std/collections/struct.HashMap.html)有一个用了`Borrow`的[`get`方法](http://doc.rust-lang.org/std/collections/struct.HashMap.html#method.get)：

```rust
fn get<Q: ?Sized>(&self, k: &Q) -> Option<&V>
    where K: Borrow<Q>,
          Q: Hash + Eq
```

这个签名非常复杂。`k`参数是我们感兴趣的。它引用了一个`HashMap`自身的参数：

```rust
struct HashMap<K, V, S = RandomState> {
```

`k`参数是`HashMap`用的`key`类型。所以，再一次查看`get()`的签名，我们可以在键实现了`Borrow<Q>`时使用`get()`。这样，我们可以创建一个`HashMap`，它使用`String`键，不过在我们搜索时使用`&str`：

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert("Foo".to_string(), 42);

assert_eq!(map.get("Foo"), Some(&42));
```

这是因为标准库中有`impl Borrow<str> for String`（为 String 实现了Borrow<str>)。

对于多数类型，当你想要获取一个自我拥有或借用的类型，`&T`就足够了。不过当有多于一种借用的值时，`Borrow`就能起作用了。引用和`slice`就是一个能体现这一点的地方：你可以有`&[T]`或者`&mut [T]`。如果我们想接受这两种类型，`Borrow`就是你需要的：

```rust
use std::borrow::Borrow;
use std::fmt::Display;

fn foo<T: Borrow<i32> + Display>(a: T) {
    println!("a is borrowed: {}", a);
}

let mut i = 5;

foo(&i);
foo(&mut i);
```

这会打印出`a is borrowed: 5`两次。

## `AsRef`
`AsRef`特性是一个转换特性。它用来在泛型中把一些值转换为引用。像这样：

```rust
let s = "Hello".to_string();

fn foo<T: AsRef<str>>(s: T) {
    let slice = s.as_ref();
}
```

## 我应该用哪个？
我们可以看到它们有些相似：它们都处理一些类型的自我拥有和借用版本。然而，它们还是有些不同。

选择`Borrow`当你想要抽象不同类型的借用，或者当你创建一个数据结构它把自我拥有和借用的值看作等同的，例如哈希和比较。

选择`AsRef`当你想要直接把一些值转换为引用，和当你在写泛型代码的时候。
