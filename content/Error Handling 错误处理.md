# 错误处理

> [error-handling.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/error-handling.md)
> <br>
> commit e26279db48cc5510a13f0e97bde97ccd2d2a1854

就像大多数编程语言，Rust 鼓励程序猿以特定的方式处理错误。一般来讲，错误处理被分割为两个大类：异常和返回值。Rust 选择了返回值。

在这一部分，我们试图提供一个全面的 Rust 如何处理错误的解决方案。不仅如此，我们也尝试一次一点的介绍错误处理，这样当你离开时会有一个所有东西如何协调的坚实理解。

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
    * [真正的`try!`macro](#the-real-try-macro)
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

保持代码的可组合性是很重要的，因为没有这个要求，我们可能在遇到没想到的情况时[panic](https://github.com/rust-lang/rust/blob/master/src/doc/std/macro.panic!.html)。（`panic`导致当前线程结束，而在大多数情况，导致整个程序结束。）这是一个例子：

```rust
// Guess a number between 1 and 10.
// If it matches the number we had in mind, return true. Else, return false.
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
thread '<main>' panicked at 'Invalid number: 11', src/bin/panic-simple.rs:5
```

这是另一个稍微不那么违和的例子。一个接受一个整型作为参数，乘以二并打印的程序。

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

`Option`类型[定义在标准库中](https://github.com/rust-lang/rust/blob/master/src/doc/std/option/enum.Option.html)：

```rust
enum Option<T> {
    None,
    Some(T),
}
```

`Option`类型是一个 Rust 类型系统用于表达*不存在的可能性（possibility of absence）*的方式。将不存在的可能性编码进类型系统是一个重要概念，因为它会强迫编译器处理不存在的情况。让我们看看一个尝试在一个字符串中找一个字符的例子：

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
# fn find(_: &str, _: char) -> Option<usize> { None }
fn main() {
    let file_name = "foobar.rs";
    match find(file_name, '.') {
        None => println!("No file extension found."),
        Some(i) => println!("File extension: {}", &file_name[i+1..]),
    }
}
```

这段代码使用[模式识别](https://github.com/rust-lang/rust/blob/master/src/doc/book/patterns.html)来对`find`函数的返回的`Option<usize>`进行 case analysis。事实上，case analysis 是唯一能获取`Option<T>`中存储的值的方式。这意味着你，作为一个程序猿，必须处理当`Option<T>`是`None`而不是`Some(t)`的情况。

不过稍等，那我们[之前](https://github.com/rust-lang/rust/blob/master/src/doc/book/error-handling.md#code-unwrap-double)使用的`unwrap`呢？那里并没有 case analysis！相反，case analysis 被放入了`unwrap`方法中。如果你想的话你可以自己定义它：

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

在[之前的例子](https://github.com/rust-lang/rust/blob/master/src/doc/book/error-handling.md#code-option-ex-string-find)中，我们看到了如何用`find`发现文件名的扩展名。当然，并不是所有文件名都有一个`.`，所以可能文件名并没有扩展名。不存在的可能性被编码进了使用`Option<T>`的类型。换句话说，编译器将会强制我们描述一个扩展名不存在的可能性。在我们的例子中，我们只打印出一个说明情况的信息。

获取一个文件名的扩展名是一个很常见的操作，所以把它放进一个函数是很有道理的：

```rust
# fn find(_: &str, _: char) -> Option<usize> { None }
// Returns the extension of the given file name, where the extension is defined
// as all characters proceeding the first `.`.
// If `file_name` has no `.`, then `None` is returned.
fn extension_explicit(file_name: &str) -> Option<&str> {
    match find(file_name, '.') {
        None => None,
        Some(i) => Some(&file_name[i+1..]),
    }
}
```

（专业建议：不要用这段代码，相反使用标准库的[extension](https://github.com/rust-lang/rust/blob/master/src/doc/std/path/struct.Path.html#method.extension)方法。）

代码是简单的，不过重要的是注意到`find`的类型强迫我们考虑不存在的可能性。这是一个好事，因为这意味着编译器不会让我们不小心忘记了文件名没有扩展名的情况。另一方面，每次都像`extension_explicit`那样进行显式 case analysis 会变得有点无聊。

事实上，`extension_explicit`的 case analysis 遵循一个非常常见的模式：将`Option<T>`中的值映射为一个函数，除非它是`None`，这时，返回`None`。

Rust 拥有参数多态（parametric polymorphism），所以定义一个组合来抽象这个模式是很容易的：

```rust
fn map<F, T, A>(option: Option<T>, f: F) -> Option<A> where F: FnOnce(T) -> A {
    match option {
        None => None,
        Some(value) => Some(f(value)),
    }
}
```

事实上，`map`是标准库中的`Option<T>`[定义的一个方法](https://github.com/rust-lang/rust/blob/master/src/doc/std/option/enum.Option.html#method.map)。

用我们的新组合，我们可以重写我们的`extension_explicit`方法来去掉 case analysis：

```rust
# fn find(_: &str, _: char) -> Option<usize> { None }
// Returns the extension of the given file name, where the extension is defined
// as all characters proceeding the first `.`.
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

（注意`unwrap_or`是标准库中的`Option<T>`[定义的一个方法](https://github.com/rust-lang/rust/blob/master/src/doc/std/option/enum.Option.html#method.unwrap_or)，所以这里我们使用它而不是我们上面定义的独立的函数。别忘了看看更通用的[`unwrap_or_else`](https://github.com/rust-lang/rust/blob/master/src/doc/std/option/enum.Option.html#method.unwrap_or_else)方法。）

这里有另一个我们认为值得特别注意的 combinator：`and_then`。它让我们更容易的组合不同的承认不存在的可能性的计算。例如，这一部分的很多代码是关于找到一个给定文件的扩展名的。为此，你首先需要一个通常截取自文件路径的文件名。虽然大部分文件路径都有一个文件名，但并不是都有。例如，`.`，`..`，`/`。

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
  // implementation elided
  unimplemented!()
}
```

你可能认为我们应该用`map`组合来减少 case analysis，不过它的类型并不匹配。也就是说，`map`获取一个只处理（Option）内部值的函数。这导致那个函数总是[重新映射成了`Some`](https://github.com/rust-lang/rust/blob/master/src/doc/book/error-handling.md#code-option-map)。因此，我们需要一些类似`map`，不过允许调用者返回另一个`Option`的方法。它的泛型实现甚至比`map`更简单：

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

`Option`类型有很多其他[定义在标准库中的](https://github.com/rust-lang/rust/blob/master/src/doc/std/option/enum.Option.html)组合。过一边这个列表并熟悉他们的功能是一个好主意 —— 通常他们可以减少你的 case analysis。熟悉这些组合将会得到回报，因为他们很多也为`Result`类型定义了实现（相似的语义），而我们接下来会讲`Result`。

组合使用像`Option`这样的符合工程学的类型来减少显式 case analysis。他们也是可组合的因为他们允许调用者以他们自己的方式处理不存在的可能性。像`unwrap`这样的方法去掉了选择因为当`Option<T>`为`None`他们会 panic。

### <a name="the-result-type"></a>`Result`类型

`Result`类型也[定义于标准库中](https://github.com/rust-lang/rust/blob/master/src/doc/std/result)：

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

它把`Result`的第二个类型参数改为总是`()`（读作“单元”或“空元组”）。`()`类型只有一个值：`()`（没错，类型和值级别的形式是一样的！）

`Result`类型是一个代表一个计算的两个可能结果的方式。通常，一个结果是期望的值或者“`Ok`”而另一个意味着非预期的或者“`Err`”。

就像`Option`，`Result`在标准库中也[定义了一个`unwrap`方法](https://github.com/rust-lang/rust/blob/master/src/doc/std/result/enum.Result.html#method.unwrap)。让我们定义它：

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

这实际上与[`Option::unwrap`的定义](https://github.com/rust-lang/rust/blob/master/src/doc/book/error-handling.md#code-option-def-unwrap)一样，除了它在`panic!`信息中包含了错误信息。这让调试变得简单，不过也要求我们为`E`类型参数（它代表我们的错误类型）添加一个[`Debug`](https://github.com/rust-lang/rust/blob/master/src/doc/std/fmt/trait.Debug.html)限制。因为绝大部分类型应该满足`Debug`限制，这使得它可以在实际中使用。（`Debug`简单的意味着这个类型有合理的方式可以打印出人类可读的描述。）

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
thread '<main>' panicked at 'called `Result::unwrap()` on an `Err` value: ParseIntError { kind: InvalidDigit }', /home/rustbuild/src/rust-buildbot/slave/beta-dist-rustc-linux/build/src/libcore/result.rs:729
```

这是很难堪的，而且如果这在你所使用的库中出现了的话，可以理解你会很烦躁。相反，我们应该尝试在我们的函数里处理错误并让调用者决定该怎么做。这意味着改变`double_number`的返回值类型。不过改编成什么呢？好吧，这需要我们看看标准库中[`parse`方法](https://github.com/rust-lang/rust/blob/master/src/doc/std/primitive.str.html#method.parse)的签名：

```rust
impl str {
    fn parse<F: FromStr>(&self) -> Result<F, F::Err>;
}
```

额嗯。所以至少我们知道了我们需要使用一个`Result`。当然，也可以返回一个`Option`。毕竟，一个字符串要么能解析成一个数字要么不能，不是吗？这当然是一个合理的方式，不过实现内部区别了为什么字符串不能解析成数字。（要么是一个空字符串，一个无效的数位，太大或太小。）因此，使用`Result`更有道理因为我们想要比单纯的“不存在”提供更多信息。我们想要表明*为什么*解析会失败。你应该尝试再现这样的推理，当你面对一个`Opation`和`Result`之间的选择时。如果你可以提供详细的错误信息，那么大概你也应该提供。（我们会在后面详细讲到。）

好的，不过我们的返回值类型该怎么写呢？上面定义的`parse`方法对所有不同的标准库定义的数字类型是泛型的。我们也可以（应该）让我们的函数也是泛型的，不过这回让我们享受显式定义的好处。我们只关心`i32`，所以我们需要寻找[`FromStr`的实现](https://github.com/rust-lang/rust/blob/master/src/doc/std/primitive.i32.html)（在你的浏览器中用`CTRL-F`搜索“FromStr”）和[与它相关的类型`](https://github.com/rust-lang/rust/blob/master/src/doc/book/associated-types.html)`Err`。这么做我可以找出具体的错误类型。在这个例子中，它是[`std::num::ParseIntError`](https://github.com/rust-lang/rust/blob/master/src/doc/std/num/struct.ParseIntError.html)。最后我们可以重写函数：

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

组合是救星！就像`Opation`一样，`Result`有很多定义的组合方法。`Result`和`option`之间的常用组合有很大的交集。特别的，`map`就是其中之一：

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

常见的组合`Result`都有，包括[unwrap_or](https://github.com/rust-lang/rust/blob/master/src/doc/std/result/enum.Result.html#method.unwrap_or)和[and_then](https://github.com/rust-lang/rust/blob/master/src/doc/std/result/enum.Result.html#method.and_then)。另外，因为`Result`有第二个类型参数，这里有只影响错误类型的组合，例如[map_err](https://github.com/rust-lang/rust/blob/master/src/doc/std/result/enum.Result.html#method.map_err)（相对于`map`）和[or_else](https://github.com/rust-lang/rust/blob/master/src/doc/std/result/enum.Result.html#method.or_else)（相对于`and_then`）。

#### <a name="the-result-type-alias-idiom"></a>`Result`类型别名习惯

在标准库中，你可能经常看到像`Result<i32>`这样的类型。不过等等，[我们定义的`Result`](https://github.com/rust-lang/rust/blob/master/src/doc/book/error-handling.md#code-result-def)有两个类型参数。我么怎么能只指定一个呢？这里的关键是定义一个`Resule`类型别名来对一个特定类型固定其中一个类型参数。通常固定的类型是错误类型。例如，我们之前的解析整数例子可以重写成这样：

```rust
use std::num::ParseIntError;
use std::result;

type Result<T> = result::Result<T, ParseIntError>;

fn double_number(number_str: &str) -> Result<i32> {
    unimplemented!();
}
```

为什么我们应该这么做？好吧，如果我们有很多可能返回`ParseIntError`的函数，那么定义一个总是使用`ParseIntError `的别名就比每回都写一遍要方便很多。

这个习惯最突出的一点是标准库中的[`io::Result`](https://github.com/rust-lang/rust/blob/master/src/doc/std/io/type.Result.html)。通常，当你使用`io::Result<T>`，很明显你就是在使用`io`模块的类型别名而不是`std::result`的原始定义。（这个习惯也用于[`fmt::Result`](https://github.com/rust-lang/rust/blob/master/src/doc/std/fmt/type.Result.html)。）

### <a name="a-brief-interlude-unwrapping-isnt-evil"></a>小插曲：unwrapping 并不邪恶

如果你一路跟了过来，你可能注意到我们花了很大力气反对使用像`unwrap`这样会`panic`并终止你的程序的方法。通常来说，这是一个好的建议。

然而，`unwrap`仍然可以被明智的使用。具体如何正当化`unwrap`的使用是一个灰色区域并且理性的人可能不会同意。我会简述这个问题的一些个人看法。

* **在例子和简单快速的编码中。**有时你要写一个例子或小程序，这时错误处理一点也不重要。这种情形要击败`unwrap`的方便易用是很难的，所以它显得很合适。
* **当 panic 意味着程序中有 bug 的时候。**

## <a name="working-with-multiple-error-types"></a>处理多种错误类型
### <a name="composing-option-and-result"></a>组合`Option`和`Result`
### <a name="the-limits-of-combinators"></a>组合的限制
### <a name="early-returns"></a>提早返回
### <a name="the-try-macro"></a>`try!`宏
### <a name="defining-your-own-error-type"></a>定义你自己的错误类型
## <a name="standard-library-traits-used-for-error-handling"></a>用于错误处理的标准库 trait
### <a name="the-error-trait"></a>`Error`trait
### <a name="the-from-trait"></a>`From`trait
### <a name="the-real-try-macro"></a>真正的`try!`macro
### <a name="composing-custom-error-types"></a>组合自定义错误类型
### <a name="advice-for-library-writers"></a>给库编写者的建议
## <a name="case-study-a-program-to-read-population-data"></a>案例学习：一个读取人口数据的程序
### <a name="initial-setup"></a>初始化
### <a name="argument-parsing"></a>参数解析
### <a name="writing-the-logic"></a>编写逻辑
### <a name="error-handling-with-boxerror"></a>使用`Box<Error>`处理错误
### <a name="reading-from-stdin"></a>从标准输入读取
### <a name="error-handling-with-a-custom-type"></a>用自定义类型处理错误
### <a name="adding-functionality"></a>增加功能
## <a name="the-short-story"></a>精简版
