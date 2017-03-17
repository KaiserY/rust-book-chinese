# 语法索引

> [syntax-index.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/syntax-index.md)
> <br>
> commit 4093bafe636bb711228e76d780d520626df28921

## 关键词（Keywords）

* `as`: 原始的类型转换。详见[类型转换（`as`）](Casting Between Types 类型转换.md)。
* `break`: 退出循环。详见[循环（提早结束迭代）](Loops 循环.md)。
* `const`: 常量和常量裸指针。详见[`const`和`static`](`const` and `static`.md)，[裸指针](Raw Pointers 裸指针.md)。
* `continue`: 继续进行下一次迭代。详见[循环（提早结束迭代）](Loops 循环.md)。
* `crate`: 外部 crate 链接。详见[crate 和模块（导入外部 crate）](Crates and Modules crate 和模块.md)。
* `else`: `if`和`if let`的失败分支。详见[`if`](If If语句.md)，[`if let`](if let.md)。
* `enum`: 定义枚举。详见[枚举](Enums 枚举.md)。
* `extern`: 外部 crate，函数，和变量链接。详见[crate 和模块（导入外部 crate）](Crates and Modules crate 和模块.md)，[外部函数接口](Foreign Function Interface 外部函数接口.md)。
* `false`: 布尔值 false 常量。详见[原生类型（布尔型）](Primitive Types 原生类型.md)。
* `fn`: 函数定义和函数指针类型。详见[函数](Functions 函数.md)。
* `for`: 循环，`impl`trait 语法的一部分，和高级生命周期语法。详见[循环（for）](Loops 循环.md)，[方法语法](Method Syntax 方法语法.md)。
* `if`: 条件分支。详见[`if`](If If语句.md)，[`if let`](if let.md)。
* `impl`: 继承和 trait 实现块。详见[方法语法](Method Syntax 方法语法.md)。
* `in`: `for`循环语法的一部分。详见[循环（for）](Loops 循环.md)。
* `let`: 变量绑定。详见[变量绑定](Variable Bindings 变量绑定.md)。
* `loop`: 无条件的无限循环。详见[循环（loop）](Loops 循环.md)。
* `match`: 模式匹配。详见[匹配](Match 匹配.md)。
* `mod`: 模块声明。详见[crate 和模块（定义模块）](Crates and Modules crate 和模块.md)。
* `move`: 闭包语法的一部分。详见[闭包（`move`闭包）](Closures 闭包.md)。
* `mut`: 表示指针类型和模式绑定的可变性。详见[可变性](Mutability 可变性.md)。
* `pub`: 表示`struct`字段，`impl`块和模块的共有可见性。详见[crate 和模块（导出共有接口）](Crates and Modules crate 和模块.md)。
* `ref`: 通过引用绑定。详见[模式（`ref`和`ref mut`）](Patterns 模式.md)。
* `return`: 从函数返回。详见[函数（提前返回）](Functions 函数.md)。
* `Self`: （trait）实现者类型的别名。详见[Traits](Traits.md)。
* `self`: 方法的主体。详见[方法语法（方法调用）](Method Syntax 方法语法.md)。
* `static`: 全局变量。详见[`const`和`static`（`static`）](`const` and `static`.md)。
* `struct`: 结构体定义。详见[结构体](Structs 结构体.md)。
* `trait`: trait 定义。详见[Traits](Traits.md)。
* `true`: 布尔值 true 常量。详见[原生类型（布尔型）](Primitive Types 原生类型.md)。
* `type`: 类型别名和关联类型定义。详见[`type`别名](`type` Aliases `type`别名.md)，[关联类型](Associated Types 关联类型.md)。
* `unsafe`: 表示不安全代码，函数，trait 和其实现。详见[不安全代码](`unsafe` 不安全代码.md)。
* `use`: 向作用域导入符号。详见[crate 和模块（使用`use`从模块导入）](Crates and Modules crate 和模块.md)。
* `where`: 类型限制从句。详见[Traits（`where`从句）](Traits.md)。
* `while`: 条件循环。详见[循环（`while`）](Loops 循环.md)。

## 运算符和记号

