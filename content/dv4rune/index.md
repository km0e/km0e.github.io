+++
date = 2025-02-27T05:13:46Z
updated = 2025-04-02T13:39:29Z
description = "A simple CLI to use dv-api with rune"
draft = false
title = "Dv4rune"

[extra]
toc = true

[taxonomies]
tags = ["PRO"]
+++

## 介绍

这个工具主要功能是用来方便的执行一系列操作，如：自动化任务、文件拷贝、应用管理、执行命令等。
由于自定义执行逻辑以配置文件的形式呈现过于复杂，本工具选择与[`Rune`](https://rune-rs.github.io/)结合，以`Module`形式嵌入，可以在`Rune`代码中直接调用`dv-api`的功能。

## API

### 外部类型

| type                    | description |
| ----------------------- | ----------- |
| [`Config`](#config)     | 用户配置    |
| [`Os`](#Os)             | 操作系统    |
| [`Packages`](#packages) | 包          |
| [`Dv`](#dv)             | API         |

#### `Config`

成员函数

| sig                                               | description                                                                                      |
| ------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| `fn cur() -> Config`                              | 创建当前用户的`Config`，默认设置`HID`为`local`，`MOUNT`为`~/.local/share/dv`，`OS`为`linux`      |
| `fn ssh(host: &str) -> Config`                    | 创建`ssh`用户的`Config`，默认设置`HOST`为`host`参数，`MOUNT`为`~/.local/share/dv`，`OS`为`linux` |
| `fn index_set(&mut self, key: &str, value: &str)` | 设置`Config`的成员变量                                                                           |

成员变量

| name        | type           | description                                                                   |
| ----------- | -------------- | ----------------------------------------------------------------------------- |
| `is_system` | `Option<bool>` | 设置用户是否为系统用户，如果不设置则当前用户自行探测，`ssh`目前为为`TODO`状态 |

Example:

```rust
let cur = Config::cur();// 创建当前用户的配置
cur["HID"] = "local_machine";// 修改当前用户的`HID`为`local_machine`
let ssh = Config::ssh("host");// 创建ssh用户的配置
ssh.is_system = Some(true);// 设置`ssh`用户为`root`用户
```

#### `Os`

成员函数

| sig                                   | description                                   |
| ------------------------------------- | --------------------------------------------- |
| `fn compat(&self, oth: &str) -> bool` | 判断`Os`是否兼容于`oth`,如`linux`兼容于`unix` |
| `fn as_str(&self) -> String`          | 将`Os`转换为字符串                            |

Example:

```rust
let os = dv.os("this")?;// 获取`this`用户的`Os`
os.compat("linux"); // 判断`os`是否兼容于`linux`
dbg!(os.as_str()); // 打印`os`的字符串表示
```

#### `Packages`

成员函数

| sig                                               | description            |
| ------------------------------------------------- | ---------------------- |
| `fn new() -> Packages`                            | 创建一个新的`Packages` |
| `fn add_assign(&mut self, pkg: &Packages)`        | 合并另一个`Packages`   |
| `fn index_set(&mut self, key: &str, value: &str)` | 设置包管理器对应的包   |

Example:

```rust
let pkg = Packages::new();// 创建一个新的`Packages`
pkg["apt"] = "zellij";// 设置`apt`包管理器对应的包为`zellij`
pkg["pacman"] = "zellij";// 设置`pacman`包管理器对应的包为`zellij`
let pkg2 = Packages::new();// 创建另一个新的`Packages`
pkg2["apt"] = "alacritty";// 设置`apt`包管理器对应的包为`alacritty`
pkg += pkg2;// 合并两个`Packages`
```

#### `Dv`

成员函数

| sig                                                                                                         | description        | link              |
| ----------------------------------------------------------------------------------------------------------- | ------------------ | ----------------- |
| `async fn add_user(id: &str, user: Config) -> Result<(), Error>`                                            | 添加用户           | [link](#add-user) |
| `async fn auto(id: &str, service: &str, action: &str, args: Option<&str>) -> Result<bool, Error>`           | 自动化任务         | [link](#auto)     |
| `async fn copy(id: &str, src:(&str, &str), dst:(&str, &str), confirm: Option<&str>) -> Result<bool, Error>` | 拷贝文件           | [link](#copy)     |
| `async fn exec(id: &str, shell: Option<&str>, cmd: &str) -> Result<bool, Error>`                            | 执行命令           | [link](#exec)     |
| `load_src(id: &str, path: &str) -> Result<Vec, Error>`                                                      | 加载源文件         | [link](#load-src) |
| `async fn once(id: &str, tid: &str, f: fn() -> Future<Result<(),Error>>) -> Result<bool, Error>`            | 执行一次           | [link](#once)     |
| `fn os(id: &str) -> Result<Os, Error>`                                                                      | 获取用户的操作系统 | [link](#os)       |
| `async fn pm(id: &str, pkg: Package) -> Result<bool, Error>`                                                | 包安装器           | [link](#pm)       |
| `async fn refresh(id: &str, key: &str) -> Result<(), Error>`                                                | 刷新某些内置的机制 | [link](#refresh)  |

##### `add_user`

添加一个用户，用户的配置可以是当前用户或`ssh`用户。

Example:

```rust
dv.add_user("this", Config::cur()).await?;
dv.add_user("system", Config::ssh("system")).await?;
```

##### `auto`

对指定的服务执行指定的动作，动作可以是`setup`，`reload`，`destroy`。

平台支持：

- `current`

  - `linux`: 使用[`systemd`](https://wiki.archlinux.org/title/Systemd)作为后端实现

    | action  | description               |
    | ------- | ------------------------- |
    | setup   | 理解为`enable`            |
    | reload  | 理解为`reload_or_restart` |
    | destroy | 理解为`disable`           |

  - `windows`: 使用[`system service`](https://learn.microsoft.com/en-us/windows/win32/services/using-services)作为后端实现（实验）

    | action  | description                  |
    | ------- | ---------------------------- |
    | setup   | 理解为`install enable start` |
    | reload  | 理解为`stop start`           |
    | destroy | 理解为`stop uninstall`       |

  - `macos`: TODO

- `ssh`

  - `linux`: 使用`systemd`作为后端实现（systemctl）

    | action  | description               |
    | ------- | ------------------------- |
    | setup   | 理解为`enable`            |
    | reload  | 理解为`reload_or_restart` |
    | destroy | 理解为`disable`           |

  - `windows`: TODO
  - `macos`: TODO

Example:

```rust
dv.auto("this", "sshd", "reload", None).await?;
```

##### `copy`

`copy`文件到指定的用户，`confirm`参数可以是`y`，`u`，`n`，分别表示覆盖，更新，不变。如果不指定`confirm`参数，则运行时会询问用户。

路径规则

- `~`会展开为用户的`home`目录
- 尝试使用`variable`与环境变量展开，格式为`${var}`
- 相对路径会尝试使用`MOUNT`展开
- 格式如下：

| src | dst | action       | description                  |
| --- | --- | ------------ | ---------------------------- |
| a/  | b/  | a/\* -> b/\* | 拷贝目录下所有文件到目标目录 |
| a/  | b   | a/\* -> b/\* | 拷贝目录下所有文件到目标目录 |
| a   | b/  | a -> b/a     | 拷贝目录/文件到目标目录      |
| a   | b   | a -> b       | 拷贝目录（文件）到目标路径   |

**_注意_**：实际上是通过两个文件的`mtime`来判断文件是否需要更新

- 源文件`mtime`变化则尝试覆盖
- 目标文件`mtime`变化则尝试更新

##### `exec`

执行命令或脚本，`shell`参数可以是`None`或`shell`的路径，如果`shell`为`None`，则直接执行命令，如果`shell`不为`None`，则将命令作为脚本传入`shell`执行。

Example:

```rust
dv.exec("this", None, "echo hello").await?;
dv.exec("this", Some("bash"), "if test -f ~/.bashrc; then echo 'exist'; else echo 'not exist'; fi").await?;
```

##### `load_src`

加载源文件，返回一个`Vec`，每个元素为一个`String`，表示源文件的内容。

##### `once`

确保某个函数只执行一次，`tid`参数为任务的`id`，`f`参数为函数指针，函数指针的返回值为`Future<Result<(),Error>>`类型。

Example:

```rust
let pkg = Packages::new();// 创建一个新的`Packages`
pkg["apt"] = "zellij";// 设置`apt`包管理器对应的包为`zellij`
dv.once("app_install", ||dv.pm("this", pkg)).await?;// 这里会执行一次`pm`函数
```

##### `os`

获得用户的操作系统，返回一个`Os`类型的对象，包含了操作系统的名称。

Example:

```rust
let os = dv.os("this")?;// 获取`this`用户的`Os`
```

##### `pm`

安装某个包，`pkg`参数为一个`Packages`类型的对象，表示要安装的包。

Example:

```rust
let pkg = Packages::new();// 创建一个新的`Packages`
pkg["apt"] = "zellij";// 设置`apt`包管理器对应的包为`zellij`
pkg["pacman"] = "zellij";// 设置`pacman`包管理器对应的包为`zellij`
dv.pm("this", pkg).await?;// 安装`zellij`
```

##### `refresh`

刷新某些内置的机制，`key`参数为要刷新的键值。如果`key`为空，则会刷新所有的键值。

| key       | description                       |
| --------- | --------------------------------- |
| `path`    | 则会删除对应的`path`的`mtime`记录 |
| `once_id` | 则会删除对应的`once_id`的记录     |

## 使用

`cli`命令行工具的使用方法如下：

```bash
Simple CLI to use dv-api with rune

Usage: dv4rune [OPTIONS] [ENTRY] [RARGS]...

Arguments:
  [ENTRY]     The entry point of the script [default: main]
  [RARGS]...  Arguments to pass to the entry point

Options:
  -d, --directory <DIRECTORY>  [default: /home/km/.config/dv/]
  -c, --config <CONFIG>        The config file to use
  -b, --dbpath <DBPATH>        Default is $directory/.cache
  -n, --dry-run                Do not actually modify anything
      --direct-run             Run the script directly without running the __build.rn script
  -h, --help                   Print help
  -V, --version                Print version
```

相关文件

| file         | description                                                |
| ------------ | ---------------------------------------------------------- |
| `.cache`     | 缓存文件，默认路径为`$directory/.cache`，可通过`-b`指定    |
| `config.rn`  | 配置文件，默认路径为`$directory/config.rn`，可通过`-c`指定 |
| `__build.rn` | 单独运行的`rune`文件，路径为`$directory/__build.rn`        |

Example:

```rust
// __build.rn
// 这是一个简单的示例
// 加载配置文件
pub async fn main(dv) {
    dv.add_user("r", Config::ssh("r")).await?;
    dv.load_src("r", "rune/lib.rn").await
}
```

```rust
// r:~/.local/share/dv/rune/lib.rn
pub async fn setup_a(dv) {
  // 配置`a`应用
}
pub async fn setup_b(dv) {
  // 配置`b`应用
}
```

```rust
// config.rn
pub fn main(dv) {
  setup_a(dv).await?;
  setup_b(dv).await?;
}
```

## 路线图

### 执行脚本

- [x] 执行命令
- [x] 执行内嵌脚本

- [ ] 指定脚本文件

### 文件操作

- [x] 拷贝文件
- [x] 拷贝目录

- [ ] 修改文件

### 服务管理

- [x] `linux`服务管理

- [ ] `windows`服务管理
- [ ] 更多后端支持

### 应用（包）管理

`platform`支持：

- [x] `apt`
- [x] `pacman`
- [x] `yay`
- [x] `paru`

基础操作

- [x] 安装
- [ ] 卸载等其他操作
