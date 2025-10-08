+++
date = 2025-09-14T14:04:08Z
updated = 2025-09-14T14:10:46Z
description = "Rust Where 相关内容"
draft = true
title = "Rust Where"

[extra]
toc = true

[taxonomies]
tags = ["RUST"]
+++

## 简介

在 Rust 中，`where` 关键字用于在函数、结构体或枚举的泛型参数中指定约束条件。它允许你为泛型类型添加更复杂的约束，使代码更加清晰和易读。`where` 子句通常用于当泛型参数较多或约束条件较复杂时，可以将这些约束条件集中在一起，而不是在泛型参数列表中逐一列出。

不使用 `where`：

```rust
fn example<T: std::fmt::Display, U: std::fmt::Debug>(t : T, u: U) {
    println!("t: {}, u: {:?}", t, u);
}
```

使用 `where`：

```rust
fn example<T, U>(t: T, u: U)
where
    T: std::fmt::Display,
    U: std::fmt::Debug,
{
    println!("t: {}, u: {:?}", t, u);
}
```

`where` 子句可以用于以下几种情况：

1. **函数**：为函数的泛型参数添加约束。
2. **结构体**：为结构体的泛型参数添加约束。
3. **枚举**：为枚举的泛型参数添加约束。
4. **实现块**：为实现块中的泛型参数添加约束。

### 示例

```rust
// 使用 where 为函数的泛型参数添加约束
fn print_info<T, U>(t: T, u: U)
where
    T: std::fmt::Display,
    U: std::fmt::Debug,
{
    println!("t: {}, u: {:?}", t, u);
}
// 使用 where 为结构体的泛型参数添加约束
struct Pair<T, U>
where
    T: std::fmt::Display,
    U: std::fmt::Debug,
{
    first: T,
    second: U,
}
// 使用 where 为枚举的泛型参数添加约束
enum Result<T, E>
where
    T: std::fmt::Display,
    E: std::fmt::Debug,
{
    Ok(T),
    Err(E),
}
// 使用 where 为实现块中的泛型参数添加约束
impl<T, U> Pair<T, U>
where
    T: std::fmt::Display,
    U: std::fmt::Debug,
{
    fn display(&self) {
        println!("first: {}, second: {:?}", self.first, self.second);
    }
}
```

## 参考资料

[keywords - Rust](https://doc.rust-lang.org/std/keyword.where.html)
[rust-by-example - where](https://doc.rust-lang.org/rust-by-example/generics/where.html)
