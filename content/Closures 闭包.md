# 闭包

> [closures.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/closures.md)
> <br>
> commit 57b53fdd9ef34bf4015b4fd7a202dfa485773c96

有时为了整洁和复用打包一个函数和**自由变量**（*free variables*）是很有用的。自由变量是指被用在函数中来自函数内部作用域并只用于函数内部的变量。对此，我们用一个新名字“闭包”而且 Rust 提供了大量关于他们的实现，正如我们将看到的。

## 语法
闭包看起来像这样：

```rust
let plus_one = |x: i32| x + 1;

assert_eq!(2, plus_one(1));
```

我们创建了一个绑定，`plus_one`，并把它赋予一个闭包。闭包的参数位于管道（`|`）之中，而闭包体是一个表达式，在这个例子中，`x + 1`。记住`{}`是一个表达式，所以我们也可以拥有包含多行的闭包：

```rust
let plus_two = |x| {
    let mut result: i32 = x;

    result += 1;
    result += 1;

    result
};

assert_eq!(4, plus_two(2));
```

你会注意到闭包的一些方面与用`fn`定义的常规函数有点不同。第一个是我们并不需要标明闭包接收和返回参数的类型。我们可以：

```rust
let plus_one = |x: i32| -> i32 { x + 1 };

assert_eq!(2, plus_one(1));
```

不过我们并不需要这么写。为什么呢？基本上，这是出于“人体工程学”的原因。因为为命名函数指定全部类型有助于像文档和类型推断，而闭包的类型则很少有文档因为它们是匿名的，并且并不会产生像推断一个命名函数的类型这样的“远距离错误”。

第二个是语法是相似的，不过有点不同。我会增加空格来使它们看起来更像一点：

```rust
fn  plus_one_v1   (x: i32) -> i32 { x + 1 }
let plus_one_v2 = |x: i32| -> i32 { x + 1 };
let plus_one_v3 = |x: i32|          x + 1  ;
```

有些小区别，不过仍然是相似的。

## 闭包及环境
之所以把它称为“闭包”是因为它们“包含在环境中”（close over their environment）。这看起来像：

```rust
let num = 5;
let plus_num = |x: i32| x + num;

assert_eq!(10, plus_num(5));
```

这个闭包，`plus_num`，引用了它作用域中的`let`绑定：`num`。更明确的说，它借用了绑定。如果我们做一些会与这个绑定冲突的事，我们会得到一个错误。比如这个：

```rust
let mut num = 5;
let plus_num = |x: i32| x + num;

let y = &mut num;
```

错误是：

```text
error: cannot borrow `num` as mutable because it is also borrowed as immutable
    let y = &mut num;
                 ^~~
note: previous borrow of `num` occurs here due to use in closure; the immutable
  borrow prevents subsequent moves or mutable borrows of `num` until the borrow
  ends
    let plus_num = |x| x + num;
                   ^~~~~~~~~~~
note: previous borrow ends here
fn main() {
    let mut num = 5;
    let plus_num = |x| x + num;

    let y = &mut num;
}
^
```

一个啰嗦但有用的错误信息！如它所说，我们不能取得一个`num`的可变借用因为闭包已经借用了它。如果我们让闭包离开作用域，我们可以：

```rust
let mut num = 5;
{
    let plus_num = |x: i32| x + num;

} // `plus_num` goes out of scope; borrow of `num` ends.

let y = &mut num;
```

然而，如果你的闭包需要它，Rust会取得所有权并移动环境。这个不能工作：

```rust
let nums = vec![1, 2, 3];

let takes_nums = || nums;

println!("{:?}", nums);
```

这会给我们：

```text
note: `nums` moved into closure environment here because it has type
  `[closure(()) -> collections::vec::Vec<i32>]`, which is non-copyable
let takes_nums = || nums;
                    ^~~~~~~
```

`Vec<T>`拥有它内容的所有权，而且由于这个原因，当我们在闭包中引用它时，我们必须取得`nums`的所有权。这与我们传递`nums`给一个取得它所有权的函数一样。

