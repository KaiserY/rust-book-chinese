# const 和 static

> [const-and-static.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/const-and-static.md)
> <br>
> commit d001e8ad179b2d0b57272d1c875d93099fc347cb

Rust 有一个用`const`关键字定义常量的方法：

```rust
const N: i32 = 5;
```

与[let](Variable Bindings 变量绑定.md)绑定不同，你必须标注一个`const`的类型。

常量贯穿于整个程序的生命周期。更具体的，Rust 中的常量并没有固定的内存地址。这是因为实际上它们会被内联到用到它们的地方。为此对同一常量的引用并不能保证引用到相同的内存地址。

## `static`

Rust 以静态量的方式提供了类似“全局变量”的功能。它们与常量类似，不过静态量在使用时并不内联。这意味着对每一个值只有一个实例，并且位于内存中的固定位置。

这是一个例子：

```rust
static N: i32 = 5;
```

与[let](Variable Bindings 变量绑定.md)绑定不同，你必须标注一个`static`的类型。

静态量贯穿于整个程序的生命周期，因此任何存储在常量中的引用有一个[`'static`生命周期](Lifetimes 生命周期.md)：

```rust
static NAME: &'static str = "Steve";
```

## 可变性
你可以用`mut`关键字引入可变性：

```rust
static mut N: i32 = 5;
```

因为这是可变的，一个线程可能在更新`N`同时另一个在读取它，导致内存不安全。因此访问和改变一个`static mut`是[不安全（unsafe）](`unsafe` 不安全代码.md)的，因此必须在`unsafe`块中操作：

```rust
# static mut N: i32 = 5;

unsafe {
    N += 1;

    println!("N: {}", N);
}
```

更进一步，任何存储在`static`的类型必须实现`Sync`。

## 初始化
`const`和`static`都要求赋予它们一个值。它们必须只能被赋予一个常量表达式的值。换句话说，你不能用一个函数调用的返回值或任何相似的复合值或在运行时赋值。

## 我应该用哪个？（Which construct should I use?）
几乎所有时候，如果你可以在两者之间选择，选择`const`。实际上你很少需要你的常量关联一个内存位置，而且使用`const`允许你不止在在自己的包装箱还可以在下游包装箱中使用像常数扩散这样的优化。

一个常量可以看作一个C中的`#define`：它有元数据开销但无运行时开销。“我应该在C中用一个#define还是一个static呢？”大体上与在Rust你应该用常量还是静态量是一个问题。
