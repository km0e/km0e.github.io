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

Rust 鼓励你把程序拆分为模块。子模块一般在同名文件夹中，或者在同一个文件中（`main.rs` 与 `lib.rs`子模块在`src/`目录下）。

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

## 基本语法

### 变量与可变性

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

### 常量

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

### 变量遮蔽

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

使用 `let` 相当于创建了一个新变量（可以改变类型）

例如：

```rust
let spaces = "   ";
let spaces = spaces.len(); // OK，类型从 &str 变成 usize
```

## 数据类型

Rust 是静态类型语言，变量的类型在编译时就确定了。Rust 有两大类数据类型：

- 标量类型（Scalar Types）：表示单一值
- 复合类型（Compound Types）：可以将多个值组合成一个类型

### 标量类型

标量类型包括：

- 整数类型（Integer Types）
- 浮点数类型（Floating-Point Types）
- 布尔类型（Boolean Type）
- 字符类型（Character Type）

#### 整数类型

整数类型表示没有小数部分的数字。Rust 提供了多种整数类型，按大小和符号分为：
| 大小 | 有符号（Signed） | 无符号（Unsigned） |
|------|------------------|--------------------|
| 8位 | i8 | u8 |
| 16位 | i16 | u16 |
| 32位 | i32 | u32 |
| 64位 | i64 | u64 |
| 128位| i128 | u128 |
| 架构相关 | isize | usize |

#### 浮点数类型

浮点数类型表示带有小数部分的数字。Rust 提供了两种浮点数类型：

- `f32`：32位单精度浮点数
- `f64`：64位双精度浮点数（默认类型）

#### 布尔类型

布尔类型表示真（`true`）或假（`false`）两个值。布尔类型使用 `bool` 关键字表示。

#### 字符类型

字符类型表示单个 Unicode 字符。Rust 使用 `char` 关键字表示字符类型，字符用单引号括起来，例如：`'a'`、`'α'`、`'∞'`。

### 复合类型

复合类型可以将多个值组合成一个类型。Rust 提供了两种主要的复合类型：

- 元组（Tuple）
- 数组（Array）

#### 元组

元组是一种将多个值组合成一个复合值的类型。元组中的值可以是不同类型的。元组使用圆括号 `()` 括起来，值之间用逗号分隔。例如：

```rust
let tup: (i32, f64, u8) = (500, 6.4, 1);
```

你可以通过模式匹配或索引来访问元组中的值：

```rust
let (x, y, z) = tup; // 模式匹配
println!("The value of y is: {y}");

let five_hundred = tup.0; // 通过索引访问
let six_point_four = tup.1;
let one = tup.2;
```

#### 数组

数组是一种将多个相同类型的值组合成一个复合值的类型。数组使用方括号 `[]` 括起来，值之间用逗号分隔。例如：

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

你可以通过索引来访问数组中的值：

```rust
let first = a[0];
let second = a[1];
```

## 函数

函数是 Rust 中的基本代码块，用于封装可重用的逻辑。函数使用 `fn` 关键字定义，后跟函数名和参数列表。参数必须指定类型，函数体用花括号 `{}` 包围。

### 定义函数

```rust
fn main() {
    greet("Alice");
}

```

### 函数参数

函数可以有多个参数，参数之间用逗号分隔。每个参数都必须指定类型：

```rust
fn greet(name: &str) {
    println!("Hello, {}!", name);
}
```

### 带返回值的函数

