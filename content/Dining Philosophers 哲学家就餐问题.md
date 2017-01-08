# 哲学家就餐问题

> [dining-philosophers.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/dining-philosophers.md)
> <br>
> commit c618c5f36a3260351a09f4b4dc51b2e5d1359fbc

> **注：** 1.7.0-stable 将此章节去掉了，因此内容可能不具有时效性，这里我们暂时保留。

作为我们的第二个项目，让我们看看一个经典的并发问题。它叫做“进餐（ji）的哲学家”。它最初由 Dijkstra 于 1965 年（网上一说 1971 年←_←）提出，不过我们将使用 Tony Hoare 写于 1985 年的[这篇论文](http://www.usingcsp.com/cspbook.pdf)的版本

> 在远古时代，一个富有的慈善家捐赠了一个学院来为 5 名知名的哲学家提供住处。每个哲学家都有一个房间来进行他专业的思考活动；这也有一个共用的餐厅，布置了一个圆桌，周围放着 5 把椅子，每一把都标出了坐在这的哲学家的名字。哲学家们按逆时针顺序围绕桌子做下。每个哲学家的左手边放着一个金叉子，而在桌子中间有一大碗意大利面，它会不时的被补充。哲学家期望用他大部分的时间思考；不过当他饿了的时候，他走向餐厅，坐在它自己的椅子上，拿起他左手边自己的叉子，然后把它插进意大利面。不过乱成一团的意大利面需要第二把叉子才能吃到嘴里。因此哲学家不得不拿起他右手边的叉子。当他吃完了他会放下两把叉子，从椅子上起来，并继续思考。当然，一把叉子一次同时只能被一名哲学家使用。如果其他哲学家需要它，他必须等待直到叉子再次可用。

这个经典的问题展示了一些不同的并发元素。原因是事实上实现它需要一些技巧：一个简单的实现可能会死锁。例如，让我们考虑一个可能解决这个问题的简单算法：

1. 一个哲学家拿起左手边的叉子
2. 他接着拿起右手边的叉子
3. 他吃
4. 他返回叉子

现在，让我们想象一下事件的序列：

1. 哲学家 1 开始算法，拿起他左手边的叉子
2. 哲学家 2 开始算法，拿起他左手边的叉子
3. 哲学家 3 开始算法，拿起他左手边的叉子
4. 哲学家 4 开始算法，拿起他左手边的叉子
5. 哲学家 5 开始算法，拿起他左手边的叉子
6. 。。。？所有的叉子都被拿走了，不过没人在吃（意大利面）！

有不同方法可以解决这个问题。在教程中我们用我们自己的解决办法。现在，让我们开始并用`cargo`创建一个新项目：

```bash
$ cd ~/projects
$ cargo new dining_philosophers --bin
$ cd dining_philosophers
```

现在我们可以对问题进行建模了。让我们在`src/main.rs`中从哲学家开始：

```rust
struct Philosopher {
    name: String,
}

impl Philosopher {
    fn new(name: &str) -> Philosopher {
        Philosopher {
            name: name.to_string(),
        }
    }
}

fn main() {
    let p1 = Philosopher::new("Judith Butler");     // 译者注：朱迪斯·巴特勒
    let p2 = Philosopher::new("Gilles Deleuze");    // 译者注：吉尔·德勒兹
    let p3 = Philosopher::new("Karl Marx");         // 译者注：卡尔·马克思
    let p4 = Philosopher::new("Emma Goldman");      // 译者注：爱玛·戈德曼
    let p5 = Philosopher::new("Michel Foucault");   // 译者注：米歇尔·福柯
}
```

这里，我们创建了一个[struct](5.11.Structs 结构体.md)来代表一个哲学家。目前，我们只需要一个名字。我们选择[String](5.17.Strings 字符串.md)类型作为名字，而不是`&str`。通常来说，处理一个拥有它自己数据的类型要比使用引用的数据来的简单。

让我们继续：

```rust
# struct Philosopher {
#     name: String,
# }
impl Philosopher {
    fn new(name: &str) -> Philosopher {
        Philosopher {
            name: name.to_string(),
        }
    }
}
```

`impl`块让我们在`Philosopher`上定义方法。在这个例子中，我们定义了一个叫做`new`的“关联函数”。第一行看起来像这样：

```rust
# struct Philosopher {
#     name: String,
# }
# impl Philosopher {
fn new(name: &str) -> Philosopher {
#         Philosopher {
#             name: name.to_string(),
#         }
#     }
# }
```

我们获取了一个参数，`name`，`&str`类型的。这是另一个字符串的引用。它返回了一个我们`Philosopher`结构体的实例。

```rust
# struct Philosopher {
#     name: String,
# }
# impl Philosopher {
#    fn new(name: &str) -> Philosopher {
Philosopher {
    name: name.to_string(),
}
#     }
# }
```

这创建了一个新的`Philosopher`，并把它的`name`设置为我们的`name`参数。不仅仅是参数自身，虽然，因为我们在它上面调用了`.to_string()`。这将创建一个我们`&str`指向的字符串的拷贝，并给我们一个新的`String`，它是我们`Philosopher`的`name`字段的类型。

为什么不直接接受一个`String`呢？它更方便调用。如果我们获取一个`String`，而我们的调用者有一个`&str`，它就不得不自己调用这个方法。这个灵活性的缺点是我们**总是**生成了一个拷贝。对于我们这个小程序，这并不是特别的重要，因为我们知道我们只会用短小的字符串。

你要注意到的最后一件事：我们刚刚定义了一个`Philosopher`，不过好像并没有对它做什么。Rust是一个“基于表达式”的语言，它意味着Rust中几乎所有的东西都是一个表达式并返回一个值。这对函数也适用，最后的表达式是自动返回的。因为我们创建了一个新的`Philosopher`作为这个函数最后的表达式，我们最终返回了它。

这个名字，`new()`，在Rust中并没有什么特殊性。不过它是创建一个结构体新实例的函数的传统名称。在我们讨论为什么之前，让我们再看看`main()`：

```rust
# struct Philosopher {
#     name: String,
# }
#
# impl Philosopher {
#     fn new(name: &str) -> Philosopher {
#         Philosopher {
#             name: name.to_string(),
#         }
#     }
# }
#
fn main() {
    let p1 = Philosopher::new("Judith Butler");
    let p2 = Philosopher::new("Gilles Deleuze");
    let p3 = Philosopher::new("Karl Marx");
    let p4 = Philosopher::new("Emma Goldman");
    let p5 = Philosopher::new("Michel Foucault");
}
```

这里，我们创建了 5 个新哲学家的变量绑定。这是我最崇拜的5个，不过你可以替换为任何你想要的。如果我们**没有**定义`new()`函数，它将看起来像这样：

```rust
# struct Philosopher {
#     name: String,
# }
fn main() {
    let p1 = Philosopher { name: "Judith Butler".to_string() };
    let p2 = Philosopher { name: "Gilles Deleuze".to_string() };
    let p3 = Philosopher { name: "Karl Marx".to_string() };
    let p4 = Philosopher { name: "Emma Goldman".to_string() };
    let p5 = Philosopher { name: "Michel Foucault".to_string() };
}
```

这看起来更乱。使用`new`还有别的优点，不过即便在这个简单的例子，它也被证明是更易于使用的。

现在我们已经掌握了够用的基础，这里有多种办法可以让我们可以处理更广泛的问题。首先我想从结尾开始：让我们准备让每个哲学家能吃完的方法。作为一个小步骤，让我们写一个方法，并接着遍历所有的哲学家，调用这个方法：

```rust
struct Philosopher {
    name: String,
}

impl Philosopher {
    fn new(name: &str) -> Philosopher {
        Philosopher {
            name: name.to_string(),
        }
    }

    fn eat(&self) {
        println!("{} is done eating.", self.name);
    }
}

fn main() {
    let philosophers = vec![
        Philosopher::new("Judith Butler"),
        Philosopher::new("Gilles Deleuze"),
        Philosopher::new("Karl Marx"),
        Philosopher::new("Emma Goldman"),
        Philosopher::new("Michel Foucault"),
    ];

    for p in &philosophers {
        p.eat();
    }
}
```

让我们先看看`main()`。与其为我们的哲学家写5个独立的变量绑定，相反我们为它们创建了一个`Vec<T>`。`Vec<T>`也叫做一个“vector”，它是一个可增长的数组类型。接着我们用[`for`](5.6.Loops 循环.md)循环遍历 vector，顺序获取每个哲学家的引用。

在循环体中，我们调用`p.eat()`，它定义在上面：

```rust
fn eat(&self) {
    println!("{} is done eating.", self.name);
}
```

在 Rust 中，方法显式获取一个`self`参数。这就是为什么`eat()`是一个方法，而`new`是一个关联函数：`new()`没有用到`self`。在我们第一个版本的`eat()`，我们仅仅打印出哲学家的名字，并提到他们吃完了。运行这个程序应该会给你如下的输出：

```text
Judith Butler is done eating.
Gilles Deleuze is done eating.
Karl Marx is done eating.
Emma Goldman is done eating.
Michel Foucault is done eating.
```

十分简单的，他们都吃完了！然而我们还没有实际上实现真正的问题，所以我们还没完！

下一步，我们想要让我们的哲学家不光说吃完了，而是实际上的吃（意大利面）。这是下一个版本：

```rust
use std::thread;
use std::time::Duration;

struct Philosopher {
    name: String,
}

impl Philosopher {
    fn new(name: &str) -> Philosopher {
        Philosopher {
            name: name.to_string(),
        }
    }

    fn eat(&self) {
        println!("{} is eating.", self.name);

        thread::sleep(Duration::from_millis(1000));

        println!("{} is done eating.", self.name);
    }
}

fn main() {
    let philosophers = vec![
        Philosopher::new("Judith Butler"),
        Philosopher::new("Gilles Deleuze"),
        Philosopher::new("Karl Marx"),
        Philosopher::new("Emma Goldman"),
        Philosopher::new("Michel Foucault"),
    ];

    for p in &philosophers {
        p.eat();
    }
}
```

只有一些变化，让我们拆开来看。

```rust
use std::thread;
```

`use`将名称引入作用域。我们将开始使用标准库的`thread`模块，所以我们需要`use`它。

```rust
    fn eat(&self) {
        println!("{} is eating.", self.name);

        thread::sleep(Duration::from_millis(1000));

        println!("{} is done eating.", self.name);
    }
```

现在我们打印出两个信息，有一个`sleep`在中间。这会模拟哲学家吃面的时间。

如果你运行这个程序，你应该会看到每个哲学家依次进餐：

```text
Judith Butler is eating.
Judith Butler is done eating.
Gilles Deleuze is eating.
Gilles Deleuze is done eating.
Karl Marx is eating.
Karl Marx is done eating.
Emma Goldman is eating.
Emma Goldman is done eating.
Michel Foucault is eating.
Michel Foucault is done eating.
```

好极了！我们做到了。这仅有一个问题：我们实际上没有进行并发处理，而这才是我们问题的核心！

为了让哲学家并发的进餐，我们需要做一个小的修改。这是下一次迭代：

```rust
use std::thread;
use std::time::Duration;

struct Philosopher {
    name: String,
}

impl Philosopher {
    fn new(name: &str) -> Philosopher {
        Philosopher {
            name: name.to_string(),
        }
    }

    fn eat(&self) {
        println!("{} is eating.", self.name);

        thread::sleep(Duration::from_millis(1000));

        println!("{} is done eating.", self.name);
    }
}

fn main() {
    let philosophers = vec![
        Philosopher::new("Judith Butler"),
        Philosopher::new("Gilles Deleuze"),
        Philosopher::new("Karl Marx"),
        Philosopher::new("Emma Goldman"),
        Philosopher::new("Michel Foucault"),
    ];

    let handles: Vec<_> = philosophers.into_iter().map(|p| {
        thread::spawn(move || {
            p.eat();
        })
    }).collect();

    for h in handles {
        h.join().unwrap();
    }
}
```

所有我们做的是改变了`main()`中的循环，并增加了第二个循环！这里是第一个变化：

```rust
let handles: Vec<_> = philosophers.into_iter().map(|p| {
    thread::spawn(move || {
        p.eat();
    })
}).collect();
```

虽然这只有 5 行，它们有 4 行密集的代码。让我们分开看。

```rust
let handles: Vec<_> =
```

我们引入了一个新的绑定，叫做`handles`。我们用这个名字因为我们将创建一些新的线程，并且它们会返回一些这些线程句柄来让我们控制它们的行为。然而这里我们需要显式注明类型，因为一个我们之后会介绍的问题。`_`是一个类型占位符。我们是在说“`handles`是一些东西的 vector，不过Rust你自己应该能发现这些东西是什么"。

```rust
philosophers.into_iter().map(|p| {
```

我们获取了哲学家列表并在其上调用`into_iter()`。它创建了一个迭代器来获取每个哲学家的所有权。我们需要这样做来把它们传递给我们的线程。我们取得这个迭代器并在其上调用`map`，他会获取一个闭包作为参数并按顺序在每个元素上调用这个闭包。

```rust
    thread::spawn(move || {
        p.eat();
    })
```

这就是并发发生的地方。`thread::spawn`获取一个闭包作为参数并在一个新线程执行这个闭包。这个闭包需要一个额外的标记，`move`，来表明这个闭包将会获取它获取的值的所有权。主要指`map`函数的`p`变量。

在线程中，所有我们做的就是在`p`上调用`eat()`。另外注意到`thread::spawn`调用最后木有分号，这使它是一个表达式。这个区别是重要的，以便生成正确的返回值。更多细节，请看[表达式VS语句](5.2.Functions 函数.md#expressions-vs.-statements)。

```rust
}).collect();
```

最后，我们获取所有这些`map`调用的结果并把它们收集起来。`collect()`将会把它们放入一个某种类型的集合，这也就是为什么我们要表明返回值的类型：我们需要一个`Vec<T>`。这些元素是`thread::spawn`调用的返回值，它们就是这些线程的句柄。噢！

```rust
for h in handles {
    h.join().unwrap();
}
```

在`main()`的结尾，我们遍历这些句柄并在其上调用`join()`，它会阻塞执行直到线程完成执行。这保证了在程序结束之前这些线程都完成了它们的工作。

如果你运行这个程序，你将会看到哲学家们无序的进餐！我们有了多线程！

```text
Judith Butler is eating.
Gilles Deleuze is eating.
Karl Marx is eating.
Emma Goldman is eating.
Michel Foucault is eating.
Judith Butler is done eating.
Gilles Deleuze is done eating.
Karl Marx is done eating.
Emma Goldman is done eating.
Michel Foucault is done eating.
```

不过叉子怎么办呢？我们还没有模型化它们呢。

为此，让我们创建一个新的`struct`：

```rust
use std::sync::Mutex;

struct Table {
    forks: Vec<Mutex<()>>,
}
```

这个`Table`有一个`Mutex`的vector，一个互斥锁是一个控制并发的方法：一次只有一个线程能访问它的内容。这正是我们需要叉子拥有的属性。我们用了一个空元组，`()`，在互斥锁的内部，因为我们实际上并不准备使用这个值，只是要持有它。

让我们修改程序来使用`Table`：

```rust
use std::thread;
use std::time::Duration;
use std::sync::{Mutex, Arc};

struct Philosopher {
    name: String,
    left: usize,
    right: usize,
}

impl Philosopher {
    fn new(name: &str, left: usize, right: usize) -> Philosopher {
        Philosopher {
            name: name.to_string(),
            left: left,
            right: right,
        }
    }

    fn eat(&self, table: &Table) {
        let _left = table.forks[self.left].lock().unwrap();
        thread::sleep(Duration::from_millis(150));
        let _right = table.forks[self.right].lock().unwrap();

        println!("{} is eating.", self.name);

        thread::sleep(Duration::from_millis(1000));

        println!("{} is done eating.", self.name);
    }
}

struct Table {
    forks: Vec<Mutex<()>>,
}

fn main() {
    let table = Arc::new(Table { forks: vec![
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
    ]});

    let philosophers = vec![
        Philosopher::new("Judith Butler", 0, 1),
        Philosopher::new("Gilles Deleuze", 1, 2),
        Philosopher::new("Karl Marx", 2, 3),
        Philosopher::new("Emma Goldman", 3, 4),
        Philosopher::new("Michel Foucault", 0, 4),
    ];

    let handles: Vec<_> = philosophers.into_iter().map(|p| {
        let table = table.clone();

        thread::spawn(move || {
            p.eat(&table);
        })
    }).collect();

    for h in handles {
        h.join().unwrap();
    }
}
```

大量的修改！然而，通过这次迭代，我们有了一个可以工作的程序。让我摸看看细节：

```rust
use std::sync::{Mutex, Arc};
```

我们将用到`std::sync`包中的另一个结构：`Arc<T>`。我们在用到时再详细解释。

```rust
struct Philosopher {
    name: String,
    left: usize,
    right: usize,
}
```

我们需要在我们的`Philosopher`中增加更多的字段。每个哲学家将拥有两把叉子：一个拿左手，一个拿右手。我们将用`usize`来表示它们，因为它是你的 vector 的索引的类型。这两个值将会是我们`Table`中的`forks`的索引。

```rust
fn new(name: &str, left: usize, right: usize) -> Philosopher {
    Philosopher {
        name: name.to_string(),
        left: left,
        right: right,
    }
}
```

现在我们需要构造这些`left`和`right`的值，所以我们把它们加到`new()`里。

```rust
fn eat(&self, table: &Table) {
    let _left = table.forks[self.left].lock().unwrap();
    thread::sleep(Duration::from_millis(150));
    let _right = table.forks[self.right].lock().unwrap();

    println!("{} is eating.", self.name);

    thread::sleep(Duration::from_millis(1000));

    println!("{} is done eating.", self.name);
}
```

我们有两个新行。我们也增加了一个参数，`table`。我们访问`Table`的叉子列表，接着使用`self.left`和`self.right`来访问特定索引位置的叉子。这让我们访问索引位置的`Mutex`，并且我们在其上调用` lock()`。如果互斥锁目前正在被别人访问，我们将阻塞直到它可用为止。我们也在第一把叉子被拿起来和第二把叉子拿起来之间调用了一个`thread::sleep`，因为拿起叉子的过程并不是立即完成的。

`lock()`可能会失败，而且如果它失败了，我们想要程序崩溃。在这个例子中，互斥锁可能发生的错误是[“被污染了（poisoned）”](http://doc.rust-lang.org/nightly/std/sync/struct.Mutex.html#poisoning)，它发生于当线程在持有锁的同时线程恐慌了。因为这不应该发生，所以我们仅仅是使用`unwrap()`。

这些代码还有另一个奇怪的事情：我们命名结果为`_left`和`_right`。为啥要用下划线？好吧，我们并不打算在锁中“使用”这些值。我们仅仅想要获取它。为此，Rust会警告我们从未使用这些值。通过使用下划线，我们告诉Rust这是我们意图做的，这样它就不会产生一个警告。

那怎么释放锁呢？好吧，这会在`_left`和`_right`离开作用域时发生，自动的。

```rust
    let table = Arc::new(Table { forks: vec![
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
        Mutex::new(()),
    ]});
```

接下来，在`main()`中，我们创建了一个新`Table`并封装在一个`Arc<T>`中。“arc”代表“原子引用计数”，并且我们需要在多个线程间共享我们的`Table`。因为我们共享了它，引用计数会增长，而当每个线程结束，它会减少。

```rust
let philosophers = vec![
    Philosopher::new("Judith Butler", 0, 1),
    Philosopher::new("Gilles Deleuze", 1, 2),
    Philosopher::new("Karl Marx", 2, 3),
    Philosopher::new("Emma Goldman", 3, 4),
    Philosopher::new("Michel Foucault", 0, 4),
];
```

我们需要传递我们的`left`和`right`的值给我们的`Philosopher`们的构造函数。不过这里有另一个细节，并且是“非常”重要。如果你观察它的模式，它们从头到尾全是连续的。米歇尔·福柯应该使用`4`，`0`作为参数，不过我们用了`0`，`4`。这事实上是为了避免死锁：我们的哲学家中有一个左撇子！这是解决这个问题的一个方法，并且在我看来，是最简单的方法。

```rust
let handles: Vec<_> = philosophers.into_iter().map(|p| {
    let table = table.clone();

    thread::spawn(move || {
        p.eat(&table);
    })
}).collect();
```

最后，在`map()`/`collect()`循环中，我们调用`table.clone()`。`Arc<T>`的`clone()`方法用来增加引用计数，而当它离开作用域，它减少计数。你会注意到这里我们可以引入一个新的`table`的绑定，而且它应该覆盖旧的一个。这经常用在你不想整出两个不同的名字的时候。

通过这些，我们的程序能工作了！任何同一时刻只有两名哲学家能进餐，因此你会得到像这样的输出：

```text
Gilles Deleuze is eating.
Emma Goldman is eating.
Emma Goldman is done eating.
Gilles Deleuze is done eating.
Judith Butler is eating.
Karl Marx is eating.
Judith Butler is done eating.
Michel Foucault is eating.
Karl Marx is done eating.
Michel Foucault is done eating.
```

恭喜！你用Rust实现了一个经典的并发问题。
