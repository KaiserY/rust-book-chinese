# 错误处理

> [error-handling.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/error-handling.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

就像大多数编程语言，Rust 鼓励程序猿以特定的方式处理错误。一般来讲，错误处理被分割为两个大类：异常和返回值。Rust 选择了返回值。

在这一部分，我们试图提供一个全面的 Rust 如何处理错误的解决方案。不仅如此，我们也尝试一次一点的介绍错误处理，这样当你离开时会有一个对所有东西如何协调的坚实理解。

Rust 的错误处理天生是冗长而烦人的。这一部分将会探索这些坑并展示如何使用标准库来让错误处理变得准确和符合工程原理。

## 内容列表

这一部分灰常的长，大部分因为我们从最基础的和类型和组合入手，并尝试一点一点的解释 Rust 错误处理的动机。为此，对有其他类似类型系统经验的童鞋可能想要跳过一些内容。

* [基础](#the-basics)
    * [理解 unwrapping](#unwrapping-explained)
    * [`Option`类型](#the-option-type)
        * [组合`Option<T>`值](#composing-optiont-values)
    * [`Result`类型](#the-result-type)
        * [解析整型](#parsing-integers)
        * [`Result`类型别名习惯](#the-result-type-alias-idiom)
    * [小插曲：unwrapping 并不邪恶](#a-brief-interlude-unwrapping-isnt-evil)
* [处理多种错误类型](#working-with-multiple-error-types)
    * [组合`Option`和`Result`](#composing-option-and-result)
    * [组合的限制](#the-limits-of-combinators)
    * [提早返回](#early-returns)
    * [`try!`宏](#the-try-macro)
    * [定义你自己的错误类型](#defining-your-own-error-type)
* [用于错误处理的标准库 trait](#standard-library-traits-used-for-error-handling)
    * [`Error`trait](#the-error-trait)
    * [`From`trait](#the-from-trait)
    * [真正的`try!`宏](#the-real-try-macro)
    * [组合自定义错误类型](#composing-custom-error-types)
    * [给库编写者的建议](#advice-for-library-writers)
* [案例学习：一个读取人口数据的程序](#case-study-a-program-to-read-population-data)
    * [初始化](#initial-setup)
    * [参数解析](#argument-parsing)
    * [编写逻辑](#writing-the-logic)
    * [使用`Box<Error>`处理错误](#error-handling-with-boxerror)
    * [从标准输入读取](#reading-from-stdin)
    * [用自定义类型处理错误](#error-handling-with-a-custom-type)
    * [增加功能](#adding-functionality)
* [精简版](#the-short-story)

## <a name="the-basics"></a>基础

你可以认为错误处理是用事例分析（case analysis）来决定一个计算成功与否。如你所见，工程性的错误处理就是要减少程序猿显式的事例分析的同时保持代码的可组合性。

保持代码的可组合性是很重要的，因为没有这个要求，我们可能在遇到没想到的情况时[panic](http://doc.rust-lang.org/std/macro.panic!.html)。（`panic`导致当前线程结束，而在大多数情况，导致整个程序结束。）这是一个例子：

```rust
// Guess a number between 1 and 10.
// If it matches the number we had in mind, return `true`. Else, return `false`.
fn guess(n: i32) -> bool {
    if n < 1 || n > 10 {
        panic!("Invalid number: {}", n);
    }
    n == 5
}

fn main() {
    guess(11);
}
```

如果你运行这段代码，程序会崩溃并输出类似如下信息：

```text
thread 'main' panicked at 'Invalid number: 11', src/bin/panic-simple.rs:5
```

这是另一个稍微不那么违和的例子。一个接受一个整型作为参数，乘以二并打印的程序。

<a name="code-unwrap-double"></a>

```rust
use std::env;

fn main() {
    let mut argv = env::args();
    let arg: String = argv.nth(1).unwrap(); // error 1
    let n: i32 = arg.parse().unwrap(); // error 2
    println!("{}", 2 * n);
}
```

如果你给这个程序 0 个参数（错误 1）或者 第一个参数并不是整型（错误 2），这个程序也会像第一个例子那样 panic。

你可以认为这种风格的错误处理类似于冲进瓷器店的公牛。它会冲向任何它想去的地方，不过会毁掉过程中的一切。

### <a name="unwrapping-explained"></a>理解 unwrapping

在之前的例子中，我们声称程序如果遇到两个错误情况之一会直接 panic，不过，程序并不像第一个程序那样包括一个显式的`panic`调用。这是因为 panic 嵌入到了`unwrap`的调用中。

Rust 中“unwrap”是说，“给我计算的结果，并且如果有错误，panic 并停止程序。”因为他们很简单如果我们能展示 unwrap 的代码就更好了，不过在这么做之前，我们首先需要探索`Option`和`Result`类型。他们俩都定义了一个叫`unwrap`的方法。

### <a name="the-option-type"></a>`Option`类型

`Option`类型[定义在标准库中](http://doc.rust-lang.org/std/option/enum.Option.html)：

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option`类型是一个 Rust 类型系统用于表达**不存在的可能性**（*possibility of absence*）的方式。将不存在的可能性编码进类型系统是一个重要概念，因为它会强迫编译器处理不存在的情况。让我们看看一个尝试在一个字符串中找一个字符的例子：

<a name="code-option-ex-string-find"></a>

```rust
// Searches `haystack` for the Unicode character `needle`. If one is found, the
// byte offset of the character is returned. Otherwise, `None` is returned.
fn find(haystack: &str, needle: char) -> Option<usize> {
    for (offset, c) in haystack.char_indices() {
        if c == needle {
            return Some(offset);
        }
    }
    None
}
```

注意当函数找到一个匹配的字符，它并不仅仅返回`offset`。相反，它返回`Some(offset)`。`Some`是一个`Option`类型的一个变体或一个值构造器。你可以认为它是一个`fn<T>(value: T) -> Option<T>`类型的函数。同理，`None`也是一个值构造器，除了它并没有参数。你可以认为`None`是一个`fn<T>() -> Option<T>`类型的函数。

这可能看起来并没有什么，不过这是故事的一半。另一半是使用我们编写的`find`函数。让我们尝试用它查找文件名的扩展名。

```rust
# fn find(haystack: &str, needle: char) -> Option<usize> { haystack.find(needle) }
fn main() {
    let file_name = "foobar.rs";
    match find(file_name, '.') {
        None => println!("No file extension found."),
        Some(i) => println!("File extension: {}", &file_name[i+1..]),
    }
}
```

这段代码使用[模式识别](Patterns 模式.md)来对`find`函数的返回的`Option<usize>`进行 case analysis。事实上，case analysis 是唯一能获取`Option<T>`中存储的值的方式。这意味着你，作为一个程序猿，必须处理当`Option<T>`是`None`而不是`Some(t)`的情况。

不过稍等，那我们[之前](#code-unwrap-double)使用的`unwrap`呢？那里并没有 case analysis！相反，case analysis 被放入了`unwrap`方法中。如果你想的话你可以自己定义它：

<a name="code-option-def-unwrap"></a>

```rust
enum Option<T> {
    None,
    Some(T),
}

impl<T> Option<T> {
    fn unwrap(self) -> T {
        match self {
            Option::Some(val) => val,
            Option::None =>
              panic!("called `Option::unwrap()` on a `None` value"),
        }
    }
}
```

`unwrap`方法抽象出了 case analysis。这正是`unwrap`的工程化用法。不幸的是，`panic!`意味着`unwrap`并不是可组合的：它是瓷器店中的公牛。

#### <a name="composing-optiont-values"></a>组合`Option<T>`值

在[之前的例子](#code-option-ex-string-find)中，我们看到了如何用`find`发现文件名的扩展名。当然，并不是所有文件名都有一个`.`，所以可能文件名并没有扩展名。不存在的可能性被编码进了使用`Option<T>`的类型。换句话说，编译器将会强制我们描述一个扩展名不存在的可能性。在我们的例子中，我们只打印出一个说明情况的信息。

获取一个文件名的扩展名是一个很常见的操作，所以把它放进一个函数是很有道理的：

```rust
# fn find(haystack: &str, needle: char) -> Option<usize> { haystack.find(needle) }
// Returns the extension of the given file name, where the extension is defined
// as all characters following the first `.`.
// If `file_name` has no `.`, then `None` is returned.
fn extension_explicit(file_name: &str) -> Option<&str> {
    match find(file_name, '.') {
        None => None,
        Some(i) => Some(&file_name[i+1..]),
    }
}
```

（专业建议：不要用这段代码，相反使用标准库的[extension](http://doc.rust-lang.org/std/path/struct.Path.html#method.extension)方法。）

代码是简单的，不过重要的是注意到`find`的类型强迫我们考虑不存在的可能性。这是一个好事，因为这意味着编译器不会让我们不小心忘记了文件名没有扩展名的情况。另一方面，每次都像`extension_explicit`那样进行显式 case analysis 会变得有点无聊。

事实上，`extension_explicit`的 case analysis 遵循一个非常常见的模式：将`Option<T>`中的值映射为一个函数，除非它是`None`，这时，返回`None`。

Rust 拥有参数多态（parametric polymorphism），所以定义一个组合来抽象这个模式是很容易的：

<a name="code-option-map"></a>

```rust
fn map<F, T, A>(option: Option<T>, f: F) -> Option<A> where F: FnOnce(T) -> A {
    match option {
        None => None,
        Some(value) => Some(f(value)),
    }
}
```

事实上，`map`是标准库中的`Option<T>`[定义的一个方法](http://doc.rust-lang.org/std/option/enum.Option.html#method.map)。值得一提的是，它有着一个略微不同的 signature：一个以`self`，`&self`或`&mut self`作为第一个参数的方法。

用我们的新组合，我们可以重写我们的`extension_explicit`方法来去掉 case analysis：

```rust
# fn find(haystack: &str, needle: char) -> Option<usize> { haystack.find(needle) }
// Returns the extension of the given file name, where the extension is defined
// as all characters following the first `.`.
// If `file_name` has no `.`, then `None` is returned.
fn extension(file_name: &str) -> Option<&str> {
    find(file_name, '.').map(|i| &file_name[i+1..])
}
```

我们通常会发现的另一个模式是为一个`Option`为`None`时赋一个默认值。例如，也许你的程序假设即便一个文件没有扩展名则它的扩展名是`rs`。正如你可能想象到的，这里的 case analysis 并不特定用于文件扩展名 - 它可以用于任何`Option<T>`：

```rust
fn unwrap_or<T>(option: Option<T>, default: T) -> T {
    match option {
        None => default,
        Some(value) => value,
    }
}
```

与上面的`map`相似，标准库中的实现是一个方法而不是一个普通的函数。

这里要注意的是默认值的类型必须与可能出现在`Option<T>`中的值类型相同。在我们的例子中使用它是非常简单的：

```rust
# fn find(haystack: &str, needle: char) -> Option<usize> {
#     for (offset, c) in haystack.char_indices() {
#         if c == needle {
#             return Some(offset);
#         }
#     }
#     None
# }
#
# fn extension(file_name: &str) -> Option<&str> {
#     find(file_name, '.').map(|i| &file_name[i+1..])
# }
fn main() {
    assert_eq!(extension("foobar.csv").unwrap_or("rs"), "csv");
    assert_eq!(extension("foobar").unwrap_or("rs"), "rs");
}
```

（注意`unwrap_or`是标准库中的`Option<T>`[定义的一个方法](http://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or)，所以这里我们使用它而不是我们上面定义的独立的函数。别忘了看看更通用的[`unwrap_or_else`](http://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap_or_else)方法。）

还有另一个我们认为值得特别注意的组合：`and_then`。它让我们更容易地组合不同的代码来应对不存在的可能。例如，这一部分的很多代码是关于找到一个给定文件的扩展名的。为此，你首先需要一个通常截取自文件路径的文件名。虽然大部分文件路径都有一个文件名，但并不是都有。例如，`.`，`..`，`/`。

所以，我们面临着从一个给定的文件路径找出一个扩展名的挑战。让我们从显式 case analysis 开始：

```rust
# fn extension(file_name: &str) -> Option<&str> { None }
fn file_path_ext_explicit(file_path: &str) -> Option<&str> {
    match file_name(file_path) {
        None => None,
        Some(name) => match extension(name) {
            None => None,
            Some(ext) => Some(ext),
        }
    }
}

fn file_name(file_path: &str) -> Option<&str> {
  // Implementation elided.
  unimplemented!()
}
```

你可能认为我们应该用`map`组合来减少 case analysis，不过它的类型并不匹配。。。

```rust,ignore
fn file_path_ext(file_path: &str) -> Option<&str> {
    file_name(file_path).map(|x| extension(x)) // This causes a compilation error.
}
```

这里的`map`函数装箱了`extension`函数返回的`Option<_>`中的值，并且因为`extension`返回一个`Option<&str>`，表达式`file_name(file_path).map(|x| extension(x))`实际上返回一个`Option<Option<&str>>`。

不过因为`file_path_ext`仅仅返回`Option<&str>`（而不是`Option<Option<&str>>`），我们会遇到一个编译错误。

被 map 函数作为输入的函数的返回值**总是**会被重新封装为`Some`。因此，我们需要一些像`map`，不过允许调用者直接返回一个`Option<_>`而不用在再套上另一个`Option<_>`的函数。

它的泛型实现甚至比`map`更简单：

```rust
fn and_then<F, T, A>(option: Option<T>, f: F) -> Option<A>
        where F: FnOnce(T) -> Option<A> {
    match option {
        None => None,
        Some(value) => f(value),
    }
}
```

现在我们可以不用显式 case analysis 重写我们的`file_path_ext`函数了：

```rust
# fn extension(file_name: &str) -> Option<&str> { None }
# fn file_name(file_path: &str) -> Option<&str> { None }
fn file_path_ext(file_path: &str) -> Option<&str> {
    file_name(file_path).and_then(extension)
}
```

边注：因为`and_then`本质上就像`map`不过返回一个`Option<_>`而不是`Option<Option<_>>`，它在一些其他语言中被称为`flatmap`。

`Option`类型有很多其他[定义在标准库中的](http://doc.rust-lang.org/std/option/enum.Option.html)组合。过一遍这个列表并熟悉他们的功能是一个好主意 —— 通常他们可以减少你的 case analysis。熟悉这些组合将会得到回报，因为他们很多也为`Result`类型定义了实现（相似的语义），而我们接下来会讲`Result`。

组合使用像`Option`这样的符合工程学的类型来减少显式 case analysis。他们也是可组合的因为他们允许调用者以他们自己的方式处理不存在的可能性。像`unwrap`这样的方法去掉了选择因为当`Option<T>`为`None`他们会 panic。

### <a name="the-result-type"></a>`Result`类型

`Result`类型也[定义于标准库中](http://doc.rust-lang.org/std/result/index.html)：

<a name="code-result-def"></a>

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`Result`是`Option`的高级版本。相比于像`Option`那样表示不存在的可能性，`Result`表示错误的可能性。通常，错误用来解释为什么一些计算会失败。严格的说这是一个更通用的`Option`。考虑如下类型别名，它的语义在任何地方都与真正的`Option<T>`相同：

```rust
type Option<T> = Result<T, ()>;
```

它把`Result`的第二个类型参数改为总是`()`（读作“单元”或“空元组”）。`()`类型只有一个值：`()`（没错，类型和值两个级别的术语表示是一样的！）

`Result`类型是一个代表一个计算的两个可能结果的方式。通常，一个结果是期望的值或者“`Ok`”而另一个意味着非预期的或者“`Err`”。

就像`Option`，`Result`在标准库中也[定义了一个`unwrap`方法](http://doc.rust-lang.org/std/result/enum.Result.html#method.unwrap)。让我们定义它：

```rust
# enum Result<T, E> { Ok(T), Err(E) }
impl<T, E: ::std::fmt::Debug> Result<T, E> {
    fn unwrap(self) -> T {
        match self {
            Result::Ok(val) => val,
            Result::Err(err) =>
              panic!("called `Result::unwrap()` on an `Err` value: {:?}", err),
        }
    }
}
```

这实际上与[`Option::unwrap`的定义](#code-option-def-unwrap)一样，除了它在`panic!`信息中包含了错误信息。这让调试变得简单，不过也要求我们为`E`类型参数（它代表我们的错误类型）添加一个[`Debug`](http://doc.rust-lang.org/std/fmt/trait.Debug.html)限制。因为绝大部分类型应该满足`Debug`限制，这使得它可以在实际中使用。（`Debug`简单的意味着这个类型有合理的方式可以打印出人类可读的描述。）

OK，让我们开始一个例子。

#### <a name="parsing-integers"></a>解析整型

Rust 标准库中让字符串转换为整型变得异常简单。事实上它太简单了，以至于你可以写出如下代码：

```rust
fn double_number(number_str: &str) -> i32 {
    2 * number_str.parse::<i32>().unwrap()
}

fn main() {
    let n: i32 = double_number("10");
    assert_eq!(n, 20);
}
```

在这里，你应该对调用`unwrap`持怀疑态度。例如，如果字符串并不能解析为一个数字，它会 panic：

```text
thread 'main' panicked at 'called `Result::unwrap()` on an `Err` value: ParseIntError { kind: InvalidDigit }', /home/rustbuild/src/rust-buildbot/slave/beta-dist-rustc-linux/build/src/libcore/result.rs:729
```

这是很难堪的，而且如果这在你所使用的库中出现了的话，可以理解你会很烦躁。相反，我们应该尝试在我们的函数里处理错误并让调用者决定该怎么做。这意味着改变`double_number`的返回值类型。不过改编成什么呢？好吧，这需要我们看看标准库中[`parse`方法](http://doc.rust-lang.org/std/primitive.str.html#method.parse)的签名：

```rust
impl str {
    fn parse<F: FromStr>(&self) -> Result<F, F::Err>;
}
```

额嗯。所以至少我们知道了我们需要使用一个`Result`。当然，也可以返回一个`Option`。毕竟，一个字符串要么能解析成一个数字要么不能，不是吗？这当然是一个合理的方式，不过实现内部区别了为什么字符串不能解析成数字。（要么是一个空字符串，一个无效的数位，太大或太小。）因此，使用`Result`更有道理因为我们想要比单纯的“不存在”提供更多信息。我们想要表明**为什么**解析会失败。你应该尝试再现这样的推理，当你面对一个`Option`和`Result`之间的选择时。如果你可以提供详细的错误信息，那么大概你也应该提供。（我们会在后面详细讲到。）

好的，不过我们的返回值类型该怎么写呢？上面定义的`parse`方法对所有不同的标准库定义的数字类型是泛型的。我们也可以（应该）让我们的函数也是泛型的，不过这回让我们享受显式定义的好处。我们只关心`i32`，所以我们需要寻找[`FromStr`的实现](http://doc.rust-lang.org/std/primitive.i32.html)（在你的浏览器中用`CTRL-F`搜索“FromStr”）和[与它相关的类型`](Associated Types 关联类型.md)`Err`。这么做我可以找出具体的错误类型。在这个例子中，它是[`std::num::ParseIntError`](http://doc.rust-lang.org/std/num/struct.ParseIntError.html)。最后我们可以重写函数：

```rust
use std::num::ParseIntError;

fn double_number(number_str: &str) -> Result<i32, ParseIntError> {
    match number_str.parse::<i32>() {
        Ok(n) => Ok(2 * n),
        Err(err) => Err(err),
    }
}

fn main() {
    match double_number("10") {
        Ok(n) => assert_eq!(n, 20),
        Err(err) => println!("Error: {:?}", err),
    }
}
```

这比之前有些进步，不过现在我们写的代码有点多了！case analysis 又一次坑了我们。

组合是救星！就像`Option`一样，`Result`有很多定义的组合方法。`Result`和`Option`之间的常用组合有很大的交集。特别的，`map`就是其中之一：

```rust
use std::num::ParseIntError;

fn double_number(number_str: &str) -> Result<i32, ParseIntError> {
    number_str.parse::<i32>().map(|n| 2 * n)
}

fn main() {
    match double_number("10") {
        Ok(n) => assert_eq!(n, 20),
        Err(err) => println!("Error: {:?}", err),
    }
}
```

常见的组合`Result`都有，包括[unwrap_or](http://doc.rust-lang.org/std/result/enum.Result.html#method.unwrap_or)和[and_then](http://doc.rust-lang.org/std/result/enum.Result.html#method.and_then)。另外，因为`Result`有第二个类型参数，所以有一些只影响错误类型的组合，例如[map_err](http://doc.rust-lang.org/std/result/enum.Result.html#method.map_err)（相对于`map`）和[or_else](http://doc.rust-lang.org/std/result/enum.Result.html#method.or_else)（相对于`and_then`）。

#### <a name="the-result-type-alias-idiom"></a>`Result`类型别名习惯

在标准库中，你可能经常看到像`Result<i32>`这样的类型。不过等等，[我们定义的`Result`](#code-result-def)有两个类型参数。我么怎么能只指定一个呢？这里的关键是定义一个`Result`类型别名来对一个特定类型固定其中一个类型参数。通常固定的类型是错误类型。例如，我们之前的解析整数例子可以重写成这样：

```rust
use std::num::ParseIntError;
use std::result;

type Result<T> = result::Result<T, ParseIntError>;

fn double_number(number_str: &str) -> Result<i32> {
    unimplemented!();
}
```

为什么我们应该这么做？好吧，如果我们有很多可能返回`ParseIntError`的函数，那么定义一个总是使用`ParseIntError `的别名就比每回都写一遍要方便很多。

这个习惯最突出的一点是标准库中的[`io::Result`](http://doc.rust-lang.org/std/io/type.Result.html)。通常，当你使用`io::Result<T>`，很明显你就是在使用`io`模块的类型别名而不是`std::result`的原始定义。（这个习惯也用于[`fmt::Result`](http://doc.rust-lang.org/std/fmt/type.Result.html)。）

### <a name="a-brief-interlude-unwrapping-isnt-evil"></a>小插曲：unwrapping 并不邪恶

如果你一路跟了过来，你可能注意到我们花了很大力气反对使用像`unwrap`这样会`panic`并终止你的程序的方法。通常来说，这是一个好的建议。

然而，`unwrap`仍然可以被明智的使用。具体如何正当化`unwrap`的使用是一个灰色区域并且理性的人可能不会同意。我会简述这个问题的一些个人看法。

* **在例子和简单快速的编码中。**有时你要写一个例子或小程序，这时错误处理一点也不重要。这种情形要击败`unwrap`的方便易用是很难的，所以它显得很合适。
* **当 panic 就意味着程序中有 bug 的时候。**当你代码中的不变量应该阻止特定情况发生的时候（比如，从一个空的栈上弹出一个值），那么 panic 就是可行的。这是因为它暴露了程序的一个 bug。这可以是显式的，例如一个`assert!`失败，或者因为一个数组越界。

这可能并不是一个完整的列表。另外，当使用`Option`的时候，通常使用[`expect`](http://doc.rust-lang.org/std/option/enum.Option.html#method.expect)方法更好。`expect`做了`unwrap`同样的工作，除了`expect`会打印你给它的信息。这让 panic 的结果更容易处理，因为相比“called unwrap on a None value.”会提供一个信息。

我的建议浓缩如下：运用你良好的判断。我的文字中并没有出现“永远不要做 X”或“Y 被认为是有害的”是有原因的。所有这些都是权衡取舍，并且这是你们程序猿的工作去决定在你的用例中哪个是可以接受的。我们目标只是尽可能的帮助你进行权衡。

现在我们介绍了 Rust 的基本的错误处理，并解释了 unwrap，让我们开始更多的探索标准库。

## <a name="working-with-multiple-error-types"></a>处理多种错误类型

到目前为止，我看到了不是`Option<T>`就是`Result<T, SomeError>`的错误处理。不过当你同时使用`Option`和`Result`时会发生什么呢？或者如果你使用`Result<T, Error1>`和`Result<T, Error2>`呢？我们接下来的挑战是处理**不同错误类型的组合**，这将会是贯穿本章余下部分的主要主题。

### <a name="composing-option-and-result"></a>组合`Option`和`Result`

到目前为止，我们讲到了为`Option`和`Result`定义的组合。我们可以用这些组合来处理不同的计算结果而不用进行显式的 case analysis。

当然，在实际的代码中，事情并不总是这么明显。有时你遇到一个`Option`和`Result`的混合类型。我们是必须求助于显式 case analysis，或者我们可以使用组合呢？

现在，让我们重温这一部分的第一个例子：

```rust
use std::env;

fn main() {
    let mut argv = env::args();
    let arg: String = argv.nth(1).unwrap(); // error 1
    let n: i32 = arg.parse().unwrap(); // error 2
    println!("{}", 2 * n);
}
```

基于我们新掌握的关于`Option`，`Result`和他们的组合的知识，我们应该尝试重写它来适当的处理错误这样出错时程序就不会 panic 了。

这里的坑是`argv.nth(1)`产生一个`Option`而`arg.parse()`产生一个`Result`。他们不是直接可组合的。当同时遇到`Option`和`Result`的时候，解决办法通常是把`Option`转换为一个`Result`。在我们的例子中，缺少命令行参数（来自`env::args()`）意味着我们的用户没有正确调用我们的程序。我们可以用一个`String`来描述这个错误。让我们试试：

<a name="code-error-double-string"></a>

```rust
use std::env;

fn double_arg(mut argv: env::Args) -> Result<i32, String> {
    argv.nth(1)
        .ok_or("Please give at least one argument".to_owned())
        .and_then(|arg| arg.parse::<i32>().map_err(|err| err.to_string()))
        .map(|n| 2 * n)
}

fn main() {
    match double_arg(env::args()) {
        Ok(n) => println!("{}", n),
        Err(err) => println!("Error: {}", err),
    }
}
```

这个例子中有几个新东西。第一个是使用了[`Option::ok_or`](http://doc.rust-lang.org/std/option/enum.Option.html#method.ok_or)组合。这是一个把`Option`转换为`Result`的方法。这个转换要求你指定当`Option`为`None`时的错误。就像我们见过的其他组合一样，它的定义是非常简单的：

```rust
fn ok_or<T, E>(option: Option<T>, err: E) -> Result<T, E> {
    match option {
        Some(val) => Ok(val),
        None => Err(err),
    }
}
```

另一个新使用的组合是[`Result::map_err`](http://doc.rust-lang.org/std/result/enum.Result.html#method.map_err)。这就像`Result::map`，除了它映射一个函数到`Result`的 error 部分。如果`Result`是一个`Ok(...)`，那么它什么也不修改。

这里我们使用`map_err`是因为它对保证相同的错误类型（因为我们使用了`and_then`）是必要的。因为我们选择把`Option<String>`（来自`argv.nth(1)`）转换为`Result<String, String>`，我们也必须把来自`arg.parse()`的`ParseIntError`转换为`String`。

### <a name="the-limits-of-combinators"></a>组合的限制

IO 和 解析输入是非常常见的任务，这也是我个人在 Rust 经常做的。因此，我们将使用（并一直使用）IO 和多种解析工作作为例子讲解错误处理。

让我们从简单的开始。我们的任务是打开一个文件，读取所有的内容并把他们转换为一个数字。接着我们把它乘以`2`并打印结果。

虽然我们劝告过你不要用`unwrap`，不过开始写代码的时候`unwrap`也是有用的。它允许你关注你的问题而不是错误处理，并且暴露出需要错误处理的点。让我们开始试试手感，再接着用更好的错误处理重构。

```rust
use std::fs::File;
use std::io::Read;
use std::path::Path;

fn file_double<P: AsRef<Path>>(file_path: P) -> i32 {
    let mut file = File::open(file_path).unwrap(); // error 1
    let mut contents = String::new();
    file.read_to_string(&mut contents).unwrap(); // error 2
    let n: i32 = contents.trim().parse().unwrap(); // error 3
    2 * n
}

fn main() {
    let doubled = file_double("foobar");
    println!("{}", doubled);
}
```

（附注：`AsRef<Path>`被使用是因为它与[`std::fs::File::open`有着相同的 bound](http://doc.rust-lang.org/std/fs/struct.File.html#method.open)。这让我们可以用任何类型的字符串作为一个文件路径。）

这里可能出现三个不同错误：

1. 打开文件出错。
2. 从文件读数据出错。
3. 将数据解析为数字出错。

头两个错误被描述为[`std::io::Error`](http://doc.rust-lang.org/std/io/struct.Error.html)类型。我们知道这些因为返回类型是[`std::fs::File::open`](http://doc.rust-lang.org/std/fs/struct.File.html#method.open)和[`std::io::Read::read_to_string`](http://doc.rust-lang.org/std/io/trait.Read.html#method.read_to_string)。（注意他们都使用了之前描述的[`Result`类型别名习惯](#the-result-type-alias-idiom)。如果你点击`Result`类型，你将会[看到这个类型别名](http://doc.rust-lang.org/std/io/type.Result.html)，以及底层的`io::Error`类型。）第三个问题被描述为[`std::num::ParseIntError`](http://doc.rust-lang.org/std/num/struct.ParseIntError.html)。特别的`io::Error`被广泛的用于标准库中。你会一次又一次的看到它。

让我们着手重构`file_double`函数。为了让这个函数可以与程序的其他组件组合，它必须不能在上述错误情况下 panic。事实上，这意味着它在任何操作失败时应该返回一个错误。我们的问题是`file_double`的返回类型是`i32`，它并没有给我们一个有效的报告错误的途径。因此，我们必须以把返回类型`i32`改成别的什么的开始。

我们需要决定的第一件事：我们应该用`Option`还是`Result`？当然我们可以简单的选择`Option`。如果出现任何错误了，我们可以简单的返回`None`。它可以工作并且比 panic 好多了，不过我们可以做的更好。我们应该在错误发生时传递一些细节。因为我们想要表达**错误的可能性**，我们应该使用`Result<i32, E>`。不过`E`应该是什么呢？因为可能出现两种不同的错误，我们需要把他们转换为一种通用类型。其中之一就是`String`。让我们看看这如何影响我们的代码：

```rust
use std::fs::File;
use std::io::Read;
use std::path::Path;

fn file_double<P: AsRef<Path>>(file_path: P) -> Result<i32, String> {
    File::open(file_path)
         .map_err(|err| err.to_string())
         .and_then(|mut file| {
              let mut contents = String::new();
              file.read_to_string(&mut contents)
                  .map_err(|err| err.to_string())
                  .map(|_| contents)
         })
         .and_then(|contents| {
              contents.trim().parse::<i32>()
                      .map_err(|err| err.to_string())
         })
         .map(|n| 2 * n)
}

fn main() {
    match file_double("foobar") {
        Ok(n) => println!("{}", n),
        Err(err) => println!("Error: {}", err),
    }
}
```

这些代码看起来有点难以理解。在能轻松编写这样的代码前可能需要更多一些实践。我们写代码的方式遵循**跟着类型走**（*following the types*）。一旦我们把`file_double `的返回类型改为`Result<i32, String>`，我们就不得不开始寻找正确的组合。在这个例子中，我们只用到了三个不同的组合：`and_then`，`map`和`map_err`。

`and_then`被用来连接多个计算，其中每一个都有可能返回一个错误。在打开文件后，还有另外两个可能失败的计算：从文件读取和把内容解析成数字。相应地，有两个`and_then`的调用。

`map`用来把一个函数用于`Result`的`Ok(...)`值。例如，最后一个`map`调用把`Ok(...)`值（它是一个`i32`）乘以`2`。如果在这之前出现了错误，这里的操作会被省略，因为`map`是这么定义的。

`map_err`是让一切可以工作的关键。`map_err`类似`map`，除了它把一个函数用于`Result`的`Err(...)`值。在这个例子中，我们想要把所有的错误转换为一个类型：`String`。因为`io::Error`和`num::ParseIntError`都实现了`ToString`，我们可以调用`to_string()`去转换他们。

说了这么多，代码仍然不好懂。掌握组合的应用是很重要的，不过他们也有限制。让我们尝试一个不同的方式：提早返回。

### <a name="early-returns"></a>提早返回

我想利用前一章节的代码并用提早返回重写它。提早返回让你提前退出函数。我们不能在另一个闭包中从`file_double`提前返回，所以我们需要退回到显式 case analysis。

```rust
use std::fs::File;
use std::io::Read;
use std::path::Path;

fn file_double<P: AsRef<Path>>(file_path: P) -> Result<i32, String> {
    let mut file = match File::open(file_path) {
        Ok(file) => file,
        Err(err) => return Err(err.to_string()),
    };
    let mut contents = String::new();
    if let Err(err) = file.read_to_string(&mut contents) {
        return Err(err.to_string());
    }
    let n: i32 = match contents.trim().parse() {
        Ok(n) => n,
        Err(err) => return Err(err.to_string()),
    };
    Ok(2 * n)
}

fn main() {
    match file_double("foobar") {
        Ok(n) => println!("{}", n),
        Err(err) => println!("Error: {}", err),
    }
}
```

理性的同学可能不同意这个比使用组合的代码更好，不过如果你并不熟悉组合方式，这些代码阅读起来更简单。它通过`match`和`if let`进行了显式 case analysis。如果错误出现了，它简单的停止执行并返回错误（通过转换为一个字符串）。

这难道不是倒退吗？之前，我们说过工程性的错误处理的关键是减少显式 case analysis，不过这里我们又退回到了显式 case analysis。这表明，有多种方式可以减少显式 case analysis。组合并不是唯一的方法。

### <a name="the-try-macro"></a>`try!`宏

Rust 中错误处理的基石是`try!`宏。`try!`宏像组合一样抽象了 case analysis，不过不像组合，它也抽象了控制流。也就是说，它可以抽象我们之前看到的提早返回的模式。

这是一个简单化的`try!`宏定义：

<a name="code-try-def-simple"></a>

```rust
macro_rules! try {
    ($e:expr) => (match $e {
        Ok(val) => val,
        Err(err) => return Err(err),
    });
}
```

（[实际的定义](http://doc.rust-lang.org/std/macro.try!.html)有一点复杂。我们会在后面详述。）

使用`try!`宏让我们的最后的例子异常的简单。因为它为我们做了 case analysis 和提早返回，我们的代码更紧凑也更易于理解：

```rust
use std::fs::File;
use std::io::Read;
use std::path::Path;

fn file_double<P: AsRef<Path>>(file_path: P) -> Result<i32, String> {
    let mut file = try!(File::open(file_path).map_err(|e| e.to_string()));
    let mut contents = String::new();
    try!(file.read_to_string(&mut contents).map_err(|e| e.to_string()));
    let n = try!(contents.trim().parse::<i32>().map_err(|e| e.to_string()));
    Ok(2 * n)
}

fn main() {
    match file_double("foobar") {
        Ok(n) => println!("{}", n),
        Err(err) => println!("Error: {}", err),
    }
}
```

根据[我们`try!`宏的定义](#code-try-def-simple)`map_err`调用仍是必要的。这是因为错误类型仍然需要被转换为`String`。好消息是我们马上就会学到如何移除这些`map_err`调用！坏消息是在我们可以移除`map_err`调用之前我们需要更深入的学习一些标准库中的重要的 trait。

### <a name="defining-your-own-error-type"></a>定义你自己的错误类型

在我们深入学习一些标准库错误 trait 之前，我想要通过移除之前例子中的作为错误类型的`String`来结束本章节。


之前我们的例子中使用`String`是为了方便，因为把错误转换为字符串是简单的，甚至把我们自己的类型转换为字符串也是如此。然而，把`String`作为你的错误有一些缺点。

第一个缺点是错误信息会倾向于另你的代码变得凌乱。把错误信息定义在别处是可能的，不顾除非你非常的（qiang）自（po）律（zheng），你很容易就会把错误信息嵌入到你的代码中。事实上，我们[上一个例子](#code-error-double-string)就是这么做的。

第二个也是更重要的缺点是`String`是**不完整的**。也就是说，如果所有错误都转换成了字符串，那么传递给调用者的错误就变得完全不透明了。调用者对于一个`String`类型的错误所能作的唯一可行的事就是把它展示给用户。当然，通过观察字符串来确定错误的类型是不健壮的。（对于一个库来说这个缺点公认要比在例如程序中来的更重要。）

例如，`io::Error`类型内嵌了一个[`io::ErrorKind`](http://doc.rust-lang.org/std/io/enum.ErrorKind.html)，它是一个表示在 IO 操作期间错误信息的**结构化类型**。这很重要因为你可能根据错误做出不同的反应。（例如，`BrokenPipe`错误可能意味着可以温和的退出程序，而`NotFound`则意味着应该返回一个错误码并向用户展示错误。）通过`io::ErrorKind`，调用者可以用 case analysis 检查错误的类型，这完全优于尝试从`String`中梳理错误的细节。

除了在我们之前从文件读取一个数字的例子中使用`String`作为错误类型外，我们可以定义我们自己的错误类型来用结构化数据代表错误。我们尽量不丢掉底层错误的信息以防调用者想要检视细节。

表示多种可能性的理想方法是用`enum`来定义我们的集合类型。在我们的例子里，错误要么是`io::Error`要么是`num::ParseIntError`，所以自然的定义如下：

```rust
use std::io;
use std::num;

// We derive `Debug` because all types should probably derive `Debug`.
// This gives us a reasonable human readable description of `CliError` values.
#[derive(Debug)]
enum CliError {
    Io(io::Error),
    Parse(num::ParseIntError),
}
```

修改我们代码非常简单。与其把错误转为字符串，我们简单的用相应的值构造器把错误转换为我们的`CliError`类型：

```rust
# #[derive(Debug)]
# enum CliError { Io(::std::io::Error), Parse(::std::num::ParseIntError) }
use std::fs::File;
use std::io::Read;
use std::path::Path;

fn file_double<P: AsRef<Path>>(file_path: P) -> Result<i32, CliError> {
    let mut file = try!(File::open(file_path).map_err(CliError::Io));
    let mut contents = String::new();
    try!(file.read_to_string(&mut contents).map_err(CliError::Io));
    let n: i32 = try!(contents.trim().parse().map_err(CliError::Parse));
    Ok(2 * n)
}

fn main() {
    match file_double("foobar") {
        Ok(n) => println!("{}", n),
        Err(err) => println!("Error: {:?}", err),
    }
}
```

这里唯一的修改是从`map_err(|e| e.to_string())`（它把错误转换为字符串）变为`map_err(CliError::Io)`或者`map_err(CliError::Parse)`。调用者要决定报告给用户的细节的级别。实际上，使用`String`作为错误类型剥夺了调用者的选择，而用一个像`CliError`这样的`enum`类型除了一个描述错误的结构化数据之外还给了调用者所有的便利。

经验之谈是定义你自己的错误类型，不过必要时`String`也能行，特别是你在写一个程序时。如果你在编写一个库，强烈建议你定义自己的错误类型这样你就不会不必要的剥夺了调用者选择。

## <a name="standard-library-traits-used-for-error-handling"></a>用于错误处理的标准库 trait

标准库定义了两个完整 trait 用于错误处理：[`http://doc.rust-lang.org/std/error/trait.Error.html`](std::error::Error)和[`std::convert::From`](http://doc.rust-lang.org/std/convert/trait.From.html)。`Error`被专门设计为描述通用错误，`From` trait 更多的用于在两个不同类型值之间转换。

### <a name="the-error-trait"></a>`Error`trait

`Error` trait [定义于标准库中](http://doc.rust-lang.org/std/error/trait.Error.html)：

```rust
use std::fmt::{Debug, Display};

trait Error: Debug + Display {
  /// A short description of the error.
  fn description(&self) -> &str;

  /// The lower level cause of this error, if any.
  fn cause(&self) -> Option<&Error> { None }
}
```

这个 trait 非常泛用因为它被设计为为所有类型实现来代表错误。它被证明对编写可组合的代码非常有帮助，正如我们后面将要看到的。这个 trait 允许你至少做如下事情：

* 获取一个错误的`Debug`表示。
* 获取一个错误的面向用户的`Display`表示
* 获取一个错误的简短描述（通过`description`方法）
* 查看错误的调用链，如果存在的话（通过`cause`方法）

头两个是因为`Error`要求实现`Debug`和`Display`。后两个来自于定义于`Error`的方法。`Error`的力量来自于所有错误类型都实现了`Error`的事实，这意味着错误可以被量化一个[trait 对象](Trait Objects trait 对象.md)。表现为`Box<Error>`或`&Error`。事实上,`cause`返回一个`&Error`，它自身就是一个 trait 对象。我们将在后面再次讨论`Error`作为 trait 对象的功能。

目前，展示一个实现了`Error` trait 的例子是足够的。让我们使用[上一部分](#defining-your-own-error-type)我们定义的错误类型：

```rust
use std::io;
use std::num;

// We derive `Debug` because all types should probably derive `Debug`.
// This gives us a reasonable human readable description of `CliError` values.
#[derive(Debug)]
enum CliError {
    Io(io::Error),
    Parse(num::ParseIntError),
}
```

这个特定的错误类型表示出现两种错误类型的可能性：一个进行 I/O 操作的错误或者一个把字符串转换为数字的错误。这个类型可以表示任何你想要添加的错误类型，通过向`enum`定义添加变量。

实现`Error`是非常直观的。这会有很多的显式 case analysis。

```rust
use std::error;
use std::fmt;

impl fmt::Display for CliError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match *self {
            // Both underlying errors already impl `Display`, so we defer to
            // their implementations.
            CliError::Io(ref err) => write!(f, "IO error: {}", err),
            CliError::Parse(ref err) => write!(f, "Parse error: {}", err),
        }
    }
}

impl error::Error for CliError {
    fn description(&self) -> &str {
        // Both underlying errors already impl `Error`, so we defer to their
        // implementations.
        match *self {
            CliError::Io(ref err) => err.description(),
            CliError::Parse(ref err) => err.description(),
        }
    }

    fn cause(&self) -> Option<&error::Error> {
        match *self {
            // N.B. Both of these implicitly cast `err` from their concrete
            // types (either `&io::Error` or `&num::ParseIntError`)
            // to a trait object `&Error`. This works because both error types
            // implement `Error`.
            CliError::Io(ref err) => Some(err),
            CliError::Parse(ref err) => Some(err),
        }
    }
}
```

我们注意到这是一个非常典型的`Error`的实现：为你不同的错误类型做匹配并满足`description`和`cause`定义的限制。

### <a name="the-from-trait"></a>`From`trait

`std::convert::From` trait [定义于标准库中](http://doc.rust-lang.org/std/convert/trait.From.html)：

<a name="code-from-def"></a>

```rust
trait From<T> {
    fn from(T) -> Self;
}
```

非常简单吧？`From`很有用因为它给了我们一个通用的方式来处理从一个特定类型`T`到其他类型的转换（在这个例子中，“其他类型”是实现的主体，或者`Self`）。`From`的核心是[标准库提供的一系列实现](http://doc.rust-lang.org/std/convert/trait.From.html)。

这里是几个展示`From`如何工作的小例子：

```rust
let string: String = From::from("foo");
let bytes: Vec<u8> = From::from("foo");
let cow: ::std::borrow::Cow<str> = From::from("foo");
```

好的，这么说`From`用来处理字符串转换，那么错误怎么办？原来有一个关键实现：

```rust
impl<'a, E: Error + 'a> From<E> for Box<Error + 'a>
```

这个实现说任何实现了`Error`的类型，我们可以把它转换一个 trait 对象`Box<Error>`。这可能看起来并不怎么令人吃惊，不过它在泛型环境中很有用。

记的我们之前处理的两个错误吗？`io::Error`和`num::ParseIntError`。因为他们都实现了`Error`，他们也能用于`From`：

```rust
use std::error::Error;
use std::fs;
use std::io;
use std::num;

// We have to jump through some hoops to actually get error values:
let io_err: io::Error = io::Error::last_os_error();
let parse_err: num::ParseIntError = "not a number".parse::<i32>().unwrap_err();

// OK, here are the conversions:
let err1: Box<Error> = From::from(io_err);
let err2: Box<Error> = From::from(parse_err);
```

这里有一个非常重要的模式。`err1`和`err2`有着相同的类型。这是因为他们实际上是定量类型，或者 trait 对象。尤其是，对编译器来说他们的底层类型被抹掉了，所以编译器认为`err1`和`err2`是完全一样的。另外，我们用完全一样的函数调用构建`err1`和`err2`：`From::from`。这是因为`From::from`的参数和返回值都可以重载。

这个模式很重要，因为它解决了一个我们之前遇到过的问题：它给了我们一个可靠的用相同的函数把错误转换为相同类型的方法。

是时候重新看看我们的老朋友：`try!`宏了。

### <a name="the-real-try-macro"></a>真正的`try!`宏

之前我们展示了`try!`的定义：

```rust
macro_rules! try {
    ($e:expr) => (match $e {
        Ok(val) => val,
        Err(err) => return Err(err),
    });
}
```

这并不是它真正的定义。它的实际定义[位于标准库中](http://doc.rust-lang.org/std/macro.try!.html)：

<a name="code-try-def"></a>

```rust
macro_rules! try {
    ($e:expr) => (match $e {
        Ok(val) => val,
        Err(err) => return Err(::std::convert::From::from(err)),
    });
}
```

这是一个很小但很有效的修改：错误值被通过`From::from`传递。这让`try!`宏变得更强大了一点，因为它免费提供给你自动类型转换。

有了更强大的`try!`宏的支持，让我们再看一眼我们之前写的读一个文件并把内容转换为数字的代码：

```rust
use std::fs::File;
use std::io::Read;
use std::path::Path;

fn file_double<P: AsRef<Path>>(file_path: P) -> Result<i32, String> {
    let mut file = try!(File::open(file_path).map_err(|e| e.to_string()));
    let mut contents = String::new();
    try!(file.read_to_string(&mut contents).map_err(|e| e.to_string()));
    let n = try!(contents.trim().parse::<i32>().map_err(|e| e.to_string()));
    Ok(2 * n)
}
```

之前，我们承诺我们可以去掉`map_err`调用。实际上，所有我们需要做的就是选一个可以用于`From`的类型。一如我们在上一个部分看到的，`From`有一个可以转换任意错误类型为`Box<Error>`的实现：

```rust
use std::error::Error;
use std::fs::File;
use std::io::Read;
use std::path::Path;

fn file_double<P: AsRef<Path>>(file_path: P) -> Result<i32, Box<Error>> {
    let mut file = try!(File::open(file_path));
    let mut contents = String::new();
    try!(file.read_to_string(&mut contents));
    let n = try!(contents.trim().parse::<i32>());
    Ok(2 * n)
}
```

我们已经非常接近理想的错误处理了。我们的代码处理错误只造成了很小的成本，因为`try!`宏同时封装了三个东西：

* case analysis。
* 控制流。
* 错误类型转换。

当结合所有这些东西，我们的代码不再受组合、`unwrap`调用或 case analysis 的困扰了。

这里还剩一点东西：`Box<Error>`是不透明的。如果我们返回一个`Box<Error>`给调用者，调用者并不能（轻易地）观察底层错误类型。当然这种情形比`String`要好，因为调用者可以调用像[`description`](http://doc.rust-lang.org/std/error/trait.Error.html#tymethod.description)和[`cause`](http://doc.rust-lang.org/std/error/trait.Error.html#method.cause)这样的方法，不过这是有限制的：`Box<Error>`是不透明的。（附注：这并不是完全正确，因为 Rust 并没有运行时反射，这在某些场景是有用的不过[超出了本部分的范畴](https://crates.io/crates/error)。）

是时候重写我们的`CliError`类型并将一切连起来了。

### <a name="composing-custom-error-types"></a>组合自定义错误类型

在这最后一部分，我们看看真正的`try!`宏以及如何通过调用`From::from`自动转换错误类型。具体的，我们把错误转换为`Box<Error>`，这是可以的，不过这个类型对调用者是不透明的。

为了修改这个问题，我们使用我们已经熟知的补救方法：一个自定义错误类型。再一次，这是读取文件内容并将其转换为数字的代码：

```rust
use std::fs::File;
use std::io::{self, Read};
use std::num;
use std::path::Path;

// We derive `Debug` because all types should probably derive `Debug`.
// This gives us a reasonable human readable description of `CliError` values.
#[derive(Debug)]
enum CliError {
    Io(io::Error),
    Parse(num::ParseIntError),
}

fn file_double_verbose<P: AsRef<Path>>(file_path: P) -> Result<i32, CliError> {
    let mut file = try!(File::open(file_path).map_err(CliError::Io));
    let mut contents = String::new();
    try!(file.read_to_string(&mut contents).map_err(CliError::Io));
    let n: i32 = try!(contents.trim().parse().map_err(CliError::Parse));
    Ok(2 * n)
}
```

注意我们仍然有`map_err`的调用。为神马？好吧，回忆[`try!`](#code-try-def)和[`From`](#code-from-def)的定义。问题是这里并没有`impl`的实现允许我们将一些像`io::Error`和`num::ParseIntError`这样的错误类型转换为我们的自定义类型`CliError`。当然，这个问题很好修改！`CliError`都是我们定义的，我们可以为其实现`From`。

```rust
# #[derive(Debug)]
# enum CliError { Io(io::Error), Parse(num::ParseIntError) }
use std::io;
use std::num;

impl From<io::Error> for CliError {
    fn from(err: io::Error) -> CliError {
        CliError::Io(err)
    }
}

impl From<num::ParseIntError> for CliError {
    fn from(err: num::ParseIntError) -> CliError {
        CliError::Parse(err)
    }
}
```

所有这些实现都是告诉`From`如何从其他类型创建一个`CliError`。在我们的例子中，构造函数就像调用相应的值构造器那样简单。确实，这通常很简单。

最后我们可以重写`file_double`：

```rust
# use std::io;
# use std::num;
# enum CliError { Io(::std::io::Error), Parse(::std::num::ParseIntError) }
# impl From<io::Error> for CliError {
#     fn from(err: io::Error) -> CliError { CliError::Io(err) }
# }
# impl From<num::ParseIntError> for CliError {
#     fn from(err: num::ParseIntError) -> CliError { CliError::Parse(err) }
# }

use std::fs::File;
use std::io::Read;
use std::path::Path;

fn file_double<P: AsRef<Path>>(file_path: P) -> Result<i32, CliError> {
    let mut file = try!(File::open(file_path));
    let mut contents = String::new();
    try!(file.read_to_string(&mut contents));
    let n: i32 = try!(contents.trim().parse());
    Ok(2 * n)
}
```

我们做的唯一一件事就是去掉了`map_err`调用。他们不再必要因为`try!`宏对错误类型调用了`From::from`。这一切可以工作因为我们对所有可能出现的错误类型提供了`From`实现。

如果我们修改我们的`file_double`函数来进行一些其他操作，例如，把字符串转换为浮点数，那么我们需要给我们的错误类型增加一个新变量：

```rust
use std::io;
use std::num;

enum CliError {
    Io(io::Error),
    ParseInt(num::ParseIntError),
    ParseFloat(num::ParseFloatError),
}
```

并增加一个新的`From`实现：

```rust
# enum CliError {
#     Io(::std::io::Error),
#     ParseInt(num::ParseIntError),
#     ParseFloat(num::ParseFloatError),
# }

use std::num;

impl From<num::ParseFloatError> for CliError {
    fn from(err: num::ParseFloatError) -> CliError {
        CliError::ParseFloat(err)
    }
}
```

一切搞定！

### <a name="advice-for-library-writers"></a>给库编写者的建议

如果你的库需要报告一些自定义错误，那么你可能应该定义你自己的错误类型。由你决定是否暴露它的表示（例如[`ErrorKind`](http://doc.rust-lang.org/std/io/enum.ErrorKind.html)）或者把它隐藏起来（例如[`ParseIntError`](http://doc.rust-lang.org/std/num/struct.ParseIntError.html)）。不过你怎么做，相比`String`表示多少提供一些关于错误的信息通常是好的实践。不过说实话，这根据使用情况大有不同。

最少，你可能应该实现[`Error`](http://doc.rust-lang.org/std/error/trait.Error.html)trait。这会给你的库的用户以[处理错误](#the-real-try-macro)的最小灵活性。实现`Error`trait 也意味着可以确保用户能够获得一个错误的字符串表示（因为它实现了`fmt::Debug`和`fmt::Display`）。

不仅如此，为你的错误类型提供`From`实现也是很有用的。这允许你（库作者）和你的用户[组合更详细的错误](#composing-custom-error-types)。例如，[`csv::Error`](http://burntsushi.net/rustdoc/csv/enum.Error.html)提供了`io::Error`和`byteorder::Error`。

最后，根据你的风格，你也许想要定义一个[`Result`类型别名](#the-result-type-alias-idiom)，尤其是如果你的库定义了一个单一的错误类型。这被用在了标准库的[`io::Result`](http://doc.rust-lang.org/std/io/type.Result.html)和[`fmt::Result`](http://doc.rust-lang.org/std/fmt/type.Result.html)中。

## <a name="case-study-a-program-to-read-population-data"></a>案例学习：一个读取人口数据的程序

这一部分很长，并且根据你的背景，它可能显得更加复杂。虽然有很多示例代码以及散文一样的解释，但大部分都被设计为教科书式的。那么，我们要开始点新东西了：一个案例学习。

为此，为此我们将要建立一个可以让你查询真实世界人口数据的命令行程序。目标是简单的：你给出一个地点接着它会告诉你人口。虽然这很简单，但仍有很多地方我们可能犯错。

我们将使用的数据来自[Data Science Toolkit](https://github.com/petewarden/dstkdata)。我为这个练习准备了一些数据。你要么可以获取[世界人口数据](http://burntsushi.net/stuff/worldcitiespop.csv.gz)（41 MB gzip 压缩，145 MB 未压缩）或者只使用[US 人口数据](http://burntsushi.net/stuff/uscitiespop.csv.gz)（2.2 MB gzip 压缩，7.2 MB 未压缩）。

直到目前为止，我们的代码一直限制在 Rust 标准库之内。但是对于一个像这样的真实的任务，我们至少想要一些解析 CSV 数据，解析程序参数以及将其自动转换为 Rust 类型的东西。为此，我们将使用[`csv`](https://crates.io/crates/csv)，以及[`rustc-serialize`](https://crates.io/crates/rustc-serialize)crate。

### <a name="initial-setup"></a>初始化

我们不打算花很多时间在使用 Cargo 创建一个项目上，因为这在 [Cargo 部分](Getting Started 准备.md#hello-cargo)和 [Cargo 文档](http://doc.crates.io/guide.html)中已被讲解。

为了从头开始，运行`cargo new --bin city-pop`并确保你的`Cargo.toml`看起来像这样：

```toml
[package]
name = "city-pop"
version = "0.1.0"
authors = ["Andrew Gallant <jamslam@gmail.com>"]

[[bin]]
name = "city-pop"

[dependencies]
csv = "0.*"
rustc-serialize = "0.*"
getopts = "0.*"
```

你应该已经可以运行了：

```bash
cargo build --release
./target/release/city-pop
# Outputs: Hello, world!
```

### <a name="argument-parsing"></a>参数解析

让我们搞定参数解析，我们不会涉及太多关于 Getopts 的细节，不过有[一些不错的文档](http://doc.rust-lang.org/getopts/getopts/index.html)。简单的说就是 Getopts 生成了一个参数解析器并通过要给选项的 vector（事实是一个隐藏于一个结构体和一堆方法之下的 vector）生成了一个帮助信息。一旦解析结束，解析器返回一个记录了匹配到定义项内容的结构体，和剩下“自由”的参数。从这里我们可以互获取 flag，实例，任何程序传递给我们的，以及他们都有什么参数。这是我们的程序，它有合适的`extern crate`语句以及 Getopts 的基本参数操作：

```rust
extern crate getopts;
extern crate rustc_serialize;

use getopts::Options;
use std::env;

fn print_usage(program: &str, opts: Options) {
    println!("{}", opts.usage(&format!("Usage: {} [options] <data-path> <city>", program)));
}

fn main() {
    let args: Vec<String> = env::args().collect();
    let program = &args[0];

    let mut opts = Options::new();
    opts.optflag("h", "help", "Show this usage message.");

    let matches = match opts.parse(&args[1..]) {
        Ok(m)  => { m }
        Err(e) => { panic!(e.to_string()) }
    };
    if matches.opt_present("h") {
        print_usage(&program, opts);
        return;
    }
    let data_path = &matches.free[0];
    let city: &str = &matches.free[1];

    // Do stuff with information.
}
```

首先，我们获取一个传递给我们程序的 vector。接着我们我们储存第一个参数，因为我们知道那是程序名。当一切搞定，我们设置我们的参数 flag，在这里是一个简单的提示信息 flag。当我们设置了参数 flag 之后，我们使用`Options.parse`解析参数列表（从 1 开始，因为 0 是程序名）。如果这成功了，我们被解析的对象赋值给`matches`，如果失败了，我们 panic。接着，我们检查用户是否传递了帮助 flag，如果是就打印使用帮助信息。帮助信息选项是 Getopts 构建的，所以为了打印用法信息所有我们需要做的就是告诉它我们想要打印什么名字和模板。如果用户并没有传递帮助 flag，我们把相应的参数赋值给合适的变量。

### <a name="writing-the-logic"></a>编写逻辑

每个人写代码的方式各有不同，不过一般错误处理都是我们最后会思考的事情。这对程序整体的设计并不好，不过它对快速原型有帮助。因为 Rust 强制我们进行显示的错误处理（通过让我们调用`unwrap`），这样很容易看出我们的程序的那一部分可以造成错误。

在这个案例学习中，逻辑真的很简单。所有我们要做的就是解析给我们的 CSV 数据并打印出匹配的行的一个字段。让我们开始吧。（确保在你的文件开头加上`extern crate csv;`。）

```rust
use std::fs::File;

// This struct represents the data in each row of the CSV file.
// Type based decoding absolves us of a lot of the nitty gritty error
// handling, like parsing strings as integers or floats.
#[derive(Debug, RustcDecodable)]
struct Row {
    country: String,
    city: String,
    accent_city: String,
    region: String,

    // Not every row has data for the population, latitude or longitude!
    // So we express them as `Option` types, which admits the possibility of
    // absence. The CSV parser will fill in the correct value for us.
    population: Option<u64>,
    latitude: Option<f64>,
    longitude: Option<f64>,
}

fn print_usage(program: &str, opts: Options) {
    println!("{}", opts.usage(&format!("Usage: {} [options] <data-path> <city>", program)));
}

fn main() {
    let args: Vec<String> = env::args().collect();
    let program = &args[0];

    let mut opts = Options::new();
    opts.optflag("h", "help", "Show this usage message.");

    let matches = match opts.parse(&args[1..]) {
        Ok(m)  => { m }
        Err(e) => { panic!(e.to_string()) }
    };

    if matches.opt_present("h") {
        print_usage(&program, opts);
        return;
    }

    let data_path = &matches.free[0];
    let city: &str = &matches.free[1];

    let file = File::open(data_path).unwrap();
    let mut rdr = csv::Reader::from_reader(file);

    for row in rdr.decode::<Row>() {
        let row = row.unwrap();

        if row.city == city {
            println!("{}, {}: {:?}",
                row.city, row.country,
                row.population.expect("population count"));
        }
    }
}
```

让我们概括下错误。我们可以从明显的开始：三个`unwrap`被调用的地方：

1. [`File::open`](http://doc.rust-lang.org/std/fs/struct.File.html#method.open)可能返回[`io::Error`](http://doc.rust-lang.org/std/io/struct.Error.html)。
2. [`csv::Reader::decode`](http://burntsushi.net/rustdoc/csv/struct.Reader.html#method.decode)一次解码一行，而且[解码一个记录](http://burntsushi.net/rustdoc/csv/struct.DecodedRecords.html)（查看`Iterator`实现的关联类型`Item`）可能产生一个[`csv::Error`](http://burntsushi.net/rustdoc/csv/enum.Error.html)。

3. 如果`row.population`是`None`，那么调用`expect`会 panic。

还有其他的吗？如果我们无法找到一个匹配的城市呢？想`grep`这样的工具会返回一个错误码，所以可能我们也应该这么做。所以我们有特定于我们的问题，IO 错误和 CSV 解析错误的逻辑错误。我们将探索两个不同方式来处理这个问题。

我像从`Box<Error>`开始。接着，我们看看如何定义有用的自定义错误类型。

### <a name="error-handling-with-boxerror"></a>使用`Box<Error>`处理错误

`Box<Error>`的好处是它刚刚够用。你并不需要定义你自己的错误类型而且也不需要任何`From`实现。缺点是因为`Box<Error>`是一个 trait 对象，这意味着编译器无法再推导出底层类型。

[之前](#the-limits-of-combinators)我们开始了把我们函数类型从`T`变成`Result<T, OurErrorType>`的重构。在这个例子中，`OurErrorType`就是`Box<Error>`。不过`T`是什么？或者我们可以给`main`添加一个返回类型吗？

第二个问题的答案是不行，我们不能这么做。这意味着我们需要写一个新函数。不过`T`是什么？最简单的办法是返回一个作为`Vec<Row>`的匹配上的`Row`的值。（更好的代码会返回一个迭代器，不过这是一个留给读者的练习。）

让我们重构函数，不过保持对`unwrap`的调用。注意我们选择处理一个不存在的人口数行的方式是单纯的忽略它。

```rust
use std::path::Path;

struct Row {
    // This struct remains unchanged.
}

struct PopulationCount {
    city: String,
    country: String,
    // This is no longer an `Option` because values of this type are only
    // constructed if they have a population count.
    count: u64,
}

fn print_usage(program: &str, opts: Options) {
    println!("{}", opts.usage(&format!("Usage: {} [options] <data-path> <city>", program)));
}

fn search<P: AsRef<Path>>(file_path: P, city: &str) -> Vec<PopulationCount> {
    let mut found = vec![];
    let file = File::open(file_path).unwrap();
    let mut rdr = csv::Reader::from_reader(file);
    for row in rdr.decode::<Row>() {
        let row = row.unwrap();
        match row.population {
            None => { } // Skip it.
            Some(count) => if row.city == city {
                found.push(PopulationCount {
                    city: row.city,
                    country: row.country,
                    count: count,
                });
            },
        }
    }
    found
}

fn main() {
    let args: Vec<String> = env::args().collect();
    let program = &args[0];

    let mut opts = Options::new();
    opts.optflag("h", "help", "Show this usage message.");

    let matches = match opts.parse(&args[1..]) {
        Ok(m)  => { m }
        Err(e) => { panic!(e.to_string()) }
    };
    if matches.opt_present("h") {
        print_usage(&program, opts);
        return;
    }

    let data_path = &matches.free[0];
    let city: &str = &matches.free[1];
    for pop in search(data_path, city) {
        println!("{}, {}: {:?}", pop.city, pop.country, pop.count);
    }
}
```

虽然我们去掉了一个`expect`调用（它是一个比`unwrap`要好的变体），我们仍要处理任何不存在的搜索结果。

为了把这转化为合适的错误处理，我们需要做如下事情：

1. 把`search`的返回值类型改为`Result<Vec<PopulationCount>, Box<Error>>`.
2.  使用[`try!`宏]()这样会返回错误给调用者而不是使程序 panic。
3. 处理`mian`中的错误。

让我们试试：

```rust
use std::error::Error;

// The rest of the code before this is unchanged.

fn search<P: AsRef<Path>>
         (file_path: P, city: &str)
         -> Result<Vec<PopulationCount>, Box<Error>> {
    let mut found = vec![];
    let file = try!(File::open(file_path));
    let mut rdr = csv::Reader::from_reader(file);
    for row in rdr.decode::<Row>() {
        let row = try!(row);
        match row.population {
            None => { } // Skip it.
            Some(count) => if row.city == city {
                found.push(PopulationCount {
                    city: row.city,
                    country: row.country,
                    count: count,
                });
            },
        }
    }
    if found.is_empty() {
        Err(From::from("No matching cities with a population were found."))
    } else {
        Ok(found)
    }
}
```

现在我们用`try!(x)`代替了`x.unwrap()`。因为我们的函数返回一个`Result<T, E>`，`try!`宏在出现错误时会提早返回。

在`search`的结尾我们也用了[相应的`From`实现](http://doc.rust-lang.org/std/convert/trait.From.html)把一个字符串转换为一个错误类型：

```rust
// We are making use of this impl in the code above, since we call `From::from`
// on a `&'static str`.
impl<'a> From<&'a str> for Box<Error>

// But this is also useful when you need to allocate a new string for an
// error message, usually with `format!`.
impl From<String> for Box<Error>
```

因为`search`现在返回`Result<T, E>`，`main`应该在调用`search`时使用 case analysis：

```rust
...
    match search(data_path, city) {
        Ok(pops) => {
            for pop in pops {
                println!("{}, {}: {:?}", pop.city, pop.country, pop.count);
            }
        }
        Err(err) => println!("{}", err)
    }
...
```

现在你看到了我们如何正确的处理`Box<Error>`，让我们尝试一种使用我们自定义错误类型的不同方式。不过首先，让我们先放下错误处理并快速的添加从`stdin`读取的功能。

### <a name="reading-from-stdin"></a>从标准输入读取

在我们的程序中，我们接受一个单文件输入并进行一次数据解析。这意味着我们可能需要能够接受标准输入。不过你可能也喜欢现在的格式——所以让我们同时拥有两者吧！

添加标准输入支持是非常简单的。我们只必需做三件事：

1. 修改程序参数，这样一个单独的参数——城市——可以被接受，同时人口数据从标准输入读取。
2. 修改程序，这样一个`-f`选项可以接受文件，如果它没有从标准输入传递。
3. 修改`search`函数接受一个可选的文件路径。当为`None`时，它应该知道从标准输入读取。

首先，这是新的使用方法函数：

```rust
fn print_usage(program: &str, opts: Options) {
    println!("{}", opts.usage(&format!("Usage: {} [options] <city>", program)));
}
```

当然我们需要加入参数处理的代码：

```rust
...
    let mut opts = Options::new();
    opts.optopt("f", "file", "Choose an input file, instead of using STDIN.", "NAME");
    opts.optflag("h", "help", "Show this usage message.");
    ...
    let data_path = matches.opt_str("f");

    let city = if !matches.free.is_empty() {
        &matches.free[0]
    } else {
        print_usage(&program, opts);
        return;
    };

    match search(&data_path, city) {
        Ok(pops) => {
            for pop in pops {
                println!("{}, {}: {:?}", pop.city, pop.country, pop.count);
            }
        }
        Err(err) => println!("{}", err)
    }
...
```

我们已经通过显示使用方法信息来提高了用户体验，而不是因为`city`这个额外的参数不存在导致索引越界而 panic，

修改`search`需要一点技巧。`csv`crate 可以用[任何实现了`io::Read`的类型]()构建一个解析器。不过我们如何对这两个类型（注：因该是`Option`的两个值）使用相同的代码呢？事实上有多种方法可以做到。其中之一是重写`search`为接受一个满足`io::Read`的`R`类型参数的泛型。另一个办法是使用 trait 对象：

```rust
use std::io;

// The rest of the code before this is unchanged.

fn search<P: AsRef<Path>>
         (file_path: &Option<P>, city: &str)
         -> Result<Vec<PopulationCount>, Box<Error>> {
    let mut found = vec![];
    let input: Box<io::Read> = match *file_path {
        None => Box::new(io::stdin()),
        Some(ref file_path) => Box::new(try!(File::open(file_path))),
    };
    let mut rdr = csv::Reader::from_reader(input);
    // The rest remains unchanged!
}
```

### <a name="error-handling-with-a-custom-type"></a>用自定义类型处理错误

之前，我们学习了如何[用自定义错误类型组合错误](#composing-custom-error-types)。我们定义了一个`enum`的错误类型并实现了`Error`和`From`。

因为我们有三个不同的错误（IO，CSV 解析和未找到），让我们定义一个三个变体的`enum`：

```rust
#[derive(Debug)]
enum CliError {
    Io(io::Error),
    Csv(csv::Error),
    NotFound,
}
```

现在让我们实现`Display`和`Error`：

```rust
use std::fmt;

impl fmt::Display for CliError {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        match *self {
            CliError::Io(ref err) => err.fmt(f),
            CliError::Csv(ref err) => err.fmt(f),
            CliError::NotFound => write!(f, "No matching cities with a \
                                             population were found."),
        }
    }
}

impl Error for CliError {
    fn description(&self) -> &str {
        match *self {
            CliError::Io(ref err) => err.description(),
            CliError::Csv(ref err) => err.description(),
            CliError::NotFound => "not found",
        }
    }

    fn cause(&self) -> Option<&Error> {
        match *self {
            CliError::Io(ref err) => Some(err),
            CliError::Csv(ref err) => Some(err),
            // Our custom error doesn't have an underlying cause,
            // but we could modify it so that it does.
            CliError::NotFound => None,
        }
    }
}
```

在我们可以在`search`函数中使用`CliError`之前，我们需要提供一系列的`From`实现。我们如何知晓该提供那个实现呢？好吧，我们得把`io::Error`和`csv::Error`都转换为`CliError`。他们都只是外部错误，所以目前我们只需要两个`From`实现：

```rust
impl From<io::Error> for CliError {
    fn from(err: io::Error) -> CliError {
        CliError::Io(err)
    }
}

impl From<csv::Error> for CliError {
    fn from(err: csv::Error) -> CliError {
        CliError::Csv(err)
    }
}
```

因为[`try!`的定义](#code-try-def)`From`的实现是很重要的。尤其是在这个例子中，如果出现错误，错误的`From::from`被调用，将被转换为我们的错误类型`CliError`。

当实现了`From`，我们只需要对`search`函数进行两个小的修改：返回值类型和“未找到”错误。这是全部的代码：

```rust
fn search<P: AsRef<Path>>
         (file_path: &Option<P>, city: &str)
         -> Result<Vec<PopulationCount>, CliError> {
    let mut found = vec![];
    let input: Box<io::Read> = match *file_path {
        None => Box::new(io::stdin()),
        Some(ref file_path) => Box::new(try!(File::open(file_path))),
    };
    let mut rdr = csv::Reader::from_reader(input);
    for row in rdr.decode::<Row>() {
        let row = try!(row);
        match row.population {
            None => { } // Skip it.
            Some(count) => if row.city == city {
                found.push(PopulationCount {
                    city: row.city,
                    country: row.country,
                    count: count,
                });
            },
        }
    }
    if found.is_empty() {
        Err(CliError::NotFound)
    } else {
        Ok(found)
    }
}
```

不再需要其他的修改。

### <a name="adding-functionality"></a>增加功能

编写泛型代码是很好的，因为泛用性是很酷的，并且之后会变得很有用。不过有时并不值得这么做。看看我们上一部分我们是怎么做的：

1. 定义了一个新的错误类型。
2.增加`Error`，`Display`和两个`From`实现。

这里最大的缺点是我们的程序并没有改进多少。这里仍然有很多用`enum`代表错误的额外操作，特别是在这样短小的程序里。

像我们这样使用自定义错误类型的一个有用的方面是`main`函数现在可以选择不同的处理错误的方式。之前使用`Box<Error>`的时候并没有什么选择：只能打印信息。我们现在仍可以这么做，不过只是在我们想这么做的时候，例如，添加一个`--quiet` flag？`--quiet` flag 应该能够消除任何冗余的输出。

现在如果程序不能匹配一个城市，它会打印一个信息说它不能。这可能有点蠢，尤其是你想要你的程序能在 shell 脚本中使用的时候。

所以让我们开始增加 flag。就像之前一样，我们需要修改用法字符串，并给选项变量添加 flag。当我们写完这些，Getopts 会搞定剩下的操作：

```rust
...
    let mut opts = Options::new();
    opts.optopt("f", "file", "Choose an input file, instead of using STDIN.", "NAME");
    opts.optflag("h", "help", "Show this usage message.");
    opts.optflag("q", "quiet", "Silences errors and warnings.");
...
```

现在我们只需要实现我们的“安静”功能。这要求我们修改`mian`中的 case analysis：

```rust
use std::process;
...
    match search(&data_path, city) {
        Err(CliError::NotFound) if matches.opt_present("q") => process::exit(1),
        Err(err) => panic!("{}", err),
        Ok(pops) => for pop in pops {
            println!("{}, {}: {:?}", pop.city, pop.country, pop.count);
        }
    }
...
```

当然，在出现 IO 错误或者数据解析失败时我们并不想变得安静。因此，我们用 case analysis 来检查错误类型是否是`NotFound`以及`--quiet`是否被启用。如果，搜索失败了，我们仍然使用一个错误码退出（使用`grep`的传统）。

如果我们还在用`Box<Error>`，那么实现`--quiet`功能将变得很复杂。

我们的案例学习讲了很多东西。从这时起，你应该能够在现实生活中编写带有合适错误处理的程序和库了。

## <a name="the-short-story"></a>精简版

因为这个章节很长，有一个 Rust 错误处理的快速总结是很有帮助的。有很多好的“经验规则”。需要强调的是他们**并非**教条。这里每一个建议都可能有适当的理由予以反驳！

* 如果你在写小的事例代码这时错误处理显得负担过重，可能使用`unwrap`（[`Result::unwrap`](http://doc.rust-lang.org/std/result/enum.Result.html#method.unwrap)，[`Option::unwrap`](http://doc.rust-lang.org/std/option/enum.Option.html#method.unwrap)，或是更可取的[`Option::expect`](http://doc.rust-lang.org/std/option/enum.Option.html#method.expect)）是足够的。你的代码的客户应该知道如何正确的处理错误。（如果他们并不知道，教会他们吧！）

* 如果你在 hack（quick 'n' dirty）程序，不要为你使用`unwrap`而感羞愧。不过你被警告过了：如果别人踩到了坑，不要因为他们对糟糕的错误信息火冒三丈而感到惊讶！

* 如果你在 hack 程序并对 panic 感到羞愧，那么使用`String`或者`Box<Error>`作为你的错误类型。

* 否则，在程序中，定义你自己的错误类型并实现合适的[`From`](http://doc.rust-lang.org/std/convert/trait.From.html)和[`Error`](http://doc.rust-lang.org/std/error/trait.Error.html)来让[`try!`](http://doc.rust-lang.org/std/macro.try!.html)宏变得更工程化。

* 如果你在写一个库并且它可能产生错误，定义你自己的错误类型并实现[`std::error::Error`](http://doc.rust-lang.org/std/error/trait.Error.html) trait。如果可以的话，实现[`From`](http://doc.rust-lang.org/std/convert/trait.From.html)来让你的库代码和调用者的代码更加容易编写。（因为 Rust 的一致性规则，调用者不能为你的错误类型实现`From`，所以你的库应该实现。）

* 学习定义于[`Option`](http://doc.rust-lang.org/std/option/enum.Option.html)和[`Result`](http://doc.rust-lang.org/std/result/enum.Result.html)中的组合。只使用他们有时可能比较累人，不过我个人发现合理的结合`try!`和组合是比较诱人的。`and_then`，`map`和`unwrap_or`是我们的最爱。
