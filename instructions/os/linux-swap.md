---
title: Linux Swap 分区
date: 2020-12-06 10:00:00
tags: 'Linux'
categories:
  - ['使用说明', '操作系统']
permalink: linux-swap
---

## 简介

Swap 分区的功能

## 刷新

```sh
# 临时关闭 swap
swapoff -a
# 开启
swapon -a
# 使操作生效, 不用重启
sysctl -p
```

上面操作可以将 Swap 的数据转储到内存, 并清空 Swap

## 不使用 Swap

配置尽量不使用交换分区

```sh
echo "vm.swappiness=0" >> /etc/sysctl.conf
# 使操作生效
sysctl -p
```

## 禁用 Swap

通过注释掉 Swap 配置来禁用交换区

```sh
# 查找交换分区
blkid
/dev/vda1: UUID="xxx" TYPE="ext3" PARTUUID="zzz"
/dev/vda2: UUID="yyy" TYPE="swap" PARTUUID="aaa"
# 识别 Swap 分区
lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0     11:0    1 1024M  0 rom
vda    254:0    0   45G  0 disk
├─vda1 254:1    0 44.8G  0 part /
└─vda2 254:2    0  256M  0 part [SWAP]
```

```sh
// 注释掉 swap 分区
vi /etc/fstab
# /etc/fstab: static file system information.
#
# Use 'blkid' to print the universally unique identifier for a
# device; this may be used with UUID= as a more robust way to name devices
# that works even if disks are added and removed. See fstab(5).
#
# <file system> <mount point>   <type>  <options>       <dump>  <pass>
# / was on /dev/sda1 during installation
/dev/vda1       /               ext3    errors=remount-ro 0       1
# swap was on /dev/sda2 during installation
# /dev/vda2       none            swap    sw              0       0
```
