+++
date = 2025-07-26T02:25:41Z
update-date = 2025-07-26T21:35:03Z
description = "A simple CLI to use dv-api with lua"
draft = false
title = "Dv4lua"

[extra]
toc = true

[taxonomies]
tags = ["PRO"]
+++

## 介绍

这个工具主要功能是用来方便的执行一系列操作，如：自动化任务、文件备份、应用管理、执行命令，配置文件管理等。
由于自定义执行逻辑以配置文件的形式呈现过于复杂，本工具选择与[`Lua`](https://github.com/mlua-rs/mlua.git)结合，以全局变量`dv`的形式提供API接口。

## API

### UM

`UM`类提供了用户管理的功能，主要用于添加用户。

以下是`UM`类的定义和成员函数：

```lua
---@class UM
---@field add_cur fun(this: UM, cfg: table)  # 声明方法
---@field add_ssh fun(this: UM, uid: string, cfg: table)  # 声明方法
```

`cfg`参数为用户配置表，可以任意添加字段。可以影响`dv`的行为字段如下：
| name | value | description |
| ------ | ------------ | ----------------------------------------- |
| `hid` | any string | 用户所在的主机标识符，`cur`用户会自动设置为`local` |
| `mount` | any string | 用户的挂载目录，影响文件操作的路径解析 |
| `os` | `linux`/`windows`/`macos`/`unix`/`ubuntu`/`...` | 用户的操作系统，影响包管理器，变量加载，等等 |

Example:

```lua
dv.um:add_cur({
    mount = "~/.local/share/dv",
    os = "linux"
})
dv.um:add_ssh("remote_user", {
    hid = "remote_machine",
    mount = "~/.local/share/dv",
    os = "linux"
})
```

Note:

- 即便设置了`os`，`dv`仍然会尝试探测用户的操作系统，并在必要时覆盖该值。目前`ssh`用户的操作系统探测只支持`linux`（且需要先设置`os`为`linux`），其他操作系统的探测为`TODO`状态。

### Op

`Op`类提供了一系列操作方法，主要用于执行命令、文件操作等。

以下是`Op`类的定义和成员函数：

```lua
---@class Op  # 定义 `Op` 类
---@field copy fun(this: Op, src: string, src_paths: string|table, dst: string, dst_paths: string|table, confirm: string?)
---@field exec fun(this: Op, uid: string, cmd: string, shell: string?)
---@field os fun(this: Op, uid: string)
```

Example:

```lua
dv.op:copy("this", "a/b/c", "remote_user", "a/b/c", "y") -- 拷贝文件
dv.op:copy("this", {"a/b/c", "a/b/d"}, "remote_user", {"a/b/c", "a/b/d"}, "u") -- 拷贝多个文件
dv.op:exec("this", "echo hello") -- 执行命令
dv.op:exec("this", "echo hello", "bash") -- 使用 bash 执行命令
local os = dv.op:os("this") -- 获取用户的操作系统
```

路径首先会尝试使用`variable`与环境变量展开，格式为`${var}`，如果是相对路径，则会尝试使用`mount`展开。

复制文件路径决定行为规则如下：

| src | dst | action       | description                  |
| --- | --- | ------------ | ---------------------------- |
| a/  | b/  | a/\* -> b/\* | 拷贝目录下所有文件到目标目录 |
| a/  | b   | a/\* -> b/\* | 拷贝目录下所有文件到目标目录 |
| a   | b/  | a -> b/a     | 拷贝目录/文件到目标目录      |
| a   | b   | a -> b       | 拷贝目录（文件）到目标路径   |

`confirm`为`y`（覆盖），`u`（更新），`n`（不变）组成的字符串。

#### `Check`

根据源文件与目标文件状态，检查可执行操作的类型。

| src | dst  | action | description                |
| --- | ---- | ------ | -------------------------- |
| \*  | none | y      | 直接覆盖文件，没有后续确定 |
| new | old  | y      | 可以覆盖文件或者不变       |
| old | new  | u      | 可以更新文件或者不变       |
| old | old  | n      | 不变                       |
| new | new  | yu     | 可以覆盖文件，更新或者不变 |

#### `Match`

根据`confirm`参数，按顺序匹配可执行操作的类型。

Example:

| action | confirm | result |
| ------ | ------- | ------ |
| y      | y       | y      |
| y      | uy      | y      |
| y      | n       | n      |
| y      | ny      | n      |
| u      | y       |        |
| yu     | y       |        |

如果可以确定`result`则执行`result`操作，否则进入交互模式。

### `Dot`

`Dot`类提供了配置文件的管理功能，主要用于加载配置文件、备份配置文件等。
以下是`Dot`类的定义和成员函数：

```lua
---@class Dot
---@field confirm fun(this: Dot, default: string)
---@field add_schema fun(this: Dot, name: string, path: string)
---@field add_source fun(this: Dot, name: string, path: string)
---@field sync fun(this: Dot, apps: table, uid: string)
```

`confirm`方法用于配置复制文件时的默认方式，参见[`Op`](#op)

`Schema`格式举例如下：

```toml
name = "default"

[schemas.fish.repo.linux.paths]
default = ["~/.config/fish"]

[schemas.alacritty.repo.windows.paths]
default = ["${APPDATA}/alacritty/alacritty.toml"]

[schemas.alacritty.repo.unix.paths]
default = [
  "${XDG_CONFIG_HOME}/alacritty/alacritty.toml",
  "${XDG_CONFIG_HOME}/alacritty.toml",
  "~/.config/alacritty/alacritty.toml",
  "~/.alacritty.toml",
]

[schemas.fcitx5.repo.linux.paths]
data = ["~/.local/share/fcitx5"]
default = ["~/.config/fcitx5"]
```

`name`为配置文件的名称，`paths`中每一项为一个配置文件的所有可能路径。
对于以上配置文件，解析如下：

- `fish`配置文件在`linux`下的配置文件`default`路径为`~/.config/fish`
- `alacritty`配置文件在`windows`下的配置文件`default`路径为`${APPDATA}/alacritty/alacritty.toml`，注意`${APPDATA}`将会被展开
- `alacritty`配置文件在`unix`下的配置文件`default`路径有四个可能的路径
- `fcitx5`配置文件在`linux`下的配置文件`data`路径为`~/.local/share/fcitx5`，`default`路径为`~/.config/fcitx5`

`add_schema`方法用于添加配置文件的schema，目前只支持以`uid`和`path`为参数加载配置文件

`Source`格式与`Schema`相同，举例如下：

```toml
name = "default"

[schemas.fish.repo.linux.paths]
default = ["fish"]

[schemas.alacritty.repo.unknown.paths]
default = ["alacritty/alacritty.toml"]

[schemas.fcitx5.repo.linux.paths]
default = ["fcitx5/config"]
data = ["fcitx5/data"]
```

`add_source`方法用于添加配置文件的源，目前只支持以`uid`和`path`为参数加载配置文件。
默认会加载`path`目录下`config.toml`文件作为配置文件，所有`paths`会被以`path`为前缀进行拼接。

`sync`方法用于同步配置文件到指定用户，`apps`参数为一个列表，每个元素为一个配置文件的名称，`uid`参数为用户的`id`。

Example:

```lua
dv.dot:add_schema("default", "path/to/schema.toml")
dv.dot:add_source("default", "path/to/source")
dv.dot:sync({"fish", "alacritty", "fcitx5"}, "remote_user")
```

## `CLI`

`dv4lua`命令行工具的使用方法如下：

```bash
Simple CLI to use dv-api with lua

Usage: dv4lua [OPTIONS] [ENTRY] [RARGS]...

Arguments:
  [ENTRY]     The entry point of the script [default: Main]
  [RARGS]...  Arguments to pass to the entry point

Options:
  -b, --dbpath <DBPATH>        cache database path [default: $directory/.cache]
  -c, --config <CONFIG>        The config file to use [default: $directory/config.lua]
  -d, --directory <DIRECTORY>  The directory to use for the config and cache [default: /home/km/.config/dv/]
  -n, --dry-run                Do not actually modify anything
  -h, --help                   Print help
  -V, --version                Print version
```
