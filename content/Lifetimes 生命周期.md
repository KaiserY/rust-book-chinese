# 生命周期

> [lifetimes.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/lifetimes.md)
> <br>
> commit ee2abe85a6ed9dacd03c4113445053b9bde1ccaf

这篇教程是现行 3 个 Rust 所有权系统章节的第三部分。所有权系统是 Rust 最独特且最引人入胜的特性之一，也是作为 Rust 开发者应该熟悉的。Rust 所追求最大的目标 -- 内存安全，关键在于所有权。所有权系统有一些不同的概念，每个概念独自成章：

* [所有权](5.8.Ownership 所有权.md)，关键章节
* [借用](5.9.References and Borrowing 引用和借用.md)，以及它关联的特性: "引用" (references)
* 生命周期，你正在阅读的这个章节

这 3 章依次互相关联，你需要完整地阅读全部 3 章来对 Rust 的所有权系统进行全面的了解。

## 原则（Meta）
在我们开始详细讲解之前，这有两点关于所有权系统重要的注意事项。

Rust 注重安全和速度。它通过很多**零开销抽象**（*zero-cost abstractions*）来实现这些目标，也就是说在 Rust 中，实现抽象的开销尽可能的小。所有权系统是一个典型的零开销抽象的例子。本文提到所有的分析都是**在编译时完成的**。你不需要在运行时为这些功能付出任何开销。

然而，这个系统确实有一个开销：学习曲线。很多 Rust 初学者会经历我们所谓的“与借用检查器作斗争”的过程，也就是指 Rust 编译器拒绝编译一个作者认为合理的程序。这种“斗争”会因为程序员关于所有权系统如何工作的基本模型与 Rust 实现的实际规则不匹配而经常发生。当你刚开始尝试 Rust 的时候，你很可能会有相似的经历。然而有一个好消息：更有经验的 Rust 开发者反映，一旦他们适应所有权系统一段时间之后，与借用检查器的冲突会越来越少。

记住这些之后，让我们来学习有关生命周期的内容。

## 生命周期
借出一个其它人所有资源的引用可以是很复杂的。例如，想象一下下列操作：

* 我获取了一个某种资源的句柄
* 我借给你了一个关于这个资源的引用
* 我决定不再需要这个资源了，然后释放了它，这时你仍然持有它的引用
* 你决定使用这个资源

噢！你的引用指向一个无效的资源。这叫做**悬垂指针**（*dangling pointer*）或者“释放后使用”，如果这个资源是内存的话。这种状况的一个小例子像这样：

```rust
let r;              // Introduce reference: `r`.
{
    let i = 1;      // Introduce scoped value: `i`.
    r = &i;         // Store reference of `i` in `r`.
}                   // `i` goes out of scope and is dropped.

println!("{}", r);  // `r` still refers to `i`.
```

要修正这个问题的话，我们必须确保第四步永远也不在第三步之后发生。在上面的小例子中 Rust 编译器能够报告问题因为它能识别出函数中不同变量的生命周期。

当我们有一个将引用作为参数的函数时情况就变得更复杂了。考虑如下例子：

```rust,compile_fail,E0106
fn skip_prefix(line: &str, prefix: &str) -> &str {
    // ...
#   line
}

let line = "lang:en=Hello World!";
let lang = "en";

let v;
{
    let p = format!("lang:{}=", lang);  // -+ `p` comes into scope.
    v = skip_prefix(line, p.as_str());  //  |
}                                       // -+ `p` goes out of scope.
println!("{}", v);
```

这里我们有个函数`skip_prefix`，它获取两个`&str`引用作为参数并返回一个`&str`引用。通过`line`和`p`的引用调用它：两个有不同生命周期的变量。现在`println!`那行代码的安全依赖于`skip_prefix`函数返回的引用是仍然存在的`line`还是已经释放掉的`p`。

因为存在上述的歧义，Rust 将会拒绝编译示例代码。为了继续我们需要向编译器提供更多关于引用生命周期的信息。这可以通过再函数签名中显式标明生命周期来完成：

```rust
fn skip_prefix<'a, 'b>(line: &'a str, prefix: &'b str) -> &'a str {
    // ...
#   line
}
```

让我们看看所做的修改，但是现在并不深入到语法--之后我们会讲到。第一个修改是再方法名后面加入了`<'a, 'b>`。这引入了两个生命周期参数`'a`和`'b`。接下来函数签名中的每个引用都关联了一个生命周期参数，通过再`&`之后加上生命周期的名字。这告诉了编译器不同引用的生命周期是如何关联的。

