+++
date = 2025-02-25T10:58:33Z
description = "A simple CLI to use dv-api with toml/yaml/json"
draft = false
title = "Dv4cfg"

[extra]
toc = true

[taxonomies]
tags = []
+++
## 手册

### 配置

#### 基本属性

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定 |
| --- |---| --- |--- | --- |---|
| `id` | 可选 |`string`| | 用于区分不同的存档 | 用于区分多个配置文件|
| `local` | 可选 |`local_device`| | 本地设备配置 | |
| `ssh` | 可选 |`[ssh_device]`| | `ssh`设备配置 | |
| `group` | 可选 |`[task_group]`| | 以组为单位执行任务 | |
| `auto` | 可选 |`[auto_task]`| | 用于自启动服务管理 | |
| `copy` | 可选 |`[copy_task]`| | 用于拷贝文件 | |
| `app` | 可选 |`[app_task]`| | 用于本机应用管理（包管理） | |
| `exec` | 可选 |`[exec_task]`| | 执行命令（脚本） | |

#### 设备配置

每个设备由普通用户与超级用户组成，目前用户只有当前用户，与`ssh`用户两种。

当前用户（local）配置如下

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- | --- |
| `uid` | 可选   | `string` |`this`|用于区分用户，指代当前用户| |
| `mount` | 可选   | `path`   |`~/.config/dv`|`copy_task`中相对路径展开| |

`ssh`用户（ssh_user）配置如下

| 名称 |必要性 | 类型 | 默认值 | 描述 |待定|
| --- |---| --- |--- | --- |---|
| `uid` | 必须   | `string` | |用于区分用户，指代`ssh`用户| |
| `host` | 必须   | `string`   | |`ssh_config`文件中的`Host`，用于查找`ssh`配置| |
| `passwd` | 可选   | `string`   | |`ssh`用户密码| |

本地设备配置（local_device）如下

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- |---|
| `hid` | 必须   | `string` |`local` |用于区分设备，指代本机| |
| `user` | 必须   | `local`   | |本机用户配置| |
| `system` | 可选   | `ssh_user`   | |本机超级用户（管理员）| 暂时复用`ssh`,未来可能写一个通用`agent` |

`ssh`设备配置（ssh_device）如下

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- |---|
| `hid` | 必须   | `string` | |用于区分设备，指代远程设备| |
| `os` | 可选   | `manjaro\|alpine\|...`   | |远程设备操作系统| |
| `users` | 可选   | `[ssh_user]`   | |远程用户配置|  |
| `system` | 可选   | `ssh_user`   | |远程设备超级用户（管理员）| |

#### 基本任务配置

每个任务的基本属性（task_attr）如下

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- |---|
| `id` | 必须   | `string` | |用于区分任务| |
| `next` | 可选   | `[string]`   | |表明任务之间的拓扑关系| |

目前任务之间只支持`next`关系，未来可能支持更多关系。

任务目标（target）类型如下

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- |---|
| `src_uid` | 可选   | `string` | |操作来源| |
| `dst_uid` | 可选   | `string`   | |操作目标| |

自启动服务（auto_task）

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- |---|
| `attr` | 必须   | `task_attr` | |任务基本属性| |
| `uid` | 可选   | `string`   | |目标用户| |
| `name` | 必须   | `string`   | |任务名称|  |
| `action` | 必须   | `setup\|reload`   | |执行的动作| |

文件拷贝（copy_task）

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- |---|
| `attr` | 必须   | `task_attr` | |任务基本属性| |
| `target` | 可选   | `target`   | |操作用户| |
| `pair` | 必须   | `[(string,string)]`   | |拷贝的文件|  |

软件管理（app_task）

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- |---|
| `attr` | 必须   | `task_attr` | |任务基本属性| |
| `uid` | 可选   | `string`   | |目标用户| |
| `pkgs` | 必须   | `[string]`   | |安装的软件|  |

执行命令（exec_task）

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- |---|
| `attr` | 必须   | `task_attr` | |任务基本属性| |
| `uid` | 可选   | `string`   | |目标用户| |
| `shell` | 可选   | `string`   | |执行命令使用的`shell`|  |
| `command` | 必须   | `string`   | |执行的命令|  |

#### 任务划分

引用（cite）

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- |---|
| `attr` | 必须   | `task_attr` | |引用基本属性| |
| `target` | 可选   | `target`   | |操作用户| |

引用基本属性的`id`与被引用任务的`id`相同。可以引用任务组，也可以引用单个任务。`target`将会填充引用任务的`target`，`dst_uid`填充引用任务的`uid`。

举例：
{{< code-toggle >}}
app:
  attr:
    id: 'task1'
  pkgs:
    - "fakeroot"
group:
  id: 'group1'
  cites:
    - attr:
        id: 'task1'
      target:
        dst_uid: 'this'
{{< /code-toggle >}}
将会替换为
{{< code-toggle >}}
group:
  id: 'group1'
  app:
    attr:
      id: 'task1'
    uid: 'this'
    pkgs:
      - "fakeroot"
{{< /code-toggle >}}

任务分组（task_group）

| 名称 |必要性 | 类型 | 默认值 | 描述 | 待定|
| --- |---| --- |--- | --- |---|
| `id` | 必须   | `string` | |区分不同任务（组）| |
| `target` | 可选   | `target`   | |目标用户| |
| `cites` | 可选   | `[cite]`   | |引用的其他任务（组）|  |
| `auto` | 可选 |`[auto_task]`| | 用于自启动服务管理 | |
| `copy` | 可选 |`[copy_task]`| | 用于拷贝文件 | |
| `app` | 可选 |`[app_task]`| | 用于本机应用管理（包管理） | |
| `exec` | 可选 |`[exec_task]`| | 执行命令（脚本） | |

分组的`taget`同上，逐级向下传递。

### 命令行参数

```bash
Simple CLI to use dv-api with toml/yaml/json

Usage: dv4cfg [OPTIONS] <COMMAND>

Commands:
  full-config  Print example config [aliases: fc]
  exec         Execute plan [aliases: e]
  help         Print this message or the help of the given subcommand(s)

Options:
  -d, --directory <DIRECTORY>  [default: /home/$USER/.config/dv/]
  -c, --config <CONFIG>        
  -h, --help                   Print help
  -V, --version                Print version
```

- `-d`：用于读取配置文件（`config.toml/yaml/json`）与`cache.db`
- `-c`：用于指定配置文件路径，优先于指定目录下配置文件

```bash
Print example config

Usage: dv4cfg full-config [EXTENSION]

Arguments:
  [EXTENSION]  

Options:
  -h, --help  Print help
```

`EXTENSION`：可选，用于指定配置文件格式（`toml/yaml/json`）

```bash
Execute plan

Usage: dv4cfg exec [OPTIONS] [PLAN_ID]

Arguments:
  [PLAN_ID]  

Options:
  -n, --dry-run  
  -h, --help     Print help
```

- `PLAN_ID`：可选，用于指定执行的计划，目前支持任务组`id`
- `-n`：可选，用于测试执行计划，不会真正执行
