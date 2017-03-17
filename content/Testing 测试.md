# 测试

> [testing.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/testing.md)
> <br>
> commit c4c86dd04ccf6e9a6ac9282ecb9bb42e13ea5dad

> Program testing can be a very effective way to show the presence of bugs, but it is hopelessly inadequate for showing their absence.

> Edsger W. Dijkstra, "The Humble Programmer" (1972)

> 软件测试是证明 bug 存在的有效方法，而证明它们不存在时则显得令人绝望的不足。

> Edsger W. Dijkstra，【谦卑的程序员】（1972）

让我们讨论一下如何测试Rust代码。在这里我们不会讨论什么是测试Rust代码的正确方法。有很多关于写测试好坏方法的流派。所有的这些途径都使用相同的基本工具，所以我们会向你展示他们的语法。

## `test`属性（The test attribute）

简单的说，测试是一个标记为`test`属性的函数。让我们用 Cargo 来创建一个叫`adder`的项目：

```bash
$ cargo new adder
$ cd adder
```

在你创建一个新项目时 Cargo 会自动生成一个简单的测试。下面是`src/lib.rs`的内容：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
    }
}
```

现在暂时去掉`mod`那部分，只关注函数：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
#[test]
fn it_works() {
}
```

注意这个`#[test]`。这个属性表明这是一个测试函数。它现在没有函数体。它肯定能编译通过！让我们用`cargo test`运行测试：

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///home/you/projects/adder)
    Finished debug [unoptimized + debuginfo] target(s) in 0.15 secs
     Running target/debug/deps/adder-941f01916ca4a642

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Cargo 编译和运行了我们的测试。这里有两部分输出：一个是我们写的测试，另一个是文档测试。我们稍后再讨论这些。现在，看看这行：

```text
test tests::it_works ... ok
```

注意那个`tests::it_works`。这是我们函数的名字：

```rust
# fn main() {
fn it_works() {
}
# }
```

然后我们有一个总结行：

```bash
test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured
```

那么为啥我们这个啥都没干的测试通过了呢？任何没有`panic!`的测试通过，`panic!`的测试失败。让我们的测试失败：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
#[test]
fn it_works() {
    assert!(false);
}
```

`assert!`是 Rust 提供的一个宏，它接受一个参数：如果参数是`true`，啥也不会发生。如果参数是`false`，它会`panic!`。让我们再次运行我们的测试：

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///home/you/projects/adder)
    Finished debug [unoptimized + debuginfo] target(s) in 0.17 secs
     Running target/debug/deps/adder-941f01916ca4a642

running 1 test
test it_works ... FAILED

failures:

---- it_works stdout ----
        thread 'it_works' panicked at 'assertion failed: false', src/lib.rs:5
note: Run with `RUST_BACKTRACE=1` for a backtrace.


failures:
    it_works

test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured

error: test failed
```

Rust指出我们的测试失败了：

```text
test it_works ... FAILED
```

这反映在了总结行上：

```text
test result: FAILED. 0 passed; 1 failed; 0 ignored; 0 measured
```

我们也得到了一个非 0 的状态码.我们在 OS X和 Linux 中使用`$?`：

```bash
$ echo $?
101
```

在 Windows 中，如果你使用`cmd`：

```dos
> echo %ERRORLEVEL%
```

而如果你使用 PowerShell：

```powershell
> echo $LASTEXITCODE # the code itself
> echo $? # a boolean, fail or succeed
```

这在你想把`cargo test`集成进其它工具时是非常有用。

我们可以使用另一个属性反转我们的失败的测试：`should_panic`：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
#[test]
#[should_panic]
fn it_works() {
    assert!(false);
}
```

现在即使我们`panic!`了测试也会通过，并且如果我们的测试通过了则会失败。让我试一下：

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///home/you/projects/adder)
    Finished debug [unoptimized + debuginfo] target(s) in 0.17 secs
     Running target/debug/deps/adder-941f01916ca4a642

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

Rust提供了另一个宏，`assert_eq!`用来比较两个参数：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
#[test]
#[should_panic]
fn it_works() {
    assert_eq!("Hello", "world");
}
```

