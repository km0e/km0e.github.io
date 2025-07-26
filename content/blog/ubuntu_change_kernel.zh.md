+++
date = 2025-03-23T07:49:46Z
description = "更换Ubuntu内核，修改`grub`配置"
draft = false
title = "更换Ubuntu内核"

[extra]
toc = true

[taxonomies]
tags = ["CFG"]
+++

## 下载内核`deb`包

[这里](https://kernel.ubuntu.com/mainline/)可以找到指定的内核版本，下载对应的`deb`包。
一般需要下载`header`和`image`两个包。

我需要安装`5.4.284-generic`内核，需要下载四个包：

- linux-headers-5.4.284-0504284-generic_5.4.284-0504284.202409121056_amd64.deb
- linux-headers-5.4.284-0504284_5.4.284-0504284.202409121056_all.deb
- linux-modules-5.4.284-0504284-generic_5.4.284-0504284.202409121056_amd64.deb
- linux-image-unsigned-5.4.284-0504284-generic_5.4.284-0504284.202409121056_amd64.deb

## 安装`deb`包

```bash
sudo dpkg -i *.deb
```

## 修改`grub`配置

一般安装之后会默认作为启动内核，可以在开机时选择内核版本。
如果希望开机时自动选用某个内核，可以修改`/etc/default/grub`文件。

### 查看`menuentry`列表

首先运行一下命令

```bash
cat /boot/grub/grub.cfg | grep menuentry
```

可以得到类似以下输出

```bash
...

menuentry 'Ubuntu' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-26f8e534-308d-4ec1-81f1-1019aa47a25d' {
submenu 'Advanced options for Ubuntu' $menuentry_id_option 'gnulinux-advanced-26f8e534-308d-4ec1-81f1-1019aa47a25d' {
 menuentry 'Ubuntu, with Linux 5.4.284-0504284-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.4.284-0504284-generic-advanced-26f8e534-308d-4ec1-81f1-1019aa47a25d' {
 menuentry 'Ubuntu, with Linux 5.4.284-0504284-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-5.4.284-0504284-generic-recovery-26f8e534-308d-4ec1-81f1-1019aa47a25d' {
 menuentry 'Ubuntu, with Linux 4.15.0-156-generic' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-4.15.0-156-generic-advanced-26f8e534-308d-4ec1-81f1-1019aa47a25d' {
 menuentry 'Ubuntu, with Linux 4.15.0-156-generic (recovery mode)' --class ubuntu --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-4.15.0-156-generic-recovery-26f8e534-308d-4ec1-81f1-1019aa47a25d' {
```

第一个`menuentry`就是默认启动的内核，编号为`0`。第二个为`Advanced options`，编号为`1`。可以看到有四个内核可供选择。编号为`1>0`，`1>1`，`1>2`，`1>3`。

### 修改`grub`配置

打开`/etc/default/grub`文件，找到`GRUB_DEFAULT`字段，修改为对应内核编号。如：

```bash
GRUB_DEFAULT="1>0"
```

以上表示默认启动第一个内核。

### 更新`grub`

执行以下命令

```bash
sudo update-grub
```
