# 宏

> [macros.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/macros.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

到目前为止你已经学到了不少Rust提供的抽象和重用代码的工具了。这些代码重用单元有丰富的语义结构。例如，函数有类型签名，类型参数有特性限制并且能重载的函数必须属于一个特定的特性。

这些结构意味着Rust核心抽象拥有强大的编译时正确性检查。不过作为代价的是灵活性的减少。如果你识别出一个重复代码的模式，你会发现把它们解释为泛型函数，特性或者任何Rust语义中的其它结构很难或者很麻烦。

宏允许我们在**句法**水平上进行抽象。宏是一个“展开后的”句法形式的速记。这个展开发生在编译的早期，在任何静态检查之前。因此，宏可以实现很多Rust核心抽象不能做到的代码重用模式。

缺点是基于宏的代码更难懂，因为它很少利用Rust的内建规则。就像常规函数，一个良好的宏可以在不知道其实现的情况下使用。然而，设计一个良好的宏困难的！另外，在宏中的编译错误更难解释，因为它在展开后的代码上描述问题，不是在开发者使用的代码级别。

这些缺点让宏成了所谓“最后求助于的功能”。这并不是说宏的坏话；只是因为它是Rust中需要真正简明，良好抽象的代码的部分。切记权衡取舍。

## 定义一个宏
你可能见过`vec!`宏。用来初始化一个任意数量元素的[vector](Vectors.md)。

```rust
let x: Vec<u32> = vec![1, 2, 3];
# assert_eq!(x, [1, 2, 3]);
```

这不可能是一个常规函数，因为它可以接受任何数量的参数。不过我们可以想象的到它是这些代码的句法简写：

```rust
let x: Vec<u32> = {
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
};
# assert_eq!(x, [1, 2, 3]);
```

我们可以使用宏来实现这么一个简写：[^实际上]

```rust
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}
# fn main() {
#     assert_eq!(vec![1,2,3], [1, 2, 3]);
# }
```

哇哦，这里有好多新语法！让我们分开来看。

```rust
macro_rules! vec { ... }
```

这里我们定义了一个叫做`vec`的宏，跟用`fn vec`定义一个`vec`函数很相似。再罗嗦一句，我们通常写宏的名字时带上一个感叹号，例如`vec!`。感叹号是调用语法的一部分用来区别宏和常规函数。

## 匹配
宏通过一系列**规则**定义，它们是模式匹配的分支。上面我们有：

```rust
( $( $x:expr ),* ) => { ... };
```

这就像一个`match`表达式分支，不过匹配发生在编译时Rust的语法树中。最后一个分支（这里只有一个分支）的分号是可选的。`=>`左侧的“模式”叫**匹配器**（*matcher*）。它有[自己的语法](http://doc.rust-lang.org/reference.html#macros)。

`$x:expr`匹配器将会匹配任何Rust表达式，把它的语法树绑定到元变量`$x`上。`expr`标识符是一个**片段分类符**（*fragment specifier*）。在[宏进阶章节](http://doc.rust-lang.org/book/advanced-macros.html)（已被本章合并，坐等官方文档更新）中列举了所有可能的分类符。匹配器写在`$(...)`中，`*`会匹配0个或多个表达式，表达式之间用逗号分隔。

除了特殊的匹配器语法，任何出现在匹配器中的Rust标记必须完全相符。例如：

```rust
macro_rules! foo {
    (x => $e:expr) => (println!("mode X: {}", $e));
    (y => $e:expr) => (println!("mode Y: {}", $e));
}

fn main() {
    foo!(y => 3);
}
```

将会打印：

```text
mode Y: 3
```

而这个：

```rust
foo!(z => 3);
```

我们会得到编译错误：

```text
error: no rules expected the token `z`
```

## 展开
宏规则的右边是正常的Rust语法，大部分是。不过我们可以拼接一些匹配器中的语法。例如最开始的例子：

```rust
$(
    temp_vec.push($x);
)*
```

每个匹配的`$x`表达式都会在宏展开中产生一个单独`push`语句。展开中的重复与匹配器中的重复“同步”进行（稍后介绍更多）。

因为`$x`已经在表达式匹配中声明了，我们并不在右侧重复`:expr`。另外，我们并不将用来分隔的逗号作为重复操作的一部分。相反，我们在重复块中使用一个结束用的分号。

另一个细节：`vec!`宏的右侧有**两对**大括号。它们经常像这样结合起来：

```rust
macro_rules! foo {
    () => {{
        ...
    }}
}
```

外层的大括号是`macro_rules!`语法的一部分。事实上，你也可以`()`或者`[]`。它们只是用来界定整个右侧结构的。

内层大括号是展开语法的一部分。记住，`vec!`在表达式上下文中使用。要写一个包含多个语句，包括`let`绑定，的表达式，我们需要使用块。如果你的宏只展开一个单独的表达式，你不需要内层的大括号。

注意我们从未**声明**宏产生一个表达式。事实上，直到宏被展开之前我们都无法知道。足够小心的话，你可以编写一个能在多个上下文中展开的宏。例如，一个数据类型的简写可以作为一个表达式或一个模式。

## 重复（Repetition）
重复运算符遵循两个原则：

1. `$(...)*`对它包含的所有`$name`都执行“一层”重复
2. 每个`$name`必须有至少这么多的`$(...)*`与其相对。如果多了，它将是多余的。

这个巴洛克宏展示了外层重复中多余的变量。

```rust
macro_rules! o_O {
    (
        $(
            $x:expr; [ $( $y:expr ),* ]
        );*
    ) => {
        &[ $($( $x + $y ),*),* ]
    }
}

fn main() {
    let a: &[i32]
        = o_O!(10; [1, 2, 3];
               20; [4, 5, 6]);

    assert_eq!(a, [11, 12, 13, 24, 25, 26]);
}
```

这就是匹配器的大部分语法。这些例子使用了`$(...)*`，它指“0次或多次”匹配。另外你可以用`$(...)+`代表“1次或多次”匹配。每种形式都可以包括一个分隔符，分隔符可以使用任何除了`+`和`*`的符号。

这个系统基于[Macro-by-Example](http://www.cs.indiana.edu/ftp/techreports/TR206.pdf)（PDF链接）。

## 卫生（Hygiene）
一些语言使用简单的文本替换来实现宏，它导致了很多问题。例如，这个C程序打印`13`而不是期望的`25`。

```c
#define FIVE_TIMES(x) 5 * x

int main() {
    printf("%d\n", FIVE_TIMES(2 + 3));
    return 0;
}
```

展开之后我们得到`5 * 2 + 3`，并且乘法比加法有更高的优先级。如果你经常使用C的宏，你可能知道标准的习惯来避免这个问题，或更多其它的问题。在Rust中，你不需要担心这个问题。

```rust
macro_rules! five_times {
    ($x:expr) => (5 * $x);
}

fn main() {
    assert_eq!(25, five_times!(2 + 3));
}
```

元变量`$x`被解析成一个单独的表达式节点，并且在替换后依旧在语法树中保持原值。

宏系统中另一个常见的问题是**变量捕捉**（*variable capture*）。这里有一个C的宏，使用了[GNU C 扩展](https://gcc.gnu.org/onlinedocs/gcc/Statement-Exprs.html)来模拟Rust表达式块。

```c
#define LOG(msg) ({ \
    int state = get_log_state(); \
    if (state > 0) { \
        printf("log(%d): %s\n", state, msg); \
    } \
})
```

这是一个非常糟糕的用例：

```c
const char *state = "reticulating splines";
LOG(state)
```

它展开为：

```c
const char *state = "reticulating splines";
{
    int state = get_log_state();
    if (state > 0) {
        printf("log(%d): %s\n", state, state);
    }
}
```

第二个叫做`state`的参数参数被替换为了第一个。当打印语句需要用到这两个参数时会出现问题。

等价的Rust宏则会有理想的表现：

```rust
# fn get_log_state() -> i32 { 3 }
macro_rules! log {
    ($msg:expr) => {{
        let state: i32 = get_log_state();
        if state > 0 {
            println!("log({}): {}", state, $msg);
        }
    }};
}

fn main() {
    let state: &str = "reticulating splines";
    log!(state);
}
```

这之所以能工作时因为Rust有一个[卫生宏系统](http://en.wikipedia.org/wiki/Hygienic_macro)。每个宏展开都在一个不同的**语法上下文**（*syntax context*）中，并且每个变量在引入的时候都在语法上下文中打了标记。这就好像是`main`中的`state`和宏中的`state`被画成了不同的“颜色”，所以它们不会冲突。

这也限制了宏在被执行时引入新绑定的能力。像这样的代码是不能工作的：

```rust
macro_rules! foo {
    () => (let x = 3;);
}

fn main() {
    foo!();
    println!("{}", x);
}
```

相反你需要在执行时传递变量的名字，这样它会在语法上下文中被正确标记。

```rust
macro_rules! foo {
    ($v:ident) => (let $v = 3;);
}

fn main() {
    foo!(x);
    println!("{}", x);
}
```

这对`let`绑定和loop标记有效，对[items](http://doc.rust-lang.org/reference.html#items)无效。所以下面的代码可以编译：

```rust
macro_rules! foo {
    () => (fn x() { });
}

fn main() {
    foo!();
    x();
}
```

## 递归宏
一个宏展开中可以包含更多的宏，包括被展开的宏自身。这种宏对处理树形结构输入时很有用的，正如这这个（简化了的）HTML简写所展示的那样：

```rust
# #![allow(unused_must_use)]
macro_rules! write_html {
    ($w:expr, ) => (());

    ($w:expr, $e:tt) => (write!($w, "{}", $e));

    ($w:expr, $tag:ident [ $($inner:tt)* ] $($rest:tt)*) => {{
        write!($w, "<{}>", stringify!($tag));
        write_html!($w, $($inner)*);
        write!($w, "</{}>", stringify!($tag));
        write_html!($w, $($rest)*);
    }};
}

fn main() {
#   // FIXME(#21826)
    use std::fmt::Write;
    let mut out = String::new();

    write_html!(&mut out,
        html[
            head[title["Macros guide"]]
            body[h1["Macros are the best!"]]
        ]);

    assert_eq!(out,
        "<html><head><title>Macros guide</title></head>\
         <body><h1>Macros are the best!</h1></body></html>");
}
```

## 调试宏代码
运行`rustc --pretty expanded`来查看宏展开后的结果。输出表现为一个完整的包装箱，所以你可以把它反馈给`rustc`，它会有时会比原版产生更好的错误信息。注意如果在同一作用域中有多个相同名字（不过在不同的语法上下文中）的变量的话`--pretty expanded`的输出可能会有不同的意义。这种情况下`--pretty expanded,hygiene`将会告诉你有关语法上下文的信息。

`rustc`提供两种语法扩展来帮助调试宏。目前为止，它们是不稳定的并且需要功能入口（feature gates）。
* `log_syntax!(...)`会打印它的参数到标准输出，在编译时，并且不“展开”任何东西。
* `trace_macros!(true)`每当一个宏被展开时会启用一个编译器信息。在展开后使用`trace_macros!(false)`来关闭它。

## 句法要求
即使Rust代码中含有未展开的宏，它也可以被解析为一个完整的[语法树](7.Glossary 词汇表.md#abstract-syntax-tree)。这个属性对于编辑器或其它处理代码的工具来说十分有用。这里也有一些关于Rust宏系统设计的推论。

一个推论是Rust必须确定，当它解析一个宏展开时，宏是否代替了

* 0个或多个项
* 0个或多个方法
* 一个表达式
* 一个语句
* 一个模式

一个块中的宏展开代表一些项，或者一个表达式/语句。Rust使用一个简单的规则来解决这些二义性。一个代表项的宏展开必须是

* 用大括号界定的，例如`foo! { ... }`
* 分号结尾的，例如`foo!(...);`

另一个展开前解析的推论是宏展开必须包含有效的Rust记号。更进一步，括号，中括号，大括号在宏展开中必须是封闭的。例如，`foo!([)`是不允许的。这让Rust知道宏何时结束。

更正式一点，宏展开体必须是一个**记号树**（*token trees*）的序列。一个记号树是一系列递归的

* 一个由`()`，`[]`或`{}`包围的记号树序列
* 任何其它单个记号

在一个匹配器中，每一个元变量都有一个**片段分类符**（*fragment specifier*），确定它匹配的哪种句法。

* `ident`：一个标识符。例如：`x`，`foo`
* `path`：一个受限的名字。例如：`T::SpecialA`
* `expr`：一个表达式。例如：`2 + 2`；`if true then { 1 } else { 2 }`；`f(42)`
* `ty`：一个类型。例如：`i32`；`Vec<(char, String)>`；`&T`
* `pat`：一个模式。例如：`Some(t)`；`(17, 'a')`；`_`
* `stmt`：一个单独语句。例如：`let x = 3`
* `block`：一个大括号界定的语句序列，或者一个表达式。例如：`{ log(error, "hi"); return 12; }`
* `item`：一个[项](http://doc.rust-lang.org/stable/reference.html#items)。例如：`fn foo() { }`，`struct Bar`
* `meta`：一个“元数据项”，可以在属性中找到。例如：`cfg(target_os = "windows")`
* `tt`：一个单独的记号树

对于一个元变量（metavariable）后面的一个记号有一些额外的规则：

* `expr`和`stmt`变量必须后跟任意一个：`=> , ;`
* `ty`和`path`变量必须后跟任意一个：`=> , = | ; : > [ { as where`
* `pat`变量必须后跟任意一个：`=> , = | if in`
* 其它变量可以后跟任何记号

这些规则为 Rust 语法提供了一些灵活性以便将来的展开不会破坏现有的宏。

宏系统完全不处理解析模糊。例如，`$($i:ident)* $e:expr`语法总是会解析失败，因为解析器会被强制在解析`$i`和解析`$e`之间做出选择。改变展开在它们之前分别加上一个记号可以解决这个问题。在这个例子中，你可以写成`$(I $i:ident)* E $e:expr`。

## 范围和宏导入/导出
宏在编译的早期阶段被展开，在命名解析之前。这有一个缺点是与语言中其它结构相比，范围对宏的作用不一样。

定义和展开都发生在同一个深度优先、字典顺序的包装箱的代码遍历中。那么在模块范围内定义的宏对同模块的接下来的代码是可见的，这包括任何接下来的子`mod`项。

一个定义在`fn`函数体内的宏，或者任何其它不在模块范围内的地方，只在它的范围内可见。

如果一个模块有`macro_use`属性，它的宏在子`mod`项之后的父模块也是可见的。如果它的父模块也有`macro_use`属性那么在父`mod`项之后的祖父模块中也是可见的，以此类推。

`macro_use`属性也可以出现在`extern crate`处。在这个上下文中它控制那些宏从外部包装箱中装载，例如

```rust
#[macro_use(foo, bar)]
extern crate baz;
```

如果属性只是简单的写成`#[macro_use]`，所有的宏都会被装载。如果没有`#[macro_use]`属性那么没有宏被装载。只有被定义为`#[macro_export]`的宏可能被装载。

装载一个包装箱的宏**而不**链接到输出，使用`#[no_link]`。

一个例子：

```rust
macro_rules! m1 { () => (()) }

// Visible here: `m1`.

mod foo {
    // Visible here: `m1`.

    #[macro_export]
    macro_rules! m2 { () => (()) }

    // Visible here: `m1`, `m2`.
}

// Visible here: `m1`.

macro_rules! m3 { () => (()) }

// Visible here: `m1`, `m3`.

#[macro_use]
mod bar {
    // Visible here: `m1`, `m3`.

    macro_rules! m4 { () => (()) }

    // Visible here: `m1`, `m3`, `m4`.
}

// Visible here: `m1`, `m3`, `m4`.
# fn main() { }
```

当这个库被用`#[macro_use] extern crate`装载时，只有`m2`会被导入。

Rust参考中有一个[宏相关的属性列表](http://doc.rust-lang.org/stable/reference.html#macro-related-attributes)。

## `$crate`变量
当一个宏在多个包装箱中使用时会产生另一个困难。来看`mylib`定义了

```rust
pub fn increment(x: u32) -> u32 {
    x + 1
}

#[macro_export]
macro_rules! inc_a {
    ($x:expr) => ( ::increment($x) )
}

#[macro_export]
macro_rules! inc_b {
    ($x:expr) => ( ::mylib::increment($x) )
}
# fn main() { }
```

`inc_a`只能在`mylib`内工作，同时`inc_b`只能在库外工作。进一步说，如果用户有另一个名字导入`mylib`时`inc_b`将不能工作。

Rust（目前）还没有针对包装箱引用的卫生系统，不过它确实提供了一个解决这个问题的变通方法。当从一个叫`foo`的包装箱总导入宏时，特殊宏变量`$crate`会展开为`::foo`。相反，当这个宏在同一包装箱内定义和使用时，`$crate`将展开为空。这意味着我们可以写

```rust
#[macro_export]
macro_rules! inc {
    ($x:expr) => ( $crate::increment($x) )
}
# fn main() { }
```

来定义一个可以在库内外都能用的宏。这个函数名字会展开为`::increment`或`::mylib::increment`。

为了保证这个系统简单和正确，`#[macro_use] extern crate ...`应只出现在你包装箱的根中，而不是在`mod`中。

## 深入（The deep end）
之前的介绍章节提到了递归宏，但并没有给出完整的介绍。还有一个原因令递归宏是有用的：每一次递归都给你匹配宏参数的机会。

作为一个极端的例子，可以，但极端不推荐，用Rust宏系统来实现一个[位循环标记](http://esolangs.org/wiki/Bitwise_Cyclic_Tag)自动机。
<a name="1"></a>
```rust
macro_rules! bct {
    // cmd 0:  d ... => ...
    (0, $($ps:tt),* ; $_d:tt)
        => (bct!($($ps),*, 0 ; ));
    (0, $($ps:tt),* ; $_d:tt, $($ds:tt),*)
        => (bct!($($ps),*, 0 ; $($ds),*));

    // cmd 1p:  1 ... => 1 ... p
    (1, $p:tt, $($ps:tt),* ; 1)
        => (bct!($($ps),*, 1, $p ; 1, $p));
    (1, $p:tt, $($ps:tt),* ; 1, $($ds:tt),*)
        => (bct!($($ps),*, 1, $p ; 1, $($ds),*, $p));

    // cmd 1p:  0 ... => 0 ...
    (1, $p:tt, $($ps:tt),* ; $($ds:tt),*)
        => (bct!($($ps),*, 1, $p ; $($ds),*));

    // Halt on empty data string:
    ( $($ps:tt),* ; )
        => (());
}
```

练习：使用宏来减少上面`bct!`宏定义中的重复。

## 常用宏（Common macros）
这里有一些你会在Rust代码中看到的常用宏。

### `panic!`
这个宏导致当前线程恐慌。你可以传给这个宏一个信息通过：

```rust,should_panic
panic!("oh no!");
```

### `vec!`
`vec!`的应用遍及本书，所以你可能已经见过它了。它方便创建`Vec<T>`：

```rust
let v = vec![1, 2, 3, 4, 5];
```

它也让你可以用重复值创建vector。例如，100个`0`：

```rust
let v = vec![0; 100];
```

### `assert!`和`assert_eq!`
这两个宏用在测试中。`assert!`获取一个布尔值，而`assert_eq!`获取两个值并比较它们。`true` 就通过，`false`就`panic!`。像这样：

```rust,should_panic
// A-ok!

assert!(true);
assert_eq!(5, 3 + 2);

// Nope :(

assert!(5 < 3);
assert_eq!(5, 3);
```

### `try!`
`try!`用来进行错误处理。它获取一些可以返回`Result<T, E>`的数据，并返回`T`如果它是` Ok<T>`，或`return`一个`Err(E)`如果出错了。像这样：

```rust
use std::fs::File;

fn foo() -> std::io::Result<()> {
    let f = try!(File::create("foo.txt"));

    Ok(())
}
```

它比这么写要更简明：

```rust
use std::fs::File;

fn foo() -> std::io::Result<()> {
    let f = File::create("foo.txt");

    let f = match f {
        Ok(t) => t,
        Err(e) => return Err(e),
    };

    Ok(())
}
```

### `unreachable!`
这个宏用于当你认为一些代码不应该被执行的时候：

```rust
if false {
    unreachable!();
}
```

有时，编译器可能会让你编写一个你认为将永远不会执行的不同分支。在这个例子中，用这个宏，这样如果最终你错了，你会为此得到一个`panic!`。

```rust
let x: Option<i32> = None;

match x {
    Some(_) => unreachable!(),
    None => println!("I know x is None!"),
}
```

### `unimplemented!`
`unimplemented!`宏可以被用来当你尝试去让你的函数通过类型检查，同时你又不想操心去写函数体的时候。一个这种情况的例子是实现一个要求多个方法的特性，而你只想一次搞定一个。用`unimplemented!`定义其它的直到你准备好去写它们了。

## 宏程序（Procedural macros）
如果Rust宏系统不能做你想要的，你可能想要写一个[编译器插件](Compiler Plugins 编译器插件.md)。与`macro_rules!`宏相比，它能做更多的事，接口也更不稳定，并且bug将更难以追踪。相反你得到了可以在编译器中运行任意Rust代码的灵活性。为此语法扩展插件有时被称为**宏程序**（*procedural macros*）。

---
[^实际上]: `vec!`在 libcollections 中的实际定义跟这里的表现并不相同，出于效率和复用的考虑。
