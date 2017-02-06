# Rust开发版

> [nightly-rust.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/nightly-rust.md)
> <br>
> commit eb1c7161dd79b55e022cd0c661f9018d406b3fe4

Rust 提供了三种发行渠道：开发版（每日构建），beta 版和稳定版。不稳定功能只在 Rust 开发版中可用。对于这个进程的更多细节，参见[可交付产品的稳定性](http://blog.rust-lang.org/2014/10/30/Stability.html)。

要安装 Rust 开发版，你可以使用`rustup.sh`：

```bash
$ curl -s https://static.rust-lang.org/rustup.sh | sh -s -- --channel=nightly
```

如果你担心使用`curl | sh`的[潜在不安全性](http://curlpipesh.tumblr.com)，请继续阅读并查看我们下面的免责声明。你也可以进行两步安装，以便于检查我们的安装脚本：

```bash
$ curl -f -L https://static.rust-lang.org/rustup.sh -O
$ sh rustup.sh --channel=nightly
```

如果你用 Windows，请直接下载[32位安装包][win32]或者[64位安装包][win64]然后运行即可。

[win32]: https://static.rust-lang.org/dist/rust-nightly-i686-pc-windows-gnu.msi
[win64]: https://static.rust-lang.org/dist/rust-nightly-x86_64-pc-windows-gnu.msi

## 卸载

如果你决定不再需要 Rust 了，我们会有点难过，但没关系。不是每种编程语言都适合所有人。运行下面的卸载脚本即可：

```bash
$ sudo /usr/local/lib/rustlib/uninstall.sh
```

如果你使用 Windows 安装包进行安装的话，重新运行`.msi`文件，它会提供卸载选项。

一些同学确实有理由对我们让他们运行`curl | sudo sh`感到反感。从根本上说，当你运行上面的脚本时，代表你相信是一些好人在维护 Rust，他们不会黑了你的电脑做坏事。对此保持警觉是很好的天性。如果你是这些人之一，请检阅以下文档：[从源码编译Rust](https://github.com/rust-lang/rust#building-from-source)或者[官方二进制文件下载](https://www.rust-lang.org/install.html)。

当然，我们还应该提到官方支持的平台：

* Windows（7+）
* Linux（2.6.18 或更高版本，各种发行版），x86 和 x86-64
* OSX 10.7（Lion）或更高版本，x86 和 x86-64

Rust 在以上平台进行了广泛的测试，当然其他一些平台也有，比如 Android。不过在上述平台工作起来最顺畅，因为它们进行了最多的测试。

最后说说 Windows。Rust 将 Windows 作为第一级平台来发布，不过说实话，WIndows 的集成体验并没有 Linux/OS X 那么好。我们正在改进！如果有不能工作的情况，就是出了 bug。一旦发生了请告知我们。任何一次提交都会在 Windows 下进行测试，和其他平台无异。

如果你已安装 Rust，你可以打开一个 Shell，然后输入：

```bash
$ rustc --version
```

你应该看到版本号，提交的 hash 值，提交时间和构建时间：

```bash
rustc 1.0.0-nightly (f11f3e7ba 2015-01-04) (built 2015-01-06)
```

如果你做到了，那么 Rust 已经成功安装！此处应有掌声！

如果你遇到什么错误，有几个你可以获取帮助的地方。最简单的是通过 [Mibbit](http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust) 访问 [Rust IRC 频道 irc.mozilla.org](irc://irc.mozilla.org/#rust)。点击上面的链接，你就可以与其他 Rustacean（我们这些人自称的绰号）聊天了，我们会帮助你。其他给力的资源包括[用户论坛](https://users.rust-lang.org/)和 [Stack Overflow](http://stackoverflow.com/questions/tagged/rust)。
