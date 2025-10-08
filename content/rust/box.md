+++
date = 2025-09-14T14:03:47Z
updated = 2025-09-14T14:03:54Z
description = "Rust Box 相关内容" 
draft = true
title = "Rust Box"

[extra]
toc = true

[taxonomies]
tags = ["RUST"]
+++

## 简介

在 Rust 中，`Box<T>` 是一种智能指针，用于在堆上分配内存。它允许你将数据存储在堆上，而不是栈上，这对于需要动态大小或递归类型的数据结构非常有用。`Box<T>` 提供了所有权和借用的语义，使得内存管理更加安全和高效。

```rust
fn main() {
    let b = Box::new(5); // 在堆上分配一个整数
    println!("b = {}", b); // 输出: b = 5
}
```

## 使用场景

1. **递归数据结构**：例如链表或树，使用 `Box<T>` 可以避免无限大小的问题。

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}
```

2. **配合特征对象使用**：当你需要在运行时决定使用哪种类型时，`Box<dyn Trait>` 可以存储实现了某个特征的不同类型。

```rust
trait Animal {
    fn speak(&self);
}
struct Dog;
impl Animal for Dog {
    fn speak(&self) {
        println!("Woof!");
    }
}
struct Cat;
impl Animal for Cat {
    fn speak(&self) {
        println!("Meow!");
    }
}
fn main() {
    let animals: Vec<Box<dyn Animal>> = vec![Box::new(Dog), Box::new(Cat)];
    for animal in animals {
        animal.speak();
    }
}
```

3. **减少栈空间使用**：对于大型数据结构，将其存储在堆上可以减少栈的使用，避免栈溢出。

## 操作

你可以像使用普通引用一样使用 `Box<T>`，通过解引用操作符 `*` 访问其内容。

```rust
fn main() {
    let b = Box::new(5);
    let x = *b; // 解引用
    println!("x = {}", x); // 输出: x = 5
}
```

[参考资料](https://doc.rust-lang.org/book/ch15-01-box.html)
