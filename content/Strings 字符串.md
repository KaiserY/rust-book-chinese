# 字符串

> [strings.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/strings.md)
> <br>
> commit d83fff3b3b9dd0fd6eef862e97f883d171367041

对于每一个程序，字符串都是需要掌握的重要内容。由于Rust主要着眼于系统编程，所以它的字符串处理系统与其它语言有些许区别。每当你碰到一个可变大小的数据结构时，情况都会变得很微妙，而字符串正是可变大小的数据结构。这也就是说，Rust的字符串与一些像C这样的系统编程语言也不相同。

让我们进一步了解一下。一个**字符串**是一串UTF-8字节编码的Unicode量级值的序列。所有的字符串都确保是有效编码的UTF-8序列。另外，字符串并不以null结尾并且可以包含null字节。

Rust有两种主要的字符串类型：`&str`和`String`。让我们先看看`&str`。这叫做**字符串片段**（*string slices*）。字符串常量是`&'static str`类型的：

```rust
let greeting = "Hello there."; // greeting: &'static str
```

`"Hello there."`是一个字符串常量而它的类型是`&'static str`。字符串常量是静态分配的字符串切片，也就是说它储存在我们编译好的程序中，并且整个程序的运行过程中一直存在。这个`greeting`绑定了一个静态分配的字符串的引用。任何接受一个字符串切片的函数也接受一个字符串常量。

字符串常量可以跨多行。有两种形式。第一种会包含新行符和之前的空格：

```rust
let s = "foo
    bar";

assert_eq!("foo\n    bar", s);
```

第二种，带有`\`，会去掉空格和新行符：

```rust
let s = "foo\
    bar";

assert_eq!("foobar", s);
```

注意通常你不能直接访问一个`str`，只能通过`&str`引用。这是因为`str`是一个不定长类型，它需要额外的运行时信息才能使用。关于更多请查看[不定长类型章节](Unsized Types 不定长类型.md)。

Rust 当然不仅仅只有`&str`。一个`String`，是一个在堆上分配的字符串。这个字符串可以增长，并且也保证是UTF-8编码的。`String`通常通过一个字符串片段调用`to_string`方法转换而来。

```rust
let mut s = "Hello".to_string(); // mut s: String
println!("{}", s);

s.push_str(", world.");
println!("{}", s);
```

`String`可以通过一个`&`强制转换为`&str`：

```rust
fn takes_slice(slice: &str) {
    println!("Got: {}", slice);
}

fn main() {
    let s = "Hello".to_string();
    takes_slice(&s);
}
```

这种强制转换并不发生在接受`&str`的trait而不是`&str`本身作为参数的函数上。例如，[TcpStream::connect](http://doc.rust-lang.org/stable/std/net/struct.TcpStream.html#method.connect)，有一个`ToSocketAddrs`类型的参数。`&str`可以不用转换不过`String`必须使用`&*`显式转换。

```rust
use std::net::TcpStream;

TcpStream::connect("192.168.0.1:3000"); // Parameter is of type &str.

let addr_string = "192.168.0.1:3000".to_string();
TcpStream::connect(&*addr_string); // Convert `addr_string` to &str.
```

把`String`转换为`&str`的代价很小，不过从`&str`转换到`String`涉及到分配内存。除非必要，没有理由这样做！

## 索引（Indexing）

因为字符串是有效UTF-8编码的，它不支持索引：

```rust
let s = "hello";

println!("The first letter of s is {}", s[0]); // ERROR!!!
```

通常，用`[]`访问一个数组是非常快的。不过，字符串中每个UTF-8编码的字符可以是多个字节，你必须遍历字符串来找到字符串的第N个字符。这个操作的代价相当高，而且我们不想误导读者。更进一步来讲，Unicode实际上并没有定义什么“字符”。我们可以选择把字符串看作一个串独立的字节，或者代码点（codepoints）：

```rust
let hachiko = "忠犬ハチ公";

for b in hachiko.as_bytes() {
    print!("{}, ", b);
}

println!("");

for c in hachiko.chars() {
    print!("{}, ", c);
}

println!("");
```

这会打印出：

```text
229, 191, 160, 231, 138, 172, 227, 131, 143, 227, 131, 129, 229, 133, 172,
忠, 犬, ハ, チ, 公,
```

如你所见，这有比`char`更多的字节。

你可以这样来获取跟索引相似的东西：

```rust
# let hachiko = "忠犬ハチ公";
let dog = hachiko.chars().nth(1); // Kinda like `hachiko[1]`.
```

这强调了我们不得不遍历整个`char`的列表。

## 切片（Slicing）

你可以使用切片语法来获取一个字符串的切片：

```rust
let dog = "hachiko";
let hachi = &dog[0..5];
```

注意这里是**字节**偏移，而不是**字符**偏移。所以如下代码在运行时会失败：

```rust
let dog = "忠犬ハチ公";
let hachi = &dog[0..2];
```

给出如下错误：

```text
thread 'main' panicked at 'byte index 2 is not a char boundary; it is inside '忠'
(bytes 0..3) of `忠犬ハチ公`'
```

## 连接（Concatenation）

如果你有一个`String`，你可以在它后面接上一个`&str`：

```rust
let hello = "Hello ".to_string();
let world = "world!";

let hello_world = hello + world;
```

不过如果你有两个`String`，你需要一个`&`：

```rust
let hello = "Hello ".to_string();
let world = "world!".to_string();

let hello_world = hello + &world;
```

这是因为`&String`可以自动转换为一个`&str`。这个功能叫做[`Deref`转换](`Deref` coercions `Deref`强制多态.md)。
