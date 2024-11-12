---
  title: "Dev Vault"
description: 
date: 2024-10-26T10:43:09+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: true
---
# Dev Vault
## 1. 设计
### 1.1. 存档属性
| 名称 |必要性 | 类型 | 默认值 | 描述 |
| --- |---| --- |--- | --- |
| `id` | 必须 |`string`|无 | 用于区分不同的存档 |
| `desc` |可选|`string`| `""` | 存档描述 |

### 1.2. 设备
#### 1.2.1. 基本属性
| 名称 |必要性 | 类型 | 默认值 | 描述 |
| --- |---| --- |--- | --- |
| `id` | 必须   | `string` |无|用于区分不同的设备|
| `desc` | 可选   | `string` |`""`|设备描述|
| `need_system` | 可选   | `bool`   |`false`|是否需要系统权限|
#### 1.2.2. 主机
目前除基本属性外，不需要额外属性

### 1.3. 环境（暂时无用）
| 属性 | 类型 | 描述 |
| --- | --- | --- |
| id | `[0-9a-zA-Z]*` | 用于区分不同的环境 |
| desc | `[string]` | 环境描述 |
| os | `["windows"\|"macos"\|"linux"\|"arch"\|"debian"\|"ubuntu"\|"centos"\|"fedora"\|...]` | 操作系统 |
| session | `["tty"\|"x11"\|"wayland"\|"mir"\|"unspecified"]` | 会话类型 |

### 1.4. 计划
| 名称 |必要性 | 类型 | 默认值 | 描述 |
| --- |---| --- |--- | --- |
| `id` | 必须 |`string`|无 | 用于区分不同的计划 |
| `desc` | 可选 |`string`|`string` | 计划描述 |
| `auto` | 可选 | `[auto_task]` |`[]` | 自动任务 |
| `copy` | 可选 | `[copy_task]` |`[]` | 复制任务 |

### 1.5. 任务
#### 1.5.1. 基本属性
| 名称 |必要性 | 类型 | 默认值 | 描述 |
| --- |---| --- |--- | --- |
| `id`     | 必须   | `string`   | 无       | 用于区分不同的任务                                       |
| `desc`   | 可选   | `string`   | `string` | 任务描述                                                 |
| `next` |可选|`[string]`| `[]` | 任务`id`数组，当前任务如果执行，则数组中所有任务应该执行 |
| `device` |可选|`string`| `"host"` | 执行设备`id` |
#### 1.5.2. 复制任务
| 名称 |必要性 | 类型 | 默认值 | 描述 |
| --- |---| --- |--- | --- |
| `pair` | 必须   | `[(string,string)]` | 无       | 复制源和目标的设备`id`对 |
#### 1.5.3. 自动任务
| 名称 |必要性 | 类型 | 默认值 | 描述 |
| --- |---| --- |--- | --- |
| `name` | 必须   | `string` | 无       | 自动任务的名称 |
| `action` | 必须   | `["setup"|"reload"]` | 无       | 自动任务的动作 |

## 2. 实现
### 2.1. 环境
首先编译时已经区分了一些环境，如`windows`、`linux`、`macos`，这里只说明细分的环境。

#### 2.1.1. Linux
- `os`：由[etc-os-release](https://crates.io/crates/etc-os-release)支持，具体文档参考[os-release](https://www.freedesktop.org/software/systemd/man/latest/os-release.html)
- `session`：由[pam_systemd](https://www.freedesktop.org/software/systemd/man/latest/pam_systemd.html)环境变量`XDG_SESSION_TYPE`支持，后续可能会有改变

