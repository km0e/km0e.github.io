+++
date = 2025-02-27T05:13:46Z
update-date = 2025-02-27T05:14:09Z
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

## 速查

### 函数介绍

```rust
async fn add_current(id: &str, user: Object) -> Result<(), Error>;
async fn add_ssh_user(id: &str, user: Object) -> Result<(), Error>;
async fn user_params(id: &str) -> Result<Object, Error>;
async fn copy(id: &str, src:&str, src_path:&str, dst:&str, dst_path:&str) -> Result<(), Error>;
async fn app(id: &str, apps: Vec<&str>) -> Result<(), Error>;
async fn exec(id: &str, shell: Option<&str>, cmd: &str) -> Result<(), Error>;
async fn once(id: &str, tid: &str, f: fn() -> Future<Result<(),Error>>) -> Result<(), Error>;
```

### 参数介绍

|`function.arg.field`|`default`|`description`|
|-|-|-|
|`add_current.id`| |指定用户的`id`|
|`add_current.user.hid`|`"local"`|设置用户的`hid`，用于判定是否属于同一设备 |
|`add_current.user.mount`|`"~/.config/dv"`|设置用户的`mount`路径，目前用于访问文件系统时相对路径的展开，如`copy`|
|`add_ssh_user.id`| |指定用户的`id`|
|`add_ssh_user.user.hid`|`$id`|设置用户的`hid`，用于判定是否属于同一设备 |
|`add_ssh_user.user.host`|`$id`|设置用户的`host`，用于`ssh`连接，需要访问当前用户的`ssh_config`文件获取配置 |
|`add_ssh_user.user.is_system`|`false`|设置用户是否为系统用户 |
|`add_ssh_user.user.os`|`"linux"`|设置用户的操作系统 |
|`user_params.id`| |指定用户的`id`|
|`copy.id`| |指定用户的`id`|
|`copy.src`| |指定源用户的`id`|
|`copy.src_path`| |指定源用户的路径|
|`copy.dst`| |指定目标用户的`id`|
|`copy.dst_path`| |指定目标用户的路径|
|`app.id`| |指定用户的`id`|
|`app.apps`| |指定应用列表|
|`exec.id`| |指定用户的`id`|
|`exec.shell`|`None`|指定执行命令的`shell`|
|`exec.cmd`| |指定执行的命令，如果`$shell`不为`None`，则直接作为脚本执行|
|`once.id`| |指定用户的`id`|
|`once.tid`| |指定任务的`id`|
|`once.f`| |指定执行的函数，一般配合`exec`，`copy`，`app`,等使用|

## 功能

命令行参数可以指定`Rune`代码的入口，一般第一个参数是一个`Any`类型的变量，所有功能均由此变量的成员函数呈现。

### 用户管理

#### 当前用户

|`name`|`type`|`default`|
|-|-|-|
|`id` |`str`| |
|`hid` |`str`| |

```rust
let user = #{};
user["hid"] = "local";
user["mount"] = "~/.config/dv";
dv.add_current("this", user).await?;
```

上述代码表示添加当前用户，`hid`为`local`，`mount`为`~/.config/dv`。所赋值均为默认值，所以可以省略简写如下：

```rust
dv.add_current("this", #{}).await?;
```

#### ssh用户

```rust
let user = #{};
user["hid"] = "system";
user["host"] = "system";
user["is_system"] = true;
user["os"] = "linux";
dv.add_ssh_user("system", user).await?;
```

上述代码表示添加`ssh`用户，`hid`与`host`默认值为`$id`，`is_system`默认值为`false`，`os`默认值为`linux`。

### Api

#### user_params

```rust
let p = dv.user_params("this").await?;
```

上述代码表示获取用户`this`的参数，目前只有`os`和`hid`两个参数。

#### copy

```rust
dv.copy("this", "some_config", "system", "target_config").await?;
```

上述代码表示`copy`用户`this`的`some_config`到`system`用户的`target_config`。具体规则如下：

|`src_path`|`dst_path`|文件移动|描述|
|-|-|-|-|
|a/|b/|a/\* -> b/\*|拷贝目录下所有文件到目标目录|
|a/|b|a/\* -> b/\*|拷贝目录下所有文件到目标文件|
|a|b/|a -> b/a|拷贝目录/文件到目标目录|
|a|b|a -> b|拷贝目录（文件）到目标目录|

#### app

```rust
dv.app("this", ["zellij"]).await?;
```

上述代码表示用户`this`安装`zellij`应用。

#### exec

```rust
dv.exec("this", None, "echo hello").await?;
dv.exec("this", Some("bash"), "if test -f ~/.bashrc; then echo 'exist'; else echo 'not exist'; fi").await?;
```

第一个函数调用表示用户`this`执行`echo hello`命令，第二个函数调用表示用户`this`执行`bash`命令，后面的命令将会作为脚本传入`bash`执行。

#### once

```rust
dv.once("app_install", ||dv.app("this", ["zellij"])).await?;
```

上述代码表示只执行一次`app`函数，如果已经安装过`zellij`则不会再次安装。当然也可以配合`exec`函数使用。
