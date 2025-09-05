+++
date = 2025-04-12T05:55:28Z
description = "一些`linux`下的库依赖问题"
draft = false
title = "Linux依赖FAQ"

[taxonomies]
tags = ["LINUX","FAQ"]
+++

### 动态库

#### 安装之后找不到

问题：
安装之后，运行时提示找不到动态库。

解决：
尝试刷新动态库缓存：

```bash
sudo ldconfig
```