* `!` (`ident!(…)`, `ident!{…}`, `ident![…]`): 表示宏扩展。详见[宏](Macros 宏.md)。
* `!` (`!expr`): 位计算或逻辑互补。可重载（`Not`）。
* `!=` (`var != expr`): 不等。可重载（`PartialEq`）。
* `%` (`expr % expr`): 算数取余。可重载（`Rem`）。
* `%=` (`var %= expr`): 算数取余并赋值。可重载（`RemAssign`）。
* `&` (`expr & expr`): 位计算和。可重载（`BitAnd`）。
* `&` (`&expr`, `&mut expr`): 借用。详见[引用和借用](References and Borrowing 引用和借用.md)。
* `&` (`&type`, `&mut type`, `&'a type`, `&'a mut type`): 借用指针类型。详见[引用和借用](References and Borrowing 引用和借用.md)。
* `&=` (`var &= expr`): 位计算和并赋值。可重载（`BitAndAssign`）。
* `&&` (`expr && expr`): 逻辑和。
* `*` (`expr * expr`): 算数乘法。可重载（`Mul`）。
* `*` (`*expr`): 解引用。
* `*` (`*const type`, `*mut type`): 裸指针。详见[裸指针](Raw Pointers 裸指针.md)。
* `*=` (`var *= expr`): 算数乘法并赋值。可重载（`MulAssign`）。
* `+` (`expr + expr`): 算数加法。可重载（`Add`）。
* `+` (`trait + trait`, `'a + trait`): 复合类型限制。详见[Traits（多个 trait bound）](Traits.md)。
* `+=` (`var += expr`): 算数加法并赋值。可重载（`AddAssign`）。
* `,`: 参数和元素分隔符。详见[属性](Attributes 属性.md)，[函数](Functions 函数.md)，[结构体](Structs 结构体.md)，[泛型](Generics 泛型.md)，[匹配](Match 匹配.md)，[闭包](Closures 闭包.md)和[crate 和模块（使用`use`从模块导入）](Crates and Modules crate 和模块.md)。
* `-` (`expr - expr`): 算数减法。可重载（`Sub`）。
* `-` (`- expr`): 算数取反。可重载（`Neg`）。
* `-=` (`var -= expr`): 算数减法并赋值。可重载（`SubAssign`）。
* `->` (`fn(…) -> type`, `|…| -> type`): 函数和闭包的返回值类型。详见[函数](Functions 函数.md)，[闭包](Closures 闭包.md)。
* `.` (`expr.ident`): 访问方法。详见[结构体](Structs 结构体.md)，[方法语法](Method Syntax 方法语法.md)。
* `..` (`..`, `expr..`, `..expr`, `expr..expr`): 右开区间的范围常量
* `..` (`..expr`): 结构体常量更新语法。详见[结构体（更新语法）](Structs 结构体.md)。
* `..` (`variant(x, ..)`, `struct_type { x, .. }`): “余下的”模式绑定。详见[模式（忽略绑定）](Patterns 模式.md)。
* `...` (`expr ... expr`): 闭区间范围模式。详见[模式（范围）](Patterns 模式.md)。
* `/` (`expr / expr`): 算数除法。可重载（`Div`）。
* `/=` (`var /= expr`): 算数除法并赋值。可重载（`DivAssign`）。
* `:` (`pat: type`, `ident: type`): 限制。详见[变量绑定](Variable Bindings 变量绑定.md)，[函数](Functions 函数.md)，[](Structs 结构体.md)
* `:` (`ident: expr`): 结构体字段初始化。详见[结构体](Structs 结构体.md)。
* `:` (`'a: loop {…}`): 循环标签。详见[循环（循环标签）](Loops 循环.md)
* `;`: 语句和项终结符。
* `;` (`[…; len]`): 定长数组语法的一部分。详见[原生类型（数组）](Primitive Types 原生类型.md)。
* `<<` (`expr << expr`): 左移。可重载（`Shl`）。
* `<<=` (`var <<= expr`): 左移并赋值。可重载（`ShlAssign`）。
* `<` (`expr < expr`): 小于。可重载（`PartialOrd`）。
* `<=` (`var <= expr`): 小于。可重载（`PartialOrd`）。
* `=` (`var = expr`, `ident = type`): 赋值 / 等价。详见[变量绑定](Variable Bindings 变量绑定.md)，[`type`别名](`type` Aliases `type`别名.md)，默认泛型参数。
* `==` (`var == expr`): 相等。可重载（`PartialEq`）。
* `=>` (`pat => expr`): 匹配分支语法的一部分。详见[匹配](Match 匹配.md)。
* `>` (`expr > expr`): 大于。可重载（`PartialOrd`）。
* `>=` (`var >= expr`): 大于。可重载（`PartialOrd`）。
* `>>` (`expr >> expr`): 右移。可重载（`Shr`）。
* `>>=` (`var >>= expr`): 右移并赋值。可重载（`ShrAssign`）。
* `@` (`ident @ pat`): 模式绑定。详见[模式（绑定）](Patterns 模式.md)。
* `^` (`expr ^ expr`): 位计算异或。可重载（`BitXor`）。
* `^=` (`var ^= expr`): 位计算异或并赋值。
* `|` (`expr | expr`): 位计算或。可重载（`BitOr`）。
* `|` (`pat | pat`): 另外的模式。详见[模式（多个模式）](Patterns 模式.md)。
* `|` (`|…| expr`): 闭包。详见[闭包](Closures 闭包.md)。
* `|=` (`var |= expr`): 位计算或并赋值。可重载（`BitOrAssign`）。
* `||` (`expr || expr`): 逻辑或。
* `_`: “忽略”的模式匹配。详见[模式（忽略绑定）](Patterns 模式.md)。也被用来增强整型常量的可读性。
* `?` (`expr?`): Error propagation。当遇到`Err(_)`时提早返回，否则不执行，类似于[`try!` macro]。

