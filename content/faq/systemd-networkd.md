+++
date = 2025-10-13T14:05:56Z
updated = 2025-10-13T18:26:29Z
description = "Some notes about systemd-networkd"
draft = false
title = "systemd-networkd"

[extra]
toc = true

[taxonomies]
tags = ["NETWORK"]
+++

## `/etc/systemd/network` 目录下配置文件未生效

配置好网络之后重启`systemd-networkd`服务，发现网络并没有生效：
`networkctl status`显示`SETUP`为`unmanaged`。

解决方法：
- 修改`/etc/systemd/network/`目录下的配置文件权限为`644`,[参考](https://github.com/systemd/systemd/issues/34436#issuecomment-2353730549)


