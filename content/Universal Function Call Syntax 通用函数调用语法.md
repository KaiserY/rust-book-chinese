# 通用函数调用语法

> [ufcs.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/ufcs.md)
> <br>
> commit 024aa9a345e92aa1926517c4d9b16bd83e74c10d

有时，函数可能有相同的名字。就像下面这些代码：

```rust
trait Foo {
    fn f(&self);
}

trait Bar {
    fn f(&self);
}

struct Baz;

impl Foo for Baz {
    fn f(&self) { println!("Baz’s impl of Foo"); }
}

impl Bar for Baz {
    fn f(&self) { println!("Baz’s impl of Bar"); }
}

let b = Baz;
```

如果我们尝试调用`b.f()`，我们会得到一个错误：

```text
error: multiple applicable methods in scope [E0034]
b.f();
  ^~~
note: candidate #1 is defined in an impl of the trait `main::Foo` for the type
`main::Baz`
    fn f(&self) { println!("Baz’s impl of Foo"); }
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
note: candidate #2 is defined in an impl of the trait `main::Bar` for the type
`main::Baz`
    fn f(&self) { println!("Baz’s impl of Bar"); }
    ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

我们需要一个区分我们需要调用哪一函数的方法。这个功能叫做“通用函数调用语法”（universal function call syntax），这看起来像这样：

```rust
# trait Foo {
#     fn f(&self);
# }
# trait Bar {
#     fn f(&self);
# }
# struct Baz;
# impl Foo for Baz {
#     fn f(&self) { println!("Baz’s impl of Foo"); }
# }
# impl Bar for Baz {
#     fn f(&self) { println!("Baz’s impl of Bar"); }
# }
# let b = Baz;
Foo::f(&b);
Bar::f(&b);
```

让我们拆开来看。

```rust
Foo::
Bar::
```

调用的这一半是两个traits的类型：`Foo`和`Bar`。这样实际上就区分了这两者：Rust调用你使用的trait里面的方法。

```rust
f(&b)
```

当我们使用[方法语法](Method Syntax 方法语法.md)调用像`b.f()`这样的方法时，如果`f()`需要`&self`，Rust实际上会自动地把`b`借用为`&self`。而在这个例子中，Rust并不会这么做，所以我们需要显式地传递一个`&b`。

## 尖括号形式（Angle-bracket Form）
我们刚才讨论的通用函数调用语法的形式：

```rust
Trait::method(args);
```

上面的形式其实是一种缩写。这是在一些情况下需要使用的扩展形式：

```rust
<Type as Trait>::method(args);
```

`<>::`语法是一个提供类型提示的方法。类型位于`<>`中。在这个例子中，类型是`Type as Trait`，表示我们想要`method`的`Trait`版本被调用。在没有二义时`as Trait`部分是可选的。尖括号也是一样。因此上面的形式就是一种缩写的形式。

这是一个使用较长形式的例子。

```rust
trait Foo {
    fn foo() -> i32;
}

struct Bar;

impl Bar {
    fn foo() -> i32 {
        20
    }
}

impl Foo for Bar {
    fn foo() -> i32 {
        10
    }
}

fn main() {
    assert_eq!(10, <Bar as Foo>::foo());
    assert_eq!(20, Bar::foo());
}
```

使用尖括号语法让你可以调用指定trait的方法而不是继承到的那个。
