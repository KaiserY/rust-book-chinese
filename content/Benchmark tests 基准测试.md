# 基准测试

> [benchmark-tests.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/benchmark-tests.md)
> <br>
> commit 28548db57d0acbc00ee80b43816953dbe31d53ba

Rust 也支持基准测试，它可以测试代码的性能。让我们把`src/lib.rs`修改成这样（省略注释）：

```rust
#![feature(test)]

extern crate test;

pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;
    use test::Bencher;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }

    #[bench]
    fn bench_add_two(b: &mut Bencher) {
        b.iter(|| add_two(2));
    }
}
```

注意`test`功能 gate，它启用了这个不稳定功能。

我们导入了`test`crate，它包含了对基准测试的支持。我们也定义了一个新函数，带有`bench`属性。与一般的不带参数的测试不同，基准测试有一个`&mut Bencher`参数。`Bencher`提供了一个`iter`方法，它接收一个闭包。这个闭包包含我们想要测试的代码。

我们可以用`cargo bench`来运行基准测试：

```bash
$ cargo bench
   Compiling adder v0.0.1 (file:///home/steve/tmp/adder)
     Running target/release/adder-91b3e234d4ed382a

running 2 tests
test tests::it_works ... ignored
test tests::bench_add_two ... bench:         1 ns/iter (+/- 0)

test result: ok. 0 passed; 0 failed; 1 ignored; 1 measured
```

我们的非基准测试将被忽略。你也许会发现`cargo bench`比`cargo test`花费的时间更长。这是因为Rust会多次运行我们的基准测试，然后取得平均值。因为我们的函数只做了非常少的操作，我们耗费了`1 ns/iter (+/- 0)`，不过运行时间更长的测试就会有出现偏差。

编写基准测试的建议：

* 把初始代码放于`iter`循环之外，只把你想要测试的部分放入它
* 确保每次循环都做了“同样的事情”，不要累加或者改变状态
* 确保外边的函数也是幂等的（idempotent），基准测试runner可能会多次运行它
* 确保`iter`循环内简短而快速，这样基准测试会运行的很快同时校准器可以在合适的分辨率上调整运转周期
* 确保`iter`循环执行简单的工作，这样可以帮助我们准确的定位性能优化（或不足）

# Gocha：优化
写基准测试有另一些比较微妙的地方：开启了优化编译的基准测试可能被优化器戏剧性的修改导致它不再是我们期望的基准测试了。举例来说，编译器可能认为一些计算并无外部影响并且整个移除它们。

```rust
#![feature(test)]

extern crate test;
use test::Bencher;

#[bench]
fn bench_xor_1000_ints(b: &mut Bencher) {
    b.iter(|| {
        (0..1000).fold(0, |old, new| old ^ new);
    });
}
```

得到如下结果：

```text
running 1 test
test bench_xor_1000_ints ... bench:         0 ns/iter (+/- 0)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured
```

基准测试运行器提供两种方法来避免这个问题：要么传递给`iter`的闭包可以返回一个随机的值这样强制优化器认为结果有用并确保它不会移除整个计算部分。这可以通过修改上面例子中的`b.iter`调用：

```rust
# struct X;
# impl X { fn iter<T, F>(&self, _: F) where F: FnMut() -> T {} } let b = X;
b.iter(|| {
    // Note lack of `;` (could also use an explicit `return`).
    (0..1000).fold(0, |old, new| old ^ new)
});
```

要么，另一个选择是调用通用的`test::black_box`函数，它会传递给优化器一个不透明的“黑盒”这样强制它考虑任何它接收到的参数。

```rust
#![feature(test)]

extern crate test;

# fn main() {
# struct X;
# impl X { fn iter<T, F>(&self, _: F) where F: FnMut() -> T {} } let b = X;
b.iter(|| {
    let n = test::black_box(1000);

    (0..n).fold(0, |a, b| a ^ b)
})
# }
```
上述两种方法均未读取或修改值，并且对于小的值来说非常廉价。对于大的只可以通过间接传递来减小额外开销（例如：`black_box(&huge_struct)`）。

执行上面任何一种修改可以获得如下基准测试结果：

```text
running 1 test
test bench_xor_1000_ints ... bench:       131 ns/iter (+/- 3)

test result: ok. 0 passed; 0 failed; 0 ignored; 1 measured
```

然而，即使使用了上述方法优化器还是可能在不合适的情况下修改测试用例。