那个测试通过了吗？因为那个`should_panic`属性，它通过了：

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///home/you/projects/adder)
    Finished debug [unoptimized + debuginfo] target(s) in 0.21 secs
     Running target/debug/deps/adder-941f01916ca4a642

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

`should_panic`测试是脆弱的，因为很难保证测试是否会因什么不可预测原因并未失败。为了解决这个问题，`should_panic`属性可以添加一个可选的`expected`参数。这个参数可以确保失败信息中包含我们提供的文字。下面是我们例子的一个更安全的版本：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
#[test]
#[should_panic(expected = "assertion failed")]
fn it_works() {
    assert_eq!("Hello", "world");
}
```

这就是全部的基础内容！让我们写一个“真实”的测试：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[test]
fn it_works() {
    assert_eq!(4, add_two(2));
}
```

`assert_eq!`是非常常见的；用已知的参数调用一些函数然后与期望的输出进行比较。

## `ignore`属性

有时一些特定的测试可能非常耗时。这时可以通过`ignore`属性来默认禁用：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[test]
fn it_works() {
    assert_eq!(4, add_two(2));
}

#[test]
#[ignore]
fn expensive_test() {
    // Code that takes an hour to run...
}
```

现在我们运行测试并发现`it_works`被执行了，而`expensive_test`没有

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///home/you/projects/adder)
    Finished debug [unoptimized + debuginfo] target(s) in 0.20 secs
     Running target/debug/deps/adder-941f01916ca4a642

running 2 tests
test expensive_test ... ignored
test it_works ... ok

test result: ok. 1 passed; 0 failed; 1 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

耗时的测试可以通过调用`cargo test -- --ignored`来执行：

```bash
$ cargo test -- --ignored
    Finished debug [unoptimized + debuginfo] target(s) in 0.0 secs
     Running target/debug/deps/adder-941f01916ca4a642

running 1 test
test expensive_test ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

`--ignored`参数是 test 程序的参数，而不是 Cargo 的，这也是为什么命令是`cargo test -- --ignored`。

## `tests`模块

然而以这样的方式来实现我们的测试的例子并不是地道的做法：它缺少`tests`模块。你可能注意到了这个测试模块在最初用`cargo new`生成时还在代码中存在，不过在我们最后一个例子中消失了。让我们解释一下。


一个比较惯用的做法应该是如下的：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::add_two;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }
}
```

这里产生了一些变化。第一个变化是引入了一个`cfg`属性的`mod tests`。这个模块允许我们把所有测试集中到一起，并且需要的话还可以定义辅助函数，它们不会成为我们包装箱的一部分。`cfg`属性只会在我们尝试去运行测试时才会编译测试代码。这样可以节省编译时间，并且也确保我们的测试代码完全不会出现在我们的正式构建中。

第二个变化是`use`声明。因为我们在一个内部模块中，我们需要把我们要测试的函数导入到当前空间中。如果你有一个大型模块的话这会非常烦人，所以这里有经常使用一个`glob`功能。让我们修改我们的`src/lib.rs`来使用这个：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }
}
```

注意`use`行的变化。现在运行我们的测试：

