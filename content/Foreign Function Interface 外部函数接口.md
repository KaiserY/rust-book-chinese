# 外部函数接口(FFI)

> [ffi.md](https://github.com/rust-lang/rust/blob/master/src/doc/book/ffi.md)
> <br>
> commit 077f4eeb8485e5a1437f6e27973a907ac772b616

## 介绍

本教程会使用[snappy](https://github.com/google/snappy)压缩/解压缩库来作为一个 Rust 编写外部语言代码绑定的介绍。目前 Rust 还不能直接调用 C++ 库，不过 snappy 库包含一个 C 接口（记录在[snappy-c.h](https://github.com/google/snappy/blob/master/snappy-c.h)中）。

## 一个关于 libc 的说明

很多这些例子使用[`libc` crate](https://crates.io/crates/libc)，它提供了很多 C 类型的类型定义，还有很多其他东西。如果你正在自己尝试这些例子，你会需要在你的`Cargo.toml`中添加`libc`：

```toml
[dependencies]
libc = "0.2.0"
```

并在你的 crate 根文件添加`extern crate libc;`

## 调用外部函数

下面是一个最简单的调用其它语言函数的例子，如果你安装了snappy的话它将能够编译：

```rust
# #![feature(libc)]
extern crate libc;
use libc::size_t;

#[link(name = "snappy")]
extern {
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
}

fn main() {
    let x = unsafe { snappy_max_compressed_length(100) };
    println!("max compressed length of a 100 byte buffer: {}", x);
}
```

`extern`块是一个外部库函数标记的列表，在这里例子中是C ABI。`#[link(...)]`属性用来指示链接器链接snappy库来解析符号。

外部函数被假定为不安全的所以调用它们需要包装在`unsafe {}`中，用来向编译器保证大括号中代码是安全的。C库经常提供不是线程安全的接口，并且几乎所有以指针作为参数的函数不是对所有输入时有效的，因为指针可以是垂悬的，而且裸指针超出了Rust安全内存模型的范围。

当声明外部语言的函数参数时，Rust编译器不能检查它是否正确，所以指定正确的类型是保证绑定运行时正常工作的一部分。

`extern`块可以扩展以包括整个snappy API：

```rust
# #![feature(libc)]
extern crate libc;
use libc::{c_int, size_t};

#[link(name = "snappy")]
extern {
    fn snappy_compress(input: *const u8,
                       input_length: size_t,
                       compressed: *mut u8,
                       compressed_length: *mut size_t) -> c_int;
    fn snappy_uncompress(compressed: *const u8,
                         compressed_length: size_t,
                         uncompressed: *mut u8,
                         uncompressed_length: *mut size_t) -> c_int;
    fn snappy_max_compressed_length(source_length: size_t) -> size_t;
    fn snappy_uncompressed_length(compressed: *const u8,
                                  compressed_length: size_t,
                                  result: *mut size_t) -> c_int;
    fn snappy_validate_compressed_buffer(compressed: *const u8,
                                         compressed_length: size_t) -> c_int;
}
# fn main() {}
```

## 创建安全接口
原始C API需要需要封装才能提供内存安全性和利用像向量这样的高级内容。一个库可以选择只暴露出安全的，高级的接口并隐藏不安全的底层细节。

包装用到了缓冲区的函数涉及使用`slice::raw`模块来将Rust向量作为内存指针来操作。Rust的向量确保是一个连续的内存块。它的长度是当前包含的元素个数，而容量则是分配内存的大小。长度小于或等于容量。

```rust
# #![feature(libc)]
# extern crate libc;
# use libc::{c_int, size_t};
# unsafe fn snappy_validate_compressed_buffer(_: *const u8, _: size_t) -> c_int { 0 }
# fn main() {}
pub fn validate_compressed_buffer(src: &[u8]) -> bool {
    unsafe {
        snappy_validate_compressed_buffer(src.as_ptr(), src.len() as size_t) == 0
    }
}
```

上面的`validate_compressed_buffer`封装使用了一个`unsafe`块，不过它通过从函数标记汇总去掉`unsafe`从而保证了对于所有输入调用都是安全的。

`snappy_compress`和`snappy_uncompress`函数更复杂，因为输出也使用了被分配的缓冲区。

`snappy_max_compressed_length`函数可以用来分配一个所需最大容量的向量来存放压缩的输出。接着这个向量可以作为一个输出参数传递给`snappy_compress`。另一个输出参数也被传递进去并设置了长度，可以用它来获取压缩后的真实长度。

```rust
# #![feature(libc)]
# extern crate libc;
# use libc::{size_t, c_int};
# unsafe fn snappy_compress(a: *const u8, b: size_t, c: *mut u8,
#                           d: *mut size_t) -> c_int { 0 }
# unsafe fn snappy_max_compressed_length(a: size_t) -> size_t { a }
# fn main() {}
pub fn compress(src: &[u8]) -> Vec<u8> {
    unsafe {
        let srclen = src.len() as size_t;
        let psrc = src.as_ptr();

        let mut dstlen = snappy_max_compressed_length(srclen);
        let mut dst = Vec::with_capacity(dstlen as usize);
        let pdst = dst.as_mut_ptr();

        snappy_compress(psrc, srclen, pdst, &mut dstlen);
        dst.set_len(dstlen as usize);
        dst
    }
}
```

解压是相似的，因为 snappy 储存了未压缩的大小作为压缩格式的一部分并且`snappy_uncompressed_length`可以取得所需缓冲区的实际大小。

```rust
# #![feature(libc)]
# extern crate libc;
# use libc::{size_t, c_int};
# unsafe fn snappy_uncompress(compressed: *const u8,
#                             compressed_length: size_t,
#                             uncompressed: *mut u8,
#                             uncompressed_length: *mut size_t) -> c_int { 0 }
# unsafe fn snappy_uncompressed_length(compressed: *const u8,
#                                      compressed_length: size_t,
#                                      result: *mut size_t) -> c_int { 0 }
# fn main() {}
pub fn uncompress(src: &[u8]) -> Option<Vec<u8>> {
    unsafe {
        let srclen = src.len() as size_t;
        let psrc = src.as_ptr();

        let mut dstlen: size_t = 0;
        snappy_uncompressed_length(psrc, srclen, &mut dstlen);

        let mut dst = Vec::with_capacity(dstlen as usize);
        let pdst = dst.as_mut_ptr();

        if snappy_uncompress(psrc, srclen, pdst, &mut dstlen) == 0 {
            dst.set_len(dstlen as usize);
            Some(dst)
        } else {
            None // SNAPPY_INVALID_INPUT
        }
    }
}
```

作为一个参考，我们在这里使用的例子可以在[GitHub的这个库](https://github.com/thestinger/rust-snappy)中找到。

## 析构函数
外部库经常把资源的所有权传递给调用函数。当这发生时，我们必须使用Rust析构函数来提供安全性和确保释放了这些资源（特别是在恐慌的时候）。

关于析构函数的更多细节，请看[`Drop`trait](https://doc.rust-lang.org/stable/std/ops/trait.Drop.html)

## 在Rust函数中处理C回调（Callbacks from C code to Rust functions）
一些外部库要求使用回调来向调用者反馈它们的当前状态或者即时数据。可以传递在Rust中定义的函数到外部库中。要求是这个回调函数被标记为`extern`并使用正确的调用约定来确保它可以在C代码中被调用。

接着回调函数可以通过一个C库的注册调用传递并在后面被执行。

一个基础的例子：

Rust代码：

```rust
extern fn callback(a: i32) {
    println!("I'm called from C with value {0}", a);
}

#[link(name = "extlib")]
extern {
   fn register_callback(cb: extern fn(i32)) -> i32;
   fn trigger_callback();
}

fn main() {
    unsafe {
        register_callback(callback);
        trigger_callback(); // Triggers the callback
    }
}
```

C代码：

```c
typedef void (*rust_callback)(int32_t);
rust_callback cb;

int32_t register_callback(rust_callback callback) {
    cb = callback;
    return 1;
}

void trigger_callback() {
  cb(7); // Will call callback(7) in Rust
}
```

这个例子中Rust的`main()`会调用C中的`trigger_callback()`，它会反过来调用Rust中的`callback()`。

## 在Rust对象上使用回调（Targeting callbacks to Rust objects）
之前的例子展示了一个全局函数是如何在C代码中被调用的。然而我们经常希望回调是针对一个特殊Rust对象的。这个对象可能代表对应C语言中的封装。

这可以通过向C库传递这个对象的不安全指针来做到。C库则可以根据这个这个通知中的指针来取得Rust对象。这允许回调不安全的访问被引用的Rust对象。

Rust代码：

```rust
#[repr(C)]
struct RustObject {
    a: i32,
    // other members
}

extern "C" fn callback(target: *mut RustObject, a: i32) {
    println!("I'm called from C with value {0}", a);
    unsafe {
        // Update the value in RustObject with the value received from the callback
        (*target).a = a;
    }
}

#[link(name = "extlib")]
extern {
   fn register_callback(target: *mut RustObject,
                        cb: extern fn(*mut RustObject, i32)) -> i32;
   fn trigger_callback();
}

fn main() {
    // Create the object that will be referenced in the callback
    let mut rust_object = Box::new(RustObject { a: 5 });

    unsafe {
        register_callback(&mut *rust_object, callback);
        trigger_callback();
    }
}
```

C代码：

```c
typedef void (*rust_callback)(void*, int32_t);
void* cb_target;
rust_callback cb;

int32_t register_callback(void* callback_target, rust_callback callback) {
    cb_target = callback_target;
    cb = callback;
    return 1;
}

void trigger_callback() {
  cb(cb_target, 7); // Will call callback(&rustObject, 7) in Rust
}
```

## 异步回调

在之前给出的例子中回调在一个外部C库的函数调用后直接就执行了。在回调的执行过程中当前线程控制权从 Rust 传到了 C 又传到了 Rust，不过最终回调和和触发它的函数都在一个线程中执行。

当外部库生成了自己的线程并触发回调时情况就变得复杂了。在这种情况下回调中对 Rust 数据结构的访问时特别不安全的并必须有合适的同步机制。除了像互斥量这种经典同步机制外，另一种可能就是使用通道（在`std::comm`中）来从触发回调的 C 线程转发数据到 Rust 线程。

如果一个异步回调指定了一个在 Rust 地址空间的特殊 Rust 对象，那么在确保在对应 Rust 对象被销毁后不会再有回调被 C 库触发就格外重要了。这一点可以通过在对象的析构函数中注销回调和设计库使其确保在回调被注销后不会再被触发来取得。

## 链接
在`extern`上的`link`属性提供了基本的构建块来指示`rustc`如何连接到原生库。现在有两种被接受的链接属性形式：

* `#[link(name = "foo")]`
* `#[link(name = "foo", kind = "bar")]`

在这两种形式中，`foo`是我们链接的原生库的名字，而在第二个形式中`bar`是编译器要链接的原生库的类型。目前有3种已知的原生库类型：

* 动态 - `#[link(name = "readline")]`
* 静态 - `#[link(name = "my_build_dependency", kind = "static")]`
* 框架 - `#[link(name = "CoreFoundation", kind = "framework")]`

注意框架只支持OSX平台。

不同`kind`的值意味着链接过程中不同原生库的参与方式。从链接的角度看，rust编译器创建了两种组件：部分的（rlib/staticlib）和最终的（dylib/binary）。原生动态库和框架会从扩展到最终组件部分，而静态库则完全不会扩展。

一些关于这些模型如何使用的例子：

* 一个原生构建依赖。有时编写部分Rust代码时需要一些C/C++代码，另外使用发行为库格式的C/C++代码只是一个负担。在这种情况下，代码会被归档为`libfoo.a`然后rust包装箱可以通过`#[link(name = "foo", kind = "static")]`声明一个依赖。

    不管包装箱输出为何种形式，原生静态库将会包含在输出中，这意味着分配一个原生静态库是没有必要的。

* 一个正常动态库依赖。通用系统库（像`readline`）在大量系统上可用，通常你找不到这类库的静态拷贝。当这种依赖被添加到包装箱里时，部分目标（比如rlibs）将不会链接这些库，但是当rlib被包含进最终目标（比如二进制文件）时，原生库将被链接。

在OSX上，框架与动态库有相同的语义。

## 不安全块（Unsafe blocks）
一些操作，像解引用不安全的指针或者被标记为不安全的函数只允许在unsafe块中使用。unsafe块隔离的不安全性并向编译器保证不安全代码不会泄露到块之外。

不安全函数，另一方面，将它公布于众。一个不安全的函数这样写：

```rust
unsafe fn kaboom(ptr: *const i32) -> i32 { *ptr }
```

这个函数只能被从`unsafe`块中或者`unsafe`函数调用。

## 访问外部全局变量（Accessing foreign globals）
外部API经常导出一个全局变量来进行像记录全局状态这样的工作。为了访问这些变量，你可以在`extern`块中用`static`关键字声明它们：

```rust
# #![feature(libc)]
extern crate libc;

#[link(name = "readline")]
extern {
    static rl_readline_version: libc::c_int;
}

fn main() {
    println!("You have readline version {} installed.",
             rl_readline_version as i32);
}
```

另外，你可能想修改外部结接口提供的全局状态。为了做到这一点，声明为`mut`这样我们就可以改变它了。

```rust
# #![feature(libc)]
extern crate libc;

use std::ffi::CString;
use std::ptr;

#[link(name = "readline")]
extern {
    static mut rl_prompt: *const libc::c_char;
}

fn main() {
    let prompt = CString::new("[my-awesome-shell] $").unwrap();
    unsafe {
        rl_prompt = prompt.as_ptr();

        println!("{:?}", rl_prompt);

        rl_prompt = ptr::null();
    }
}
```

注意与`static mut`变量的所有交互都是不安全的，包括读或写。与全局可变量打交道需要足够的注意。

## 外部调用约定（Foreign calling conventions）
大部分外部代码导出为一个C的ABI，并且Rust默认使用平台C的调用约定来调用外部函数。一些外部函数，尤其是大部分Windows API，使用其它的调用约定。Rust提供了一个告诉编译器应该用哪种调用约定的方法：

```rust
# #![feature(libc)]
extern crate libc;

#[cfg(all(target_os = "win32", target_arch = "x86"))]
#[link(name = "kernel32")]
#[allow(non_snake_case)]
extern "stdcall" {
    fn SetEnvironmentVariableA(n: *const u8, v: *const u8) -> libc::c_int;
}
# fn main() { }
```

这适用于整个`extern`块。被支持的ABI约束的列表为：

* `stdcall`
* `aapcs`
* `cdecl`
* `fastcall`
* `vectorcall` 目前隐藏于`abi_vectorcall` gate 之后并倾向于改变。
* `Rust`
* `rust-intrinsic`
* `system`
* `C`
* `win64`

列表中大部分ABI都是自解释的，不过`system`ABI可能看起来有点奇怪。这个约束会选择任何能和目标库正确交互的ABI。例如，在x86架构上，这意味着会使用`stdcall`ABI。然而，在x86_64上windows使用`C`调用约定，所以`C`会被使用。这意味在我们之前的例子中，我们可以使用`extern "system" { ... }`定义一个适用于所有 windows 系统的块，而不仅仅是 x86 系统。

## 外部代码交互性（Interoperability with foreign code）

只有当`#[repr(C)]`属性被用于结构体时Rust能确保`struct`的布局兼容平台的 C 的表现。`#[repr(C, packed)]`可以用来不对齐的排列结构体成员。`#[repr(C)]`也可以被用于一个枚举。

Rust拥有的装箱（`Box<T>`）使用非空指针作为指向他包含的对象的句柄。然而，它们不应该手动创建因为它们由内部分分配器托管。引用可以被安全的假设为直接指向数据的非空指针。然而，打破借用检查和可变性规则并不能保证安全，所以倾向于只在需要时使用裸指针（`*`）因为编译器不能为它们做更多假设。

向量和字符串共享同样基础的内存布局，`vec`和`str`模块中可用的功能可以操作 C API。然而，字符串不是`\0`结尾的。如果你需要一个NUL结尾的字符串来与 C 交互，你需要使用`std::ffi`模块中的`CString`类型。

标准库中的`libc`模块包含类型别名和C标准库中的函数定义，Rust 默认链接`libc`和`libm`。

## “可空指针优化”（The "nullable pointer optimization"）
特定类型被定义为不为`null`。这包括引用（`&T`，`&mut T`），装箱（`Box<T>`），和函数指针（`extern "abi" fn()`）。当使用C接口时，可能为空的指针经常被使用。作为一个特殊的例子，一个泛化的`enum`包含两个变体，其中一个没有数据，而另一个包含一个单独的字段，非常适合“可空指针优化”。当这么一个枚举被用一个非空指针类型实例化时，它表现为一个指针，而无数据的变体表现为一个空指针。那么`Option<extern "C" fn(c_int) -> c_int>`可以用来表现一个C ABI中的可空函数指针。

## 在C中调用 Rust 代码
你可能会希望这么编译 Rus t代码以便可以在 C 中调用。这是很简单的，不过需要一些东西：

```rust
#[no_mangle]
pub extern fn hello_rust() -> *const u8 {
    "Hello, world!\0".as_ptr()
}
# fn main() {}
```

`extern`使这个函数遵循 C 调用约定，就像之前讨论[外部调用约定](#外部调用约定（foreign-calling-conventions）)时一样。`no_mangle`属性关闭Rust的命名改编，这样它更容易链接。

### FFI 和 panic
当使用FFI时留意`panic!`是很重要的。一个跨越FFI边界的`panic!`是未定义行为。如果你的代码可能panic，你应该在另一个线程运行它，这样panic不会出现在C代码中：

```rust
use std::thread;

#[no_mangle]
pub extern fn oh_no() -> i32 {
    let h = thread::spawn(|| {
        panic!("Oops!");
    });

    match h.join() {
        Ok(_) => 1,
        Err(_) => 0,
    }
}
# fn main() {}
```

### 表示 opaque 结构体

有时一个 C 库想要提供某种指针，不过并不想让你知道它需要的内部细节。最简单的方法是使用一个`void *`参数：

```c
void foo(void *arg);
void bar(void *arg);
```

我们可以使用`c_void`在 Rust 中表示它：

```rust
# #![feature(libc)]
extern crate libc;

extern "C" {
    pub fn foo(arg: *mut libc::c_void);
    pub fn bar(arg: *mut libc::c_void);
}
# fn main() {}
```

这是处理这种情形完美有效的方式。然而，我们可以做的更好一点。为此，一些 C 库会创建一个`struct`，结构体的细节和内存布局是私有的。这提供了一些类型安全性。这种结构体叫做`opaque`。这是一个 C 的例子：

```c
struct Foo; /* Foo is a structure, but its contents are not part of the public interface */
struct Bar;
void foo(struct Foo *arg);
void bar(struct Bar *arg);
```

在 Rust 中，让我们用`enum`创建自己的 opaque 类型：

```rust
pub enum Foo {}
pub enum Bar {}

extern "C" {
    pub fn foo(arg: *mut Foo);
    pub fn bar(arg: *mut Bar);
}
# fn main() {}
```

通过一个没有变量的`enum`，我们创建了一个不能实例化的 opaque 类型，因为它没有变量。不过因为我们的`Foo`和`Bar`是不同类型，我们可以安全的获取这两个类型，所以我们不可能不小心向`bar()`传递一个`Foo`的指针。
