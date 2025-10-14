+++
date = 2025-02-25T11:13:51Z
updated = 2025-10-14T15:24:13Z
description = "如何使用Linux系统搭建软路由"
draft = false
title = "Linux软路由配置"

[extra]
toc = true

[taxonomies]
tags = ["Network","Router"]
+++
## `systemd-networkd` + `systemd-resolved`

本节参考[Arch Manual](https://man.archlinux.org/man/systemd.network.5)。

假设我们有两个网口`enp1s0`，`enp2s0`，`enp1s0`作为`WAN`口，`enp2s0`作为`LAN`口。

多个`LAN`口可以通过桥接的方式来实现，具体配置方式根据网络管理器自行查阅。

### 基本网络配置

将以下配置写入`/etc/systemd/network/10-enp1s0.network`文件

```ini
[Match]
MACAddress=xx:xx:xx:xx:xx:xx

[Network]
DHCP=ipv4
IPForward=yes
# ipv6配置
IPv6AcceptRA=yes

[DHCP]
RouteMetric=100
UseMTU=true
```

以上配置中`MACAddress`填写`enp1s0`的`MAC`地址。
配置`enp1s0`通过`DHCP`获取`ipv4`地址，并且开启`ip`转发功能。
`ipv6`配置中开启`IPv6AcceptRA`，表示接受路由器通告。


将以下配置写入`/etc/systemd/network/20-enp2s0.network`文件

```ini
[Match]
MACAddress=xx:xx:xx:xx:xx:xx

[Network]
DHCPServer=yes
IPMasquerade=both
Address=192.168.5.1/24
# ipv6配置
Address=fd00:1::/64
IPv6SendRA=yes

[DHCPServer]
PoolOffset=3
DNS=_server_address

# 静态分配
[DHCPServerStaticLease]
MACAddress=xx:xx:xx:xx:xx:xx
Address=192.168.5.2

# ipv6配置
[IPv6SendRA]
DNS=_link_local

[IPv6Prefix]
Prefix=fd00:1::/64

[IPv6RoutePrefix]
Route=fd00:1::/64
```

以上配置中`MACAddress`填写`enp2s0`的`MAC`地址。
配置`enp2s0`开启`DHCP`服务，并且开启`ip`伪装功能。
`Address`配置`enp2s0`的`ipv4`地址为`192.168.5.1/24`。
`DHCPServer`配置中`DNS=_server_address`表示将`LAN`口的地址作为`DNS`服务器地址，需配合`DNS`服务使用。
`ipv6`配置中配置`ipv6`地址为`fd00:1::/64`，并且开启`IPv6SendRA`，表示发送路由器通告。

注意:如果以上配置不生效，检查`net.ipv4.ip_forward`和`net.ipv6.conf.all.forwarding`是否开启。具体配置方式自行查阅。

[Note - FAQ - systemd-networkd](@/faq/systemd-networkd.md)

### DNS配置

将以下配置写入（修改至）`/etc/systemd/resolved.conf`文件

```ini
...
[Resolve]
...
DNSStubListener=yes
DNSStubListenerExtra=192.168.5.1
...
```

开启`DNSStubListenerExtra`，并指定`LAN`口的地址。

更多`DNS`配置请参考[Note - DNS](@/blog/dns.md)。

### 启动服务

```bash
sudo systemctl enable --now systemd-networkd systemd-resolved
```


## 其它搭配软件

### `netplan`

本节参考[Netplan.io](https://netplan.readthedocs.io/en/stable/)。

注意`netplan`目前还不支持`DHCPServer`和`IPMasquerade`配置。
可以通过`netplan`来配置网口

```yaml
# /etc/netplan/01-netcfg.yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp1s0:
      dhcp4: true
      accept-ra: true
    enp2s0:
      addresses: 
        - 192.168.5.1/24
        - fd00:1::1/64
```

如果选择`systemd-networkd`作为渲染器，则不要在`/etc/systemd/network/`目录下配置网口，否则将会覆盖`netplan`的配置。

应用配置

```bash
sudo netplan apply
```

### `dnsmasq`

本节参考[Arch Wiki - Dnsmasq](https://wiki.archlinux.org/title/Dnsmasq)。

`dnsmasq`可以作为`DNS`和`DHCP`服务来使用。
将以下配置写入`/etc/dnsmasq.d/lan.conf`文件

```ini
interface=enp2s0

# 只启用 DHCP
port=0

listen-address=fd00:1::1,192.168.5.1

# DHCP 地址池
dhcp-range=192.168.5.3,192.168.5.255,24h

# 网关（可选）
dhcp-option=3,192.168.5.1

# DNS 服务器
dhcp-option=6,192.168.5.1

dhcp-host=98:fc:84:e5:13:6a,192.168.5.2

enable-ra

dhcp-range=fd00:1::100,fd00:1::ffff,64,12h
```

注意这里`dnsmasq`没有配置自带的`DNS`服务。

应用同时配置自动启动

```bash
sudo systemctl enable --now dnsmasq
```

## `Masquerade`配置

如果不使用`systemd-networkd`的`IPMasquerade`功能，同样需要配置`NAT`转发功能。

同时注意`net.ipv4.ip_forward`和`net.ipv6.conf.all.forwarding`需要开启。

这里可以选择用`iptables`或者`nftables`来配置`Masquerade`。
注意如果机器上同时配有其他使用`iptables`的服务如`Docker`和`TailScale`，则建议使用`iptables`,或者安装`nftables`但是用`iptables`的兼容模式。

#### `iptables`

```bash
sudo iptables -t nat -A POSTROUTING -o enp1s0 -j MASQUERADE
sudo iptables -A FORWARD -i enp2s0 -o enp1s0 -j ACCEPT
sudo iptables -A FORWARD -i enp1s0 -o enp2s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
# ipv6
sudo ip6tables -t nat -A POSTROUTING -o enp1s0 -j MASQUERADE
sudo ip6tables -A FORWARD -i enp2s0 -o enp1s0 -j ACCEPT
sudo ip6tables -A FORWARD -i enp1s0 -o enp2s0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

可以使用`iptables-persistent`来保存`iptables`规则

```bash
sudo apt install iptables-persistent
```

应该在安装过程中选择保存当前规则。没有则查看是否有`/etc/iptables/rules.v4`和`/etc/iptables/rules.v6`文件，如果没有则手动保存

```bash
sudo iptables-save > /etc/iptables/rules.v4
sudo ip6tables-save > /etc/iptables/rules.v6
```

#### `nftables`

```nft
#!/usr/sbin/nft -f
#/etc/nftables.conf
table ip nat {
    chain postrouting {
        type nat hook postrouting priority 100;
        counter oif "enp1s0" masquerade
    }
}
table ip filter {
    chain forward {
        type filter hook forward priority 0; policy drop;
        iif "enp2s0" oif "enp1s0" accept
        iif "enp1s0" oif "enp2s0" ct state related,established accept
    }
}
table ip6 nat {
    chain postrouting {
        type nat hook postrouting priority 100;
        counter oif "enp1s0" masquerade
    }
}
table ip6 filter {
    chain forward {
        type filter hook forward priority 0; policy drop;
        iif "enp2s0" oif "enp1s0" accept
        iif "enp1s0" oif "enp2s0" ct state related,established accept
    }
}
```
应用同时配置自动启动

```bash
sudo systemctl enable --now nftables
```
