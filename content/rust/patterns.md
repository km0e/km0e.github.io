+++
date = 2025-09-14T13:48:30Z
update-date = 2025-09-14T13:48:30Z
description = "Rust Patterns 相关内容"
draft = true
title = "Rust Patterns"

[extra]
toc = true

[taxonomies]
tags = ["RUST"]
+++

### `@` 模式匹配

在 Rust 中，`@` 模式匹配允许你在模式匹配中同时绑定一个变量并检查它是否符合某个模式。这对于需要在匹配时保留对值的引用，同时又想检查该值是否符合特定条件的情况非常有用。

```rust
enum Message {
    Hello { id: i32 },
}
fn main() {
    let msg = Message::Hello { id: 5 };

    match msg {
        Message::Hello { id: id_variable @ 3..=7 } => {
            println!("Found an id in range: {}", id_variable);
        }
        Message::Hello { id: 10..=12 } => {
            println!("Found an id in another range");
        }
        Message::Hello { id } => {
            println!("Found some other id: {}", id);
        }
    }
}
```

在这个例子中，`id_variable @ 3..=7` 绑定了 `id` 的值到 `id_variable`，同时检查它是否在 3 到 7 的范围内。如果匹配成功，你可以在匹配分支中使用 `id_variable`。

[参考资料](https://doc.rust-lang.org/reference/patterns.html#identifier-patterns)
