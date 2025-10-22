+++
date = 2025-07-26T02:25:41Z
updated = 2025-08-06T15:58:22Z
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

Note:

- 🚧：表示该功能尚未实现或存在问题

## API

### Dv

`Dv`类是整个API的核心，包含了用户管理（`um`）、配置文件管理（`dot`）和包管理（`pm`）等功能模块。
以下是`Dv`类的定义和成员变量：

```lua
---@class Dv        # `dv` 的类型
---@field sync fun(this: Dv, src: string, src_paths: string|table, dest: string, dest_paths: string|table, confirm: string?)
---@field dl fun(this: Dv, url: string, expire?: string): string
---@field json fun(this: Dv, str: string|any): table|string
---@field um fun(this: Dv):UM  # 关联 `um` 字段的类型
---@field dot fun(this: Dv):Dot  # 关联 `dot` 字段的类型
---@field pm fun(this: Dv):Pm  # 关联 `pm` 字段的类型
```

#### sync

Example:

```lua
dv:sync("this", "a/b/c", "remote_user", "a/b/c", "1") -- 拷贝文件
dv:sync("this", {"a/b/c", "a/b/d"}, "remote_user", {"a/b/c", "a/b/d"}, "2") -- 拷贝多个文件
```

路径首先会尝试使用`variable`与环境变量展开，格式为`${var}`，如果是相对路径，则还会尝试使用`mount`展开。

`confirm`为一个由特殊字符组成的字符串，表示在复制文件时的默认行为。

注意每个行为都在特定条件下才会被执行，具体请参考下方的`Check`与`Match`步骤。

目前支持的字符有：

| char | description                |
| ---- | -------------------------- |
| 0    | 不进行任何操作             |
| 1    | 直接覆盖目标文件         |
| 2    | 更新文件                   |
| 3    | 删除目标文件               |
| 4    | 删除源文件               |
| 5    | 上传文件               |
| 6    | 下载文件               |


对于`sync`操作，主要有以下几个步骤：

##### `Check`

检查源文件与目标文件的状态（对于目录会扫描目录下的所有文件），确定可执行的操作类型。

| src  | dst  | executable action | description                |
| ---- | ---- | ----------------- | -------------------------- |
| \*   | none | 0,4,5 | 无视，删除源文件，上传到目标文件               |
| none | \*   | 0,3,6 | 无视，删除目标文件，下载到目标文件               |
| new  | old | 0,1 | 无视，覆盖目标文件               |
| old  | new | 0,2 | 无视，更新源文件               |
| old  | old | | 直接无视                       |
| new  | new | 0,1,2 | 无视，覆盖目标文件，更新源文件 |


##### `Match`

根据`confirm`参数，按顺序匹配可执行操作的类型，没有匹配则进行交互。

Example:

| executable action | confirm | matched action | result description        |
| ----------------- | ------- | -------------- | ------------------------- |
| 0,4,5             | 0 | 0              | 无视                     |
| 0,4,5             | 4 | 4              | 删除源文件               |
| 0,4,5             | 5 | 5              | 上传文件               |
| 0,4,5             | 04 | 0              | 无视                     |
| 0,4,5             | 1 | none           | 进入交互模式             |
| 0,4,5             | 10 | 0              | 无视                     |
| 0,4,5             | 01 | 0              | 无视                     |


#### `dl`

Example:

```lua
dv:dl("https://example.com/file.txt", "1h") -- 下载文件，过期时间为1小时。格式参考这里[Duration](https://docs.rs/humantime/latest/humantime/)
dv:dl("https://example.com/file.txt") -- 下载文件，默认更新
```

#### `json`

Example:

```lua
local tbl = dv:json('{"key": "value"}') -- 解析 JSON 字符串为 Lua 表
local str = dv:json({key = "value"}) -- 将 Lua 表转换为 JSON 字符串
```

#### um, dot, pm

Example:

```lua
local um = dv:um()
local dot = dv:dot()
local pm = dv:pm()
```

### UM

`UM`类提供了用户管理的功能，实际数据存储在`Dv`类中。

以下是`UM`类的定义和成员函数：

```lua
---@class ExecOptions
---@field reply boolean # 是否为交互式命令
---@field etor string? # 脚本执行器

---@class User
---@field exec fun(this: User, cmd: string, opt:boolean|ExecOptions?) : (number, string?, string?) 
---@field read fun(this: User, path: string): string|nil
---@field write fun(this: User, path: string, content: string)
---@field user string
---@field os string
---@field [string] string # env variables or user config

---@class UM
---@field add_cur fun(this: UM, cfg: table)  # 声明方法
---@field add_ssh fun(this: UM, uid: string, cfg: table)  # 声明方法
---@field [string] User # 用户列表
```

