# 类型转换

> [casting-between-types.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/casting-between-types.md)
> <br>
> commit 40b9352aa371d5984c7e84cad5e2f597dbecf177

Rust，和它对安全的关注，提供了两种不同的在不同类型间转换的方式。第一个，`as`，用于安全转换。相反，`transmute`允许任意的转换，而这是 Rust 中最危险的功能之一！

## 强制转换（Coercion）

类型间的强制转换是隐式的并没有自己的语法，不过可以写作[`as`](#显式转换（explicit-coercions）)。

强转出现在`let`，`const`和`static`语句；函数调用参数；结构体初始化的字符值；和函数返回值中。

最常用的强转的例子是从引用中去掉可变性：

* `&mut T`到`&T`

一个相似的转换时去掉一个[裸指针](Raw Pointers 裸指针.md)的可变性：

* `*mut T`到`*const T`

引用也能被强转为裸指针：

* `&T`到`*const T`
* `&mut T`到`*mut T`

自定义强转可以用[`Deref`](`Deref` coercions `Deref`强制多态.md)定义。

强转是可传递的。

## `as`

`as`关键字进行安全的转换：

```rust
let x: i32 = 5;

let y = x as i64;
```

有三种形式的安全转换：显式转换，数字类型之间的转换，和指针转换。

转换并不是可传递的：即便是`e as U1 as U2`是一个有效的表达式，`e as U2`也不必要是（事实上只有在`U1`强转为`U2`时才有效）。

### 显式转换（Explicit coercions）

`e as U`是有效的仅当`e`是`T`类型而且`T`能强转为`U`。

### 数值转换

`e as U`的转换在如下情况下也是有效的：

* `e`是`T`类型而且`T`和`U`是任意数值类型：`numeric-cast`
* `e`是一个类 C 语言枚举（变量并没有附加值），而且`U`是一个整型：`enum-cast`
* `e`是`bool`或`char`而且`T`是一个整型：`prim-int-cast`
* `e`是`u8`而且`U`是`char`：`u8-char-cast`

例如：

```rust
let one = true as u8;
let at_sign = 64 as char;
let two_hundred = -56i8 as u8;
```

数值转换的语义是：

* 两个相同大小的整型之间（例如：`i32`->`u32`）的转换是一个`no-op`
* 从一个大的整型转换为一个小的整型（例如：`u32`->`u8`）会截断
* 从一个小的整型转换为一个大的整型（例如：`u8`->`u32`）会
  * 如果源类型是无符号的会补零（zero-extend）
  * 如果源类型是有符号的会符号（sign-extend）
* 从一个浮点转换为一个整型会向 0 舍入
  * [注意：目前如果舍入的值并不能用目标整型表示的话会导致未定义行为（Undefined Behavior）](https://github.com/rust-lang/rust/issues/10184)。这包括 Inf 和 NaN。这是一个 bug 并会被修复。
* 从一个整型转换为一个浮点会产生整型的浮点表示，如有必要会舍入（未指定舍入策略）
* 从 f32 转换为 f64 是完美无缺的
* 从 f64 转换为 f32 会产生最接近的可能值（未指定舍入策略）
  * [注意：目前如果值是有限的不过大于或小于 f32 所能表示的最大最小值会导致未定义行为（Undefined Behavior）](https://github.com/rust-lang/rust/issues/10184)。这是一个 bug 并会被修复。

### 指针转换

你也许会惊讶，[裸指针](Raw Pointers 裸指针.md)与整型之间的转换是安全的，而且不同类型的指针之间的转换遵循一些限制。只有解引用指针是不安全的：

```rust
let a = 300 as *const char; // `a` is a pointer to location 300.
let b = a as u32;
```

`e as U`在如下情况是一个有效的指针转换：

* `e`是`*T`类型，`U`是`*U_0`类型，且要么`U_0: Sized`要么`unsize_kind(T) == unsize_kind(U_0)`：`ptr-ptr-cast`
* `e`是`*T`类型且`U`是数值类型，同时`T: Sized`：`ptr-addr-cast`
* `e`是一个整型且`U`是`*U_0`类型，同时`U_0: Sized`：`addr-ptr-cast`
* `e`是`&[T; n]`类型且`U`是`*const T`类型：`array-ptr-cast`
* `e`是函数指针且`U`是`*T`类型，同时`T: Sized`：`fptr-ptr-cast`
* `e`是函数指针且`U`是一个整型：`fptr-addr-cast`


## `transmute`

`as`只允许安全的转换，并会拒绝例如尝试将 4 个字节转换为一个`u32`：

```rust
let a = [0u8, 0u8, 0u8, 0u8];

let b = a as u32; // Four u8s makes a u32.
```

这个错误为：

```text
error: non-scalar cast: `[u8; 4]` as `u32`
let b = a as u32; // Four u8s makes a u32.
        ^~~~~~~~
```

这是一个“非标量转换（non-scalar cast）”因为这里我们有多个值：四个元素的数组。这种类型的转换是非常危险的，因为他们假设多种底层结构的实现方式。为此，我们需要一些更危险的东西。

`transmute`函数由[编译器固有功能](Intrinsics 固有功能.md)提供，它做的工作非常简单，不过非常可怕。它告诉Rust对待一个类型的值就像它是另一个类型一样。它这样做并不管类型检查系统，并完全信任你。

在我们之前的例子中，我们知道一个有4个`u8`的数组可以正常代表一个`u32`，并且我们想要进行转换。使用`transmute`而不是`as`，Rust允许我们：

```rust
use std::mem;

fn main() {
    unsafe {
        let a = [0u8, 1u8, 0u8, 0u8];
        let b = mem::transmute::<[u8; 4], u32>(a);
        println!("{}", b); // 256
        // Or, more concisely:
        let c: u32 = mem::transmute(a);
        println!("{}", c); // 256
    }
}
```

为了使它编译通过我们要把这些操作封装到一个`unsafe`块中。技术上讲，只有`mem::transmute`调用自身需要位于块中，不过在这个情况下包含所有相关的内容是有好处的，这样你就知道该看哪了。在这例子中，`a`的细节也是重要的，所以它们放到了块中。你会看到各种风格的代码，有时上下文离得太远，因此在`unsafe`中包含所有的代码并不是一个好主意。

虽然`transmute`做了非常少的检查，至少它确保了这些类型是相同大小的，这个错误：

```rust
use std::mem;

unsafe {
    let a = [0u8, 0u8, 0u8, 0u8];

    let b = mem::transmute::<[u8; 4], u64>(a);
}
```

和：

```text
error: transmute called with differently sized types: [u8; 4] (32 bits) to u64
(64 bits)
```

除了这些，你可以自行随意转换，只能帮你这么多了！
