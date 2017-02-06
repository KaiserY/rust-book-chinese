# `Drop`

> [drop.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/drop.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

现在我们讨论了 trait，让我们看看一个由 Rust 标准库提供的特殊 trait，[`Drop`](http://doc.rust-lang.org/nightly/std/ops/trait.Drop.html)。`Drop` trait提供了一个当一个值离开作用域后运行一些代码的方法。例如：

```rust
struct HasDrop;

impl Drop for HasDrop {
    fn drop(&mut self) {
        println!("Dropping!");
    }
}

fn main() {
    let x = HasDrop;

    // Do stuff.

} // `x` goes out of scope here.
```

当在` main()`的末尾`x`离开作用域的时候，`Drop`的代码将会执行。`Drop`有一个方法，他也叫做`drop()`。它获取一个`self`的可变引用。

就是这样！`Drop`的机制非常简单，不过这有一些细节。例如，值会以与它们声明相反的顺序被丢弃（dropped）。这是另一个例子：

```rust
struct Firework {
    strength: i32,
}

impl Drop for Firework {
    fn drop(&mut self) {
        println!("BOOM times {}!!!", self.strength);
    }
}

fn main() {
    let firecracker = Firework { strength: 1 };
    let tnt = Firework { strength: 100 };
}
```

这会输出：

```text
BOOM times 100!!!
BOOM times 1!!!
```

`tnt`在`firecracker`之前离开作用域（原文大意：TNT在爆竹之前爆炸），因为它在之后被声明。后进先出。

那么`Drop`有什么好处呢？通常来说，`Drop`用来清理任何与`struct`关联的资源。例如，[`Arc<T>`类型](http://doc.rust-lang.org/nightly/std/sync/struct.Arc.html)是一个引用计数类型。当`Drop`被调用，它会减少引用计数，并且如果引用的总数为0，将会清除底层的值。
