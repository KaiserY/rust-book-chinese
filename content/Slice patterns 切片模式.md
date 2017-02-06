# 切片模式

> [slice-patterns.md](https://github.com/rust-lang/rust/blob/stable/src/doc/book/slice-patterns.md)
> <br>
> commit 5cf4139d21073731fa7f7226d941349dbacc16d6

如果你想在一个切片或数组上匹配，你可以通过`slice_patterns`功能使用`&`：

```rust
#![feature(slice_patterns)]

fn main() {
    let v = vec!["match_this", "1"];

    match &v[..] {
        &["match_this", second] => println!("The second element is {}", second),
        _ => {},
    }
}
```

`advanced_slice_patterns`gate 让你使用`..`表明在一个切片的模式匹配中任意数量的元素。这个通配符对一个给定的数组只能只用一次。如果在`..`之前有一个标识符，结果会被绑定到那个名字上。例如：

```rust
#![feature(advanced_slice_patterns, slice_patterns)]

fn is_symmetric(list: &[u32]) -> bool {
    match list {
        &[] | &[_] => true,
        &[x, ref inside.., y] if x == y => is_symmetric(inside),
        _ => false
    }
}

fn main() {
    let sym = &[0, 1, 4, 2, 4, 1, 0];
    assert!(is_symmetric(sym));

    let not_sym = &[0, 1, 7, 2, 4, 1, 0];
    assert!(!is_symmetric(not_sym));
}
```
