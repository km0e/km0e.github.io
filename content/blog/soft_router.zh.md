+++
date = 2025-02-25T11:13:51Z
description = "如何配置软路由"
draft = false
title = "软路由配置"

[extra]
toc = true

[taxonomies]
tags = ["ROUTE","CFG"]
+++
## 记配置软路由的过程

### 1. 环境

使用修改后的[`bench.sh`]({{< ref "utils/index.zh-cn.md#1.1 linux 服务器测试脚本" >}})脚本，探测到的环境如下：

{{ img(src="/images/image.png") }}

### 2. Linux配置

换源，来自[MirrorZ](https://help.mirrorz.org/debian/)

```bash
su -c "cat <<'EOF' > /etc/apt/sources.list
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware

# deb https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
# # deb-src https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
EOF"
```

- 更新源

```bash
su -c "apt update && apt upgrade -y"
```

- 安装配置无密码`sudo`

```bash
su -c "apt install sudo -y"
su -c "echo '$USER ALL=(ALL) NOPASSWD: ALL' > /etc/sudoers.d/$USER"
```

首先需要开启Linux内核的一些功能，以支持软路由的配置。

- 开启`ipv4`转发

```bash
sudo sed -i 's/^#net.ipv4.ip_forward.*$/net.ipv4.ip_forward=1/g' /etc/sysctl.conf
# 使配置生效
sudo sysctl -p
```

## 3. 配置网口

查看本机网口

```bash
ip link
```

{{ img(src="/images/image-1.png") }}

安装必要的工具

```bash
sudo apt install bridge-utils isc-dhcp-server iptables -y
```

配置网口`enp2s0`为`WAN`口，`enp3s0`，`enp4s0`，`enp5s0`为`LAN`口。并且将`enp3s0`，`enp4s0`，`enp5s0`绑定为`br0`网桥。编辑`/etc/network/interfaces`文件

```
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
auto enp2s0
iface enp2s0 inet dhcp

auto br0
iface br0 inet static
    address 10.1.0.1
    netmask 255.255.0.0
    bridge_ports enp3s0 enp4s0 enp5s0

```

配置`DHCP`服务，编辑`/etc/dhcp/dhcpd.conf`文件

```
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#

# option definitions common to all supported networks...
option domain-name "example.org";
option domain-name-servers ns1.example.org, ns2.example.org;

default-lease-time 600;
max-lease-time 7200;

# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
#authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
#log-facility local7;

# No service will be given on this subnet, but declaring it helps the
# DHCP server to understand the network topology.

#subnet 10.152.187.0 netmask 255.255.255.0 {
#}

# This is a very basic subnet declaration.

#subnet 10.254.239.0 netmask 255.255.255.224 {
#  range 10.254.239.10 10.254.239.20;
#  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
#}

# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.

#subnet 10.254.239.32 netmask 255.255.255.224 {
#  range dynamic-bootp 10.254.239.40 10.254.239.60;
#  option broadcast-address 10.254.239.31;
#  option routers rtr-239-32-1.example.org;
#}

# A slightly different configuration for an internal subnet.
#subnet 10.5.5.0 netmask 255.255.255.224 {
#  range 10.5.5.26 10.5.5.30;
#  option domain-name-servers ns1.internal.example.org;
#  option domain-name "internal.example.org";
#  option routers 10.5.5.1;
#  option broadcast-address 10.5.5.31;
#  default-lease-time 600;
#  max-lease-time 7200;
#}

# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.

#host passacaglia {
#  hardware ethernet 0:0:c0:5d:bd:95;
#  filename "vmunix.passacaglia";
#  server-name "toccata.example.com";
#}

# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.
#host fantasia {
#  hardware ethernet 08:00:07:26:c0:a5;
#  fixed-address fantasia.example.com;
#}

# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.

#class "foo" {
#  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
#}

#shared-network 224-29 {
#  subnet 10.17.224.0 netmask 255.255.255.0 {
#    option routers rtr-224.example.org;
#  }
#  subnet 10.0.29.0 netmask 255.255.255.0 {
#    option routers rtr-29.example.org;
#  }
#  pool {
#    allow members of "foo";
#    range 10.17.224.10 10.17.224.250;
#  }
#  pool {
#    deny members of "foo";
#    range 10.0.29.10 10.0.29.230;
#  }
#}

subnet 10.1.0.0 netmask 255.255.0.0 {
  range 10.1.0.2 10.1.255.255;
  option subnet-mask 255.255.255.0;#这个理论上是不需要的，但是配另一个主机的时候需要，加上也没关系
  option routers 10.1.0.1;
  option domain-name-servers 8.8.8.8;
}
```

修改`/etc/default/isc-dhcp-server`文件使`isc-dhcp-server`服务监听`br0`网桥

```
INTERFACESv4="br0"
```

配置`NAT`转发

```bash
# 清空iptables规则
sudo iptables -F 
sudo iptables -F -t nat
sudo iptables -t nat -A POSTROUTING -o enp2s0 -j MASQUERADE

sudo iptables -A FORWARD -i br0 -o enp2s0 -j ACCEPT
sudo iptables -A FORWARD -i enp2s0 -o br0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

保存`iptables`规则

```bash
sudo iptables-save > /etc/iptables/rules.v4
```

恢复时命令

```bash
sudo iptables-restore < /etc/iptables/rules.v4
```

启动服务

```bash
sudo systemctl restart networking isc-dhcp-server
```
