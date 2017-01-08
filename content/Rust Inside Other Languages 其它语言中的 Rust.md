# 其他语言中的 Rust

> [rust-inside-other-languages.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/rust-inside-other-languages.md)
> <br>
> commit 024aa9a345e92aa1926517c4d9b16bd83e74c10d

> **注：** 1.7.0-stable 将此章节去掉了，因此内容可能不具有时效性，这里我们暂时保留。

作为我们的第三个项目，我们决定选择一个可以展示Rust最强实力的功能：缺少实质的运行时。

当组织增长，他们越来越依赖大量的编程语言。不同的编程语言有不同的能力和弱点，而一个多语言栈让你在某个特定的编程语言的优点起作用的时候能使用它，当它有缺陷时使用其他编程语言。

一个非常常见的问题是很多编程语言在程序的运行时性能是很差的。通常来讲，使用一个更慢的，不过提供了更强大的程序员生产力的语言是一个值得的权衡的问题。为了帮助缓和这个问题，它们提供了一个用C编写部分你的系统，然后接着用高级语言编写的代码调用C代码。这叫做一个“外部函数接口”，通常简写为“FFI”。

Rust 在所有两个方向支持 FFI：它可以简单的调用C代码，而且至关重要的，它也可以简单的**在**C中被调用。与 Rust 缺乏垃圾回收和底层运行时要求相结合，这让 Rust 成为一个当你需要嵌入其他语言中以提供一些额外的活力时的强大的候选。

这有一个全面[专注于FFI的章节](4.9.Foreign Function Interface 外部函数接口.md)和详情位于本书的其他位置。不过在这一章，我们会检查这个特定的FFI用例，通过 3 个例子，分别在 Ruby，Python 和 JavaScript 中。

## 问题
有很多不同的项目我们可以选择，不过我们将选择一个 Rust 相比其他语言有明确优势的例子：数值计算和线程。

很多语言，为了一致性，将数字放在堆上，而不是放在栈上。特别是在专注面向对象编程和使用垃圾回收的语言中，堆分配是默认行为。有时优化会栈分配特定的数字，不过与其依赖优化器做这个工作，我们可能想要确保我们总是使用原始类型而不是使用各种对象类型。

第二，很多语言有一个“全局解释器锁（GIL）”，它在很多情况下限制了并发。这在安全的名义下被使用，也有一定的积极影响，不过它限制了同时可以进行的工作的数量，这是一个很负面的影响。

为了强调这两方面，我们将创建一个大量使用这两方面的项目。因为这个例子关注的是将Rust嵌入到其他语言中，而不是问题自身，我们只使用一个玩具例子：

> 启动10个线程。在每个线程中，从1数到500万。在所有10个线程结束后，打印“done”。

我选择 500 万基于我特定的电脑。这里是一个例子的 Ruby 代码：

```ruby
threads = []

10.times do
  threads << Thread.new do
    count = 0

    5_000_000.times do
      count += 1
    end

    count
  end
end

threads.each do |t|
  puts "Thread finished with count=#{t.value}"
end
puts "done!"
```

尝试运行这个例子，并选择一个将运行几秒钟的数字。基于你电脑的硬件配置，你可能需要增大或减小这个数字。

在我的系统中，运行这个例子花费`2.156`秒。并且，如果我用一些进程监视工具，像`top`，我可以看到它只用了我的机器的一个核。这是 GIL 在起作用。

虽然这确实是一个虚构的程序，你可以想象许多问题与现实世界中的问题相似。为了我们的目标，启动一些繁忙的线程来代表一些并行的，昂贵的计算。

## 一个Rust库
让我们用Rust重写这个问题。首先，让我们用Cargo创建一个新项目：

```bash
$ cargo new embed
$ cd embed
```

这个程序在Rust中很好写：

```rust
use std::thread;

fn process() {
    let handles: Vec<_> = (0..10).map(|_| {
        thread::spawn(|| {
            let mut x = 0;
            for _ in 0..5_000_000 {
                x += 1
            }
            x
        })
    }).collect();

    for h in handles {
        println!("Thread finished with count={}",
	    h.join().map_err(|_| "Could not join a thread!").unwrap());
    }
}
```

一些代码可能与前面的例子类似。我们启动了 10个 线程，把它们收集到一个`handles`向量中。在每一个线程里，我们循环 500 万次，并每次给`_x`加一。最后，我们同步每个线程。

然而现在，这是一个 Rust 库，而且它并没有暴露任何可以从C中调用的东西。如果现在我们尝试在别的语言中链接这个库，这并不能工作。我们只需做两个小的改变来修复这个问题，第一个是修改我们代码的开头：

```rust
#[no_mangle]
pub extern fn process() {
```

我们必须增加一个新的属性，`no_mangle`。当你创建了一个Rust库，编译器会在输出中修改函数的名称。这么做的原因超出了本教程的范围，不过为了其他语言能够知道如何调用这些函数，我们需要禁止这么做。这个属性将它关闭。

