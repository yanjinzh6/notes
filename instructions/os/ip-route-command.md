---
title: ip-route-command
date: 2021-03-13 16:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: ip-route-command
---

https://cloud.tencent.com/developer/article/1508756


# Linux 下常用的配置 IP 及 route 的命令汇编

2019-09-182019-09-18 15:04:23 阅读 6480

Linux 下用于配置或者查看 IP 地址, 路由表的命令有很多, 本文打算将其都罗列出来, 后面想到其它的命令再一一补充.

内容有点杂乱.....

**1. 配置, 及查看 IP 地址的命令**

常用的有 ifconfig, ifcfg, ip 命令.

下面主要以常用的实例来说明其用法. 具体的参数请参考 man 帮助文档.

**1.1 ifconfig 命令:**

ifconfig -a                   # 显示所有网络接口信息 [含 down 状态的接口]

ifconfig eth0     # 显示 eth0 接口信息

ifcfg eth0 down| up           # 关闭 | 启用 eth0

ifconfig eth0 192.168.10.2/24 up    # 修改 eth0 的 IP 地址为 192.168.10.2/24[重启网络服务 i 配置即失效]

ifconfig eth0 192.168.10.2 netmask 255.255.255.0        # 设置 ip 地址, 同上一条命令效果一样

ifconfig eth0:0 192.168.20.2/24 up          # 添加虚拟网卡 eth0:0, IP 地址为 192.168.20.2/24[重启网络服务 i 配置即失效]

**1.2 ifcfg 命令:**

ifcfg 是用来取代 ifconfig 命令的, 和 ifconfig 命令功用基本一致.

ifcfg eth0:0 add 192.168.10.2/24    # 添加虚拟网卡 eth0:0, IP 地址为 192.168.10.2/24[重启网络服务 i 配置即失效]

ifcfg eth0:0 del 192.168.10.2/24

**1.3 ip 命令:**

ip link sh           # 显示的所有接口的链路状态 [down 或者 up, MTU 等信息]

ip link sh eth0  # 显示 eth0 的链路状态

ip link sh eth0  # 显示 eth0 的链路状态

ip link sh up      # 仅显示状态为 up 的接口信息

ip link set eth1 down| up           # 激活或禁止 eth1 接口

ip addr sh          # 显示网卡及 IP 等信息

ip addr sh eth1         # 显示 eth1 的 IP 信息

ip addr sh label eth1\*      # 显示匹配 label 为 eth1 的网卡信息 [支持通配符]

ip addr add 192.168.2.11/24 dev eth0 label eth0:0         # 添加虚拟网卡, 并指定网卡别名

ip addr del 192.168.2.11/24 dev eth0               # 删除虚拟网卡

**Note: 下面的几个 ip addr 命令不常用**

ip addr add 10.10.10.20/24 dev eth1 scope link

上面的 \[scope {global| link| host}\]: 指明作用域 [global: 全局可用; link: 仅链接可用; host: 本机可用;]

此外, 还可以设置 broadcast 广播地址, 如: ip addr add 10.10.20.20 broadcast 10.255.255.255 dev eth1

ip addr flush eth0     # 清空 eth0 网卡的配置

ip addr flush eth1 to 192.168.2.10/24     # 清空 eth1 上的 192.168.2.10/24

ip addr flush 的各种参数格式和 ip addr show 类似, 请参考上面.

**2. 配置及查看路由的命令**

常用的有 route, ip 命令.

下面主要以常用的实例来说明其用法. 更多的参数请参考 man 帮助文档.

**2.1 route 命令:**

route -n    # 以数字形式显示路由条目 [自动将域名解析为 IP 地址]

**添加一条主机路由**

route add -host 192.168.3.1 gw 172.16.0.1 dev eth0       

\# 添加一条到达主机 192.168.3.1 的路由, 经由 eth0 接口, , 网关为 172.16.0.1

**注意: 添加主机路由时候, 不要加掩码**

**添加一条网段路由**

route add -net 192.168.4.0**/24** gw 172.16.0.1 dev eth0  

\# 添加一条到达 192.168.4.0/24 网段的路由, 经由 eth0 接口, 网关为 172.16.0.1

**添加默认路由**

route add default gw 172.16.0.1

或者 route add -net 0.0.0.0 netmask 0.0.0.0 gw 172.16.0.1

**删除路由**

route del -host 192.168.1.3            # 删除到 192.168.1.3 的主机路由

route del -net 192.168.0.0 netmask 255.255.255.0          # 删除到 192.168.0.0/24 网段的路由

**2.2 ip 命令:**

ip route sh    # 显示本机路由表信息

常用的还有: ip route { add | del | change | append | replace | monitor } ROUTE

**添加路由**

ip route add TARGET via GW dev IFACE src SOURCE\_IP

TARGET 可以是:

主机路由: 具体 IP 地址

网络路由: NETWORK/MASK

例如:

ip route add 192.168.20.2 via  192.168.2.2 dev eth0 src 192.168.2.13      # 添加主机路由 [不加掩码]

\# 如果只有一块网卡的话, 其中的 src Source\_IP 可以不写

ip route add 192.168.20.0/24 via 192.168.2.2 dev eth0 src 192.168.2.13   # 添加网络路由 [需要加掩码]

**添加网关:**

ip route add defalt via GW dev IFACE

**删除路由:**

格式: ip route del TARGET

ip route del 192.168.20.2        # 删除主机路由 [del 后面接的是单个 IP 地址]

ip route del 192.168.20.0/24   # 删除网络路由 [del 后面接的是网段]

ip route del \[-net|-host\] target \[gw Gw\] \[netmask Nm\] \[\[dev\] If\]

**清空路由表**

ip route flush

       \[dev IFACE\]

       \[via PREFIX\]

例如:

ip route flush       # 清空所有路由表

ip route flush dev eth1      # 清空与 eth1 接口

ip route flush dev eth1 via 192.168.10.10

**补充一:**

**路由表格式:**

**Destination 列: 是远程路由地址, 可以是主机地址或者网段**

**Gateway 列:**

**GenMask 列: Destination 的掩码**

**Flags 列:**

U 表示该路由可用

G 表示该路由需要经网关转发

H 表示该行的路由为一台主机, 而非一个网段

**补充二:**

配置 IP 过程中, 一些也会用到的命令, 如:

ifup eth1       # 启用 eth1

ifdown eth1  # 禁掉 eth1

本文参与 [腾讯云自媒体分享计划](/developer/support-plan), 欢迎正在阅读的你也加入, 一起分享.

[展开阅读全文](javascript:;)

[TCP/IP](/developer/tag/10750? entry= article)

[举报](javascript:;)

点赞 2 分享

### 我来说两句

0 条评论

[登录](javascript:;) 后参与评论