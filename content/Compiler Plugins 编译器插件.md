# 编译器插件

> [compiler-plugins.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/compiler-plugins.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

## 介绍

`rustc`可以加载编译器插件，它是由用户提供的库用来扩充编译器的行为，例如新的语法扩展，lint检查等。

一个插件是带有设计好的用来在`rustc`中注册扩展的**注册**（*registrar*）函数的一个动态库包装箱。其它包装箱可以使用`#![plugin(...)]`属性来装载这个扩展。查看[rustc::plugin](http://doc.rust-lang.org/rustc/plugin/)文档来获取更多关于定义和装载插件的机制。

如果属性存在的话，`#![plugin(foo(... args ...))]`传递的参数并不由`rustc`自身解释。它们被传递给插件的`Registry`[args方法](http://doc.rust-lang.org/rustc/plugin/registry/struct.Registry.html#method.args)。

在绝大多数情况中，一个插件应该**只**通过`#![plugin]`而不通过`extern crate`来使用。链接一个插件会将`libsyntax`和`librustc`加入到你的包装箱的依赖中。基本上你不会希望如此除非你在构建另一个插件。`plugin_as_library`lint会检查这些原则。

通常的做法是将插件放到它们自己的包装箱中，与任何那些会被库的调用者使用的`macro_rules!`宏或 Rust 代码分开。

## 语法扩展

插件可以有多种方法来扩展 Rust 的语法。一种语法扩展是宏过程。它们与[普通宏](5.35.Macros 宏.md)的调用方法一样，不过扩展是通过执行任意Rust代码在编译时操作[语法树](http://doc.rust-lang.org/syntax/ast/)进行的。

让我们写一个实现了罗马数字的插件[roman_numerals.rs]((https://github.com/rust-lang/rust/blob/master/src/test/run-pass-fulldeps/auxiliary/roman_numerals.rs)。

```rust
#![crate_type="dylib"]
#![feature(plugin_registrar, rustc_private)]

extern crate syntax;
extern crate rustc;
extern crate rustc_plugin;

use syntax::parse::token;
use syntax::tokenstream::TokenTree;
use syntax::ext::base::{ExtCtxt, MacResult, DummyResult, MacEager};
use syntax::ext::build::AstBuilder;  // A trait for expr_usize.
use syntax::ext::quote::rt::Span;
use rustc_plugin::Registry;

fn expand_rn(cx: &mut ExtCtxt, sp: Span, args: &[TokenTree])
        -> Box<MacResult + 'static> {

    static NUMERALS: &'static [(&'static str, usize)] = &[
        ("M", 1000), ("CM", 900), ("D", 500), ("CD", 400),
        ("C",  100), ("XC",  90), ("L",  50), ("XL",  40),
        ("X",   10), ("IX",   9), ("V",   5), ("IV",   4),
        ("I",    1)];

    if args.len() != 1 {
        cx.span_err(
            sp,
            &format!("argument should be a single identifier, but got {} arguments", args.len()));
        return DummyResult::any(sp);
    }

    let text = match args[0] {
        TokenTree::Token(_, token::Ident(s)) => s.to_string(),
        _ => {
            cx.span_err(sp, "argument should be a single identifier");
            return DummyResult::any(sp);
        }
    };

    let mut text = &*text;
    let mut total = 0;
    while !text.is_empty() {
        match NUMERALS.iter().find(|&&(rn, _)| text.starts_with(rn)) {
            Some(&(rn, val)) => {
                total += val;
                text = &text[rn.len()..];
            }
            None => {
                cx.span_err(sp, "invalid Roman numeral");
                return DummyResult::any(sp);
            }
        }
    }

    MacEager::expr(cx.expr_usize(sp, total))
}

#[plugin_registrar]
pub fn plugin_registrar(reg: &mut Registry) {
    reg.register_macro("rn", expand_rn);
}
```

我们可以像其它宏那样使用`rn!()`：

```rust
#![feature(plugin)]
#![plugin(roman_numerals)]

fn main() {
    assert_eq!(rn!(MMXV), 2015);
}
```

与一个简单的`fn(&str) -> u32`函数相比的优势有：

* （任意复杂程度的）转换都发生在编译时
* 输入验证也在编译时进行
* 可以扩展并允许在模式中使用，它可以有效的为任何数据类型定义新语法。

除了宏过程，你可以定义新的类[derive](http://doc.rust-lang.org/reference.html#derive)属性和其它类型的扩展。查看[Registry::register_syntax_extension](http://doc.rust-lang.org/rustc/plugin/registry/struct.Registry.html#method.register_syntax_extension)和[SyntaxExtension enum](http://doc.rust-lang.org/syntax/ext/base/enum.SyntaxExtension.html)。对于更复杂的宏例子，查看[regex_macros](https://github.com/rust-lang/regex/blob/master/regex_macros/src/lib.rs)。

## 提示与技巧

这里提供一些[宏调试的提示](5.35.Macros 宏.md#debugging-macro-code)。

你可以使用[syntax::parse](http://doc.rust-lang.org/syntax/parse/)来将记号树转换为像表达式这样的更高级的语法元素：

```rust
fn expand_foo(cx: &mut ExtCtxt, sp: Span, args: &[TokenTree])
        -> Box<MacResult+'static> {

    let mut parser = cx.new_parser_from_tts(args);

    let expr: P<Expr> = parser.parse_expr();
```

看完[libsyntax解析器代码](https://github.com/rust-lang/rust/blob/master/src/libsyntax/parse/parser.rs)会给你一个解析基础设施如何工作的感觉。

保留你解析所有的[Span](http://doc.rust-lang.org/syntax/codemap/struct.Span.html)，以便更好的报告错误。你可以用[Spanned](http://doc.rust-lang.org/syntax/codemap/struct.Spanned.html)包围你的自定数据结构。

调用[ExtCtxt::span_fatal](http://doc.rust-lang.org/syntax/ext/base/struct.ExtCtxt.html#method.span_fatal)将会立即终止编译。相反最好调用[ExtCtxt::span_err](http://doc.rust-lang.org/syntax/ext/base/struct.ExtCtxt.html#method.span_err)并返回[DummyResult](http://doc.rust-lang.org/syntax/ext/base/struct.DummyResult.html)，这样编译器可以继续并找到更多错误。

为了打印用于调试的语法段，你可以同时使用[span_note](http://doc.rust-lang.org/syntax/ext/base/struct.ExtCtxt.html#method.span_note)和[syntax::print::pprust::*_to_string](http://doc.rust-lang.org/syntax/print/pprust/#functions)。

上面的例子使用[AstBuilder::expr_usize](http://doc.rust-lang.org/syntax/ext/build/trait.AstBuilder.html#tymethod.expr_usize)产生了一个普通整数。作为一个`AstBuilder`特性的额外选择，`libsyntax`提供了一个[准引用宏](http://doc.rust-lang.org/syntax/ext/quote/)的集合。它们并没有文档并且非常边缘化。然而，这些将会是实现一个作为一个普通插件库的改进准引用的好的出发点。

## Lint 插件

插件可以扩展[Rust Lint基础设施](http://doc.rust-lang.org/reference.html#lint-check-attributes)来添加额外的代码风格，安全检查等。你可以查看[src/test/auxiliary/lint_plugin_test.rs](https://github.com/rust-lang/rust/blob/master/src/test/run-pass-fulldeps/auxiliary/lint_plugin_test.rs)来了解一个完整的例子，我们在这里重现它的核心部分：

```rust
#![feature(plugin_registrar)]
#![feature(box_syntax, rustc_private)]

extern crate syntax;

// Load rustc as a plugin to get macros
#[macro_use]
extern crate rustc;
extern crate rustc_plugin;

use rustc::lint::{EarlyContext, LintContext, LintPass, EarlyLintPass,
                  EarlyLintPassObject, LintArray};
use rustc_plugin::Registry;
use syntax::ast;

declare_lint!(TEST_LINT, Warn, "Warn about items named 'lintme'");

struct Pass;

impl LintPass for Pass {
    fn get_lints(&self) -> LintArray {
        lint_array!(TEST_LINT)
    }
}

impl EarlyLintPass for Pass {
    fn check_item(&mut self, cx: &EarlyContext, it: &ast::Item) {
        if it.ident.name.as_str() == "lintme" {
            cx.span_lint(TEST_LINT, it.span, "item is named 'lintme'");
        }
    }
}

#[plugin_registrar]
pub fn plugin_registrar(reg: &mut Registry) {
    reg.register_early_lint_pass(box Pass as EarlyLintPassObject);
}
```

那么像这样的代码：

```rust
#![plugin(lint_plugin_test)]

fn lintme() { }
```

将产生一个编译警告：

```text
foo.rs:4:1: 4:16 warning: item is named 'lintme', #[warn(test_lint)] on by default
foo.rs:4 fn lintme() { }
         ^~~~~~~~~~~~~~~
```

Lint插件的组件有：

* 一个或多个`declare_lint!`调用，它定义了[Lint](http://doc.rust-lang.org/rustc/lint/struct.Lint.html)结构
* 一个用来存放lint检查所需的所有状态（在我们的例子中，没有）
* 一个定义了如何检查每个语法元素的[LintPass](http://doc.rust-lang.org/rustc/lint/trait.LintPass.html)实现。一个单独的`LintPass`可能会对多个不同的`Lint`调用`span_lint`，不过它们都需要用`get_lints`方法进行注册。

Lint过程是语法遍历，不过它们运行在编译的晚期，这时类型信息时可用的。`rustc`的[内建lint](https://github.com/rust-lang/rust/blob/master/src/librustc/lint/builtin.rs)与lint插件使用相同的基础构架，并提供了如何访问类型信息的例子。

由插件定义的语法通常通过[属性和插件标识](http://doc.rust-lang.org/reference.html#lint-check-attributes)控制，例如，`[#[allow(test_lint)]]`，`-A test-lint`。这些标识符来自于`declare_lint!`的第一个参数，经过合适的大小写和标点转换。

你可以运行`rustc -W help foo.rs`来见检查lint列表是否为`rustc`所知，包括由`foo.rs`加载的插件。