另一个变化是`pub extern`。`pub`意味着这个函数应当从模块外被调用，而`extern`说它应当能被C调用。这就是了！没有更多的修改。

我们需要做的第二件事是修改我们的`Cargo.toml`的一个设定。在底部加上这些：

```toml
[lib]
name = "embed"
crate-type = ["dylib"]
```

这告诉Rust我们想要将我们的库编译为一个标准的动态库。默认，Rust 编译为一个“rlib”，一个 Rust 特定的格式。

现在让我们构建这个项目：

```bash
$ cargo build --release
   Compiling embed v0.1.0 (file:///home/steve/src/embed)
```

我们选择了`cargo build --release`，它打开了优化进行构建。我们想让它越快越好！你可以在`target/release`找到输出的库：

```bash
$ ls target/release/
build  deps  examples  libembed.so  native
```

那个`libembed.so`就是我们的“共享目标（动态）”库。我们可以像任何用C写的动态库使用这个文件！另外，这也可能有`embed.dll` (Microsoft Windows) 或`libembed.dylib` (Mac OS X)，基于不同的平台。

现在我们构建了我们的 Rust 库，让我们在 Ruby 中使用它。

## Ruby
在我们的项目中打开一个`embed.rb`文件。并这么做：

```ruby
require 'ffi'

module Hello
  extend FFI::Library
  ffi_lib 'target/release/libembed.so'
  attach_function :process, [], :void
end

Hello.process

puts 'done!'
```

在我们可以运行之前，我们需要安装`ffi`gem：

```bash
$ gem install ffi # this may need sudo
Fetching: ffi-1.9.8.gem (100%)
Building native extensions.  This could take a while...
Successfully installed ffi-1.9.8
Parsing documentation for ffi-1.9.8
Installing ri documentation for ffi-1.9.8
Done installing documentation for ffi after 0 seconds
1 gem installed
```

最后，我们可以尝试运行它：

```bash
$ ruby embed.rb
Thread finished with count=5000000
Thread finished with count=5000000
Thread finished with count=5000000
Thread finished with count=5000000
Thread finished with count=5000000
Thread finished with count=5000000
Thread finished with count=5000000
Thread finished with count=5000000
Thread finished with count=5000000
Thread finished with count=5000000
done!
done!
$
```

哇哦，这很快欸！在我系统中，它花费了`0.086`秒，而不是纯 Ruby 所需的 2 秒。让我们分析下我们的 Ruby 代码：

```ruby
require 'ffi'
```

首先我们需要`ffi`gem。这让我们可以像 C 库一样使用 Rust 的接口。

```ruby
module Hello
  extend FFI::Library
  ffi_lib 'target/release/libembed.so'
```

`Hello`模块被用来从共享库中附加原生函数。在其中，我们`extend`必要的`FFI::Library`模块，接着调用`ffi_lib`加载我们的动态库。我们仅仅传递我们库存储的路径，它是我们之前见过的，是`target/release/libembed.so`。

```ruby
attach_function :process, [], :void
```

`attach_function`方法由`ffi`gem提供。它用来把我们Rust中`process()`连接到Ruby中同名函数。因为`process()`没有参数，第二个参数是一个空数组，并且因为它也没有返回值，我传递`:void`作为最后的参数。

```ruby
Hello.process
```

这是实际的Rust调用。我们的`module`和`attach_function`调用的组合设置了环境。它看起来像一个Ruby函数，不过它实际是Rust！

```ruby
puts 'done!'
```

最后，作为我们每个项目的要求，我们打印`done!`。

这就是全部！就像我们看到的，连接两个语言真是很简单，并为我们带来了很多性提升。

接下来，让我们试试 Python！

## Python
在这个目录中创建一个`embed.py`，并写入这些：

```python
from ctypes import cdll

lib = cdll.LoadLibrary("target/release/libembed.so")

lib.process()

print("done!")
```

甚至更简单了！我们使用`ctypes`模块的`cdll`。之后是一个快速的`LoadLibrary`，然后我可以调用`process()`。

在我的系统，这花费了`0.017`秒。非常快！

## Node.js
Node 并不是一个语言，不过目前它是服务端 JavaScript 居统治地位的实现。

为了在 Node 中进行 FFI，首先我们需要安装这个库：

```bash
$ npm install ffi
```

安装之后，我们就可以使用它了：

```javascript
var ffi = require('ffi');

var lib = ffi.Library('target/release/libembed', {
  'process': ['void', []]
});

lib.process();

console.log("done!");
```

这看起来比 Python 的例子更像Ruby的例子。我们使用`ffi`模块来获取`ffi.Library()`，它加载我们的动态库。我们需要标明函数的返回值和参数值，它们是返回“void”，和一个空数组表明没有参数。这样，我们就可以调用它并打印结果。

在我的系统上，这会花费`0.092`秒,很快。

## 结论
如你所见，基础操作是**很**简单的。当然，这里有很多我们可以做的。查看[FFI](4.9.Foreign Function Interface 外部函数接口.md)章节以了解更多细节。
