# 内联汇编

> [inline-assembly.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/inline-assembly.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

为了极端底层操作和性能要求，你可能希望直接控制 CPU。Rust 通过`asm!`宏来支持使用内联汇编。

```rust
asm!(assembly template
   : output operands
   : input operands
   : clobbers
   : options
   );
```

任何`asm`的使用需要功能通道（需要在包装箱上加上`#![feature(asm)]`来允许使用）并且当然也需要写在`unsafe`块中

> **注意**：这里的例子使用了 x86/x86-64 汇编，不过所有平台都受支持。

## 汇编模板
`assembly template`是唯一需要的参数并且必须是原始字符串（就是`""`）

```rust
#![feature(asm)]

#[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
fn foo() {
    unsafe {
        asm!("NOP");
    }
}

// Other platforms:
#[cfg(not(any(target_arch = "x86", target_arch = "x86_64")))]
fn foo() { /* ... */ }

fn main() {
    // ...
    foo();
    // ...
}
```

（`feature(asm)`和`#[cfg]`从现在开始将被忽略。）

输出操作数，输入操作数，覆盖和选项都是可选的，然而如果你要省略它们的话，你必选加上正确数量的`:`：

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# fn main() { unsafe {
asm!("xor %eax, %eax"
    :
    :
    : "eax"
   );
# } }
# #[cfg(not(any(target_arch = "x86", target_arch = "x86_64")))]
# fn main() {}
```

有空格在中间也没关系：

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# fn main() { unsafe {
asm!("xor %eax, %eax" ::: "eax");
# } }
# #[cfg(not(any(target_arch = "x86", target_arch = "x86_64")))]
# fn main() {}
```

## 操作数

输入和输出操作数都有相同的格式：`: "constraints1"(expr1), "constraints2"(expr2), ..."`。输出操作数表达式必须是可变的左值，或还未赋值的：

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
fn add(a: i32, b: i32) -> i32 {
    let c: i32;
    unsafe {
        asm!("add $2, $0"
             : "=r"(c)
             : "0"(a), "r"(b)
             );
    }
    c
}
# #[cfg(not(any(target_arch = "x86", target_arch = "x86_64")))]
# fn add(a: i32, b: i32) -> i32 { a + b }

fn main() {
    assert_eq!(add(3, 14159), 14162)
}
```

如果你想在这里使用真正的操作数，然而，要求你在你想使用的寄存器上套上大括号`{}`，并且要求你指明操作数的大小。这在非常底层的编程中是很有用的，这时你使用哪个寄存器是很重要的：

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# unsafe fn read_byte_in(port: u16) -> u8 {
let result: u8;
asm!("in %dx, %al" : "={al}"(result) : "{dx}"(port));
result
# }
```

## 覆盖（Clobbers）

一些指令修改的寄存器可能保存有不同的值，所以我们使用覆盖列表来告诉编译器不要假设任何装载在这些寄存器的值是有效的。

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# fn main() { unsafe {
// Put the value 0x200 in eax:
asm!("mov $$0x200, %eax" : /* no outputs */ : /* no inputs */ : "eax");
# } }
# #[cfg(not(any(target_arch = "x86", target_arch = "x86_64")))]
# fn main() {}
```

输入和输出寄存器并不需要列出因为这些信息已经通过给出的限制沟通过了。因此，任何其它的被使用的寄存器应该隐式或显式的被列出。

如果汇编修改了代码状态寄存器`cc`则需要在覆盖中被列出，如果汇编修改了内存，`memory`也应被指定。

## 选项（Options）

最后一部分，`options`是 Rust 特有的。格式是逗号分隔的基本字符串（也就是说，`:"foo", "bar", "baz"`）。它被用来指定关于内联汇编的额外信息：

目前有效的选项有：

1. *volatile* - 相当于 gcc/clang 中的`__asm__ __volatile__ (...)`
2. *alignstack* - 特定的指令需要栈按特定方式对齐（比如，SSE）并且指定这个告诉编译器插入通常的栈对齐代码
3. *intel* - 使用 intel 语法而不是默认的 AT&T 语法

```rust
# #![feature(asm)]
# #[cfg(any(target_arch = "x86", target_arch = "x86_64"))]
# fn main() {
let result: i32;
unsafe {
   asm!("mov eax, 2" : "={eax}"(result) : : : "intel")
}
println!("eax is currently {}", result);
# }
# #[cfg(not(any(target_arch = "x86", target_arch = "x86_64")))]
# fn main() {}
```

## 更多信息
目前`asm!`的实现是一个[LLVM内联汇编表达式](http://llvm.org/docs/LangRef.html#inline-assembler-expressions)的直接绑定，所以请确保充分的阅读[他们的文档](http://llvm.org/docs/LangRef.html#inline-assembler-expressions)来获取关于覆盖，限制等概念的更多信息。
