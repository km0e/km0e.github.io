+++
date = 2025-11-04T20:51:31Z
updated = 2025-11-04T20:51:31Z
description = "Arch Linux下使用`systemd`自动挂载SMB网络共享目录的配置方法。"
draft = false
title = "SMB客户端"

[extra]
toc = true

[taxonomies]
tags = ["Note"]
+++

## SMB服务器配置

```bash
export MOUNT_DIR=/mnt/my
export SMB_SHARE="//host/dir"
export CREDENTIALS_FILE=/etc/samba/credentials
export SMB_TIMEOUT=30
```

## 快速配置

快速为当前用户配置SMB挂载：

```bash
export SMB_UID=$(id -u your_username)
export UNIT_NAME=$(echo $MOUNT_DIR | sed 's/^\///; s/\//-/g')

sudo pacman -S cifs-utils
sudo mkdir -p $MOUNT_DIR
sudo mkdir -p $(dirname $CREDENTIALS_FILE)
sudo vim $CREDENTIALS_FILE
# 在文件中添加：
# username=your_username
# password=your_password
sudo chmod 600 $CREDENTIALS_FILE

echo "[Unit]
Description=Mount SMB Share at $MOUNT_DIR
After=network-online.target
Wants=network-online.target

[Mount]
What=$SMB_SHARE
Where=$MOUNT_DIR
Type=cifs
Options=_netdev,credentials=$CREDENTIALS_FILE,uid=$SMB_UID,iocharset=utf8,rw
Timeout=30

[Install]
WantedBy=multi-user.target" | sudo tee /etc/systemd/system/$UNIT_NAME.mount

sudo systemctl enable --now $UNIT_NAME.mount

# 可选自动挂载
sudo systemctl disable --now $UNIT_NAME.mount
echo "[Unit]
Description=Auto mount SMB Share at $MOUNT_DIR

[Automount]
Where=$MOUNT_DIR
TimeoutIdleSec=600

[Install]
WantedBy=multi-user.target" | sudo tee /etc/systemd/system/$UNIT_NAME.automount

sudo systemctl enable --now $UNIT_NAME.automount
```


## 准备
确保系统已安装`cifs-utils`包：

```bash
sudo pacman -S cifs-utils
```

## 创建挂载点
选择一个合适的位置创建挂载点，例如：
```bash
sudo mkdir -p /mnt/my
```

## 配置凭据文件
为了安全起见，建议将SMB的用户名和密码存储在一个单独的凭据文件中，例如`/etc/samba/credentials`：

```bash
sudo vim /etc/samba/credentials
```

在文件中添加以下内容：
```ini
username=your_username
password=your_password
```

设置文件权限以保护凭据：
```bash
sudo chmod 600 /etc/samba/credentials
```

## 编辑`mount`配置

使用`systemd`的挂载单元文件来配置SMB挂载。创建一个新的挂载单元文件，例如`/etc/systemd/system/mnt-my.mount`：
注意这里的路径需要将`/`替换为`-`，例如`/mnt/my`对应的单元文件名为`mnt-my.mount`。

```bash
sudo vim /etc/systemd/system/mnt-my.mount
```

在文件中添加以下内容：
```ini
[Unit]
Description=Mount SMB Share at /mnt/my
After=network-online.target
Wants=network-online.target

[Mount]
What=//host/dir
Where=/mnt/my
Type=cifs
Options=_netdev,credentials=/etc/samba/credentials,uid=1000,iocharset=utf8,rw
Timeout=30

[Install]
WantedBy=multi-user.target
```

将`//host/dir`替换为实际的SMB共享路径，`uid=1000`替换为你的用户ID。

## 启用并启动挂载
启用并启动挂载单元：
```bash
sudo systemctl enable --now mnt-my.mount
```

## 可选：自动挂载

如果希望在访问挂载点时自动挂载，可以创建一个自动挂载单元文件，例如`/etc/systemd/system/mnt-my.automount`：

```bash
sudo vim /etc/systemd/system/mnt-my.automount
```

在文件中添加以下内容：
```ini
[Unit]
Description=Auto mount SMB Share at /mnt/my

[Automount]
Where=/mnt/my
TimeoutIdleSec=600

[Install]
WantedBy=multi-user.target
```
启用并启动自动挂载单元：

```bash
# sudo systemctl disable --now mnt-my.mount # 如果之前启用了挂载单元，先禁用它
sudo systemctl enable --now mnt-my.automount
```

## 验证挂载
可以使用以下命令验证挂载是否成功：
```bash
mount | grep /mnt/my
```

如果看到相关的挂载信息，说明SMB共享已经成功挂载。

## 卸载共享
如果需要卸载SMB共享，可以使用以下命令：
```bash
sudo systemctl disable --now mnt-my.mount
# 或者如果使用了自动挂载
# sudo systemctl disable --now mnt-my.automount
```

## 问题排查

如果遇到挂载问题，可以查看`systemd`的日志以获取更多信息：
```bash
journalctl -u mnt-my.mount
# 或者
journalctl -u mnt-my.automount
```

## 参考
- [Arch Wiki - Samba](https://wiki.archlinux.org/title/Samba)