这样编译器就能推断出`skip_prefix`函数的返回值与`line`参数有着相同的生命周期，这样就使得之前例子中`v`引用即使在`p`离开作用域之后也能安全使用。

另外编译器能够检查`skip_prefix`返回值的用途，它也能确保之后的实现也遵守函数声明建立的约束。这在实现[之后会介绍到][traits]的 trait 时显得尤为实用。

[traits]: Traits.md

**注意：** 认识到生命周期注解是`_descriptive_`（描述性）而不是`_prescriptive_`（规定性）是很重要的（译者注：各位可以自行搜索这两个术语）。这意味着引用的生命周期是由代码决定的，而不是由生命周期注解决定的。注解，提供了供编译器用来检查引用的有效性的信息。编译器在简单的情况下无需注解就能进行这种检查，不过在复杂的场景需要程序猿的协助。

## 语法

`'a`读作“生命周期 a”。技术上讲，每一个引用都有一些与之相关的生命周期，不过编译器在通常情况让你可以省略（也就是，省略，查看下面的[生命周期省略](#生命周期省略（lifetime-elision）)）它们。在我们讲到它之前，让我们看看一个显式生命周期的例子：

```rust
fn bar<'a>(...)
```

之前我们讨论了一些[函数语法](Functions 函数.md)，不过我们并没有讨论函数名后面的`<>`。一个函数可以在`<>`之间有“泛型参数”，生命周期也是其中一种。我们在[本书的后面](Generics 泛型.md)讨论其他类型的泛型。不过现在让我们着重看生命周期。

我们用`<>`声明了生命周期。这是说`bar`有一个生命周期`'a`。如果我们有两个拥有不同生命周期的引用参数，它应该看起来像这样：

```rust
fn bar<'a, 'b>(...)
```

接着在我们的参数列表中，我们使用了我们命名的生命周期：

```rust
...(x: &'a i32)
```

如果我们想要一个`&mut`引用，我们这么做：

```rust
...(x: &'a mut i32)
```

如果你对比一下`&mut i32`和`&'a mut i32`，他们是一样的，只是后者在`&`和`mut i32`之间夹了一个`'a`生命周期。`&mut i32`读作“一个`i32`的可变引用”，而`&'a mut i32`读作“一个带有生命周期'a的i32的可变引用”。

## 在`struct`中

当你处理[结构体](5.12.Structs 结构体.md)时你也需要显式的生命周期：

```rust
struct Foo<'a> {
    x: &'a i32,
}

fn main() {
    let y = &5; // This is the same as `let _y = 5; let y = &_y;`.
    let f = Foo { x: y };

    println!("{}", f.x);
}
```

如你所见，`struct`也可以有生命周期。跟函数类似的方法，

```rust
struct Foo<'a> {
# x: &'a i32,
# }
```

声明一个生命周期，接着

```rust
# struct Foo<'a> {
x: &'a i32,
# }
```

使用它。然而为什么这里我们需要一个生命周期呢？因为我们需要确保任何`Foo`的引用不能比它包含的`i32`的引用活的更久。

## `impl`块

让我们在`Foo`中实现一个方法：

```rust
struct Foo<'a> {
    x: &'a i32,
}

impl<'a> Foo<'a> {
    fn x(&self) -> &'a i32 { self.x }
}

fn main() {
    let y = &5; // This is the same as `let _y = 5; let y = &_y;`.
    let f = Foo { x: y };

    println!("x is: {}", f.x());
}
```

如你所见，我们需要在`impl`行为`Foo`声明一个生命周期。我们重复了`'a`两次，就像在函数中：`impl<'a>`定义了一个生命周期`'a`，而`Foo<'a>`使用它。

## 多个生命周期

如果你有多个引用，你可以多次使用同一个生命周期：

```rust
fn x_or_y<'a>(x: &'a str, y: &'a str) -> &'a str {
#    x
# }
```

这意味着`x`和`y`存活在同样的作用域内，并且返回值也同样存活在这个作用域内。如果你想要`x`和`y`有不同的生命周期，你可以使用多个生命周期参数：

```rust
fn x_or_y<'a, 'b>(x: &'a str, y: &'b str) -> &'a str {
#    x
# }
```

在这个例子中，`x`和`y`有不同的有效的作用域，不过返回值和`x`有相同的生命周期

## 理解作用域（Thinking in scopes）
理解生命周期的一个办法是想象一个引用有效的作用域。例如：

```rust
fn main() {
    let y = &5;     // -+ `y` comes into scope.
                    //  |
    // Stuff...     //  |
                    //  |
}                   // -+ `y` goes out of scope.
```

加入我们的`Foo`：

```rust
struct Foo<'a> {
    x: &'a i32,
}

fn main() {
    let y = &5;           // -+ `y` comes into scope.
    let f = Foo { x: y }; // -+ `f` comes into scope.
    // Stuff...           //  |
                          //  |
}                         // -+ `f` and `y` go out of scope.
```

我们的`f`生存在`y`的作用域之中，所以一切正常。那么如果不是呢？下面的代码不能工作：

```rust
struct Foo<'a> {
    x: &'a i32,
}

fn main() {
    let x;                    // -+ `x` comes into scope.
                              //  |
    {                         //  |
        let y = &5;           // ---+ `y` comes into scope.
        let f = Foo { x: y }; // ---+ `f` comes into scope.
        x = &f.x;             //  | | This causes an error.
    }                         // ---+ `f` and y go out of scope.
                              //  |
    println!("{}", x);        //  |
}                             // -+ `x` goes out of scope.
```

噢！就像你在这里看到的一样，`f`和`y`的作用域小于`x`的作用域。不过当我们尝试`x = &f.x`时，我们让`x`引用一些将要离开作用域的变量。

命名作用域用来赋予作用域一个名字。有了名字是我们可以谈论它的第一步。

## 'static
叫做`static`的作用域是特殊的。它代表某样东西具有横跨整个程序的生命周期。大部分 Rust 程序员当他们处理字符串时第一次遇到`'static`：

```rust
let x: &'static str = "Hello, world.";
```

基本字符串是`&'static str`类型的因为它的引用一直有效：它们被写入了最终库文件的数据段。另一个例子是全局量：

```rust
static FOO: i32 = 5;
let x: &'static i32 = &FOO;
```

它在二进制文件的数据段中保存了一个`i32`，而`x`是它的一个引用。

## 生命周期省略（Lifetime Elision）
Rust支持强大的在函数体中的局部类型推断，不过这在项签名中是禁止的以便允许只通过项签名本身推导出类型。然而，出于人体工程学方面的考虑，有第二个非常限制的叫做“生命周期省略”的推断算法适用于函数签名。它只基于签名部分自身推断而不涉及函数体，只推断生命周期参数，并且只基于 3 个易于记忆和无歧义的规则，虽然并不隐藏它涉及到的实际类型，因为局部推断可能会适用于它。

当我们讨论生命周期省略的时候，我们使用**输入生命周期和输出生命周期**（*input lifetime and output lifetime.*）。**输入生命周期**是关于函数参数的，而**输出生命周期**是关于函数返回值的。例如，这个函数有一个输入生命周期：

```rust
fn foo<'a>(bar: &'a str)
```

这个有一个输出生命周期：

```rust
fn foo<'a>() -> &'a str
```

这个两者皆有：

```rust
fn foo<'a>(bar: &'a str) -> &'a str
```

有3条规则：

* 每一个被省略的函数参数成为一个不同的生命周期参数。
* 如果刚好有一个输入生命周期，不管是否省略，这个生命周期被赋予所有函数返回值中被省略的生命周期。
* 如果有多个输入生命周期，不过它们当中有一个是`&self`或者`&mut self`，`self`的生命周期被赋予所有省略的输出生命周期。

否则，省略一个输出生命周期将是一个错误。

## 例子
这里有一些省略了生命周期的函数的例子。我们用它们的扩展形式配对了每个省略了生命周期的例子。

```rust,ignore
fn print(s: &str); // elided
fn print<'a>(s: &'a str); // expanded

fn debug(lvl: u32, s: &str); // elided
fn debug<'a>(lvl: u32, s: &'a str); // expanded
```

在上面的例子中，`lvl`并不需要一个生命周期，因为它不是一个引用（`&`）。只有与引用（例如一个包含引用的`struct`）相关的变量才需要生命周期。

```rust,ignore
fn substr(s: &str, until: u32) -> &str; // elided
fn substr<'a>(s: &'a str, until: u32) -> &'a str; // expanded

fn get_str() -> &str; // ILLEGAL, no inputs

fn frob(s: &str, t: &str) -> &str; // ILLEGAL, two inputs
fn frob<'a, 'b>(s: &'a str, t: &'b str) -> &str; // Expanded: Output lifetime is ambiguous

fn get_mut(&mut self) -> &mut T; // elided
fn get_mut<'a>(&'a mut self) -> &'a mut T; // expanded

fn args<T: ToCStr>(&mut self, args: &[T]) -> &mut Command; // elided
fn args<'a, 'b, T: ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command; // expanded

fn new(buf: &mut [u8]) -> BufWriter; // elided
fn new<'a>(buf: &'a mut [u8]) -> BufWriter<'a>; // expanded
```