## `move`闭包
我们可以使用`move`关键字强制使我们的闭包取得它环境的所有权：

```rust
let num = 5;

let owns_num = move |x: i32| x + num;
```

现在，即便关键字是`move`，变量遵循正常的移动语义。在这个例子中，`5`实现了`Copy`，所以`owns_num`取得一个`5`的拷贝的所有权。那么区别是什么呢？

```rust
let mut num = 5;

{
    let mut add_num = |x: i32| num += x;

    add_num(5);
}

assert_eq!(10, num);
```

那么在这个例子中，我们的闭包取得了一个`num`的可变引用，然后接着我们调用了`add_num`，它改变了其中的值，正如我们期望的。我们也需要将`add_num`声明为`mut`，因为我们会改变它的环境。

如果我们改为一个`move`闭包，这有些不同：

```rust
let mut num = 5;

{
    let mut add_num = move |x: i32| num += x;

    add_num(5);
}

assert_eq!(5, num);
```

我们只会得到`5`。与其获取一个我们`num`的可变借用，我们取得了一个拷贝的所有权。

另一个理解`move`闭包的方法：它给出了一个拥有自己栈帧的闭包。没有`move`，一个闭包可能会绑定在创建它的栈帧上，而`move`闭包则是独立的。例如，这意味着大体上你不能从函数返回一个非`move`闭包。

不过在我们讨论获取或返回闭包之前，我们应该更多的了解一下闭包实现的方法。作为一个系统语言，Rust给予你了大量的控制你代码的能力，而闭包也是一样。

## 闭包实现

Rust 的闭包实现与其它语言有些许不同。它们实际上是trait的语法糖。在此之前你可能要确定已经读过[trait章节](Traits.md)和[trait对象](Trait Objects trait 对象.md)。

都读过了？很好。

理解闭包底层是如何工作的关键有点奇怪：使用`()`调用函数，像`foo()`，是一个可重载的运算符。到此，其它的一切都会明了。在Rust中，我们使用trait系统来重载运算符。调用函数也不例外。我们有三个trait来分别重载：

```rust
# #![feature(unboxed_closures)]
# mod foo {
pub trait Fn<Args> : FnMut<Args> {
    extern "rust-call" fn call(&self, args: Args) -> Self::Output;
}

pub trait FnMut<Args> : FnOnce<Args> {
    extern "rust-call" fn call_mut(&mut self, args: Args) -> Self::Output;
}

pub trait FnOnce<Args> {
    type Output;

    extern "rust-call" fn call_once(self, args: Args) -> Self::Output;
}
# }
```

你会注意到这些 trait 之间的些许区别，不过一个大的区别是`self`：`Fn`获取`&self`，`FnMut`获取`&mut self`，而`FnOnce`获取`self`。这包含了所有3种通过通常函数调用语法的`self`。不过我们将它们分在 3 个 trait 里，而不是单独的 1 个。这给了我们大量的对于我们可以使用哪种闭包的控制。

闭包的`|| {}`语法是上面 3 个 trait 的语法糖。Rust 将会为了环境创建一个结构体，`impl`合适的 trait，并使用它。

## 闭包作为参数（Taking closures as arguments）

现在我们知道了闭包是 trait，我们已经知道了如何接受和返回闭包；就像任何其它的 trait！

这也意味着我们也可以选择静态或动态分发。首先，让我们写一个函数，它接受可调用的参数，调用之，然后返回结果：

```rust
fn call_with_one<F>(some_closure: F) -> i32
    where F: Fn(i32) -> i32 {

    some_closure(1)
}

let answer = call_with_one(|x| x + 2);

assert_eq!(3, answer);
```

我们传递我们的闭包，`|x| x + 2`，给`call_with_one`。它正做了我们说的：它调用了闭包，`1`作为参数。

让我们更深层的解析`call_with_one`的签名：

```rust
fn call_with_one<F>(some_closure: F) -> i32
#    where F: Fn(i32) -> i32 {
#    some_closure(1) }
```

