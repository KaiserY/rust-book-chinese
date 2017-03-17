# 过程宏（和自定义导出）

> [procedural-macros.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/procedural-macros.md)
> <br>
> commit 3075c1f65e08e0b52dcf872588358daffef8b47c

在本书接下来的部分，你将看到 Rust 提供了一个叫做“导出（derive）”的机制来轻松的实现 trait。例如，

```rust
#[derive(Debug)]
struct Point {
    x: i32,
    y: i32,
}
```

is a lot simpler than

```rust
struct Point {
    x: i32,
    y: i32,
}

use std::fmt;

impl fmt::Debug for Point {
    fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
        write!(f, "Point {{ x: {}, y: {} }}", self.x, self.y)
    }
}
```

Rust 包含很多可以导出的 trait，不过也允许定义你自己的 trait。我们可以通过一个叫做“过程宏”的 Rust 功能来实现这个效果。最终，过程宏将会允许 Rust 所有类型的高级元编程，不过现在只能自定义导出。

## Hello World

首先需要做的就是为我们的项目新建一个 crate。

```bash
$ cargo new --bin hello-world
```

我们想要实现的就是能够在导出的类型上调用`hello_world()`。就想这样：

```rust,ignore
#[derive(HelloWorld)]
struct Pancakes;

fn main() {
    Pancakes::hello_world();
}
```

再来一些给力的输出，比如“Hello, World! 我叫煎饼（←_←）。”

继续并从用户的角度编写我们的宏。在`src/main.rs`中：

```rust,ignore
#[macro_use]
extern crate hello_world_derive;

trait HelloWorld {
    fn hello_world();
}

#[derive(HelloWorld)]
struct FrenchToast;

#[derive(HelloWorld)]
struct Waffles;

fn main() {
    FrenchToast::hello_world();
    Waffles::hello_world();
}
```

好的。现在我们只需实际编写我们的过程宏。目前，过程宏需要位于它自己的 crate 中。最终这个限制会解除，不过现在是必须的。为此，有一个惯例是，对于一个叫`foo`的 crate，一个自定义的过程宏叫做`foo-derive`。让我们在`hello-world`项目中新建一个叫做`hello-world-derive`的 crate。

```bash
$ cargo new hello-world-derive
```

为了确保`hello-world` crate 能够找到这个新创建的 crate 我们把它加入到项目 toml 文件中：

```toml
[dependencies]
hello-world-derive = { path = "hello-world-derive" }
```

这里是一个`hello-world-derive` crate 源码的例子：

```rust,ignore
extern crate proc_macro;
extern crate syn;
#[macro_use]
extern crate quote;

use proc_macro::TokenStream;

#[proc_macro_derive(HelloWorld)]
pub fn hello_world(input: TokenStream) -> TokenStream {
    // Construct a string representation of the type definition
    let s = input.to_string();
    
    // Parse the string representation
    let ast = syn::parse_macro_input(&s).unwrap();

    // Build the impl
    let gen = impl_hello_world(&ast);
    
    // Return the generated impl
    gen.parse().unwrap()
}
```

这里有很多内容。我们引入了两个新的 crate：[`syn`]和[`quote`]。你可能注意到了，`input: TokenSteam`直接就被转换成了一个`String`。这个字符串是我们要导出的`HelloWorld`Rust 代码的字符串形式。现在，能对`TokenStream`做的唯一的事情就是把它转换为一个字符串。将来会有更丰富的 API。

所以我们真正需要做的是能够把 Rust 代码_解析_成有用的东西。这正是`syn`出场机会。`syn`是一个解析 Rust 代码的 crate。我们引入的另外一个 crate 是`quote`。它本质上与`syn`是成双成对的，因为它可以轻松的生成 Rust 代码。也可以自己编写这些功能，不过使用这些库会更加轻松。编写一个完整 Rust 代码解析器可不是一个简单的工作。

[`syn`]: https://crates.io/crates/syn
[`quote`]: https://crates.io/crates/quote

这些代码注释提供了我们总体策略的很好的解释。我们将为导出的类型提供一个`String`类型的 Rust 代码，用`syn`解析它，（使用`quote`）构建`hello_world`的实现，接着把它传递回给 Rust 编译器。

最后一个要点：这里有一些`unwrap()`，如果你要为过程宏提供一个错误，那么你需要`panic!`并提供错误信息。这里，我们从简实现。

好的，让我们编写`impl_hello_world(&ast)`。

```rust,ignore
fn impl_hello_world(ast: &syn::MacroInput) -> quote::Tokens {
    let name = &ast.ident;
    quote! {
        impl HelloWorld for #name {
            fn hello_world() {
                println!("Hello, World! My name is {}", stringify!(#name));
            }
        }
    }
}
```

这里就是`quote`出场的地方。`ast`参数是一个代表我们类型（可以是一个`struct`或`enum`）的结构体。查看[文档](https://docs.rs/syn/0.10.5/syn/struct.MacroInput.html)。这里有一些有用的信息。我们可以通过`ast.ident`获取类型的信息。`quote!`宏允许我们编写想要返回的 Rust 代码并把它转换为`Tokens`。`quote!`让我们可以使用一些炫酷的模板机制；简单的使用`#name`，`quote!`就会把它替换为叫做`name`的变量。你甚至可以类似常规宏那样进行一些重复。请查看这些[文档](https://docs.rs/quote)，这里有一些好的介绍。

应该就这些了。噢，对了，我们需要在`hello-world-derive` crate 的`cargo.toml`中添加`syn`和`quote`的依赖。

```toml
[dependencies]
syn = "0.10.5"
quote = "

这样就 OK 了。尝试编译`hello-world`。

```bash
error: the `#[proc_macro_derive]` attribute is only usable with crates of the `proc-macro` crate type
 --> hello-world-derive/src/lib.rs:8:3
  |
8 | #[proc_macro_derive(HelloWorld)]
  |   ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
```

好吧，看来我们需要把`hello-world-derive` crate 声明为`proc-macro`类型。怎么做呢？像这样：


```toml
[lib]
proc-macro = true
```

现在好了，编译`hello-world`。现在执行`cargo run`将会输出：

```bash
Hello, World! My name is FrenchToast
Hello, World! My name is Waffles
```

本小节到此为止！