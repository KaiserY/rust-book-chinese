# trait对象

> [trait-objects.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/trait-objects.md)
> <br>
> commit c7b092b47d8e50b58c975040af0a84a544f7fa7a

当涉及到多态的代码时，我们需要一个机制来决定哪个具体的版本应该得到执行。这叫做“分发”（dispatch）。大体上有两种形式的分发：静态分发和动态分发。虽然 Rust 喜欢静态分发，不过它也提供了一个叫做“trait 对象”的机制来支持动态分发。

## 背景
在本章接下来的内容中，我们需要一个 trait 和一些实现。让我们来创建一个简单的`Foo`。它有一个返回`String`的方法。

```rust
trait Foo {
    fn method(&self) -> String;
}
```

我们也在`u8`和`String`上实现了这个trait：

```rust
# trait Foo { fn method(&self) -> String; }
impl Foo for u8 {
    fn method(&self) -> String { format!("u8: {}", *self) }
}

impl Foo for String {
    fn method(&self) -> String { format!("string: {}", *self) }
}
```

## 静态分发

我们可以使用 trait 的限制来进行静态分发：

```rust
# trait Foo { fn method(&self) -> String; }
# impl Foo for u8 { fn method(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn method(&self) -> String { format!("string: {}", *self) } }
fn do_something<T: Foo>(x: T) {
    x.method();
}

fn main() {
    let x = 5u8;
    let y = "Hello".to_string();

    do_something(x);
    do_something(y);
}
```

在这里 Rust 用“单态”来进行静态分发。这意味着 Rust 会为`u8`和`String`分别创建一个特殊版本的的`do_something()`，然后将对`do_something`的调用替换为这些特殊函数。也就是说，Rust 生成了一些像这样的函数：

```rust
# trait Foo { fn method(&self) -> String; }
# impl Foo for u8 { fn method(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn method(&self) -> String { format!("string: {}", *self) } }
fn do_something_u8(x: u8) {
    x.method();
}

fn do_something_string(x: String) {
    x.method();
}

fn main() {
    let x = 5u8;
    let y = "Hello".to_string();

    do_something_u8(x);
    do_something_string(y);
}
```

这样做的一个很大的优点在于：静态分发允许函数被内联调用，因为调用者在编译时就知道它，内联对编译器进行代码优化十分有利。静态分发能提高程序的运行效率，不过相应的也有它的弊端：会导致“代码膨胀”（code bloat）。因为在编译出的二进制程序中，同样的函数，对于每个类型都会有不同的拷贝存在。

此外，编译器也不是完美的并且“优化”后的代码可能更慢。例如，过度的函数内联会导致指令缓存膨胀（缓存控制着我们周围的一切）。这也是为何要谨慎使用`#[inline]`和`#[inline(always)]`的部分原因。另外一个使用动态分发的原因是，在一些情况下，动态分发更有效率。

然而，常规情况下静态分发更有效率，并且我们总是可以写一个小的静态分发的封装函数来进行动态分发，不过反过来不行，这就是说静态调用更加灵活。因为这个原因标准库尽可能的使用了静态分发。

## 动态分发

Rust 通过一个叫做“trait 对象”的功能提供动态分发。比如说`&Foo`、`Box<Foo>`这些就是trait对象。它们是一些值，值中储存实现了特定 trait 的**任意**类型。它的具体类型只能在运行时才能确定。

从一些实现了特定`trait`的类型的指针中，可以从通过**转型**(casting)（例如，`&x as &Foo`）或者**强制转型**(coercing it)（例如，把`&x`当做参数传递给一个接收`&Foo`类型的函数）来取得trait对象。

这些 trait 对象的强制多态和转型也适用于类似于`&mut Foo`的`&mut T`以及`Box<Foo>`的`Box<T>`这样的指针，也就是目前为止我们讨论到的所有指针。强制转型和转型是一样的。

这个操作可以被看作“清除”编译器关于特定类型指针的信息，因此trait对象有时被称为“类型清除”（type erasure）。

回到上面的例子，我们可以使用相同的 trait，通过 trait 对象的转型（casting）来进行动态分发：

```rust
# trait Foo { fn method(&self) -> String; }
# impl Foo for u8 { fn method(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn method(&self) -> String { format!("string: {}", *self) } }

fn do_something(x: &Foo) {
    x.method();
}

fn main() {
    let x = 5u8;
    do_something(&x as &Foo);
}
```

或者通过强制转型（by concercing）：

```rust
# trait Foo { fn method(&self) -> String; }
# impl Foo for u8 { fn method(&self) -> String { format!("u8: {}", *self) } }
# impl Foo for String { fn method(&self) -> String { format!("string: {}", *self) } }

fn do_something(x: &Foo) {
    x.method();
}

fn main() {
    let x = "Hello".to_string();
    do_something(&x);
}
```

一个使用trait对象的函数并没有为每个实现了`Foo`的类型专门生成函数：它只有一份函数的代码，一般（但不总是）会减少代码膨胀。然而，因为调用虚函数，会带来更大的运行时开销，也会大大地阻止任何内联以及相关优化的进行。

### 为什么用指针？