```bash
$ cargo test
    Updating registry `https://github.com/rust-lang/crates.io-index`
   Compiling adder v0.1.0 (file:///home/you/projects/adder)
     Running target/debug/deps/adder-91b3e234d4ed382a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

它能工作了！

目前的习惯是使用`test`模块来存放你的“单元测试”。任何只是测试一小部分功能的测试理应放在这里。那么“集成测试”怎么办呢？我们有`tests`目录来处理这些。

## `tests`目录

每一个`tests/*.rs`文件都被当作一个独立的 crate。因此，为了进行集成测试，让我们创建一个`tests`目录，然后放一个`tests/integration_test.rs`文件进去，输入如下内容：

```rust
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
# // Sadly, this code will not work in play.rust-lang.org, because we have no
# // crate adder to import. You'll need to try this part on your own machine.
extern crate adder;

#[test]
fn it_works() {
    assert_eq!(4, adder::add_two(2));
}
```

这看起来与我们刚才的测试很像，不过有些许的不同。我们现在有一行`extern crate adder`在开头。这是因为在`tests`目录中的每个测试（文件）是一个完全不同的 crate，所以我们需要导入我们的库。这也是为什么`tests`是一个写集成测试的好地方：它们就像其它程序一样使用我们的库。

让我们运行一下：

```bash
$ cargo test
   Compiling adder v0.1.0 (file:///home/you/projects/adder)
     Running target/debug/deps/adder-91b3e234d4ed382a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

     Running target/debug/integration_test-68064b69521c828a

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured
```

现在我们有了三个部分：我们之前的两个测试，然后还有我们新添加的。

Cargo （不？）会忽略`tests/`目录的子目录的文件。因此在集成测试中共享模块是可能的。例如`tests/common/mod.rs`并不会被 Cargo 单独编译并可以被任何包含`mod common`的测试（文件）引用。

这就是`tests`目录的全部内容。它不需要`test`模块因为它整个就是关于测试的。

让我们最后看看第三部分：文档测试。

## 文档测试
没有什么是比带有例子的文档更好的了。当然也没有什么比不能工作的例子更糟的，因为文档完成之后代码已经被改写。为此，Rust支持自动运行你文档中的例子（**注意：**这只在库 crate中有用，而在二进制 crate 中没用）。这是一个完整的有例子的`src/lib.rs`：

~~~rust,ignore
# // The next line exists to trick play.rust-lang.org into running our code as a
# // test:
# // fn main
#
//! The `adder` crate provides functions that add numbers to other numbers.
//!
//! # Examples
//!
//! ```
//! assert_eq!(4, adder::add_two(2));
//! ```

/// This function adds two to its argument.
///
/// # Examples
///
/// ```
/// use adder::add_two;
///
/// assert_eq!(4, add_two(2));
/// ```
pub fn add_two(a: i32) -> i32 {
    a + 2
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(4, add_two(2));
    }
}
~~~

注意模块级的文档以`//!`开头然后函数级的文档以`///`开头。Rust文档在注释中支持Markdown语法，所以它支持3个反单引号代码块语法。想上面例子那样，加入一个`# Examples`部分被认为是一个惯例。

让我们再次运行测试：

```bash
$ cargo test
   Compiling adder v0.1.0. (file:///home/you/projects/adder)
     Running target/debug/deps/adder-91b3e234d4ed382a

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

     Running target/debug/integration_test-68064b69521c828a

running 1 test
test it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured

   Doc-tests adder

running 2 tests
test add_two_0 ... ok
test _0 ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured
```

现在我们运行了3种测试！注意文档测试的名称：`_0`生成为模块测试，而`add_two_0`函数测试。如果你添加更多用例的话它们会像`add_two_1`这样自动加一。

我们还没有讲到所有编写文档测试的所有细节。关于更多，请看[文档章节](Documentation 文档.md)。

## 测试与并发

特别需要注意的是测试使用线程来并发的运行。为此需要注意测试之间不能相互依赖也不能依赖任何共享状态。“共享状态”可以包括运行环境，例如当前工作目录（cwd），或者环境变量。

如果这样做有问题控制这些并发也是可能的，要么设置环境变量`RUST_TEST_THREADS`，或者向测试传递`--test-threads`用来比较两个参数：

```bash
$ RUST_TEST_THREADS=1 cargo test   # Run tests with no concurrency
...
$ cargo test -- --test-threads=1   # Same as above
...
```

## 测试输出

默认 Rust 测试标准库捕获并将输出丢弃到标准输出/错误中。例如来自`println!()`的输出。这也可以通过环境变量或者参数来控制：

```bash
$ RUST_TEST_NOCAPTURE=1 cargo test   # Preserve stdout/stderr
...
$ cargo test -- --nocapture          # Same as above
...
```

然而一个避免输出被捕获的更好的方式是采用日志而不是使用原始的输出。Rust 有一个 [standard logging API][log]，它提供了一个多种日志实现的前端。这可以被用来与默认的 [env_logger] 相结合，以一种可以在运行时控制的方式输出任何调试信息。

[log]: https://crates.io/crates/log
[env_logger]: https://crates.io/crates/env_logger