我们获取一个参数，而它有类型`F`。我们也返回一个`i32`。这一部分并不有趣。下一部分是：

```rust
# fn call_with_one<F>(some_closure: F) -> i32
    where F: Fn(i32) -> i32 {
#   some_closure(1) }
```

因为`Fn`是一个trait，我们可以用它限制我们的泛型。在这个例子中，我们的闭包取得一个`i32`作为参数并返回`i32`，所以我们用泛型限制是`Fn(i32) -> i32`。

还有一个关键点在于：因为我们用一个trait限制泛型，它会是单态的，并且因此，我们在闭包中使用静态分发。这是非常简单的。在很多语言中，闭包固定在堆上分配，所以总是进行动态分发。在Rust中，我们可以在栈上分配我们闭包的环境，并静态分发调用。这经常发生在迭代器和它们的适配器上，它们经常取得闭包作为参数。

当然，如果我们想要动态分发，我们也可以做到。trait对象处理这种情况，通常：

```rust
fn call_with_one(some_closure: &Fn(i32) -> i32) -> i32 {
    some_closure(1)
}

let answer = call_with_one(&|x| x + 2);

assert_eq!(3, answer);
```

现在我们取得一个trait对象，一个`&Fn`。并且当我们将我们的闭包传递给`call_with_one`时我们必须获取一个引用，所以我们使用`&||`。

下面是一个使用显式生命周期的闭包的例子。有时你可能需要一个获取这样引用的闭包：

```rust
fn call_with_ref<F>(some_closure:F) -> i32
    where F: Fn(&i32) -> i32 {

    let value = 0;
    some_closure(&value)
}
```

通常你可以指定闭包的参数的生命周期。我们可以在函数声明上指定它：

```rust,ignore
fn call_with_ref<'a, F>(some_closure:F) -> i32
    where F: Fn(&'a i32) -> i32 {
```

然而这导致了一个问题。当一个函数拥有一个显式生命周期参数，那个生命周期必须跟**整个**调用这个函数的生命周期一样长。借用检查器会抱怨说`value`的生命周期并不够长，因为它只位于声明后在函数体的作用域内。

我们需要的是只为它的参数借用其自己的作用域的闭包，而不是整个外层函数的作用域。为此，我们可以使用更高级的 Trait Bound，使用`for<...>`语法：

```rust
fn call_with_ref<F>(some_closure:F) -> i32
    where F: for<'a> Fn(&'a i32) -> i32 {
```

这会让 rust 编译器找到最小的生命周期来调用闭包并满足借用检查器的规则。我们的函数将能顺利编译：

```rust
fn call_with_ref<F>(some_closure:F) -> i32
    where F: for<'a> Fn(&'a i32) -> i32 {

    let value = 0;
    some_closure(&value)
}
```

## 函数指针和闭包

一个函数指针有点像一个没有环境的闭包。因此，你可以传递函数指针给任何期待闭包参数的函数，且能够工作：

```rust
fn call_with_one(some_closure: &Fn(i32) -> i32) -> i32 {
    some_closure(1)
}

fn add_one(i: i32) -> i32 {
    i + 1
}

let f = add_one;

let answer = call_with_one(&f);

assert_eq!(2, answer);
```

在这个例子中，我们并不是严格的需要这个中间变量`f`，函数的名字就可以了：

```rust
let answer = call_with_one(&add_one);
```

## 返回闭包（Returning closures）

对于函数式风格代码来说在各种情况返回闭包是非常常见的。如果你尝试返回一个闭包，你可能会得到一个错误。在刚接触的时候，这看起来有点奇怪，不过我们会搞清楚。当你尝试从函数返回一个闭包的时候，你可能会写出类似这样的代码：

```rust
fn factory() -> (Fn(i32) -> i32) {
    let num = 5;

    |x| x + num
}

let f = factory();

let answer = f(1);
assert_eq!(6, answer);
```

编译的时候会给出这一长串相关错误：

