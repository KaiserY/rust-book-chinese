# 裸指针

> [raw-pointers.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/raw-pointers.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

Rust 的标准库中有一系列不同的智能指针类型，不过这有两个类型是十分特殊的。Rust的安全大多来源于编译时检查，不过裸指针并没有这样的保证，使用它们是[`unsafe`](`unsafe` 不安全代码.md)的。

`*const T`和`*mut T`在Rust中被称为“裸指针”。有时当编写特定类型的库时，为了某些原因你需要绕过Rust的安全保障。在这种情况下，你可以使用裸指针来实现你的库，同时暴露一个安全的接口给你的用户。例如，`*`指针允许别名，允许用来写共享所有权类型，甚至是内存安全的共享内存类型（`Rc<T>`和`Arc<T>`类型都是完全用Rust实现的）。

有一些你需要记住的裸指针不同于其它指针的地方。它们是：

* 不能保证指向有效的内存，甚至不能保证是非空的（不像`Box`和`&`）；
* 没有任何自动清除，不像`Box`，所以需要手动管理资源；
* 是普通旧式类型，也就是说，它不移动所有权，这又不像`Box`，因此Rust编译器不能保证不出像释放后使用这种bug；
* ~~被认为是可发送的（如果它的内容是可发送的），因此编译器不能提供帮助确保它的使用是线程安全的；例如，你可以从两个线程中并发的访问`*mut i32`而不用同步。~~
* 缺少任何形式的生命周期，不像`&`，因此编译器不能判断出悬垂指针；
* 除了不允许直接通过`*const T`改变外，没有别名或可变性的保障。

## 基础

创建一个裸指针是非常安全的：

```rust
let x = 5;
let raw = &x as *const i32;

let mut y = 10;
let raw_mut = &mut y as *mut i32;
```

然而，解引用它则不行。这个并不能工作：

```rust
let x = 5;
let raw = &x as *const i32;

println!("raw points at {}", *raw);
```

它给出这个错误：

```text
error: dereference of raw pointer requires unsafe function or block [E0133]
     println!("raw points at {}", *raw);
                                  ^~~~
```

当你解引用一个裸指针，你要为它并不指向正确的地方负责。为此，你需要`unsafe`：

```rust
let x = 5;
let raw = &x as *const i32;

let points_at = unsafe { *raw };

println!("raw points at {}", points_at);
```

关于裸指针的更多操作，查看[它们的API文档](http://doc.rust-lang.org/stable/std/primitive.pointer.html)。

## FFI
裸指针在FFI中很有用：Rust的`*const T`和`*mut T`分别与C中的`const T*`和`T*`类似。关于它们的应用，查看[FFI章节](Foreign Function Interface 外部函数接口.md)。

## 引用和裸指针
在运行时，指向一份相同数据的裸指针`*`和引用有相同的表现。事实上，在安全代码中`&T`引用会隐式的转换为一个`*const T`同时它们的`mut`变体也有类似的行为（这两种转换都可以显式执行，分别为`value as *const T`和`value as *mut T`）。

反其道而行之，从`*const`到`&`引用，是不安全的。一个`&T`总是有效的，所以，最少，`*const T`裸指针必须指向一个`T`的有效实例。进一步，结果指针必须满足引用的别名和可变性法则。编译器假设这些属性对任何引用都是有效的，不管它们是如何创建的，因而所以任何从裸指针来的转换都断言它们成立。程序员**必须**保证它。

推荐的转换方法是

```rust
// Explicit cast:
let i: u32 = 1;
let p_imm: *const u32 = &i as *const u32;

// Implicit coercion:
let mut m: u32 = 2;
let p_mut: *mut u32 = &mut m;

unsafe {
    let ref_imm: &u32 = &*p_imm;
    let ref_mut: &mut u32 = &mut *p_mut;
}
```

与使用`transmute`相比更倾向于`&*x`解引用风格。`transmute`远比需要的强大，并且（解引用）更受限的操作会更难以错误使用；例如，它要求`x`是一个指针（不像`transmute`）。