## 其他语法

<!-- Various bits of standalone stuff. -->

* `'ident`: 命名的生命周期或循环标签。详见[模式（绑定）](Patterns 模式.md)。    
* `…u8`, `…i32`, `…f64`, `…usize`, …: 特定类型的数字常量。
* `"…"`: 字符串常量。详见[字符串](Strings 字符串.md)。
* `r"…"`, `r#"…"#`, `r##"…"##`, …: 原始字符串常量，转义字符不会被处理。详见[参考手册（原始字符串常量）](http://doc.rust-lang.org/reference.html#raw-string-literals)。
* `b"…"`: 字节字符串常量，生成一个`[u8]`而不是一个字符串。详见[参考手册（字节字符串常量）](http://doc.rust-lang.org/reference.html#byte-string-literals)。
* `br"…"`, `br#"…"#`, `br##"…"##`, …: 原始字节字符串常量，原始和字节字符串常量的组合。详见[参考手册（原始字节字符串常量）](http://doc.rust-lang.org/reference.html#raw-byte-string-literals)。
* `'…'`: 字符常量。详见[原生类型（`char`）](Primitive Types 原生类型.md)。
* `b'…'`: ASCII 字节常量。
* `|…| expr`: 闭包。详见[闭包](Closures 闭包.md)。

<!-- Path-related syntax -->

* `ident::ident`: 路径。详见[crate 和模块（定义模块）](Crates and Modules crate 和模块.md)。
* `::path`: 相对 crate 根的路径（也就是说，一个明确的绝对路径）。详见[crate 和模块（`pub use`重导出）](Crates and Modules crate 和模块.md)。
* `self::path`: 相对当前模块的路径（也就是说，一个明确的相对路径）。详见[crate 和模块（`pub use`重导出）](Crates and Modules crate 和模块.md)。
* `super::path`: 相对当前模块父模块的路径。详见[crate 和模块（`pub use`重导出）](Crates and Modules crate 和模块.md)。
* `type::ident`: 关联常量，函数和类型。详见[关联类型](Associated Types 关联类型.md)。
* `<type>::…`: 一个不能直接命名的类型的关联项（例如，`<&T>::…`，`<[T]>::…`等）。详见[关联类型](Associated Types 关联类型.md)。

<!-- Generics -->

* `path<…>` (*e.g.* `Vec<u8>`): 用类型指定泛型的参数类型。详见[泛型](Generics 泛型.md)。
* `path::<…>`, `method::<…>` (*e.g.* `"42".parse::<i32>()`): 用表达式指定泛型类型，函数或方法的参数。
* `fn ident<…> …`: 定义泛型函数。详见[泛型](Generics 泛型.md)。
* `struct ident<…> …`: 定义泛型结构体。详见[泛型](Generics 泛型.md)。
* `enum ident<…> …`: 定义泛型枚举。详见[泛型](Generics 泛型.md)。
* `impl<…> …`: 定义泛型实现。
* `for<…> type`: 高级生命周期 bound。
* `type<ident=type>` (*e.g.* `Iterator<Item=T>`): 一个泛型类型，它有一个或多个有特定赋值的关联类型。详见[关联类型](Associated Types 关联类型.md)。

<!-- Constraints -->

* `T: U`: 泛型参数`T`被限制为实现了`U`的类型。详见[Traits](Traits.md)。
* `T: 'a`: 泛型类型`T`必须超过声明周期`'a`。当我们说一个类型“超出”它的作用域时，意味着它不能间接的包含短于`'a`作用域的任何引用。
* `T : 'static`: 泛型类型`T`不包含除`'static`之外的被借用的引用。
* `'b: 'a`: 泛型生命周期`'b`必须超过声明周期`'a`
* `T: ?Sized`: 允许泛型类型是一个不定长度类型。详见[不定长类型](Unsized Types 不定长类型.md)。
* `'a + trait`, `trait + trait`: 复合类型限制。详见[Traits（多个 trait bound）](Traits.md)

<!-- Macros and attributes -->

* `#[meta]`: 外部属性。详见[属性](Attributes 属性.md)。
* `#![meta]`: 内部属性。详见[属性](Attributes 属性.md)。
* `$ident`: 宏替代（部分）。详见[宏](Macros 宏.md)。
* `$ident:kind`: 宏 capture。详见[宏](Macros 宏.md)。
* `$(…)…`: 宏重复（部分）。详见[宏](Macros 宏.md)。

<!-- Comments -->

* `//`: 行注释。详见[注释](Comments 注释.md)。
* `//!`: 内部行文档注释。详见[注释](Comments 注释.md)。
* `///`: 外部行文档注释。详见[注释](Comments 注释.md)。
* `/*…*/`: 块注释。详见[注释](Comments 注释.md)。
* `/*!…*/`: 内部块文档注释。详见[注释](Comments 注释.md)。
* `/**…*/`: 内部块文档注释。详见[注释](Comments 注释.md)。

<!-- Special types -->

* `!`: 一个空的 Never type。详见[发散函数](Functions 函数.md)

<!-- Various things involving parens and tuples -->

* `()`: 空元组（也就是单元），常量和类型。
* `(expr)`: 自带括号的表达式。
* `(expr,)`: 单元素元组表达式。详见[原生类型（元组）](Primitive Types 原生类型.md)。
* `(type,)`: 单元素元组类型。详见[原生类型（元组）](Primitive Types 原生类型.md)。
* `(expr, …)`: 元组类型。详见[原生类型（元组）](Primitive Types 原生类型.md)。
* `(type, …)`: 元组类型。详见[原生类型（元组）](Primitive Types 原生类型.md)。
* `expr(expr, …)`: 函数调用表达式。也用于初始化元组`struct`和元组`enum`变量。详见[函数](Functions 函数.md)。
* `ident!(…)`, `ident!{…}`, `ident![…]`: 宏调用。详见[宏](Macros 宏.md)。
* `expr.0`, `expr.1`, …: 元组索引。详见[原生类型（元组索引）](Primitive Types 原生类型.md)。

<!-- Bracey things -->

* `{…}`: 表达式块
* `Type {…}`: `struct`常量。详见[结构体](Structs 结构体.md)。

<!-- Brackety things -->

* `[…]`: 数组常量。详见[原生类型（数组）](Primitive Types 原生类型.md)。
* `[expr; len]`: 包含`expr`的`len`次拷贝的数组常量。详见[原生类型（数组）](Primitive Types 原生类型.md)。
* `[type; len]`: 包含`len`个`type`实例的数组类型。详见[原生类型（数组）](Primitive Types 原生类型.md)。
* `expr[expr]`: 集合索引。可重载（`Index`，`IndexMut`）。
* `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]`: 用来生成集合切片的集合索引，分别使用`Range`，`RangeFrom`，`RangeTo`，`RangeFull`作为“索引”。

[`const` and `static` (`static`)]: const-and-static.html#static
[`const` and `static`]: const-and-static.html
[`if let`]: if-let.html
[`if`]: if.html
[`type` Aliases]: type-aliases.html
[Associated Types]: associated-types.html
[Attributes]: attributes.html
[Casting Between Types (`as`)]: casting-between-types.html#as
[Closures (`move` closures)]: closures.html#move-closures
[Closures]: closures.html
[Comments]: comments.html
[Crates and Modules (Defining Modules)]: crates-and-modules.html#defining-modules
[Crates and Modules (Exporting a Public Interface)]: crates-and-modules.html#exporting-a-public-interface
[Crates and Modules (Importing External Crates)]: crates-and-modules.html#importing-external-crates
[Crates and Modules (Importing Modules with `use`)]: crates-and-modules.html#importing-modules-with-use
[Crates and Modules (Re-exporting with `pub use`)]: crates-and-modules.html#re-exporting-with-pub-use
[Diverging Functions]: functions.html#diverging-functions
[Enums]: enums.html
[Foreign Function Interface]: ffi.html
[Functions (Early Returns)]: functions.html#early-returns
[Functions]: functions.html
[Generics]: generics.html
[Iterators]: iterators.html
[`try!` macro]: error-handling.html#the-try-macro
[Lifetimes]: lifetimes.html
[Loops (`for`)]: loops.html#for
[Loops (`loop`)]: loops.html#loop
[Loops (`while`)]: loops.html#while
[Loops (Ending Iteration Early)]: loops.html#ending-iteration-early
[Loops (Loops Labels)]: loops.html#loop-labels
[Macros]: macros.html
[Match]: match.html
[Method Syntax (Method Calls)]: method-syntax.html#method-calls
[Method Syntax]: method-syntax.html
[Mutability]: mutability.html
[Operators and Overloading]: operators-and-overloading.html
[Patterns (`ref` and `ref mut`)]: patterns.html#ref-and-ref-mut
[Patterns (Bindings)]: patterns.html#bindings
[Patterns (Ignoring bindings)]: patterns.html#ignoring-bindings
[Patterns (Multiple patterns)]: patterns.html#multiple-patterns
[Patterns (Ranges)]: patterns.html#ranges
[Primitive Types (`char`)]: primitive-types.html#char
[Primitive Types (Arrays)]: primitive-types.html#arrays
[Primitive Types (Booleans)]: primitive-types.html#booleans
[Primitive Types (Tuple Indexing)]: primitive-types.html#tuple-indexing
[Primitive Types (Tuples)]: primitive-types.html#tuples
[Raw Pointers]: raw-pointers.html
[Reference (Byte String Literals)]: ../reference.html#byte-string-literals
[Reference (Integer literals)]: ../reference.html#integer-literals
[Reference (Raw Byte String Literals)]: ../reference.html#raw-byte-string-literals
[Reference (Raw String Literals)]: ../reference.html#raw-string-literals
[References and Borrowing]: references-and-borrowing.html
[Strings]: strings.html
[Structs (Update syntax)]: structs.html#update-syntax
[Structs]: structs.html
[Traits (`where` clause)]: traits.html#where-clause
[Traits (Multiple Trait Bounds)]: traits.html#multiple-trait-bounds
[Traits]: traits.html
[Universal Function Call Syntax]: ufcs.html
[Universal Function Call Syntax (Angle-bracket Form)]: ufcs.html#angle-bracket-form
[Unsafe]: unsafe.html
[Unsized Types (`?Sized`)]: unsized-types.html#sized
[Variable Bindings]: variable-bindings.html
