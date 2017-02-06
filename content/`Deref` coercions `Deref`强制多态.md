# `Deref`强制多态

> [deref-coercions.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/deref-coercions.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

标准库提供了一个特殊的特性，[`Deref`](http://doc.rust-lang.org/stable/std/ops/trait.Deref.html)。它一般用来重载`*`，解引用运算符：

```rust
use std::ops::Deref;

struct DerefExample<T> {
    value: T,
}

impl<T> Deref for DerefExample<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.value
    }
}

fn main() {
    let x = DerefExample { value: 'a' };
    assert_eq!('a', *x);
}
```

这对编写自定义指针类型很有用。然而，有一个与`Deref`相关的语言功能：“解引用强制多态（deref coercions）”。规则如下：如果你有一个`U`类型，和它的实现`Deref<Target=T>`，（那么）`&U`的值将会自动转换为`&T`。这是一个例子：

```rust
fn foo(s: &str) {
    // Borrow a string for a second.
}

// String implements Deref<Target=str>.
let owned = "Hello".to_string();

// Therefore, this works:
foo(&owned);
```

在一个值的前面用`&`号获取它的引用。所以`owned`是一个`String`，`&owned`是一个`&String`，而因为`impl Deref<Target=str> for String`，`&String`将会转换为`&str`，而它是`foo()`需要的。

这就是了。这是Rust唯一一个为你进行一个自动转换的地方，不过它增加了很多灵活性。例如，`Rc<T>`类型实现了`Deref<Target=T>`，所以这可以工作：

```rust
use std::rc::Rc;

fn foo(s: &str) {
    // Borrow a string for a second.
}

// String implements Deref<Target=str>.
let owned = "Hello".to_string();
let counted = Rc::new(owned);

// Therefore, this works:
foo(&counted);
```

我们所做的一切就是把我们的`String`封装到了一个`Rc<T>`里。不过现在我们可以传递`Rc<String>`给任何我们有一个`String`的地方。`foo`的签名并无变化，不过它对这两个类型都能正常工作。这个例子有两个转换：`&Rc<String>`转换为`&String`接着是`&String`转换为`&str`。只要类型匹配Rust将可以做任意多次这样的转换。

标准库提供的另一个非常通用的实现是：

```rust
fn foo(s: &[i32]) {
    // Borrow a slice for a second.
}

// Vec<T> implements Deref<Target=[T]>.
let owned = vec![1, 2, 3];

foo(&owned);
```

向量可以`Deref`为一个切片。

## `Deref`和方法调用
当调用一个方法时`Deref`也会出现。考虑下面的例子：

```rust
struct Foo;

impl Foo {
    fn foo(&self) { println!("Foo"); }
}

let f = &&Foo;

f.foo();
```

即便`f`是`&&Foo`，而`foo`接受`&self`，这也是可以工作的。因为这些都是一样的：

```rust
f.foo();
(&f).foo();
(&&f).foo();
(&&&&&&&&f).foo();
```

一个`&&&&&&&&&&&&&&&&Foo`类型的值仍然可以调用`Foo`定义的方法，因为编译器会插入足够多的`*`来使类型正确。而正因为它插入`*`，它用了`Deref`。
