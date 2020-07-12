---
title: MacOS netstat 命令
date: 2020-07-05 20:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: macos-netstat-command
---

## 简介

netstat 命令在 MacOS 中使用与 Linux 中不一样, 当习惯性的输入 `netstat -nltp` 时, 会提示如下错误

```sh
netstat -nltp
netstat: option requires an argument -- p
Usage:  netstat [-AaLlnW] [-f address_family | -p protocol]
        netstat [-gilns] [-f address_family]
        netstat -i | -I interface [-w wait] [-abdgRtS]
        netstat -s [-s] [-f address_family | -p protocol] [-w wait]
        netstat -i | -I interface -s [-f address_family | -p protocol]
        netstat -m [-m]
        netstat -r [-Aaln] [-f address_family]
        netstat -rs [-s]
```

可以看到提示中 `-p` 参数需要加上 protocol 协议, 常见就是 `tcp`, `udp`, `-f` 参数需要带上地址族

<!-- more -->

## 使用方法

- 查询 tcp 端口 8080: `netstat -anvp tcp | grep 8080`
- 查询 udp 端口 7788: `netstat -anvp udp | grep 7788`
- 查询 inet: `netstat -anvf inet`

## 参考

- [MacOS 端口占用情况，其中 netstat 命令与 CentOS 略有出入](https://www.lovesofttech.com/mac/macPort.html)
