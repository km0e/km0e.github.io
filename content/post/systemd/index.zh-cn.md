+++
title = 'Systemd User Service'
date = 2024-10-03T22:53:13+08:00
draft = false
categories = ["Linux"]
tags = ["systemd"]
+++
# 自动停止

`user`类型的`systemd`服务在`ssh``session`断开后自动停止，解决命令如下：

```bash
sudo loginctl enable-linger $USER
```

# 服务不存在

`user`类型的`systemd`服务不存在，错误如下：

```bash
Failed to connect to bus: $DBUS_SESSION_BUS_ADDRESS and $XDG_RUNTIME_DIR not defined (consider using --machine=<user>@.host --user to connect to bus of other user)
```

解决方法为修改`systemd`配置文件`/etc/ssh/sshd_config`部分如下：

```bash
...
UsePAM yes
...

```
别忘了重启`sshd`服务：

```bash
sudo systemctl restart sshd
```