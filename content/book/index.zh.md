+++
date = 2025-06-13T17:00:00Z
update-date = 2025-06-13T17:00:00Z
description = ""
draft = true
title = "Rust"

[extra]
toc = true

[taxonomies]
tags = ["BOOK"]
+++

## 简介

简单学习`Rust`

## Rust 项目结构与包管理（Cargo）

Rust 自带的构建工具和包管理器叫做 **Cargo**。你几乎在所有 Rust 项目中都会使用它，它可以帮你：

- 创建项目
- 编译代码
- 引入依赖
- 运行测试
- 构建发布版本

---

### 创建项目

要创建一个新项目，只需一条命令：

```bash
cargo new my_project
```

或者，如果你只想创建一个库（没有主程序）：

```bash
cargo new --lib my_library
```

Cargo 会帮你自动生成以下结构：

```
my_project/
├── Cargo.toml      // 配置文件（包名、依赖等）
└── src/
    └── main.rs     // 主程序入口（lib.rs 是库项目的入口）
```

你可以进入项目目录，并使用以下命令运行程序：

```bash
cd my_project
cargo run
```

---

### 引入依赖（使用第三方库）

`reqwest`是一个常用的 HTTP 客户端库，以添加`reqwest` 为例
我们可以在 `Cargo.toml` 文件中添加依赖：

```toml
[dependencies]
reqwest = "0.12"
```

也可以直接使用命令行添加依赖：

```bash
cargo add reqwest
```

这里为了使用`blocking`版本的`reqwest`，我们可以在 `Cargo.toml` 中指定：

```toml
[dependencies]
reqwest = { version = "0.12", features = ["blocking"] }
```

然后在代码中使用它：

```rust
use reqwest::blocking::get;

fn main() {
    let res = get("https://www.baidu.com").unwrap();
    println!("状态码: {}", res.status());
}
```

保存后运行：

```bash
cargo run
```

Cargo 会自动下载并编译依赖。

---

### 模块化代码（多文件结构）

Rust 鼓励你把程序拆分为模块。子模块一般在同名文件夹中，或者在同一个文件中（`main.rs` 与 `lib.rs`子模块在`src/`目录下）。`

`src/main.rs`:

```rust
mod utils;

fn main() {
    utils::hello();
}
```

`src/utils.rs`（旧的方式：`src/utils/mod.rs`）:

```rust
mod helper;
pub fn hello() {
    println!("{}", helper::hello_string());
}
```

`src/utils/helper.rs`:

```rust
pub fn hello_string() -> String {
    "Hello from helper!".to_string()
}
```

---

### 编译与构建

你可以构建调试版本：

```bash
cargo build
```

或者构建发布版本（带优化）：

```bash
cargo build --release
```

构建后的可执行文件在：

- 调试模式：`target/debug/my_project`
- 发布模式：`target/release/my_project`

---

### 跑测试

Rust 有内建的测试框架。你可以在任何模块里写测试：

在 `src/utils/helper.rs` 中添加测试：

```rust
///...
#[cfg(test)]
mod tests {
    #[test]
    fn test_hello_string() {
        let result = super::hello_string();
        assert_eq!(result, "Hello from helper!");
    }
}

```

运行测试：

```bash
cargo test
```

---

### 发布自己的库（可选）

如果你写的是工具包或库，并希望发布到 [crates.io](https://crates.io)，可以：

1. 注册账户并获取 token（只需一次）：

```bash
   cargo login <your-token>
```

2. 在 `Cargo.toml` 中填写好包名、版本、作者、描述等。

3. 发布：

```bash
cargo publish
```

---

### 总结

你现在应该可以：

✅ 创建项目
✅ 添加依赖
✅ 拆分模块
✅ 跑测试和构建
✅ 准备发布库

借助 Cargo，Rust 的项目管理变得非常简单。你可以快速搭建一个 API 工具包、命令行工具，或任何库项目。

如果你需要我补充构建 API 项目、引入本地模块、或者如何写文档注释等内容，也可以继续告诉我。

## 变量与可变性

在 Rust 中，**变量默认是不可变的**。这是一种设计选择，可以帮助你写出更安全、并发更容易的代码。但如果需要，你也可以让变量变成**可变的（mutable）**。

来看一个例子：

```rust
fn main() {
    let x = 5;// 不可变变量
    x = 6; // 编译错误
    let mut y = 5; // 可变变量
    y = 6; // OK
}
```

## 常量

常量和不可变变量类似，但也有一些不同点：

- 使用 `const` 关键字声明，而不是 `let`
- **永远是不可变的**，不能加 `mut`
- 必须指定类型
- 只能用**编译时能确定的值**来初始化

例如：

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

常量常用于定义全局通用的值，比如：

- 一天有多少秒
- 游戏中的最高得分
- 圆周率等

命名习惯是使用全大写字母和下划线分隔，比如 `MAX_POINTS`。

---

## 变量遮蔽

Rust 中还支持“**遮蔽（shadowing）**”：你可以用同一个变量名多次声明新变量，旧的变量会被“遮蔽”掉：

```rust
fn main() {
    let x = 5;

    let x = x + 1;

    {
        let x = x * 2;
        println!("内部作用域的 x 是: {x}"); // 12
    }

    println!("外部作用域的 x 是: {x}"); // 6
}
```

**遮蔽和 `mut` 的区别：**

- 使用 `let` 相当于创建了一个新变量（可以改变类型）
- 使用 `mut` 是修改原来的变量（**不能改变类型**）

例如：

```rust
let spaces = "   ";
let spaces = spaces.len(); // OK，类型从 &str 变成 usize
```

而如果使用 `mut`：

```rust
let mut spaces = "   ";
spaces = spaces.len(); // 错误！不能把字符串变成数字
```
