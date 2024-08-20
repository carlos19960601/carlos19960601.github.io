---
title: '学习rust'
date: 2024-07-25T10:56:02+08:00
draft: false
---

# 生命周期

rust自动推断变量的生命周期

```rust
fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

但是上面这段代码，rust无法推断出变量的生命周期。所以需要显示的标注生命周期。

> 生命周期标注并不会改变任何引用的实际作用域


# 参考资料

* [通过例子学 Rust](https://rustwiki.org/zh-CN/rust-by-example/index.html)
* [Rust 程序设计语言](https://kaisery.github.io/trpl-zh-cn/title-page.html)
* [Rust语言圣经(](https://course.rs/about-book.html)