# 链接进阶

> [advanced-linking.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/advanced-linking.md)
> <br>
> commit 226bcdf7d1e774f5967f92b0bd0bf237179f95c9

Rust 的常用链接形式在本书的之前部分已经介绍过了，不过支持多种其他语言可用的可能的链接对 Rust 获取与原生库的无缝交互是很重要的。

## 链接参数（Link args）

这里还有一个方法来告诉 rustc 如何自定义链接，这就是通过`link_args`属性。这个属性作用于`extern`块并指定当产生构件时需要传递给连接器的原始标记。一个用例将是：

```rust,no_run
#![feature(link_args)]

#[link_args = "-foo -bar -baz"]
extern {}
# fn main() {}
```

注意现在这个功能隐藏在`feature(link_args)`gate 之后因为它并不是一个被认可的执行链接的方法。目前 rustc 从 shell 调用系统的连接器（大多数系统是`gcc`，MSVC 是`link.exe`），所以使用额外的命令行参数是可行的，不过这并一定永远可行。将来 rustc 可能使用 LLVM 直接链接原生库这样一来`link_args`就毫无意义了。你可以向`rustc`传递`-C link-args`参数来获得和`link_args`属性同样的效果。

强烈建议你**不要**使用这个属性，而是使用一个更正式的`[link(...)]`属性作用于`extern`块。

## 静态链接

静态链接代表创建包含所有所需库的输出的过程，这样你在任何系统上使用你编译的项目时就不需要安装相应的库了。纯 Rust 的依赖默认都是静态链接的这样你可以使用你创建的二进制和库而不需要安装 Rust。相反，原生库（例如，`libc`和`libm`）通常是动态链接的，不过也可以修改为静态链接。

链接是一个非常依赖平台的问题--在一些平台上，静态链接可能根本就是不可能的！这个部分假设你对你选择的平台的链接一些基础的认识。

### Linux

在 Linux 上 Rust 程默认会链接系统的`libc`以及一些其他的库。让我们看看一个使用 GCC 和`glibc`的 64 位 Linux（目前为止 Linux 上最常见的`libc`）的例子：

```bash
$ mkdir musldist
$ PREFIX=$(pwd)/musldist
$
$ # Build musl
$ curl -O http://www.musl-libc.org/releases/musl-1.1.10.tar.gz
$ tar xf musl-1.1.10.tar.gz
$ cd musl-1.1.10/
musl-1.1.10 $ ./configure --disable-shared --prefix=$PREFIX
musl-1.1.10 $ make
musl-1.1.10 $ make install
musl-1.1.10 $ cd ..
$ du -h musldist/lib/libc.a
2.2M    musldist/lib/libc.a
$
$ # Build libunwind.a
$ curl -O http://llvm.org/releases/3.7.0/llvm-3.7.0.src.tar.xz
$ tar xf llvm-3.7.0.src.tar.xz
$ cd llvm-3.7.0.src/projects/
llvm-3.7.0.src/projects $ curl http://llvm.org/releases/3.7.0/libunwind-3.7.0.src.tar.xz | tar xJf -
llvm-3.7.0.src/projects $ mv libunwind-3.7.0.src libunwind
llvm-3.7.0.src/projects $ mkdir libunwind/build
llvm-3.7.0.src/projects $ cd libunwind/build
llvm-3.7.0.src/projects/libunwind/build $ cmake -DLLVM_PATH=../../.. -DLIBUNWIND_ENABLE_SHARED=0 ..
llvm-3.7.0.src/projects/libunwind/build $ make
llvm-3.7.0.src/projects/libunwind/build $ cp lib/libunwind.a $PREFIX/lib/
llvm-3.7.0.src/projects/libunwind/build $ cd ../../../../
$ du -h musldist/lib/libunwind.a
164K    musldist/lib/libunwind.a
$
$ # Build musl-enabled rust
$ git clone https://github.com/rust-lang/rust.git muslrust
$ cd muslrust
muslrust $ ./configure --target=x86_64-unknown-linux-musl --musl-root=$PREFIX --prefix=$PREFIX
muslrust $ make
muslrust $ make install
muslrust $ cd ..
$ du -h musldist/bin/rustc
12K     musldist/bin/rustc
```

现在你有了一个启用了`musl`的Rust！因为我们用了一个自定义的目录，当我们尝试并运行它的时候我们需要确保我们的系统能够找到二进制文件和正确的库：

```bash
$ export PATH=$PREFIX/bin:$PATH
$ export LD_LIBRARY_PATH=$PREFIX/lib:$LD_LIBRARY_PATH
```

让我们试一下！

```bash
$ echo 'fn main() { println!("hi!"); panic!("failed"); }' > example.rs
$ rustc --target=x86_64-unknown-linux-musl example.rs
$ ldd example
        not a dynamic executable
$ ./example
hi!
thread 'main' panicked at 'failed', example.rs:1
```

成功了！这个二进制文件可以被拷贝到几乎所有拥有相同构架的 Linux 机器上无故障的运行。

`cargo build`也允许`--target`选项所以你也能用它来正常的构建你的 crate。然而，你可能需要先链接你的原生库到`musl`，在你可以链接到它之前。
