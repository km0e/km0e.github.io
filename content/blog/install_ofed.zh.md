+++
date = 2025-04-07T11:02:51Z
description = "安装`OFED`"
draft = false
title = "Ubuntu 安装OFED"

[extra]
toc = true

[taxonomies]
tags = ["PRO"]
+++

## `Ubuntu18.04`

### 环境

- `Linux np01 5.4.0-150-generic #167~18.04.1-Ubuntu SMP Wed May 24 00:51:42 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux`
- `MLNX_OFED_SRC-debian-4.9-7.1.0.0.tgz`

### 安装步骤

解压压缩包之后执行：

```bash
sudo ./install.pl -vvv
```

### 遇到的问题

解决办法整合

```bash
sudo apt install python3-distutils bison flex
```

一般情况下如果安装失败，会有以下内容

```bash
...
Failed to build mlnx-ofed-kernel DEB
Collecting debug info...
See /tmp/OFED.1639.logs/mlnx-ofed-kernel.debbuild.log
```

查看`log`再加以搜索一般就可以解决，这里只列举我遇到的

#### "No module named 'distutils.core'"

```bash
...
cd ofed_scripts/utils; python3 ./setup.py install --install-layout=deb --root=../../debian/mlnx-ofed-kernel-utils
Traceback (most recent call last):
  File "./setup.py", line 28, in <module>
    from distutils.core import setup
ModuleNotFoundError: No module named 'distutils.core'
...
```

解决办法：
[原帖](https://stackoverflow.com/a/55796603)，简单来说就是安装`python3-distutils`

```bash
sudo apt install python3-distutils
```

#### "bison: not found"

```bash
...
/bin/sh: 1: bison: not found
Makefile:182: recipe for target 'emp_ematch.yacc.c' failed
...
```

解决办法：
安装`bison`

```bash
sudo apt install bison
```

#### "flex: not found"

```bash
...
/bin/sh: 1: flex: not found
Makefile:185: recipe for target 'emp_ematch.lex.c' failed
...
```

解决办法：
安装`flex`

```bash
sudo apt install flex
```

## `Ubuntu22.04`

### 环境

- `Linux np01 5.15.0-125-generic #135-Ubuntu SMP Fri Sep 27 13:53:58 UTC 2024 x86_64 x86_64 x86_64 GNU/Linux`
- `MLNX_OFED_SRC-debian-24.04-0.7.0.0.tgz`

### 安装步骤

解压压缩包之后执行：

```bash
sudo ./install.pl -vvv
```
