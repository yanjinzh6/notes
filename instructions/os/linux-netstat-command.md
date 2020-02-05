---
title: Linux netstat 命令
date: 2019-02-05 18:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: linux-netstat-command
---

# 简介

netstat 命令用于显示与 IP, TCP, UDP 和 ICMP 协议相关的统计数据, 一般用于检验本机各端口的网络连接情况. netstat 是在内核中访问网络及相关信息的程序, 它能提供 TCP 连接, TCP 和 UDP 监听, 进程内存管理的相关报告. 

# 常用

查看当前所有 tcp 端口

```sh
netstat -ntlp
```

查看所有 3306 端口使用情况

```sh
netstat -ntulp | grep 3306
```

<!-- more -->

# 命令详情

```sh
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]

  -a 或 –all 显示所有连线中的 Socket
  -A<网络类型>或 –<网络类型> 列出该网络类型连线中的相关地址
  -c 或 –continuous 持续列出网络状态
  -C 或 –cache 显示路由器配置的快取信息
  -e 或 –extend 显示网络其他相关信息
  -F 或 –fib 显示 FIB
  -g 或 –groups 显示多重广播功能群组组员名单
  -h 或 –help 在线帮助
  -i 或 –interfaces 显示网络界面信息表单
  -l 或 –listening 显示监控中的服务器的 Socket
  -M 或 –masquerade 显示伪装的网络连线
  -n 或 –numeric 直接使用 IP 地址, 而不通过域名服务器
  -N 或 –netlink 或 –symbolic 显示网络硬件外围设备的符号连接名称
  -o 或 –timers 显示计时器
  -p 或 –programs 显示正在使用 Socket 的程序识别码和程序名称
  -r 或 –route 显示 Routing Table
  -s 或 –statistice 显示网络工作信息统计表
  -t 或 –tcp 显示 TCP 传输协议的连线状况
  -u 或 –udp 显示 UDP 传输协议的连线状况
  -v 或 –verbose 显示指令执行过程
  -V 或 –version 显示版本信息
  -w 或 –raw 显示 RAW 传输协议的连线状况
  -x 或 –unix 此参数的效果和指定 "-A unix" 参数相同
  –ip 或 –inet 此参数的效果和指定 "-A inet" 参数相同
```

# 结果说明

- Active Internet connections, 称为有源 TCP 连接, 其中 "Recv-Q" 和 "Send-Q" 指的是接收队列和发送队列. 这些数字一般都应该是 0. 如果不是则表示软件包正在队列中堆积. 这种情况只能在非常少的情况见到. 
- Active UNIX domain sockets, 称为有源 Unix 域套接口(和网络套接字一样, 但是只能用于本机通信, 性能可以提高一倍). 

Proto 显示连接使用的协议,RefCnt 表示连接到本套接口上的进程号,Types 显示套接口的类型,State 显示套接口当前的状态,Path 表示连接到套接口的其它进程使用的路径名. 

## 套接口类型: 

- -t: TCP
- -u: UDP
- -raw: RAW 类型
- --unix: UNIX 域类型
- --ax25: AX25 类型
- --ipx: ipx 类型
- --netrom: netrom 类型

## 状态说明: 

- LISTEN: 侦听来自远方的 TCP 端口的连接请求
- SYN-SENT: 再发送连接请求后等待匹配的连接请求 (如果有大量这样的状态包, 检查是否中招了)
- SYN-RECEIVED: 再收到和发送一个连接请求后等待对方对连接请求的确认 (如有大量此状态, 估计被 flood 攻击了)
- ESTABLISHED: 代表一个打开的连接
- FIN-WAIT-1: 等待远程 TCP 连接中断请求, 或先前的连接中断请求的确认
- FIN-WAIT-2: 从远程 TCP 等待连接中断请求
- CLOSE-WAIT: 等待从本地用户发来的连接中断请求
- CLOSING: 等待远程 TCP 对连接中断的确认
- LAST-ACK: 等待原来的发向远程 TCP 的连接中断请求的确认 (不是什么好东西, 此项出现, 检查是否被攻击)
- TIME-WAIT: 等待足够的时间以确保远程 TCP 接收到连接中断请求的确认
- CLOSED: 没有任何连接状态