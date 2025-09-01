+++
date = 2025-08-21T16:42:40Z
description = "KVM 安装Ubuntu"
draft = true
title = "KVM 安装Ubuntu"

[extra]
toc = true

[taxonomies]
tags = ["NOTE"]
+++

## 相关

### `virbr0` 桥接网络配置

`virbr0` 是 libvirt 默认创建的桥接网络接口，允许虚拟机通过宿主机的网络连接到外部网络。它通常配置为 NAT 模式，这意味着虚拟机可以访问外部网络，但外部网络无法直接访问虚拟机。
配置文件位于 `/etc/libvirt/qemu/networks/default.xml`。

## 环境

Linux $host 5.15.0-138-generic #148~20.04.1-Ubuntu SMP Fri Mar 28 14:32:35 UTC 2025 x86_64 x86_64 x86_64 GNU/Linux

## 准备KVM环境

### 检查CPU虚拟化支持

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

### 安装相关软件包

```bash
sudo apt -y install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager cloud-image-utils
```

### 添加当前用户到 `libvirt` 和 `kvm` 组

```bash
sudo adduser $USER libvirt
sudo adduser $USER kvm
```

### 检查并启动 `libvirtd` 服务

```bash
sudo systemctl status libvirtd
sudo systemctl start libvirtd
kvm-ok
```

## 安装Ubuntu虚拟机

### 下载Ubuntu ISO

- [国内镜像](https://launchpad.net/ubuntu/+mirror/mirror.nyist.edu.cn-release)

```bash
wget https://cloud-images.ubuntu.com/bionic/current/bionic-server-cloudimg-amd64.img
```

### 准备硬盘

```bash
qemu-img create -F qcow2 -b ./bionic-server-cloudimg-amd64.img -f qcow2 ./np-ubuntu18.qcow2 40G
```

### 配置

创建 `user-data` 文件：

```yaml
#cloud-config
hostname: np-ubuntu18
manage_etc_hosts: true
users:
  - name: phx2
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    home: /home/phx2
    shell: /bin/bash
    lock_passwd: false
ssh_pwauth: true
disable_root: false
chpasswd:
  list: |
    vmadm:password
  expire: false
```

按实际情况修改 `hostname`、`users` 和 `chpasswd`。

创建 `meta-data` 空文件

```bash
touch meta-data
```

### 生成 seed 文件

```bash
cloud-localds -v ./np-ubuntu18-seed.qcow2 user-data meta-data
```

### 创建虚拟机

```bash
sudo virt-install --name np-ubuntu18 \
  --os-variant ubuntu18.04 \
  --vcpus 2 \
  --ram 2048 \
  --network bridge=virbr0,model=virtio \
  --disk path=np-ubuntu18-seed.qcow2,device=disk \
  --disk path=/var/lib/libvirt/images/ubuntu18.qcow2,size=40,format=qcow2,backing_store=np-ubuntu18.qcow2 \
  --import \
  --nographics none
```

```bash
# sudo virt-install --name np-ubuntu18 --os-variant ubuntu18.04 --vcpus 2 --ram 2048 --location $iso_path --network bridge=virbr0,model=virtio --graphics none --extra-args='console=ttyS0,115200n8 serial' --disk path=/var/lib/libvirt/images/ubuntu18.qcow2,size=40,format=qcow2,bus=virtio
```

- `--name`：虚拟机名称
- `--os-variant`：操作系统版本
- `--vcpus`：虚拟CPU数量
- `--ram`：内存大小（MB）
- `--location`：安装介质路径（ISO文件路径）
- `--network`：网络配置，这里使用桥接模式
- `--graphics none`：无图形界面，使用命令行安装
- `--extra-args`：传递给安装程序的额外参数，这里配置为使用串口控制台
- `--disk`：磁盘配置，这里创建一个40GB的QCOW2格式磁盘

## 完整脚本

```bash
set -e
export VM_NAME=np-ubuntu18
export UBUNTU_RELEASE=bionic
export VM_IMAGE=$UBUNTU_RELEASE-server-cloudimg-amd64.img
wget https://mirror.nyist.edu.cn/ubuntu-cloud-images/$UBUNTU_RELEASE/current/$UBUNTU_RELEASE-server-cloudimg-amd64.img
# qemu-img create -F qcow2 -b ./$VM_IMAGE -f qcow2 ./$VM_NAME.qcow2 10G
# cat >network-config <<EOF
# ethernets:
#     $INTERFACE:
#         addresses:
#         - $IP_ADDR/24
#         dhcp4: false
#         gateway4: 192.168.122.1
#         match:
#             macaddress: $MAC_ADDR
#         nameservers:
#             addresses:
#             - 1.1.1.1
#             - 8.8.8.8
#         set-name: $INTERFACE
# version: 2
# EOF

cat >user-data <<EOF
#cloud-config
manage_etc_hosts: true
users:
  - name: vmadm
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users, admin
    home: /home/vmadm
    shell: /bin/bash
    lock_passwd: false
ssh_pwauth: true
disable_root: false
chpasswd:
  list: |
    vmadm:vmadm
  expire: false
EOF

touch meta-data
# cloud-localds -v --network-config=network-config ./$VM_NAME-seed.qcow2 user-data meta-data
cloud-localds -v ./$VM_NAME-seed.qcow2 user-data meta-data

virt-install --connect qemu:///system \
  --virt-type kvm \
  --name $VM_NAME \
  --ram 1024 \
  --vcpus=2 \
  --os-type linux \
  --os-variant ubuntu18.04 \
  --disk path=/var/lib/libvirt/images/$VM_NAME.qcow2,size=40,format=qcow2,backing_store=$VM_IMAGE \
  --disk path=$VM_NAME-seed.qcow2,device=disk \
  --import \
  --network network=default,model=virtio \
  --noautoconsole
```