和很多托管语言不一样，Rust 默认不用指针来存放数据，因此类型有着不同的大小。在编译时知道值的大小（size），以及了解把值作为参数传递给函数、值在栈上移动、值在堆上分配（或释放）并储存等情况，对于 Rust 程序员来说是很重要的。

对于`Foo`，我们需要一个值至少是一个`String`（24字节）或一个`u8`（1字节），或者其它crate中可能实现了`Foo`（任意字节）的其他类型。如果值没有使用指针存储，我们无法保证代码能对其他类型正常运作，因为其它类型可以是任意大小的。

用指针来储存值意味着当我们使用 trait 对象时值的大小（size）是无关的，只与指针的大小（size）有关。

### 表现（Representation）

可以在一个 trait 对象上通过一个特殊的函数指针的记录调用的特性函数通常叫做“虚函数表”（由编译器创建和管理）。

trait 对象既简单又复杂：它的核心表现和设计是十分直观的，不过这有一些难懂的错误信息和诡异行为有待发掘。

让我们从一个简单的，带有 trait 对象的运行时表现开始。`std::raw`模块包含与复杂的内建类型有相同结构的结构体，[包括trait对象](http://doc.rust-lang.org/std/raw/struct.TraitObject.html)：

```rust
# mod foo {
pub struct TraitObject {
    pub data: *mut (),
    pub vtable: *mut (),
}
# }
```

这就是了，一个trait对象就像包含一个“数据”指针和“虚函数表”指针的`&Foo`。

数据指针指向 trait 对象保存的数据（某个未知的类型`T`），和一个虚表指针指向对应`T`的`Foo`实现的虚函数表。

一个虚表本质上是一个函数指针的结构体，指向每个函数实现的具体机器码。一个像`trait_object.method()`的函数调用会从虚表中取出正确的指针然后进行一个动态调用。例如：

```rust
struct FooVtable {
    destructor: fn(*mut ()),
    size: usize,
    align: usize,
    method: fn(*const ()) -> String,
}

// u8:

fn call_method_on_u8(x: *const ()) -> String {
    // The compiler guarantees that this function is only called
    // with `x` pointing to a u8.
    let byte: &u8 = unsafe { &*(x as *const u8) };

    byte.method()
}

static Foo_for_u8_vtable: FooVtable = FooVtable {
    destructor: /* compiler magic */,
    size: 1,
    align: 1,

    // Cast to a function pointer:
    method: call_method_on_u8 as fn(*const ()) -> String,
};


// String:

fn call_method_on_String(x: *const ()) -> String {
    // The compiler guarantees that this function is only called
    // with `x` pointing to a String.
    let string: &String = unsafe { &*(x as *const String) };

    string.method()
}

static Foo_for_String_vtable: FooVtable = FooVtable {
    destructor: /* compiler magic */,
    // Values for a 64-bit computer, halve them for 32-bit ones
    size: 24,
    align: 8,

    method: call_method_on_String as fn(*const ()) -> String,
};
```

在每个虚表中的`destructor`字段指向一个会清理虚表类型的任何资源的函数，对于`u8`是普通的，不过对于`String`它会释放内存。这对于像`Box<Foo>`这类有所有权的trait对象来说是必要的，它需要在离开作用域后清理`Box`以及它内部的类型所分配的。`size`和`align`字段储存需要清除类型的大小和它的对齐需求。

假设我们有一些实现了`Foo`的值，那么显式的创建和使用`Foo`trait对象可能看起来有点像这个（忽略不匹配的类型，它们只是指针而已）：

```rust
let a: String = "foo".to_string();
let x: u8 = 1;

// let b: &Foo = &a;
let b = TraitObject {
    // Store the data:
    data: &a,
    // Store the methods:
    vtable: &Foo_for_String_vtable
};

// let y: &Foo = x;
let y = TraitObject {
    // Store the data:
    data: &x,
    // Store the methods:
    vtable: &Foo_for_u8_vtable
};

// b.method();
(b.vtable.method)(b.data);

// y.method();
(y.vtable.method)(y.data);
```

## 对象安全（Object Safety）

并不是所有 trait 都可以被用来作为一个 trait 对象。例如，vector 实现了`Clone`，不过如果我们尝试创建一个 trait 对象：

```rust
let v = vec![1, 2, 3];
let o = &v as &Clone;
```

我们得到一个错误：

```text
error: cannot convert to a trait object because trait `core::clone::Clone` is not object-safe [E0038]
let o = &v as &Clone;
        ^~
note: the trait cannot require that `Self : Sized`
let o = &v as &Clone;
        ^~
```

错误表明`Clone`并不是“对象安全的（object-safe）”。只有对象安全的 trait 才能成为 trait 对象。一个对象安全的 trait 需要如下两条为真：

* trait 并不要求`Self: Sized`
* 所有的方法是对象安全的

那么什么让一个方法是对象安全的呢？每一个方法必须要求`Self: Sized`或者如下所有：

* 必须没有任何类型参数
* 必须不使用`Self`

好的。如你所见，几乎所有的规则都谈到了`Self`。一个直观的理解是“除了特殊情况，如果你的 trait 的方法使用了`Self`，它就不是对象安全的”。
