+++
date = 2025-09-14T14:44:05Z
update-date = 2025-09-14T15:01:26Z
description = "Rust 生命周期相关内容"
draft = true
title = "Rust 生命周期"

[extra]
toc = true

[taxonomies]
tags = ["RUST"]
+++

## 简介

在 Rust 中，生命周期（lifetime）是一个非常重要的概念，用于管理引用的有效性和内存安全。生命周期确保引用在使用时始终指向有效的数据，防止悬垂引用和数据竞争等问题。Rust 的编译器通过生命周期标注来跟踪引用的作用域，从而保证内存安全。

## 生命周期标注

生命周期标注使用撇号（`'`）加上一个标识符来表示。例如，`'a` 是一个生命周期标注。

### 函数中的生命周期

先举一个简单的例子：

```rust
fn foo(x: &i32, y: &i32) -> &i32 {
    if x > y {
        x
    } else {
        y
    }
}
```

这个时候会报错：

```bash
rustc: missing lifetime specifier
this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y` [E0106]
rustc: consider introducing a named lifetime parameter: `<'a>`, `'a `, `'a `, `'a ` [E0106]
```

这个函数有两个引用参数 `x` 和 `y`，并返回一个引用。然而，这段代码无法编译，因为编译器无法确定返回的引用的生命周期。我们需要为参数和返回值添加生命周期标注：

```rust
fn foo<'a>(x: &'a i32, y: &'a i32) -> &'a i32 {
    if x > y {
        x
    } else {
        y
    }
}
```

在这个例子中，`'a` 是一个生命周期标注，表示 `x` 和 `y` 以及返回值都具有相同的生命周期 `'a`。这意味着返回的引用在 `x` 和 `y` 中较短的生命周期内有效。
比如：

```rust
fn main() {
    let x = 5;
    let r;
    {
        let y = 10;
        r = foo(&x, &y);
        println!("r: {}", r); // 这里 r 是有效的
    }
    // println!("r: {}", r); // 这里 r 无效，因为 y 已经超出作用域
}
```

### 结构体中的生命周期

结构体也可以包含引用，因此需要为结构体添加生命周期标注：

```rust
struct Foo<'a> {
    x: &'a i32,
}
fn main() {
    let value = 42;
    let foo = Foo { x: &value };
    println!("foo.x: {}", foo.x);
}
```

在这个例子中，`Foo` 结构体包含一个引用 `x`，并且该引用的生命周期由 `'a` 标注。

同时，结构体的方法也可以使用生命周期标注：

```rust
impl<'a> Foo<'a> {
    fn get_x(&self) -> &'a i32 {
        self.x
    }
}
```

### 生命周期省略规则

Rust 有一些生命周期省略规则，可以在某些情况下省略显式的生命周期标注：

1. 每个引用参数都有它自己的生命周期参数。例如，`fn foo(x: &i32, y: &i32)` 实际上是 `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`。
2. 如果只有一个输入生命周期参数，那么它会被赋予所有输出生命周期参数。例如，`fn foo(x: &i32) -> &i32` 实际上是 `fn foo<'a>(x: &'a i32) -> &'a i32`。
3. 如果方法有 `&self` 或 `&mut self`，则 `self` 的生命周期会被赋予所有输出生命周期参数。例如，`fn get_x(&self) -> &i32` 实际上是 `fn get_x<'a>(&'a self) -> &'a i32`。

### 静态生命周期

静态生命周期 `'static` 表示引用在整个程序运行期间都是有效的。字符串字面量就是一个例子：

```rust
let s: &'static str = "Hello, world!";
```

## 参考资料

- [Rust By Example - Lifetimes](https://doc.rust-lang.org/rust-by-example/scope/lifetime.html)