```text
error: the trait bound `core::ops::Fn(i32) -> i32 : core::marker::Sized` is not satisfied [E0277]
fn factory() -> (Fn(i32) -> i32) {
                ^~~~~~~~~~~~~~~~
note: `core::ops::Fn(i32) -> i32` does not have a constant size known at compile-time
fn factory() -> (Fn(i32) -> i32) {
                ^~~~~~~~~~~~~~~~
error: the trait bound `core::ops::Fn(i32) -> i32 : core::marker::Sized` is not satisfied [E0277]
let f = factory();
    ^
note: `core::ops::Fn(i32) -> i32` does not have a constant size known at compile-time
let f = factory();
    ^
```

为了从函数返回一些东西，Rust 需要知道返回类型的大小。不过`Fn`是一个 trait，它可以是各种大小(size)的任何东西。比如说，返回值可以是实现了`Fn`的任意类型。一个简单的解决方法是：返回一个引用。因为引用的大小(size)是固定的，因此返回值的大小就固定了。因此我们可以这样写：

```rust
fn factory() -> &(Fn(i32) -> i32) {
    let num = 5;

    |x| x + num
}

let f = factory();

let answer = f(1);
assert_eq!(6, answer);
```

不过这样会出现另外一个错误：

```text
error: missing lifetime specifier [E0106]
fn factory() -> &(Fn(i32) -> i32) {
                ^~~~~~~~~~~~~~~~~
```

对。因为我们有一个引用，我们需要给它一个生命周期。不过我们的`factory()`函数不接收参数，所以省略不能用在这。我们可以使用什么生命周期呢？`'static`：

```rust
fn factory() -> &'static (Fn(i32) -> i32) {
    let num = 5;

    |x| x + num
}

let f = factory();

let answer = f(1);
assert_eq!(6, answer);
```

不过这样又会出现另一个错误：

```text
error: mismatched types:
 expected `&'static core::ops::Fn(i32) -> i32`,
    found `[closure@<anon>:7:9: 7:20]`
(expected &-ptr,
    found closure) [E0308]
         |x| x + num
         ^~~~~~~~~~~

```

这个错误让我们知道我们并没有返回一个`&'static Fn(i32) -> i32`，而是返回了一个`[closure <anon>:7:9: 7:20]`。等等，什么？

因为每个闭包生成了它自己的环境`struct`并实现了`Fn`和其它一些东西，这些类型是匿名的。它们只在这个闭包中存在。所以Rust把它们显示为`closure <anon>`，而不是一些自动生成的名字。

这个错误也指出了返回值类型期望是一个引用，不过我们尝试返回的不是。更进一步，我们并不能直接给一个对象`'static`声明周期。所以我们换一个方法并通过`Box`装箱`Fn`来返回一个 trait 对象。这个**几乎**可以成功运行：

```rust
fn factory() -> Box<Fn(i32) -> i32> {
    let num = 5;

    Box::new(|x| x + num)
}


let f = factory();

let answer = f(1);
assert_eq!(6, answer);

```

这还有最后一个问题：

```text
error: closure may outlive the current function, but it borrows `num`,
which is owned by the current function [E0373]
Box::new(|x| x + num)
         ^~~~~~~~~~~
```

好吧，正如我们上面讨论的，闭包借用他们的环境。而且在这个例子中。我们的环境基于一个栈分配的`5`，`num`变量绑定。所以这个借用有这个栈帧的生命周期。所以如果我们返回了这个闭包，这个函数调用将会结束，栈帧也将消失，那么我们的闭包获得了被释放的内存环境！再有最后一个修改，我们就可以让它运行了：

```rust
fn factory() -> Box<Fn(i32) -> i32> {
    let num = 5;

    Box::new(move |x| x + num)
}

let f = factory();

let answer = f(1);
assert_eq!(6, answer);

```

通过把内部闭包变为`move Fn`，我们为闭包创建了一个新的栈帧。通过`Box`装箱，我们提供了一个已知大小的返回值，并允许它离开我们的栈帧。
