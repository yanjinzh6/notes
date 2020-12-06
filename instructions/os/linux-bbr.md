---
title: Linux 开启 BBR
date: 2020-11-29 14:00:00
tags: 'Linux'
categories:
  - ['使用说明', '操作系统']
permalink: linux-bbr
---

## 简介

BBR 是 Google 开发的一款优化网络传输的算法, 主要是针对 TCP 的, 能在拥堵的环境下减少丢包, 加速 TCP 连接

<!-- more -->

## 需求

需要内核版本为 4.9 以上并且拥有 root 权限, 内核不同的需要更换内核

## 配置

```sh
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```

## 验证

```sh
sysctl net.ipv4.tcp_available_congestion_control
net.ipv4.tcp_available_congestion_control = reno cubic bbr

sysctl net.ipv4.tcp_congestion_control
net.ipv4.tcp_congestion_control = bbr
```

打印的结果中输出 BBR 则表示已经开启

重启并查看配置是否成功生效

```sh
lsmod | grep bbr
Module                  Size  Used by
tcp_bbr                20480  2
```

Used 是会随着使用而改变的