函数可以返回一个值，使用 `->` 指定返回类型。函数体的最后一个表达式的值会作为返回值返回，不需要使用 `return` 关键字（除非你想提前返回）。

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b // 没有分号，表示这是一个表达式，返回值
}
```

## 控制流

Rust 提供了多种控制流结构来控制代码的执行顺序，包括条件语句和循环。

### 条件语句

条件语句使用 `if`、`else if` 和 `else` 关键字来执行不同的代码块：

```rust
fn main() {
    let number = 6;

    if number % 4 == 0 {
        println!("number is divisible by 4");
    } else if number % 3 == 0 {
        println!("number is divisible by 3");
    } else if number % 2 == 0 {
        println!("number is divisible by 2");
    } else {
        println!("number is not divisible by 4, 3, or 2");
    }
}
```

`if` 语句可以作为表达式使用，这意味着它们可以返回值：

```rust
let condition = true;
let number = if condition { 5 } else { 6 };
```

### 循环

Rust 提供了三种主要的循环结构：`loop`、`while` 和 `for`。

#### 无限循环（loop）

`loop` 创建一个无限循环，除非使用 `break` 语句退出：

```rust
fn main() {
    let mut count = 0;
    loop {
        count += 1;
        if count == 5 {
            break; // 退出循环
        }
        println!("count is: {count}");
    }
}
```

`loop` 也可以作为表达式使用，返回一个值：

```rust
let result = loop {
    count += 1;
    if count == 10 {
        break count * 2; // 返回值
    }
};
println!("The result is {result}");
```

多维循环标签：

```rust
'outer: loop {
    println!("Entered the outer loop");
    'inner: loop {
        println!("Entered the inner loop");
        break 'outer; // 退出外层循环
    }
    println!("This point will never be reached");
}
println!("Exited the outer loop");
```

#### 条件循环（while）

`while` 循环在条件为真时重复执行代码块：

```rust
fn main() {
    let mut number = 3;

    while number != 0 {
        println!("{number}!");
        number -= 1;
    }
    println!("LIFTOFF!!!");
}
```

#### 遍历集合（for）

`for` 循环用于遍历集合（如数组、向量等）：

```rust
fn main() {
    let a = [10, 20, 30, 40, 50];
    for element in a {
        println!("the value is: {element}");
    }
}
```

## 所有权（Ownership）

Rust 的所有权系统是其核心特性之一，旨在确保内存安全和并发安全。理解所有权对于编写高效且安全的 Rust 代码至关重要。

### 基本概念

Rust 通过一套规则来管理内存，这些规则在编译时进行检查，确保程序在运行时不会出现内存错误。所有权系统的三个主要概念是：

1. **所有者（Owner）**：每个值都有一个变量作为其所有者。
2. **借用（Borrowing）**：可以通过引用来借用值，而不是获取其所有权。
3. **生命周期（Lifetimes）**：引用的有效期必须不超过其所有者的生命周期。

### 所有权规则

1. 每个值都有一个所有者。
2. 每个值在任一时刻只能有一个所有者。
3. 当所有者离开作用域时，值会被丢弃，内存会被释放。

### 示例

```rust
fn main() {
    let s1 = String::from("hello"); // s1 是所有者
    let s2 = s1; // 所有权转移到 s2，s1 不再有效
    // println!("{}", s1); // 编译错误，s1 不再有效
    println!("{}", s2); // 输出 "hello"
}
```

### 借用

```rust
fn main() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // 借用 s1 的引用
    println!("The length of '{}' is {}.", s1, len); // s1 仍然有效
}
fn calculate_length(s: &String) -> usize { // s 是一个引用
    s.len()
}
```

### 可变借用

```rust
fn main() {
    let mut s = String::from("hello");
    change(&mut s); // 可变借用
    println!("{}", s); // 输出 "hello, world"
}
fn change(s: &mut String) {
    s.push_str(", world");
}
```

`Rust` 允许在同一时间有多个不可变引用或一个可变引用，但不能同时存在两者：

```rust
fn main() {
    let mut s = String::from("hello");
    let r1 = &s; // 不可变引用
    let r2 = &s; // 另一个不可变引用
    // let r3 = &mut s; // 编译错误，不能同时有可变引用
    println!("{}, {}", r1, r2);
}
```

### 生命周期

生命周期是 Rust 用来确保引用在有效范围内的机制。每个引用都有一个生命周期，表示引用有效的作用域。

```rust
fn main() {
    let r;                // r 没有生命周期
    {
        let x = 5;
        r = &x;          // r 的生命周期与 x 相关联
    }                   // x 离开作用域，r 变为悬垂引用
    // println!("r: {}", r); // 编译错误，r 不再有效
}
```
