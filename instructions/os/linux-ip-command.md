---
title: Linux ip 命令
date: 2021-03-06 11:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: linux-ip-command
---

https://zhuanlan.zhihu.com/p/28155886

# ip 命令用法归纳

[! [{已注销}](https://pic4.zhimg.com/da8e974dc_xs.jpg? source=172ae18b)](//www.zhihu.com/people/yuzenan888)

[{已注销}](//www.zhihu.com/people/yuzenan888)

5 人赞同了该文章

## **一, 前言**

ip 是 iproute2 工具包里面的一个命令行工具, 用于配置网络接口以及路由表.

iproute2 正在逐步取代旧的 net-tools(ifconfig), 所以是时候学习下 iproute2 的使用方法啦~

## **二, 接口信息查看**

### **2.1 查看接口状态和详细统计**

> 不指定接口则显示所有接口的详细统计.

```
ip -d -s -s link show [dev <接口名>]
```

**举例**

查看 `ens34` 接口信息.

```
[root: ~]# ip -d -s -s link show ens34
3: ens34: <BROADCAST, MULTICAST, UP, LOWER_UP> mtu 1500 qdisc pfifo_fast state UP mode DEFAULT qlen 1000
    link/ether 88:32:9b: ca:3f:4a brd ff: ff: ff: ff: ff: ff promiscuity 0 addrgenmode eui64
    RX: bytes  packets  errors  dropped overrun mcast
    581645     6100     0       0       0       0
    RX errors: length   crc     frame   fifo    missed
               0        0       0       0       0
    TX: bytes  packets  errors  dropped carrier collsns
    3743584    3939     0       0       0       0
    TX errors: aborted  fifo   window heartbeat transns
               0        0       0       0       2
```

## **三, IP 地址设置**

### **3.1 查看接口 IP 地址**

不指定接口则显示所有接口的 IP 地址.

```
ip addr show [dev <接口名>]
```

### **3.2 查看接口 IPv6 地址**

不指定接口则显示所有接口的 IPv6 地址.

```
ip -6 addr show [dev <接口名>]
```

### **3.3 为接口添加 IP 地址**

```
ip addr add <IP 地址 / 前缀长度> [broadcast <广播地址>] dev <接口名>
```

### **3.4 为接口添加 IPv6 地址**

```
ip -6 addr add <IPv6 地址 / 前缀长度> dev <接口名>
```

### **3.5 为接口删除 IP 地址**

```
ip addr del <IP 地址 / 前缀长度> dev <接口名>
```

### **3.6 为接口删除 IPv6 地址**

```
ip -6 addr del <IP 地址 / 前缀长度> dev <接口名>
```

举例

为 `ens34` 添加 IP 地址 `192.168.1.111/24` 并检查.

```
[root: ~]# ip addr add 192.168.1.111/24 dev ens34
3: ens34: <BROADCAST, MULTICAST, UP, LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 88:32:9b: ca:3f:4a brd ff: ff: ff: ff: ff: ff
    inet 10.16.1.2/24 brd 10.16.1.255 scope global ens34
       valid_lft forever preferred_lft forever
    inet 192.168.1.111/24 scope global ens34
       valid_lft forever preferred_lft forever
    inet6 fe80:: f65c:89ff: fecd:3ab5/64 scope link
       valid_lft forever preferred_lft forever
```

## **四, 接口设置**

### **4.1 启用接口**

```
ip link set <接口名> up
```

### **4.2 禁用接口**

```
ip link set <接口名> down
```

### **4.3 设置接口 MAC 地址**

> 设置前请先禁用接口.

```
ip link set <接口名> address <值>
```

### **4.4 设置接口 MTU**

> 设置前请先禁用接口.

```
ip link set <接口名> mtu <值>
```

举例

把 `ens33` 的 MTU 改成 `9000` 并检查.

```
[root: ~]# ip link show dev ens33 #修改前
2: ens33: <BROADCAST, MULTICAST> mtu 1500 qdisc pfifo_fast state DOWN mode DEFAULT qlen 1000
    link/ether 88:32:9b: ca:3f:49 brd ff: ff: ff: ff: ff: ff

============================================

[root: ~]# ip link set ens33 mtu 9000

[root: ~]# ip link show dev ens33  #修改后
2: ens33: <BROADCAST, MULTICAST> mtu 9000 qdisc pfifo_fast state DOWN mode DEFAULT qlen 1000
    link/ether 88:32:9b: ca:3f:49 brd ff: ff: ff: ff: ff: ff
```

## **五, VLAN 设置**

### **5.1 添加 802.1Q VLAN 子接口**

```
ip link add link <接口名> name <子接口名> type vlan id <VLAN ID>
```

### **5.2 删除 802.1Q VLAN 子接口**

```
ip link del <接口名>
```

举例

为 `ens33` 添加 `VLAN100` 子接口并检查.

```
[root: ~]# ip link add link ens33 name ens33.100 type vlan id 100

[root: ~]# ip -d -s -s link show ens33.100
7: ens33.100@ens33: <BROADCAST, MULTICAST, UP, LOWER_UP> mtu 9000 qdisc noqueue state UP mode DEFAULT qlen 1000
    link/ether 88:32:9b: ca:3f: aa brd ff: ff: ff: ff: ff: ff promiscuity 0
    vlan protocol 802.1Q id 100 <REORDER_HDR> addrgenmode eui64
    RX: bytes  packets  errors  dropped overrun mcast
    0          0        0       0       0       0
    RX errors: length   crc     frame   fifo    missed
               0        0       0       0       0
    TX: bytes  packets  errors  dropped carrier collsns
    738        9        0       0       0       0
    TX errors: aborted  fifo   window heartbeat transns
               0        0       0       0       3
```

## **六, 路由表设置**

### **6.1 查看路由表**

> 不指定接口则显示所有接口的路由表.

```
ip route show [dev <接口名>]
```

### **6.2 查看指定目标地址用的是哪条路由表**

```
ip route get <目标 IP>
```

### **6.3 添加路由表**

```
ip route add <目标 IP 地址 / 前缀长度> via <下一跳> [dev <出接口>]
```

### **6.4 添加默认网关**

```
ip route add default via <默认网关> [dev <出接口>]
```

### **6.5 删除路由表**

```
ip route del <目标 IP 地址 / 前缀长度> via <下一跳> [dev <出接口>]
```

举例

查看目标地址为 `8.8.8.8` 用的是哪条路由表.

```
[root: ~]# ip route get 8.8.8.8
8.8.8.8 via 192.168.1.1 dev ens33  src 192.168.1.143
    cache
#下一跳是 192.168.1.1, 出接口是 ens33, 接口的 IP 是 192.168.1.143.
```

## **七, ARP 设置**

### **7.1 查看 ARP 表**

> 不指定接口则显示所有接口的 ARP 表.

```
ip neigh show [dev <接口名>]
```

### **7.2 添加永久 ARP 条目**

```
ip neigh add <IP 地址> lladdr <以冒号分割的 MAC 地址> dev <接口名> nud permanent
```

### **7.3 把动态 ARP 条目转换为永久 ARP 条目 (仅限已存在条目)**

```
ip neigh change <IP 地址> dev <接口名> nud permanent
```

### **7.4 删除 ARP 条目**

```
ip neigh del <IP 地址> dev <接口名>
```

### **7.5 清空 ARP 表 (不影响永久条目)**

```
ip neigh flush all
```

编辑于 2019-05-24

[

Linux



](//www.zhihu.com/topic/19554300)

[

Linux 运维



](//www.zhihu.com/topic/19648078)

[

计算机网络



](//www.zhihu.com/topic/19572894)

​赞同 5​

​添加评论

​分享

​喜欢​收藏​申请转载

​