`cfg`参数为用户配置表，可以任意添加字段。可以影响`dv`的行为字段如下：
| name | value | description |
| ------ | ------------ | ----------------------------------------- |
| `hid` | any string | 用户所在的主机标识符，`cur`用户会自动设置为`local` |
| `mount` | any string | 用户的挂载目录，影响文件操作的路径解析 |
| `os` | `linux`/`windows`/`macos`/`unix`/`ubuntu`/`...` | 用户的操作系统，影响包管理器，变量加载，等等 |

Example:

```lua
local um = dv:um()
um:add_cur({
    mount = "~/.local/share/dv",
    os = "linux"
})
um:add_ssh("remote_user", {
    hid = "remote_machine",
    mount = "~/.local/share/dv",
    os = "linux"
})
print(um["cur"].user) -- 输出当前用户
print(um["remote_user"].user) -- 输出远程用户
um["remote_user"]:exec("echo hello") -- 在远程用户下执行命令
um["remote_user"]:exec("bash", true) -- 交互式执行 bash
um["remote_user"]:exec("echo hello", {reply = false, etor = "bash"}) -- 使用 bash 执行命令
um["remote_user"]:write("a.txt", "hello world") -- 写文件
print(um["remote_user"]:read("a.txt")) -- 读文件
```

Note:

- 即便设置了`os`，`dv`仍然会尝试探测用户的操作系统，并在必要时覆盖该值。目前`ssh`用户的操作系统探测只支持`linux`（且需要先设置`os`为`linux`），其他操作系统的探测为`TODO`状态。

### `Dot`

`Dot`类提供了配置文件的管理功能，主要用于加载配置文件、备份配置文件等。

注意所有配置文件相关数据均存储在此类中。

以下是`Dot`类的定义和成员函数：

```lua
---@class Dot
---@field confirm fun(this: Dot, default: string)
---@field add_schema fun(this: Dot, name: string, path: string)
---@field add_source fun(this: Dot, name: string, path: string)
---@field sync fun(this: Dot, apps: table, uid: string)
---@field upload fun(this: Dot, apps: table, uid: string)
```

`confirm`方法用于配置复制文件时的默认方式，参见[`Op`](#sync)

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

`add_schema`方法用于添加配置文件的schema
支持加载方式

- `uid` + `path`：加载指定用户的配置文件，`path`为配置文件的路径。

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
`upload`方法用于上传指定用户配置文件到`source`，`apps`参数为一个列表，每个元素为一个配置文件的名称，`uid`参数为用户的`id`。

Example:

```lua
local dot = dv:dot()
dot:add_schema("default", "path/to/schema.toml")
local schema = dv:dl("https://raw.githubusercontent.com/km0e/schema/main/dot.toml")
dot:add_schema("cur", schema)
dot:add_source("default", "path/to/source")
dot:sync({"fish", "alacritty", "fcitx5"}, "remote_user")
dot:upload({"fish", "alacritty", "fcitx5"}, "remote_user")
```

### `Pm`

`Pm`类提供了一些包管理器的功能，主要用于安装、更新软件包，实际并没有存储任何数据。

以下是`Pm`类的定义和成员函数：

```lua
---@class Pm
---@field install fun(this: Pm, hid: string, pkg: string, yes: boolean?)
---@field update fun(this: Pm, hid: string, yes: boolean?)
---@field upgrade fun(this: Pm, hid: string, yes: boolean?)
```

`install`方法用于安装软件包，`pkg`参数为软件包名称（可以是多个软件包，用空格分隔），`yes`参数为是否自动确认安装。

对包管理器的支持如下：

- `apt`（Debian/Ubuntu）
- `apk`（Alpine）
- `yay`（Arch Based Linux）
- `dnf`（Fedora）🚧
- `pacman`（Arch Based Linux）
- `paru`（Arch Based Linux）
- `winget`（Windows）🚧

Note：注意需要的是设备`hid`，而不是用户的`uid`。

Example:

```lua
dv:um():add_ssh("remote_user", {
    hid = "remote_machine",
    mount = "~/.local/share/dv",
    os = "linux"
})
local pm = dv:pm()
pm:install("remote_machine", "git neovim", true) -- 安装 git 和 vim
pm:update("remote_machine", true) -- 更新软件包
pm:upgrade("remote_machine", true) -- 升级软件包
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
