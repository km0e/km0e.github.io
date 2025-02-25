+++
date = 2025-02-25T10:58:17Z
description = "A simple CLI to use dv-api with toml/yaml/json"
draft = false
title = "Dv4cfg"

[extra]
toc = true

[taxonomies]
tags = []
+++

## Usage

### Config

#### Basic

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `id` | `opt` |`string`| | use to distinguish different vault | |
| `local` | `opt` |`local_device`| | local device config | |
| `ssh` | `opt` |`[ssh_device]`| | `ssh` device config | |
| `group` | `opt` |`[task_group]`| | execution unit | |
|<span id="auto_desc">`auto`</span> | `opt` |`[auto_task]`| | some daemon services or autostart applications  | |
|<span id="copy_desc">`copy`</span> | `opt` |`[copy_task]`| | copy files/directories between devices | |
|<span id="app_desc">`app`</span> | `opt` |`[app_task]`| | device package management  | |
|<span id="exec_desc">`exec`</span> | `opt` |`[exec_task]`| | exec_desc">execute commands/script | |

#### Device Config

Each device is composed of ordinary users and super users. Currently, users only have the current user and `ssh` users.

Current user (`local`) config as follows

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- | --- |
| `uid` | `opt`   | `string` |`this`|use to distinguish different user| |
| `mount` | `opt`   | `path`   |`~/.config/dv`|relative path expansion in `copy_task`| |

`ssh` user (`ssh_user`) config as follows

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `uid` | `must`   | `string` | |use to distinguish different user| |
| `host` | `must`   | `string`   | |The `Host` field in the `ssh_config` file is used to find the `ssh` configuration| |
| `passwd` | `opt`   | `string`   | |`ssh` user passwd| |

Local device config (`local_device`) as follows

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `hid` | `must`   | `string` |`local` |use to distinguish different user| |
| `user` | `must`   | `local`   | |current user config| |
| `system` | `opt`   | `ssh_user`   | |local device super user(administrator)| Temporarily reuse `ssh`, and may write a general `agent` in the future |

`ssh` device config (`ssh_device`) as follows

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `hid` | `must`   | `string` | |use to distinguish different user| |
| `os` | `opt`   | `manjaro\|alpine\|...`   | |remote device os| |
| `users` | `opt`   | `[ssh_user]`   | |`ssh` user config|  |
| `system` | `opt`   | `ssh_user`   | |remote device super user(administrator)| |

#### Basic Task

Each task has basic attributes (`task_attr`) as follows

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `id` | `must`   | `string` | |use to distinguish different user| |
| `next` | `opt`   | `[string]`   | |indicates the topological relationship between tasks| |

Currently, only the `next` relationship is supported between tasks, and more relationships may be supported in the future.

The task target (`target`) type is as follows

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `src_uid` | `opt`   | `string` | |operation source| |
| `dst_uid` | `opt`   | `string`   | |operation destination| |

Auto start service (`auto_task`)

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `attr` | `must`   | `task_attr` | |basic attributes| |
| `uid` | `opt`   | `string`   | |operation destination| |
| `name` | `must`   | `string`   | |service name|  |
| `action` | `must`   | `setup\|reload`   | |operation| |

File copy (`copy_task`)

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `attr` | `must`   | `task_attr` | |basic attributes| |
| `target` | `opt`   | `target`   | |operation destination| |
| `pair` | `must`   | `[(string,string)]`   | |the path of the file to be copied|  |

App management (`app_task`)

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `attr` | `must`   | `task_attr` | |basic attributes| |
| `uid` | `opt`   | `string`   | |operation destination| |
| `pkgs` | `must`   | `[string]`   | |apps need to be installed|  |

Execute command (`exec_task`)

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `attr` | `must`   | `task_attr` | |basic attributes| |
| `uid` | `opt`   | `string`   | |operation destination| |
| `shell` | `opt`   | `string`   | |the `shell` used to execute command|  |
| `command` | `must`   | `string`   | |commands|  |

#### Task Group

Cite (cite)

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `attr` | `must`   | `task_attr` | |basic attributes| |
| `target` | `opt`   | `target`   | |related users| |

The `id` of the basic attribute of the task to be referenced is the same as the `id` of the referenced task. You can reference task groups or individual tasks. `target` will fill the `target` of the referenced task, and `dst_uid` will fill the `uid` of the referenced task.

<!-- e.g.
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
will be replaced by
{{< code-toggle >}}
group:
  id: 'group1'
  app:
    attr:
      id: 'task1'
    uid: 'this'
    pkgs:
      - "fakeroot"
{{< /code-toggle >}} -->

Task group (`task_group`)

| name | necessity | type | default | description | todo |
| --- |---| --- |--- | --- |---|
| `id` | `must`   | `string` | |use to distinguish different user(group)| |
| `target` | `opt`   | `target`   | |related users| |
| `cites` | `opt`   | `[cite]`   | | Other tasks referenced (groups)|  |
| `auto`([see previous](#auto_desc)) |  | | | | |
| `copy`([see previous](#copy_desc)) | | | | | |
| `app`([see previous](#app_desc)) |  | | |  | |
| `exec`([see previous](#exec_desc)) |  | | |  | |

The grouped `taget` is the same as above, passed down level by level

### Command

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

- `-d`：used to read the configuration file (`config.toml/yaml/json`) and `cache.db`
- `-c`：used to specify the configuration file path, which takes precedence over the configuration file in the specified directory

```bash
Print example config

Usage: dv4cfg full-config [EXTENSION]

Arguments:
  [EXTENSION]  

Options:
  -h, --help  Print help
```

`EXTENSION`：`opt`，used to specify the configuration file format (`toml/yaml/json`)

```bash
Execute plan

Usage: dv4cfg exec [OPTIONS] [PLAN_ID]

Arguments:
  [PLAN_ID]  

Options:
  -n, --dry-run  
  -h, --help     Print help
```

- `PLAN_ID`：`opt`，used to specify the plan to be executed, currently supports task group `id`
- `-n`：`opt`，used to test the execution plan, will not actually execute
