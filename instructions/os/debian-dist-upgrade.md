---
title: Debian 系统升级
date: 2020-12-06 15:00:00
tags: 'Linux'
categories:
  - ['使用说明', '操作系统']
permalink: linux-dist-upgrade
---

## 简介

Debian 8 已经被标记为更旧的稳定版 (oldoldstable), 现在最新的文档版为 Debian 10 (buster), 该文档主要记录升级过程

注意: **升级过程最好使用 screen 等工具, 预防因为 ssh 中断导致的升级中断出现的不可预料的情况**

<!-- more -->

## 升级到 Debian 9 (stretch)

```sh
# 查看当前系统版本
uname -a
3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt9-3~deb8u1 (2015-04-24) x86_64 GNU/Linux
# 备份
cp /etc/apt/sources.list /etc/apt/sources.list.jessie
# 更新 Debian 8 到最新版本
apt-get update && apt-get upgrade
# 替换成 Debian 9 版本的源
sed -i 's/jessie/stretch/g' /etc/apt/sources.list
# 更新
apt-get update && apt-get upgrade
# 升级
apt-get dist-upgrade
# 重启
reboot
```

## 升级到 Debian 10 (buster)

操作流程是一致的, 只是需要将源中的 `stretch` 版本修改为 `buster`

```sh
# 查看当前系统版本
uname -a
Linux your-hostname 4.9.0-14-amd64 #1 SMP Debian 4.9.240-2 (2020-10-30) x86_64 GNU/Linux
# 备份
cp /etc/apt/sources.list /etc/apt/sources.list.stretch
# 更新 Debian 9 到最新版本
apt-get update && apt-get upgrade
# 替换成 Debian 10 版本的源
sed -i 's/stretch/buster/g' /etc/apt/sources.list
# 更新
apt-get update && apt-get upgrade
# 升级
apt-get dist-upgrade
# 重启
reboot
```

重启后查看系统版本

```sh
uname -a
Linux your-hostname 4.19.0-13-amd64 #1 SMP Debian 4.19.160-2 (2020-11-28) x86_64 GNU/Linux
```

9 和 10 的内核好像是一样的
