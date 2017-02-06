# 关联类型

> [associated-types.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/associated-types.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

关联类型是Rust类型系统中非常强大的一部分。它涉及到‘类型族’的概念，换句话说，就是把多种类型归于一类。这个描述可能比较抽象，所以让我们深入研究一个例子。如果你想编写一个`Graph`trait，你需要泛型化两个类型：点类型和边类型。所以你可能会像这样写一个trait，`Graph<N, E>`：

```rust
trait Graph<N, E> {
    fn has_edge(&self, &N, &N) -> bool;
    fn edges(&self, &N) -> Vec<E>;
    // Etc.
}
```

虽然这可以工作，不过显得很尴尬，例如，任何需要一个`Graph`作为参数的函数都需要泛型化的`N`ode和`E`dge类型：

```rust
fn distance<N, E, G: Graph<N, E>>(graph: &G, start: &N, end: &N) -> u32 { ... }
```

我们的距离计算并不需要`Edge`类型，所以函数签名中`E`只是写着玩的。

我们需要的是对于每一种`Graph`类型，都使用一个特定的的`N`ode和`E`dge类型。我们可以用关联类型来做到这一点：

```rust
trait Graph {
    type N;
    type E;

    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
    // Etc.
}
```

现在，我们使用一个抽象的`Graph`了：

```rust
fn distance<G: Graph>(graph: &G, start: &G::N, end: &G::N) -> uint { ... }
```

这里不再需要处理`E`dge类型了。

让我们更详细的回顾一下。

## 定义关联类型
让我们构建一个`Graph`trait。这里是定义：

```rust
trait Graph {
    type N;
    type E;

    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
}
```

十分简单。关联类型使用`type`关键字，并出现在trait体和函数中。

这些`type`声明跟函数定义一样。例如，如果我们想`N`类型实现`Display`，这样我们就可以打印出点类型，我们可以这样写：

```rust
use std::fmt;

trait Graph {
    type N: fmt::Display;
    type E;

    fn has_edge(&self, &Self::N, &Self::N) -> bool;
    fn edges(&self, &Self::N) -> Vec<Self::E>;
}
```

## 实现关联类型

就像任何 trait，使用关联类型的 trait 用`impl`关键字来提供实现。下面是一个`Graph`的简单实现：

```rust
# trait Graph {
#     type N;
#     type E;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::E>;
# }
struct Node;

struct Edge;

struct MyGraph;

impl Graph for MyGraph {
    type N = Node;
    type E = Edge;

    fn has_edge(&self, n1: &Node, n2: &Node) -> bool {
        true
    }

    fn edges(&self, n: &Node) -> Vec<Edge> {
        Vec::new()
    }
}
```

这个可笑的实现总是返回`true`和一个空的`Vec<Edge>`，不过它提供了如何实现这类 trait 的思路。首先我们需要3个`struct`，一个代表图，一个代表点，还有一个代表边。如果使用别的类型更合理，也可以那样做，我们只是准备使用`struct`来代表这 3 个类型。

接下来是`impl`行，它就像其它任何 trait 的实现。

在这里，我们使用`=`来定义我们的关联类型。trait 使用的名字出现在`=`的左边，而我们`impl`的具体类型出现在右边。最后，我们在函数声明中使用具体类型。

## trait 对象和关联类型

这里还有另外一个我们需要讨论的语法：trait对象。如果你试图从一个带有关联类型的 trait 创建一个 trait 对象，像这样：

```rust
# trait Graph {
#     type N;
#     type E;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::E>;
# }
# struct Node;
# struct Edge;
# struct MyGraph;
# impl Graph for MyGraph {
#     type N = Node;
#     type E = Edge;
#     fn has_edge(&self, n1: &Node, n2: &Node) -> bool {
#         true
#     }
#     fn edges(&self, n: &Node) -> Vec<Edge> {
#         Vec::new()
#     }
# }
let graph = MyGraph;
let obj = Box::new(graph) as Box<Graph>;
```

你会得到两个错误：

```text
error: the value of the associated type `E` (from the trait `main::Graph`) must
be specified [E0191]
let obj = Box::new(graph) as Box<Graph>;
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
24:44 error: the value of the associated type `N` (from the trait
`main::Graph`) must be specified [E0191]
let obj = Box::new(graph) as Box<Graph>;
          ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~
```

我们不能这样创建一个trait对象，因为我们并不知道关联的类型。相反，我们可以这样写：

```rust
# trait Graph {
#     type N;
#     type E;
#     fn has_edge(&self, &Self::N, &Self::N) -> bool;
#     fn edges(&self, &Self::N) -> Vec<Self::E>;
# }
# struct Node;
# struct Edge;
# struct MyGraph;
# impl Graph for MyGraph {
#     type N = Node;
#     type E = Edge;
#     fn has_edge(&self, n1: &Node, n2: &Node) -> bool {
#         true
#     }
#     fn edges(&self, n: &Node) -> Vec<Edge> {
#         Vec::new()
#     }
# }
let graph = MyGraph;
let obj = Box::new(graph) as Box<Graph<N=Node, E=Edge>>;
```

`N=Node`语法允许我们提供一个具体类型，`Node`，作为`N`类型参数。`E=Edge`也是一样。如果我们不提供这个限制，我们不能确定应该`impl`那个来匹配trait对象。
