# 并发

> [concurrency.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/concurrency.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

并发与并行是计算机科学中相当重要的两个主题，并且在当今生产环境中也十分热门。计算机正拥有越来越多的核心，然而很多程序员还没有准备好去完全的利用它们。

Rust 的内存安全功能也适用于并发环境。甚至并发的 Rust 程序也会是内存安全的，并且没有数据竞争。Rust 的类型系统也能胜任，并且在编译时能提供你强大的方式去推论并发代码。

在我们讨论 Rust 提供的并发功能之前，理解一些问题是很重要的：Rust 非常底层以至于所有这些都是由标准库，而不是由语言提供的。这意味着如果你在某些方面不喜欢 Rust 处理并发的方式，你可以自己实现一个。[mio](https://github.com/carllerche/mio)是关于这个原则实践的一个实际的例子。

## 背景：`Send`和`Sync`

并发难以推理。在 Rust 中，我们有一个强大、静态类型系统来帮助我们推理我们的代码。Rust 自身提供了两个特性来帮助我们理解可能是并发的代码的意思。

### `Send`

第一个我们要谈到的特性是[Send](http://doc.rust-lang.org/std/marker/trait.Send.html)。当一个`T`类型实现了`Send`，它向编译器指示这个类型的所有权可以在线程间安全的转移。

强制实施一些通用的限制是很重要的。例如，我们有一个连接两个线程的通道，我们想要能够向通道发送些数据到另一个线程。因此，我们要确保这个类型实现了`Send`。

相反的，如果我们通过 FFI 封装了一个不是线程安全的库，我们并不想实现`Send`，那么编译器会帮助我们强制确保它不会离开当前线程。

### `Sync`

第二个特性是`Sync`。当一个类型`T`实现了`Sync`，它向编译器指示这个类型在多线程并发时没有导致内存不安全的可能性。这隐含了没有[内部可变性](Mutability 可变性.md)的类型天生是`Sync`的，这包含了基本类型（如 `u8`）和包含他们的聚合类型。

为了在线程间共享引用，Rust 提供了一个叫做`Arc<T>`的 wrapper 类型。`Arc<T>`实现了`Send`和`Sync`当且仅当`T`实现了`Send`和`Sync`。例如，一个`Arc<RefCell<U>>`类型的对象不能在线程间传送因为[RefCell](Choosing your Guarantees 选择你的保证.md#refcellt)并没有实现`Sync`，因此`Arc<RefCell<U>>`并不会实现`Send`。

这两个特性允许你使用类型系统来确保你代码在并发环境的特性。在我们演示为什么之前，我们需要先学会如何创建一个并发 Rust 程序！

## 线程

Rust标准库提供了一个“线程”库，它允许你并行的执行 Rust 代码。这是一个使用`std::thread`的基本例子：

```rust
use std::thread;

fn main() {
    thread::spawn(|| {
        println!("Hello from a thread!");
    });
}
```

`thread::spawn()`方法接受一个闭包，它将会在一个新线程中执行。它返回一线程的句柄，这个句柄可以用来等待子线程结束并提取它的结果：

```rust
use std::thread;

fn main() {
    let handle = thread::spawn(|| {
        "Hello from a thread!"
    });

    println!("{}", handle.join().unwrap());
}
```

正如闭包可以从它的环境中获取变量，我们也可以把一些数据带到其他线程中：

```rust
use std::thread;

fn main() {
    let x = 1;
    thread::spawn(|| {
        println!("x is {}", x);
    });
}
```

然而，这会给我们一个错误：

```text
5:19: 7:6 error: closure may outlive the current function, but it
                 borrows `x`, which is owned by the current function
...
5:19: 7:6 help: to force the closure to take ownership of `x` (and any other referenced variables),
          use the `move` keyword, as shown:
      thread::spawn(move || {
          println!("x is {}", x);
      });
```

这是因为默认闭包通过引用获取变量，因此闭包只会获取一个`x`的引用。这样有一个问题，因为线程可能会存在超过`x`的作用域，导致产生一个悬垂指针。

为了解决这个问题，我们使用错误中提示的`move`闭包。`move`闭包将在[这里](Closures 闭包.md)详细讲解。从基本上讲，它把变量从环境中移动到自身。这意味着`x`现在归闭包所有，并不能在调用`spawn()`后的`main()`中使用。

```rust
use std::thread;

fn main() {
    let x = 1;
    thread::spawn(move || {
        println!("x is {}", x);
    });
}
```

很多语言有执行多线程的能力，不过是很不安全的。有完整的书籍是关于如何避免在共享可变状态下出现错误的。在此，借助类型系统，Rust也通过在编译时避免数据竞争来帮助我们。让我们具体讨论下如何在线程间共享数据。

## 安全共享的可变状态（Safe Shared Mutable State）

根据 Rust的 类型系统，我们有个听起来类似谎言的概念叫做：“安全共享的可变状态”。很多程序员都同意共享可变状态是非常，非常不好的。

有人曾说道：
> 共享可变状态是一切罪恶的根源。大部分语言尝试解决这个问题的“可变”部分，而Rust则尝试解决“共享”部分。

同样[所有权系统](5.8.Ownership 所有权.md)也通过防止不当的使用指针来帮助我们排除数据竞争，最糟糕的并发bug之一。

作为一个例子，这是一个在很多语言中可能会产生数据竞争的 Rust 版本程序。它不能编译：

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let mut data = vec![1, 2, 3];

    for i in 0..3 {
        thread::spawn(move || {
            data[0] += i;
        });
    }

    thread::sleep(Duration::from_millis(50));
}
```

这会给我们一个错误：

```text
8:17 error: capture of moved value: `data`
        data[0] += i;
        ^~~~
```

Rust 知道这并不是安全的！如果每个线程中都有一个`data`的引用，并且这些线程获取了引用的所有权，我们就有了3个所有者！`data`在第一次调用`spawn()`时被移出了`main`，所以循环中接下来的调用不能使用这个变量。

所以，我们需要一些类型可以让我们拥有一个值的多个有所有权的引用。通常，我们使用`Rc<T>`，它是一个引用计数类型用以提供共享的所有权。它有一些运行时记录来跟踪引用它的数量，也就是“引用计数”。

调用`Rc<T>`的`clone()`方法会返回一个有所有权的引用并增加其内部引用计数。我们为每一个线程创建一个：

```rust
use std::thread;
use std::time::Duration;
use std::rc::Rc;

fn main() {
    let mut data = Rc::new(vec![1, 2, 3]);

    for i in 0..3 {
        // Create a new owned reference:
        let data_ref = data.clone();

        // Use it in a thread:
        thread::spawn(move || {
            data_ref[0] += i;
        });
    }

    thread::sleep(Duration::from_millis(50));
}
```

这并不能运行，不过它会给我们这个错误：

```text
13:9: 13:22 error: the trait bound `alloc::rc::Rc<collections::vec::Vec<i32>> : core::marker::Send`
            is not satisfied
...
13:9: 13:22 note: `alloc::rc::Rc<collections::vec::Vec<i32>>`
            cannot be sent between threads safely
```

如错误中提到的，`Rc`并不能在线程间安全的传递。这是因为其内部的引用计数并不是通过一个线程安全的方式维护的（非原子性操作）并可能产生数据竞争。

为了解决这个问题，我们使用`Arc<T>`，Rust 标准的原子引用计数类型。

`Arc<T>`的原子部分可以在多线程中安全的访问。为此编译器确保了内部计数的改变都是不可分割的操作这样就不会产生数据竞争。

本质上，`Arc<T>`是一个可以让我们在线程间安全的共享所有权的类型。

```rust
use std::thread;
use std::sync::Arc;
use std::time::Duration;

fn main() {
    let mut data = Arc::new(vec![1, 2, 3]);

    for i in 0..3 {
        let data = data.clone();
        thread::spawn(move || {
            data[0] += i;
        });
    }

    thread::sleep(Duration::from_millis(50));
}
```

与之前类似，我们使用`clone()`来创建一个新的有所有权的句柄。接着这个句柄被移动到了新线程。

不过。。。仍然出错了。

```text
<anon>:11:24 error: cannot borrow immutable borrowed content as mutable
<anon>:11                    data[0] += i;
                             ^~~~
```

`Arc<T>`默认是不可变的。它允许在线程间**共享**数据，不过可变的共享数据是不安全的并且在涉及到多线程时会造成数据竞争！

通常当我们希望让某个不可变的东西变成可变时，我们使用`Cell<T>`或者`RefCell<T>`，它们通过运行时检查或其他手段提供安全的可变性。然而，与`Rc`类似，它们不是线程安全的。如果我们尝试使用它们，我们将会得到一个错误说这些类型不能被`Sync`，代码会编译失败。

看起来我们需要一些允许我们安全的在线程间改变共享值的类型，例如同一时刻只允许一个线程能够改变它内部值的类型。

为此，我们可以使用`Mutex<T>`类型！

下面是一个可以工作的版本：

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::time::Duration;

fn main() {
    let data = Arc::new(Mutex::new(vec![1, 2, 3]));

    for i in 0..3 {
        let data = data.clone();
        thread::spawn(move || {
            let mut data = data.lock().unwrap();
            data[0] += i;
        });
    }

    thread::sleep(Duration::from_millis(50));
}
```

注意`i`的值被限制（拷贝）到了闭包里并不是在线程间共享。

这里我们“锁定”了互斥锁（mutex）。一个互斥锁，正如其名，同时只允许一个线程访问一个值。当我们想要访问一个值时，我们`lock()`它。这会“锁定” mutex，并且其他线程不能锁定它（也就是改变它的值），直到我们处理完之后。如果一个线程尝试锁定一个已经被锁定的 mutex，它将会等待直到其他线程释放这个锁为止。

这里锁的“释放”是隐式的；当锁的结果（在这里是`data`）离开作用域，锁将会被自动释放。

注意[Mutex](http://doc.rust-lang.org/std/sync/struct.Mutex.html)的[lock](http://doc.rust-lang.org/std/sync/struct.Mutex.html#method.lock)方法有如下签名：

```rust
fn lock(&self) -> LockResult<MutexGuard<T>>
```

并且因为`MutexGuard<T>`并没有实现`Send`，guard 并不能跨线程，确保了线程局部性锁的获取和释放。

让我们更仔细的检查一个线程代码：

```rust
# use std::sync::{Arc, Mutex};
# use std::thread;
# use std::time::Duration;
# fn main() {
#     let data = Arc::new(Mutex::new(vec![1, 2, 3]));
#     for i in 0..3 {
#         let data = data.clone();
thread::spawn(move || {
    let mut data = data.lock().unwrap();
    data[0] += i;
});
#     }
#     thread::sleep(Duration::from_millis(50));
# }
```

首先，我们调用`lock()`，它获取了互斥锁。因为这可能失败，它返回一个`Result<T, E>`，并且因为这仅仅是一个例子，我们`unwrap()`结果来获得一个数据的引用。现实中的代码在这里应该有更健壮的错误处理。下面我们可以随意修改它，因为我们持有锁。

最后，在线程运行的同时，我们等待在一个较短的定时器上。不过这并不理想：我们可能选择等待了一个合理的时间不过它更可能比所需的时间要久或并不足够长，这依赖程序运行时线程完成它的计算所需的时间。

一个比定时器更精确的替代是使用一个 Rust 标准库提供的用来同步各个线程的机制。让我们聊聊其中一个：通道。

## 通道（Channels）

下面是我们代码使用通道同步的版本，而不是等待特定时间：

```rust
use std::sync::{Arc, Mutex};
use std::thread;
use std::sync::mpsc;

fn main() {
    let data = Arc::new(Mutex::new(0));

    // `tx` is the "transmitter" or "sender".
    // `rx` is the "receiver".
    let (tx, rx) = mpsc::channel();

    for _ in 0..10 {
        let (data, tx) = (data.clone(), tx.clone());

        thread::spawn(move || {
            let mut data = data.lock().unwrap();
            *data += 1;

            tx.send(()).unwrap();
        });
    }

    for _ in 0..10 {
        rx.recv().unwrap();
    }
}
```

我们使用`mpsc::channel()`方法创建了一个新的通道。我们仅仅向通道中`send`了一个简单的`()`，然后等待它们10个都返回。

因为这个通道只是发送了一个通用信号，我们也可以通过通道发送任何实现了`Send`的数据！

```rust
use std::thread;
use std::sync::mpsc;

fn main() {
    let (tx, rx) = mpsc::channel();

    for i in 0..10 {
        let tx = tx.clone();

        thread::spawn(move || {
            let answer = i * i;

            tx.send(answer).unwrap();
        });
    }

    for _ in 0..10 {
        println!("{}", rx.recv().unwrap());
    }
}
```

这里我们创建了 10 个线程，分别计算一个数字的平方（`spawn()`时的`i`），接着通过通道把结果`send()`回主线程。

## 恐慌（Panics）

`panic!`会使当前执行线程崩溃。你可以使用 Rust 的线程来作为一个简单的隔离机制：

```rust
use std::thread;

let handle = thread::spawn(move || {
    panic!("oops!");
});

let result = handle.join();

assert!(result.is_err());
```

我们的`Thread`返回一个`Result`,它允许我们检查我们的线程是否发生了恐慌。
