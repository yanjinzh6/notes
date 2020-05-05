---
title: TCP/IP 四层网络模型
date: 2020-05-02 13:00:00
tags: 'Network'
categories:
  - ['运维', 'Network']
permalink: tcp-ip-4-layer
photo:
---

## 简介

`TCP/IP` 指因特网的整个 `TCP/IP` 协议簇

从协议分层模型方面来讲, `TCP/IP` 由 4 个层次组成

- 网络接口层 (Network Access Layer) : 定义了主机间网络连通的协议, 具体包括 Echernet, FDDI, ATM 等通信协议
- 网络层 (Internet Layer) : 主要用于数据的传输, 路由及地址的解析, 以保障主机可以把数据发送给任何网络上的目标, 数据经过网络传输, 发送的顺序和到达的顺序可能发生变化, 在网络层使用 IP (Internet Protocol) 和地址解析协议 (ARP)
- 传输层 (Transport Layer) : 使源端和目的端机器上的对等实体可以基于会话相互通信, 在这一层定义了两个端到端的协议 TCP 和 UDP, TCP 是面向连接的协议, 提供可靠的报文传输和对上层应用的连接服务, 除了基本的数据传输, 它还有可靠性保证, 流量控制, 多路复用, 优先权和安全性控制等功能, UDP 是面向无连接的不可靠传输的协议, 主要用于不需要 TCP 的排序和流量控制等功能的应用程序
- 应用层 (Application Layer) : 负责具体应用层协议的定义, 包括 Telnet (TELecommunications NETwork, 虚拟终端协议) , FTP (File Transfer Protocol, 文件传输协议) , SMTP (Simple Mail Transfer Protocol, 电子邮件传输协议) , DNS (Domain Name Service, 域名服务) , NNTP (Net News Transfer Protocol, 网上新闻传输协议) 和 HTTP (HyperText Transfer Protocol, 超文本传输协议) 等
