---
title: linux-iproute2
date: 2021-03-21 19:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: linux-iproute2
---

https://blog.csdn.net/weixin_42767604/article/details/106251844

# iproute2 命令详解

! [](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[初心@\_@](https://tianyao.blog.csdn.net) 2020-05-21 12:00:30 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 1151 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollectionActive.png) 收藏  9

分类专栏: [\# 每天一个 linux 命令](https://blog.csdn.net/weixin_42767604/category_9879533.html) 文章标签: [linux](https://www.csdn.net/tags/MtjaQg5sMDY0MC1ibG9n.html) [运维](https://www.csdn.net/tags/MtTaEg0sMDQyNTMtYmxvZwO0O0OO0O0O.html)

版权声明: 本文为博主原创文章, 遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议, 转载请附上原文出处链接和本声明.

本文链接: [https://blog.csdn.net/weixin\_42767604/article/details/106251844](https://blog.csdn.net/weixin_42767604/article/details/106251844)

版权

### iproute2 命令详解

*   *   *   [一, 和 netstat 说再见](#netstat_5)
        *   [二, 篡权的 ss](#ss_9)
        *   [三, 被 ip 取代的命令](#ip_15)

> 博客环境说明:
> 系统版本: CentOS Linux release 7.7.1908 (Core)
> yum 源: 阿里源

### 一, 和 netstat 说再见

> [netstat 命令详解点击这里查看](https://blog.csdn.net/weixin_42767604/article/details/105745227)

! [在这里插入图片描述](https://img-blog.csdnimg.cn/2020052111443737.png? x-oss-process= image/watermark, type_ZmFuZ3poZW5naGVpdGk, shadow_10, text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mjc2NzYwNA==, size_16, color_FFFFFF, t_70)

### 二, 篡权的 ss

centos 安装 iproute2 命令:
`yum install -y iproute iproute-doc`

[点击这里查看 ss 命令详细讲解](https://blog.csdn.net/weixin_42767604/article/details/105745227)

### 三, 被 ip 取代的命令

> [ip 命令详解点击这里查看](https://blog.csdn.net/weixin_42767604/article/details/105745227)

| 功能 | 老用法 | 新用法 |
| --- | --- | --- |
| 路由表 | netstat -r/route | ip route |
| 网络接口统计信息 | netstat -i | ip -s link |
| 组播 | netstat -g | ip maddr |
| 网络接口地址和链路 | ifconfig | ip addr /ip link |
| ARP | arp | ip neigh |
| 隧道 | iptunnel | ip tunnel |

! [在这里插入图片描述](https://img-blog.csdnimg.cn/20200521104839263.png? x-oss-process= image/watermark, type_ZmFuZ3poZW5naGVpdGk, shadow_10, text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80Mjc2NzYwNA==, size_16, color_FFFFFF, t_70)
