+++
date = 2025-09-14T14:11:44Z
updated = 2025-09-14T14:42:12Z
description = "Rust 变量共享专题"
draft =true 
title = "Rust 变量共享专题"

[extra]
toc = true

[taxonomies]
tags = ["RUST"]
+++

## 非线程安全的引用计数（Rc）

在 Rust 中，`Rc<T>` 是一种引用计数智能指针，用于在单线程环境中实现共享所有权。它允许多个所有者共享对同一数据的访问，而不需要使用复杂的生命周期管理。`Rc<T>` 通过维护一个引用计数来跟踪有多少个指针指向同一块内存，当最后一个指针被丢弃时，内存会被自动释放。

```rust
use std::rc::Rc;
fn main() {
    let a = Rc::new(5); // 创建一个新的 Rc 指针，指向值 5
    let b = Rc::clone(&a); // 克隆 Rc 指针，增加引用计数
    let c = Rc::clone(&a); // 再次克隆 Rc 指针

    println!("a: {}, b: {}, c: {}", a, b, c); // 输出: a: 5, b: 5, c: 5
    println!("Reference count: {}", Rc::strong_count(&a)); // 输出引用计数
} // 当 a, b, c 超出作用域时，引用计数变为 0，内存被释放
```

与其相关的类型还有 `Weak<T>`，它是一种弱引用，不会增加引用计数，适用于避免循环引用的场景。

循环引用场景如下：

```rust
use std::rc::Rc;
struct Foo {
    bar: Option<Rc<Bar>>,
}
struct Bar {
    foo: Option<Rc<Foo>>,
}
fn main() {
    let foo = Rc::new(Foo { bar: None });
    let bar = Rc::new(Bar { foo: Some(Rc::clone(&foo)) });
    // 创建循环引用
    foo.bar = Some(Rc::clone(&bar));
}
```

使用 `Weak<T>` 可以避免这种情况：

```rust
use std::rc::{Rc, Weak};
struct Foo {
    bar: Option<Weak<Bar>>, // 使用 Weak 避免循环引用
}
struct Bar {
    foo: Option<Rc<Foo>>,
}
fn main() {
    let foo = Rc::new(Foo { bar: None });
    let bar = Rc::new(Bar { foo: Some(Rc::clone(&foo)) });
    foo.bar = Some(Rc::downgrade(&bar)); // 使用 Rc::downgrade 创建弱引用
}
```

使用`cargo-valgrind`可以检测内存泄漏：

```bash
cargo install cargo-valgrind
cargo valgrind run
```

`Rc<T>` 一般配合 `RefCell<T>` 使用，以实现可变的共享所有权：

```rust
use std::rc::Rc;
use std::cell::RefCell;
fn main() {
    let a = Rc::new(RefCell::new(5)); // Rc 包裹 RefCell
    {
        let mut b = a.borrow_mut(); // 可变借用
        *b += 10; // 修改值
    }
    println!("a: {}", a.borrow()); // 输出: a: 15
}
```

### 参考资料

- [Rust 官方文档 - Rc](https://doc.rust-lang.org/std/rc/index.html)

## 线程安全的引用计数（Arc）

`Arc<T>` 是 `Rc<T>` 的线程安全版本，适用于多线程环境。它使用原子操作来管理引用计数，确保在多个线程中安全地共享数据。

```rust
use std::sync::Arc;
use std::thread;
fn main() {
    let a = Arc::new(5); // 创建一个新的 Arc 指针，指向值 5
    let mut handles = vec![];
    for _ in 0..10 {
        let a_clone = Arc::clone(&a); // 克隆 Arc 指针，增加引用计数
        let handle = thread::spawn(move || {
            println!("a: {}", a_clone); // 输出: a: 5
        });
        handles.push(handle);
    }
    for handle in handles {
        handle.join().unwrap(); // 等待所有线程完成
    }
    println!("Reference count: {}", Arc::strong_count(&a)); // 输出引用计数
} // 当所有 Arc 指针超出作用域时，引用计数变为 0，内存被释放
```

与 `Rc<T>` 类似，`Arc<T>` 也有 `Weak<T>` 类型，用于避免循环引用。

一般来说，`Arc<T>` 也常与 `Mutex<T>` ，`RwLock<T>` 或 `Atomic<T>` 一起使用，以实现线程间的可变共享：
