---
title: check-open-port
date: 2021-03-16 16:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: check-open-port
---

https://zh.codepre.com/debian-12138.html

[首页](/) » [Debian](https://zh.codepre.com/debian) » 如何检查 Debian 10 中的开放端口

# 如何检查 Debian 10 中的开放端口

检查端口对于查看哪些端口已打开并在系统上进行侦听非常重要. 侦听服务可能是黑客的切入点, 他们可以利用系统漏洞来访问或破坏您的系统. 我们不建议您继续运行未使用的服务. 它还消耗额外的资源. 因此, 您应该不断检查系统上的开放端口.

本文介绍了如何使用四种不同的方法检查 Debian 10 系统上的开放端口.

注意: 本文中描述的命令和过程已在 Debian 10 Buster 系统上进行了测试.

## 使用 ss 命令检查打开的端口

Linux ss(套接字统计信息) 命令提供有关网络连接的重要信息, 例如开放端口和侦听套接字. 从 Linux 内核获取此信息. 使用不带命令行参数的 ss 命令可显示有关所有当前连接的详细信息, 而不管当前连接的状态如何. ss 命令替换 netstat 命令. ss 命令与 iproute2 软件包捆绑在一起, 可以在 Debian 系统上使用. 但是, 无论哪种情况, 如果您都无法在系统上找到它, 则可以轻松安装它.

在 Debian 10 系统上打开一个终端, 然后在其中发出以下命令:

```
$ sudo apt install iproute2
```

要查看 Debian 系统上打开了哪些端口, 请在终端中发出以下命令.

```
$ sudo ss -tulpn
```

哪里:

*   **\-t,–tcp:** 显示所有 TCP 套接字
*   **\-u,–udp:** 显示所有 UDP 套接字
*   **\-l,–收听:** 显示所有监听套接字
*   **\-p, 处理:** 查看哪个进程正在使用套接字
*   **\-n,- 数字:** 如果要显示端口号而不是服务名称, 请使用此选项

输出显示所有侦听的 TCP 和 UDP 连接的列表.

上面的输出显示系统上只有端口 22 打开.

**注意**: 如果您在 ss 命令中使用 -p 或–processes 选项, 则您必须是 root 用户或具有 sudo 特权的用户. 否则, 您将看不到端口上运行的进程的进程标识号 (PID).

## 使用 netstat 命令检查打开的端口

Linux Netstat 命令提供有关当前网络连接和统计信息. Netstat 的命令选项类似于 ss 命令. 您必须安装 net-tools 才能使用 netstat 命令. 为此, 请在终端中发出以下命令.

```
$ sudo apt-get install net-tools
```

! [安装网络工具](data: image/svg+ xml; base64, PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI4NTAiIGhlaWdodD0iNDE4Ij48L3N2Zz4= " 如何检查 Debian 10 中的开放端口 ")

安装后, 您可以在 Debian Terminal 中使用 netstat 命令.

要查看在 Debian 系统上打开了哪些端口, 请在其中发出以下命令:

```
$ sudo netstat –tulnp
```

哪里:

*   **\-t,–tcp:** 显示所有 TCP 套接字
*   **\-u,–udp:** 显示所有 UDP 套接字
*   **\-l,–收听:** 显示所有监听套接字
*   **\-p, 处理:** 查看哪个进程正在使用套接字
*   **\-n,- 数字:** 如果要显示端口号而不是服务名称, 请使用此选项

! [使用 netstat 命令检查打开的端口](data: image/svg+ xml; base64, PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIxMTMzIiBoZWlnaHQ9IjI4MSI+ PC9zdmc+ " 如何检查 Debian 10 中的开放端口 ")

上面的输出显示系统上只有端口 22 打开.

**注意**: 如果您在 netstat 命令中使用 -p 或–processes 选项, 则您必须是 root 用户或具有 sudo 特权的用户. 否则, 您将看不到端口上运行的进程的进程标识号 (PID).

## 使用 lsof 命令检查打开的端口

Linux 上的 lsof 命令表示打开文件的列表 (因为 Linux 上的所有文件都是包含设备, 目录, 端口等的文件). 您可以使用 lsof 命令查找有关各种进程打开的文件的信息.

lsof 命令可以在 Debian 系统上使用. 但是, 无论哪种情况, 如果您的系统上都没有它, 则可以在终端中使用以下命令轻松地安装它:

```
$ apt-get install lsof
```

要使用 lsof 查看所有侦听的 TCP 端口, 请在终端中发出以下命令:

```
$ sudo lsof -nP -iTCP -sTCP: LISTEN
```

##

<img class="alignnone size-full wp-image-12137" src="https://static.codepre.com/uploads-zh/1598017210.png" width="709" height="120" alt=" 使用 lsof 查找开放端口 " srcset="https://static.codepre.com/uploads-zh/1598017210.png 709w, https://static.codepre.com/uploads-zh/1598017210-300x51.png 300w" sizes="(max-width: 709px) 100vw, 709px" title=" 如何检查 Debian 10 中的开放端口 ">

! [使用 lsof 查找开放端口](data: image/svg+ xml,%3Csvg%20xmlns=%22http://www.w3.org/2000/svg%22%20viewBox=%220%200%20709%20120%22%3E%3C/svg%3E " 如何检查 Debian 10 中的开放端口 ")

上面的输出显示系统上只有端口 22 打开.

## 使用 Nmap 实用程序检查打开的端口

Nmap 是用于执行系统和网络扫描的 Linux 命令行实用程序. 主要用于网络审核和安全扫描. 在 Linux 系统上, 默认情况下未安装它, 但是您可以在终端中使用以下命令来安装它:

```
$ sudo apt install nmap
```

! [使用 Nmap 查找开放端口](data: image/svg+ xml; base64, PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI3NDIiIGhlaWdodD0iMzI0Ij48L3N2Zz4= " 如何检查 Debian 10 中的开放端口 ")

当您运行上述命令时, 您可能会看到一条消息, 询问您是否要继续安装. 按 y 继续. 然后, 系统将开始安装.

安装后, 您可以使用 Nmap 查看系统上打开了哪些端口. 为此, 请在终端中发出以下命令.

```
$ sudo nmap –sT –p-65535 ip-address
```

系统 IP 地址为 192.168.72.158, 因此命令为:

```
$ sudo nmap –sT –p-65535 192.168.72.158
```

! [使用 nmap 扫描端口](data: image/svg+ xml; base64, PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI2OTkiIGhlaWdodD0iMjA5Ij48L3N2Zz4= " 如何检查 Debian 10 中的开放端口 ")

上面的输出显示系统上只有端口 22 打开.

就这样. 本文介绍了如何找出在 Debian 10 系统上打开了哪些端口. 我认为您喜欢这篇文章!

如何检查 Debian 10 中的开放端口

* * *

#### 近期更新

*   [croc 是用于在计算机之间传输可恢复的加密文件和文件夹的工具 (命令行)](https://zh.codepre.com/how-to-28387.html)
*   [如何在 CentOS 8 上安装和配置 Docker Swarm 集群](https://zh.codepre.com/centos-28385.html)
*   [croc 是用于在计算机之间传输可恢复的加密文件和文件夹的工具 (命令行)](https://zh.codepre.com/how-to-28383.html)
*   [ytfzf- 从终端搜索 (使用缩略图) 并播放 YouTube 视频](https://zh.codepre.com/how-to-28381.html)
*   [如何使用 Linux 命令创建可读的输出](https://zh.codepre.com/linux-28379.html)
*   [如何使用 Linux 命令创建可读的输出](https://zh.codepre.com/linux-28378.html)
*   [顺利遇到: 看起来很不错的待办事项列表应用程序](https://zh.codepre.com/open-source-28377.html)
*   [ytfzf- 从终端搜索 (使用缩略图) 并播放 YouTube 视频](https://zh.codepre.com/how-to-28371.html)
*   [顺利遇到: 看起来很不错的待办事项列表应用程序](https://zh.codepre.com/open-source-28369.html)
*   [ytfzf- 从终端搜索 (使用缩略图) 并播放 YouTube 视频](https://zh.codepre.com/how-to-28363.html)