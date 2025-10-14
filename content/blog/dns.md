+++
date = 2025-10-14T14:42:27Z
updated = 2025-10-14T15:32:48Z
description = "如何正确配置DNS"
draft = false
title = "Linux DNS配置"

[extra]
toc = true

[taxonomies]
tags = ["Network","Router"]
+++

## 本机DNS配置
在`Linux`系统中，DNS配置主要通过`/etc/resolv.conf`文件进行管理。该文件包含了DNS服务器的地址和搜索域等信息。

### 配置`resolv.conf`

`resolv.conf`文件的基本格式如下：

```ini
nameserver 223.5.5.5
nameserver 223.6.6.6
search example.com
```


### 配置`systemd-resolved`


主要配置文件位于`/etc/systemd/resolved.conf`，可以通过编辑该文件来配置DNS服务器。
大致格式如下

```ini
[Resolve]
# Some examples of DNS servers which may be used for DNS= and FallbackDNS=:
# Cloudflare: 1.1.1.1#cloudflare-dns.com 1.0.0.1#cloudflare-dns.com 2606:4700:4700::1111#cloudflare-dns.com 2606:4700:4700::1001#cloudflare-dns.com
# Google:     8.8.8.8#dns.google 8.8.4.4#dns.google 2001:4860:4860::8888#dns.google 2001:4860:4860::8844#dns.google
# Quad9:      9.9.9.9#dns.quad9.net 149.112.112.112#dns.quad9.net 2620:fe::fe#dns.quad9.net 2620:fe::9#dns.quad9.net
# DNS0:       193.110.81.0#dns0.eu 185.253.5.0#dns0.eu 2a0f:fc80::#dns0.eu 2a0f:fc81::#dns0.eu
#
# Using DNS= configures global DNS servers and does not suppress link-specific
# configuration. Parallel requests will be sent to per-link DNS servers
# configured automatically by systemd-networkd.service(8), NetworkManager(8), or
# similar management services, or configured manually via resolvectl(1). See
# resolved.conf(5) and systemd-resolved(8) for more details.
#DNS=
#FallbackDNS=9.9.9.9#dns.quad9.net 2620:fe::9#dns.quad9.net 1.1.1.1#cloudflare-dns.com 2606:4700:4700::1111#cloudflare-dns.com 8.8.8.8#dns.google 2001:4860:4860::8888#dns.google
#Domains=
#DNSSEC=no
#DNSOverTLS=no
#MulticastDNS=yes
#LLMNR=yes
#Cache=yes
#CacheFromLocalhost=no
#DNSStubListener=yes
#DNSStubListenerExtra=
#ReadEtcHosts=yes
#ResolveUnicastSingleLabel=no
#StaleRetentionSec=0
#RefuseRecordTypes=
```

并且`systemd-resolved`会自动生成`/etc/resolv.conf`文件，用来接管系统的DNS解析功能。
通过以下命令可以覆盖默认的`resolv.conf`文件：

```bash
sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
```

注意该文件以全局配置形式，类似于`Fallback`的方式。如果你的网络接口有单独的DNS配置（手动配置或通过`DHCP`获取），则优先于该文件。
可以通过以下命令查看当前的DNS配置：

```bash
resolvectl status
...
Link 3 (enp0s20f0u2u1)
    Current Scopes: DNS LLMNR/IPv4 LLMNR/IPv6 mDNS/IPv4 mDNS/IPv6
         Protocols: +DefaultRoute +LLMNR +mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 192.168.5.1
       DNS Servers: 192.168.5.1 fe80::62be:b4ff:fe19:2641
     Default Route: yes
...
```
类似于上面的输出，`Current DNS Server`表示当前使用的DNS服务器地址。

#### 具体问题

##### 外网局域网有自定义的`DNS zone`，即某些域名需要通过特定的`DNS`服务器解析,但是又想避免污染

先在`/etc/systemd/resolved.conf`中添加你希望使用的`DNS`服务器：

```ini
[Resolve]
DNS=223.5.5.5
Domains=~.
```

`~.`表示所有其他域名的请求会通过该`DNS`服务器进行解析。

如果网络管理器为`systemd-networkd`，则在`/etc/systemd/network/`目录下`WAN`口的配置文件中添加以下内容：

```ini
[Network]
Domains=~example.com
```
`~`表示该域名后缀的请求会通过该网络接口的`DNS`服务器进行解析。

最后重启生效。

##### 自定义的`DNS`服务器希望使用`DNS over TLS`协议，但是局域网内的`DNS`服务器不支持`DoT`协议
先时在`/etc/systemd/resolved.conf`中开启`DoT`协议：

```ini
[Resolve]
DNS=9.9.9.9#dns.quad9.net
DNSOverTLS=yes
```

`DNSOverTLS=yes`表示全局的`DNS`请求会使用`DoT`协议。

如果使用`systemd-networkd`，则在`/etc/systemd/network/`目录下`WAN`口的配置文件中添加以下内容：

```ini
[Network]
DNSOverTLS=no
```

`DNSOverTLS=no`表示该网络接口的`DNS`请求不会使用`DoT`协议。

如果使用`NetworkManager`，则可以通过`nmcli`命令配置：

```bash
sudo nmcli connection modify <connection_name> connection.dns-over-tls 0
```




