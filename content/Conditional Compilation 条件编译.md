# 条件编译

> [conditional-compilation.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/conditional-compilation.md)
> <br>
> commit d30662f3e78ddc65f6ecafd20e4b6ecd3033e466

Rust有一个特殊的属性，`#[cfg]`，它允许你基于一个传递给编译器的标记编译代码。它有两种形式：

```rust
#[cfg(foo)]
# fn foo() {}

#[cfg(bar = "baz")]
# fn bar() {}
```

它还有一些帮助选项：

```rust
#[cfg(any(unix, windows))]
# fn foo() {}

#[cfg(all(unix, target_pointer_width = "32"))]
# fn bar() {}

#[cfg(not(foo))]
# fn not_foo() {}
```

这些选项可以任意嵌套：

```rust
#[cfg(any(not(unix), all(target_os="macos", target_arch = "powerpc")))]
# fn foo() {}
```

至于如何启用和禁用这些开关，如果你使用Cargo的话，它们可以在你`Cargo.toml`中的[`[features]`部分](http://doc.crates.io/manifest.html#the-%5Bfeatures%5D-section)设置：

```toml
[features]
# no features by default
default = []

# Add feature "foo" here, then you can use it.
# Our "foo" feature depends on nothing else.
foo = []

# The “secure-password” feature depends on the bcrypt package.
# secure-password = ["bcrypt"]
```

当你这么做的时候，Cargo传递给`rustc`一个标记：

```bash
--cfg feature="${feature_name}"
```

这些`cfg`标记集合会决定哪些功能被启用，并且因此，哪些代码会被编译。让我们看看这些代码：

```rust
#[cfg(feature = "foo")]
mod foo {
}
```

如果你用`cargo build --features "foo"`编译，他会向`rustc`传递`--cfg feature="foo"`标记，并且输出中将会包含`mod foo`。如果我们使用常规的`cargo build`编译，则不会传递额外的标记，因此，（输出）不会存在`foo`模块。

## cfg_attr
你也可以通过一个基于`cfg`变量的`cfg_attr`来设置另一个属性：

```rust
#[cfg_attr(a, b)]
# fn foo() {}
```

如果`a`通过`cfg`属性设置了的话这与`#[b]`相同，否则不起作用。

# cfg!
`cfg!`[语法扩展](Compiler Plugins 编译器插件.md)也让你可以在你的代码中使用这类标记：

```rust
if cfg!(target_os = "macos") || cfg!(target_os = "ios") {
    println!("Think Different!");
}
```

这会在编译时被替换为一个`true`或`false`，依配置设定而定。
