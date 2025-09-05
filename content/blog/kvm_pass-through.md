+++
date = 2025-09-05T17:01:51Z
description = "KVM 设备直通"
draft = false
title = "KVM 设备直通" 

[extra]
toc = true

[taxonomies]
tags = ["NOTE"]
+++

## 查看直通设备

```bash
lshw -c network -businfo
...
pci@0000:5e:00.2                   network        ConnectX Family mlx5Gen Virtual Function
```

## 配置直通设备

```bash
sudo virsh attach-interface <domain> hostdev 0000:5e:00.2 --managed --live --config
```

Note:

- `--live` 选项表示应用到正在运行的虚拟机
