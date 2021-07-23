---
title: cni-calico
date: 2021-03-16 14:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: cni-calico
photo:
---

https://www.cnblogs.com/goldsunshine/p/10701242.html

# [k8s 网络之 Calico 网络](https://www.cnblogs.com/goldsunshine/p/10701242.html)

Calico 是一种容器之间互通的网络方案. 在虚拟化平台中, 比如 OpenStack, Docker 等都需要实现 workloads 之间互连, 但同时也需要对容器做隔离控制, 就像在 Internet 中的服务仅开放 80 端口, 公有云的多租户一样, 提供隔离和管控机制. 而在多数的虚拟化平台实现中, 通常都使用二层隔离技术来实现容器的网络, 这些二层的技术有一些弊端, 比如需要依赖 VLAN, bridge 和隧道等技术, 其中 bridge 带来了复杂性, vlan 隔离和 tunnel 隧道则消耗更多的资源并对物理环境有要求, 随着网络规模的增大, 整体会变得越加复杂. 我们尝试把 Host 当作 Internet 中的路由器, 同样使用 BGP 同步路由, 并使用 iptables 来做安全访问策略, 最终设计出了 Calico 方案.

k8s 网络主题系列:

[一, k8s 网络之设计与实现](https://www.cnblogs.com/goldsunshine/p/10740090.html)

[二, k8s 网络之 Flannel 网络](https://www.cnblogs.com/goldsunshine/p/10740928.html%20)

[三, k8s 网络之 Calico 网络](https://www.cnblogs.com/goldsunshine/p/10701242.html)

# 简介

Calico 是一种容器之间互通的网络方案. 在虚拟化平台中, 比如 OpenStack, Docker 等都需要实现 workloads 之间互连, 但同时也需要对容器做隔离控制, 就像在 Internet 中的服务仅开放 80 端口, 公有云的多租户一样, 提供隔离和管控机制. 而在多数的虚拟化平台实现中, 通常都使用二层隔离技术来实现容器的网络, 这些二层的技术有一些弊端, 比如需要依赖 VLAN, bridge 和隧道等技术, 其中 bridge 带来了复杂性, vlan 隔离和 tunnel 隧道则消耗更多的资源并对物理环境有要求, 随着网络规模的增大, 整体会变得越加复杂. 我们尝试把 Host 当作 Internet 中的路由器, 同样使用 BGP 同步路由, 并使用 iptables 来做安全访问策略, 最终设计出了 Calico 方案.

**适用场景**: k8s 环境中的 pod 之间需要隔离

**设计思想**: Calico 不使用隧道或 NAT 来实现转发, 而是巧妙的把所有二三层流量转换成三层流量, 并通过 host 上路由配置完成跨 Host 转发.

**设计优势**:

1. 更优的资源利用

二层网络通讯需要依赖广播消息机制, 广播消息的开销与 host 的数量呈指数级增长, Calico 使用的三层路由方法, 则完全抑制了二层广播, 减少了资源开销.

另外, 二层网络使用 VLAN 隔离技术, 天生有 4096 个规格限制, 即便可以使用 vxlan 解决, 但 vxlan 又带来了隧道开销的新问题. 而 Calico 不使用 vlan 或 vxlan 技术, 使资源利用率更高.

2. 可扩展性

Calico 使用与 Internet 类似的方案, Internet 的网络比任何数据中心都大, Calico 同样天然具有可扩展性.

3. 简单而更容易 debug

因为没有隧道, 意味着 workloads 之间路径更短更简单, 配置更少, 在 host 上更容易进行 debug 调试.

4. 更少的依赖

Calico 仅依赖三层路由可达.

5. 可适配性

Calico 较少的依赖性使它能适配所有 VM, Container, 白盒或者混合环境场景.

# Calico 架构

架构图:

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413152300545-538840176.png)

Calico 网络模型主要工作组件:

1.Felix: 运行在每一台 Host 的 agent 进程, 主要负责网络接口管理和监听, 路由, ARP 管理, ACL 管理和同步, 状态上报等.

2.etcd: 分布式键值存储, 主要负责网络元数据一致性, 确保 Calico 网络状态的准确性, 可以与 kubernetes 共用;

3.BGP Client(BIRD): Calico 为每一台 Host 部署一个 BGP Client, 使用 BIRD 实现, BIRD 是一个单独的持续发展的项目, 实现了众多动态路由协议比如 BGP, OSPF, RIP 等. 在 Calico 的角色是监听 Host 上由 Felix 注入的路由信息, 然后通过 BGP 协议广播告诉剩余 Host 节点, 从而实现网络互通.

4.BGP Route Reflector: 在大型网络规模中, 如果仅仅使用 BGP client 形成 mesh 全网互联的方案就会导致规模限制, 因为所有节点之间俩俩互联, 需要 N^2 个连接, 为了解决这个规模问题, 可以采用 BGP 的 Router Reflector 的方法, 使所有 BGP Client 仅与特定 RR 节点互联并做路由同步, 从而大大减少连接数.

**Felix**

Felix 会监听 ECTD 中心的存储, 从它获取事件, 比如说用户在这台机器上加了一个 IP, 或者是创建了一个容器等. 用户创建 pod 后, Felix 负责将其网卡, IP, MAC 都设置好, 然后在内核的路由表里面写一条, 注明这个 IP 应该到这张网卡. 同样如果用户制定了隔离策略, Felix 同样会将该策略创建到 ACL 中, 以实现隔离.

**BIRD**

BIRD 是一个标准的路由程序, 它会从内核里面获取哪一些 IP 的路由发生了变化, 然后通过标准 BGP 的路由协议扩散到整个其他的宿主机上, 让外界都知道这个 IP 在这里, 你们路由的时候得到这里来.

**架构特点**

由于 Calico 是一种纯三层的实现, 因此可以避免与二层方案相关的数据包封装的操作, 中间没有任何的 NAT, 没有任何的 overlay, 所以它的转发效率可能是所有方案中最高的, 因为它的包直接走原生 TCP/IP 的协议栈, 它的隔离也因为这个栈而变得好做. 因为 TCP/IP 的协议栈提供了一整套的防火墙的规则, 所以它可以通过 IPTABLES 的规则达到比较复杂的隔离逻辑.

# **Calico** **网络 Node 之间两种网络**

IPIP

从字面来理解, 就是把一个 IP 数据包又套在一个 IP 包里, 即把 IP 层封装到 IP 层的一个 tunnel. 它的作用其实基本上就相当于一个基于 IP 层的网桥! 一般来说, 普通的网桥是基于 mac 层的, 根本不需 IP, 而这个 ipip 则是通过两端的路由做一个 tunnel, 把两个本来不通的网络通过点对点连接起来.

BGP

边界网关协议 (Border Gateway Protocol, BGP) 是互联网上一个核心的去中心化自治路由协议. 它通过维护 IP 路由表或 ' 前缀 ' 表来实现自治系统 (AS) 之间的可达性, 属于矢量路由协议.BGP 不使用传统的内部网关协议 (IGP) 的指标, 而使用基于路径, 网络策略或规则集来决定路由. 因此, 它更适合被称为矢量性协议, 而不是路由协议.BGP, 通俗的讲就是讲接入到机房的多条线路 (如电信, 联通, 移动等) 融合为一体, 实现多线单 IP, BGP 机房的优点: 服务器只需要设置一个 IP 地址, 最佳访问路由是由网络上的骨干路由器根据路由跳数与其它技术指标来确定的, 不会占用服务器的任何系统.

# IPIP 工作模式

## 1. 测试环境

一个 msater 节点, ip 172.171.5.95, 一个 node 节点 ip 172.171.5.96

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150641464-621788389.png)

创建一个 daemonset 的应用, pod1 落在 master 节点上 ip 地址为 192.168.236.3, pod2 落在 node 节点上 ip 地址为 192.168.190.203

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150652618-320617155.png)

pod1 ping pod2

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413153642465-2094530861.png)

## 2.ping 包的旅程

pod1 上的路由信息

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150713064-1083686463.png)

根据路由信息, ping 192.168.190.203, 会匹配到第一条. 第一条路由的意思是: 去往任何网段的数据包都发往网管 169.254.1.1, 然后从 eth0 网卡发送出去.

路由表中 Flags 标志的含义:

U up 表示当前为启动状态

H host 表示该路由为一个主机, 多为达到数据包的路由

G Gateway 表示该路由是一个网关, 如果没有说明目的地是直连的

D Dynamicaly 表示该路由是重定向报文修改

M 表示该路由已被重定向报文修改

master 节点上的路由信息

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150720233-1995872444.png)

当 ping 包来到 master 节点上, 会匹配到路由 tunl0. 该路由的意思是: 去往 192.169.190.192/26 的网段的数据包都发往网关 172.171.5.96. 因为 pod1 在 5.95, pod2 在 5.96. 所以数据包就通过设备 tunl0 发往到 node 节点上.

 node 节点上路由信息

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413154329954-1165326319.png)

当 node 节点网卡收到数据包之后, 发现发往的目的 ip 为 192.168.190.203, 于是匹配到红线的路由. 该路由的意思是:192.168.190.203 是本机直连设备, 去往设备的数据包发往 caliadce112d250.

 那么该设备是什么呢? 如果到这里你能猜出来是什么, 那说明你的网络功底是不错的. 这个设备就是 veth pair 的一端. 在创建 pod2 时 calico 会给 pod2 创建一个 veth pair 设备. 一端是 pod2 的网卡, 另一端就是我们看到的 caliadce112d250. 下面我们验证一下. 在 pod2 中安装 ethtool 工具, 然后使用 ethtool -S eth0, 查看 veth pair 另一端的设备号.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413154707177-2093232618.png)

pod2 网卡另一端的设备好号是 18, 在 node 上查看编号为 18 的网络设备, 可以发现该网络设备就是 caliadce112d250.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413155040787-1729369707.png)

所以, node 上的路由, 发送 caliadce112d250 的数据其实就是发送到 pod2 的网卡中.ping 包的旅行到这里就到了目的地.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150729776-1365068043.png)

查看一下 pod2 中的路由信息, 发现该路由信息和 pod1 中是一样的.

 ! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150746241-395523835.png)

顾名思义, IPIP 网络就是将 IP 网络封装在 IP 网络里.IPIP 网络的特点是所有 pod 的数据流量都从隧道 tunl0 发送, 并且在 tunl0 这增加了一层传输层的封包.

在 master 网卡上抓包分析该过程.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190414232527057-801852995.png)

 ! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150813156-308581738.png)

打开 ICMP 285, pod1 ping pod2 的数据包, 能够看到该数据包一共 5 层, 其中 IP 所在的网络层有两个, 分别是 pod 之间的网络和主机之间的网络封装.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150820715-66735737.png)

根据数据包的封装顺序, 应该是在 pod1 ping pod2 的 ICMP 包外面多封装了一层主机之间的数据包.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190414173605263-65569449.png)

之所以要这样做是因为 tunl0 是一个隧道端点设备, 在数据到达时要加上一层封装, 便于发送到对端隧道设备中.

 两层 IP 封装的具体内容

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190413150826196-1894311200.png)

IPIP 的连接方式:

 ! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415165144848-1984358878.png)

# BGP 工作模式

## 1. 修改配置

在安装 calico 网络时, 默认安装是 IPIP 网络.calico.yaml 文件中, 将 CALICO\_IPV4POOL\_IPIP 的值修改成 "off", 就能够替换成 BGP 网络.

 ! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190414174416871-1167710181.png)

## 2. 对比

BGP 网络相比较 IPIP 网络, 最大的不同之处就是没有了隧道设备 tunl0. 前面介绍过 IPIP 网络 pod 之间的流量发送 tunl0, 然后 tunl0 发送对端设备.BGP 网络中, pod 之间的流量直接从网卡发送目的地, 减少了 tunl0 这个环节.

master 节点上路由信息. 从路由信息来看, 没有 tunl0 设备.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415113440788-275117848.png)

同样创建一个 daemonset, pod1 在 master 节点上, pod2 在 node 节点上.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415114541797-340745707.png)

## 3.ping 包之旅

pod1 ping pod2.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415114230073-496262441.png)

根据 pod1 中的路由信息, ping 包通过 eth0 网卡发送到 master 节点上.

master 节点上路由信息. 根据匹配到的 192.168.190.192 路由, 该路由的意思是: 去往网段 192.168.190.192/26 的数据包, 发送网段 172.171.5.96. 而 5.96 就是 node 节点. 所以, 该数据包直接发送了 5.96 节点.

 ! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415114439204-1886327378.png)

node 节点上的路由信息. 根据匹配到的 192.168.190.192 的路由, 数据将发送给 cali6fcd7d1702e 设备, 该设备和上面分析的是一样, 为 pod2 的 veth pair 的一端. 数据就直接发送给 pod2 的网卡.

 ! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415115008617-409053518.png)

当 pod2 对 ping 包做出回应之后, 数据到达 node 节点上, 匹配到 192.168.236.0 的路由, 该路由说的是: 去往网段 192.168.236.0/26 的数据, 发送给网关 172.171.5.95. 数据包就直接通过网卡 ens160, 发送到 master 节点上.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415115329941-1036275159.png)

通过在 master 节点上抓包, 查看经过的流量, 筛选出 ICMP, 找到 pod1 ping pod2 的数据包.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415115718930-42251257.png)

可以看到 BGP 网络下, 没有使用 IPIP 模式, 数据包是正常的封装.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415113605760-703346562.png)

值得注意的是 mac 地址的封装.192.168.236.0 是 pod1 的 ip,192.168.190.198 是 pod2 的 ip. 而源 mac 地址是 master 节点网卡的 mac, 目的 mac 是 node 节点的网卡的 mac. 这说明, 在 master 节点的路由接收到数据, 重新构建数据包时, 使用 arp 请求, 将 node 节点的 mac 拿到, 然后封装到数据链路层.

! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415113611124-513029729.png)

BGP 的连接方式:

 ! [](https://img2018.cnblogs.com/blog/1060878/201904/1060878-20190415165320714-135136611.png)

# **两种网络对比**

**IPIP 网络**:

流量: tunlo 设备封装数据, 形成隧道, 承载流量.

适用网络类型: 适用于互相访问的 pod 不在同一个网段中, 跨网段访问的场景. 外层封装的 ip 能够解决跨网段的路由问题.

效率: 流量需要 tunl0 设备封装, 效率略低

**BGP 网络**:

流量: 使用路由信息导向流量

适用网络类型: 适用于互相访问的 pod 在同一个网段, 适用于大型网络.

效率: 原生 hostGW, 效率高

# 存在问题 

(1) 缺点租户隔离问题

Calico 的三层方案是直接在 host 上进行路由寻址, 那么对于多租户如果使用同一个 CIDR 网络就面临着地址冲突的问题.

(2) 路由规模问题

通过路由规则可以看出, 路由规模和 pod 分布有关, 如果 pod 离散分布在 host 集群中, 势必会产生较多的路由项.

(3) iptables 规则规模问题

1 台 Host 上可能虚拟化十几或几十个容器实例, 过多的 iptables 规则造成复杂性和不可调试性, 同时也存在性能损耗.

(4) 跨子网时的网关路由问题

当对端网络不为二层可达时, 需要通过三层路由机时, 需要网关支持自定义路由配置, 即 pod 的目的地址为本网段的网关地址, 再由网关进行跨三层转发.

**部分数据或文字引用至 http://ju.outofmemory.cn/entry/367749**

posted @ 2019-04-16 10:08  [金色旭光](https://www.cnblogs.com/goldsunshine/)  阅读 (28164)  评论 (16)  [编辑](https://i.cnblogs.com/EditPosts.aspx? postid=10701242)  [收藏](javascript: void(0))

https://www.cnblogs.com/ssgeek/p/13194687.html

# [Kubernetes 网络之 Calico](https://www.cnblogs.com/ssgeek/p/13194687.html)

目录

- [k8s 网络之 Calico 网络](#k8s-网络之-calico-网络)
- [简介](#简介)
- [Calico 架构](#calico-架构)
- [**Calico** **网络 Node 之间两种网络**](#calico-网络-node-之间两种网络)
- [IPIP 工作模式](#ipip-工作模式)
  - [1. 测试环境](#1-测试环境)
  - [2.ping 包的旅程](#2ping-包的旅程)
- [BGP 工作模式](#bgp-工作模式)
  - [1. 修改配置](#1-修改配置)
  - [2. 对比](#2-对比)
  - [3.ping 包之旅](#3ping-包之旅)
- [**两种网络对比**](#两种网络对比)
- [存在问题](#存在问题)
- [Kubernetes 网络之 Calico](#kubernetes-网络之-calico)
  - [1, Calico 概述](#1-calico-概述)
  - [2, Calico 架构及 BGP 实现](#2-calico-架构及-bgp-实现)
  - [3, Calico 部署](#3-calico-部署)
  - [4, Calico 管理工具](#4-calico-管理工具)
  - [5, Calico BGP 模式](#5-calico-bgp-模式)
  - [6, Calico Route Reflector 模式 (RR)](#6-calico-route-reflector-模式-rr)
  - [7, Calico IPIP 模式](#7-calico-ipip-模式)
  - [8, Calico 网络策略](#8-calico-网络策略)
- [Calico 介绍, 原理与使用](#calico-介绍-原理与使用)
  - [**什么是 Calico ?**](#什么是-calico-)
  - [**Calico 组件概述**](#calico-组件概述)
  - [**Calico 网络模式**](#calico-网络模式)
  - [**BGP 概述**](#bgp-概述)
  - [**BGP 两种模式**](#bgp-两种模式)
    - [**全互联模式 (node-to-node mesh)**](#全互联模式-node-to-node-mesh)
    - [**路由反射模式 Router Reflection(RR)**](#路由反射模式-router-reflectionrr)
  - [**Calico BGP 概述**](#calico-bgp-概述)
    - [**BGP 是怎么工作的?**](#bgp-是怎么工作的)
    - [**为什么叫边界网关协议呢?**](#为什么叫边界网关协议呢)
    - [**Pod 1 访问 Pod 2 流程如下**](#pod-1-访问-pod-2-流程如下)
  - [**Route Reflector 模式 (RR) (路由反射)**](#route-reflector-模式-rr-路由反射)
  - [**IPIP 模式概述**](#ipip-模式概述)
    - [**Pod1 访问 Pod2 流程如下:**](#pod1-访问-pod2-流程如下)
    - [**实际查看路由情况**](#实际查看路由情况)
  - [**Calico 优势 与 劣势**](#calico-优势-与-劣势)
    - [**优势**](#优势)
    - [**劣势**](#劣势)
  - [**Calico 管理工具**](#calico-管理工具)
  - [**CNI 网络方案优缺点及最终选择**](#cni-网络方案优缺点及最终选择)
  - [**参考链接**](#参考链接)
    - [我来说两句](#我来说两句)

## 1, Calico 概述

`Calico` 是 `Kubernetes` 生态系统中另一种流行的网络选择. 虽然 `Flannel` 被公认为是最简单的选择, 但 `Calico` 以其性能, 灵活性而闻名.`Calico` 的功能更为全面, 不仅提供主机和 `pod` 之间的网络连接, 还涉及网络安全和管理.`Calico CNI` 插件在 `CNI` 框架内封装了 `Calico` 的功能.

`Calico` 是一个基于 `BGP` 的纯三层的网络方案, 与 `OpenStack`,`Kubernetes`,`AWS`,`GCE` 等云平台都能够良好地集成.`Calico` 在每个计算节点都利用 `Linux Kernel` 实现了一个高效的虚拟路由器 `vRouter` 来负责数据转发. 每个 `vRouter` 都通过 `BGP1` 协议把在本节点上运行的容器的路由信息向整个 `Calico` 网络广播, 并自动设置到达其他节点的路由转发规则.`Calico` 保证所有容器之间的数据流量都是通过 `IP` 路由的方式完成互联互通的.`Calico` 节点组网时可以直接利用数据中心的网络结构 (L2 或者 L3), 不需要额外的 `NAT`, 隧道或者 `Overlay Network`, 没有额外的封包解包, 能够节约 `CPU` 运算, 提高网络效率.

! [](https://image.ssgeek.com/20200626-01.png)

`Calico` 在小规模集群中可以直接互联, 在大规模集群中可以通过额外的 `BGP route reflector` 来完成.

! [](https://image.ssgeek.com/20200626-02.png)

此外,`Calico` 基于 `iptables` 还提供了丰富的网络策略, 实现了 `Kubernetes` 的 `Network Policy` 策略, 提供容器间网络可达性限制的功能.

## 2, Calico 架构及 BGP 实现

`BGP` 是互联网上一个核心的去中心化自治路由协议, 它通过维护 `IP` 路由表或 " 前缀 " 表来实现自治系统 `AS` 之间的可达性, 属于矢量路由协议. 不过, 考虑到并非所有的网络都能支持 `BGP`, 以及 Calico 控制平面的设计要求物理网络必须是二层网络, 以确保 `vRouter` 间均直接可达, 路由不能够将物理设备当作下一跳等原因, 为了支持三层网络,`Calico` 还推出了 `IP-in-IP` 叠加的模型, 它也使用 Overlay 的方式来传输数据.`IPIP` 的包头非常小, 而且也是内置在内核中, 因此理论上它的速度要比 `VxLAN` 快一点 , 但安全性更差.`Calico 3.x` 的默认配置使用的是 `IPIP` 类型的传输方案而非 `BGP`.

`Calico` 的系统架构如图所示

! [](https://image.ssgeek.com/20200626-03.png)

`Calico` 主要由 `Felix`,`Orchestrator Plugin`,`etcd`,`BIRD` 和 `BGP Router Reflector` 等组件组成.

*   `Felix`: Calico Agent, 运行于每个节点.
*   `Orchestrator Plugi`: 编排系统 (如 Kubernetes , OpenStack 等) 以将 `Calico` 整合进系统中的插件, 例如 `Kubernetes` 的 `CNI`.
*   `etcd`: 持久存储 `Calico` 数据的存储管理系统.
*   `BIRD`: 用于分发路由信息的 `BGP` 客户端.
*   `BGP Route Reflector`: `BGP` 路由反射器, 可选组件, 用于较大规模的网络场景.

## 3, Calico 部署

在 `Kubernetes` 中部署 `Calico` 的主要步骤如下:

*   修改 `Kubernetes` 服务的启动参数, 并重启服务

    *   设置 Master 上 kube-apiserver 服务的启动参数:--allowprivileged= true(因为 calico-node 需要以特权模式运行在各 Node 上).
    *   设置各 Node 上 kubelet 服务的启动参数:--networkplugin= cni(使用 CNI 网络插件)
*   创建 `Calico` 服务, 主要包括 calico-node 和 calico policy controller. 需要创建的资源对象如下

    *   创建 ConfigMap calico-config, 包含 Calico 所需的配置参数
    *   创建 Secret calico-etcd-secrets, 用于使用 TLS 方式连接 etcd.
    *   在每个 Node 上都运行 calico/node 容器, 部署为 DaemonSet
    *   在每个 Node 上都安装 Calico(由 install-cni 容器完成)
    *   部署一个名为 calico/kube-policy-controller 的 Deployment, 以对 接 Kubernetes 集群中为 Pod 设置的 Network Policy

具体部署的步骤如下
下载 yaml

```
curl https://docs.projectcalico.org/v3.11/manifests/calico-etcd.yaml -o calico-etcd.yaml
```

下载完后修改配置项

*   配置连接 etcd 地址, 如果使用 https, 还需要配置证书.(ConfigMap, Secret)

```
# cat /opt/etcd/ssl/ca.pem | base64 -w 0
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURlakNDQW1LZ0F3SUJBZ0lVRHQrZ21iYnhzWmoxRGNrbGl3K240MkI5YW5Nd0RRWUpLb1pJaHZjTkFRRUwKQlFBd1F6RUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdUQjBKbGFXcHBibWN4RURBT0JnTlZCQWNUQjBKbAphV3BwYm1jeEVEQU9CZ05WQkFNVEIyVjBZMlFnUTBFd0hoY05NVGt4TWpBeE1UQXdNREF3V2hjTk1qUXhNVEk1Ck1UQXdNREF3V2pCRE1Rc3dDUVlEVlFRR0V3SkRUakVRTUE0R0ExVUVDQk1IUW1WcGFtbHVaekVRTUE0R0ExVUUKQnhNSFFtVnBhbWx1WnpFUU1BNEdBMVVFQXhNSFpYUmpaQ0JEUVRDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRApnZ0VQQURDQ0FRb0NnZ0VCQUtEaGFsNHFaVG5DUE0ra3hvN3pYT2ZRZEFheGo2R3JVSWFwOGd4MTR4dFhRcnhrCmR0ZmVvUXh0UG5EbDdVdG1ZUkUza2xlYXdDOVhxM0hPZ3J1YkRuQ2ZMRnJZV05DUjFkeG1KZkNFdXU0YmZKeE4KVHNETVF1aUlxcnZ2aVN3QnQ3ZHUzczVTbEJUc2NOV0Y4TWNBMkNLTkVRbzR2Snp5RFZXRTlGTm1kdC8wOEV3UwpmZVNPRmpRV3BWWnprQW1Fc0VRaldtYUVHZjcyUXZvbmRNM2Raejl5M2x0UTgrWnJxOGdaZHRBeWpXQmdrZHB1ClVXZ2NaUTBZWmQ2Q2p4YWUwVzBqVkt5RER4bGlSQ3pLcUFiaUNucW9XYW1DVDR3RUdNU2o0Q0JiYTkwVXc3cTgKajVyekFIVVdMK0dnM2dzdndQcXFnL2JmMTR2TzQ2clRkR1g0Q2hzQ0F3RUFBYU5tTUdRd0RnWURWUjBQQVFILwpCQVFEQWdFR01CSUdBMVVkRXdFQi93UUlNQVlCQWY4Q0FRSXdIUVlEVlIwT0JCWUVGRFJTakhxMm0wVWVFM0JmCks2bDZJUUpPU2Vzck1COEdBMVVkSXdRWU1CYUFGRFJTakhxMm0wVWVFM0JmSzZsNklRSk9TZXNyTUEwR0NTcUcKU0liM0RRRUJDd1VBQTRJQkFRQUsyZXhBY2VhUndIRU9rQXkxbUsyWlhad1Q1ZC9jRXFFMmZCTmROTXpFeFJSbApnZDV0aGwvYlBKWHRSeWt0aEFUdVB2dzBjWVFPM1gwK09QUGJkOHl6dzRsZk5Ka1FBaUlvRUJUZEQvZWdmODFPCmxZOCtrRFhxZ1FZdFZLQm9HSGt5K2xRNEw4UUdOVEdaeWIvU3J5N0g3VXVDcTN0UmhzR2E4WGQ2YTNIeHJKYUsKTWZna1ZsNDA0bW83QXlWUHl0eHMrNmpLWCtJSmd3a2dHcG9DOXA2cDMyZDI1Q0NJelEweDRiZCtqejQzNXY1VApvRldBUmcySGdiTTR0aHdhRm1VRDcrbHdqVHpMczMreFN3Tys0S3Bmc2tScTR5dEEydUdNRDRqUTd0bnpoNi8wCkhQRkx6N0FGazRHRXoxaTNmMEtVTThEUlhwS0JKUXZNYzk4a3IrK24KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
# cat /opt/etcd/ssl/server-key.pem | base64 -w 0
LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBdUZJdlNWNndHTTROZURqZktSWDgzWEVTWHhDVERvSVdpcFVJQmVTL3JnTHBFek10Cmk3enp6SjRyUGplbElJZ2ZRdVJHMHdJVXNzN3FDOVhpa3JGcEdnNXp5d2dMNmREZE9KNkUxcUIrUWprbk85ZzgKalRaenc3cWwxSitVOExNZ0k3TWNCU2VtWVo0TUFSTmd6Z09xd2x2WkVBMnUvNFh5azdBdUJYWFgrUTI2SitLVApYVEJ4MnBHOXoxdnVmU0xzMG52YzdKY2gxT2lLZ2R2UHBqSktPQjNMTm83ZnJXMWlaenprTExWSjlEV3U1NFdMCk4rVE5GZWZCK1lBTTdlSHVHTjdHSTJKdW1YL3hKczc3dnQzRjF2VStYSitzVTZ3cmRGMStULzVyamNUN1dDdmgKbkZVWlBxTk9NWUZTWUdlOVBIY1l0Y1Y1MENnUVV2NUx2OTE1elFJREFRQUJBb0lCQUY1YkRBUG1LZ1Y0cmVLRwpVbzhJeDNwZ29NUHppeVJaS2NybGdjYnFrOGt6aWpjZThzamZBSHNWMlJNdmp5TjVLMitseGkvTWwrWDFFRkRnCnUreldUdlJjdzZBQ3pYNXpRbHZ5b2hQdzh0Rlp5cURURUNSRjVMc2t1REdCUTlCNEVoTFVaSnFxOG54MFdMYlEKUWJVVW9YeC9ZajNhazJRUklOM0R5YnRYMlNpUHBPN1hVMmFiVkNzYkZBWW1uN2lweW16M25WWFRseDJuVk1sZQpmYzhXbERsd09pL3FJUThwZjNpRnowRDVoUGl5ZDY5eXp2b2ZrVk5CbCtodGFPbGdwdVNqSEFrNnhIcFpBUExTCkIxclVJaDk1RWozTUk5U3BuSnNWcUFFVHFSSmpYOHl3bFZYa2dvd3I2TXJuTnVXelRZUnlSNDY5UFVmKzhaSzQKUjE1WTdvMENnWUVBM21HdjErSmRuSkVrL1R4M2ZaMkFmOWJIazJ1dE5FemxUakN5YlZtYkxKamx0M1pjSG96UgphZVR2azJSQ0Q4VDU0NU9EWmIzS3Zxdzg2TXkxQW9lWmpqV3pTR1VIVHZJYTRDQ3lMenBXaVNaQkRHSE9KbDBtCk9nbnRRclFPK0UwZjNXOHZtbkp0NGoySGxMWHByL1R6Zk12R05lTWVkSUlIMC8xZXV0WkJjNnNDZ1lFQTFDK0gKaDVtQ0pnbllNcm5zK3dZZ2lWVTJFdjZmRUc2VGl2QU5XUUlldVpHcDRoclZISXc0UTV3SHhZNWgrNE15bXFORAprMmVDYU15RjFxb1NCS1hOckFZS3RtWCtxR3ltaVBpWlRJWEltZlppcENocWl3dm1udjMxbWE1Njk2NkZ6SjdaCjJTLzZkTGtweWI2OTUxRTl5azRBOEYzNVdQRlY4M01DanJ1bjBHY0NnWUFoOFVFWXIybGdZMXNFOS95NUJKZy8KYXZYdFQyc1JaNGM4WnZ4azZsOWY4RHBueFQ0TVA2d2JBS0Y4bXJubWxFY2I4RUVHLzIvNXFHcG5rZzh5d3FXeQphZ25pUytGUXNHMWZ0ajNjTFloVnlLdjNDdHFmU21weVExK2VaY00vTE81bkt2aFdGNDhrRUFZb3NaZG9qdmUzCkhaYzBWR1VxblVvNmxocW1ZOXQ3bndLQmdGbFFVRm9Sa2FqMVI5M0NTVEE0bWdWMHFyaEFHVEJQZXlkbWVCZloKUHBtWjZNcFZ4UktwS3gyNlZjTWdkYm5xdGFoRnhMSU5SZVZiQVpNa0wwVnBqVE0xcjlpckFoQmUrNUo0SWY4Rgo2VFIxYzN2cHp6OE1HVjBmUlB3Vlo0bE9HdC9RbFo1SUJjS1FGampuWXdRMVBDOGx1bHR6RXZ3UFNjQ1p6cC9KCitZOU5Bb0dBVkpybjl4QmZhYWF5T3BlMHFTTjNGVzRlaEZyaTFaRUYvVHZqS3lnMnJGUng3VDRhY1dNWWdSK20KL2RsYU9CRFN6bjNheVVXNlBwSnN1UTBFanpIajFNSVFtV3JOQXNGSVJiN0Z6YzdhaVQzMFZmNFFYUmMwQUloLwpXNHk0OW5wNWNDWUZ5SXRSWEhXMUk5bkZPSjViQjF2b1pYTWNMK1dyMVZVa2FuVlIvNEE9Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg==
# cat /opt/etcd/ssl/server.pem | base64 -w 0
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURyekNDQXBlZ0F3SUJBZ0lVZUFZTHdLMkxVdnE0V2ZiSG92cTlzVS8rWlJ3d0RRWUpLb1pJaHZjTkFRRUwKQlFBd1F6RUxNQWtHQTFVRUJoTUNRMDR4RURBT0JnTlZCQWdUQjBKbGFXcHBibWN4RURBT0JnTlZCQWNUQjBKbAphV3BwYm1jeEVEQU9CZ05WQkFNVEIyVjBZMlFnUTBFd0hoY05NVGt4TWpBeE1UQXdNREF3V2hjTk1qa3hNVEk0Ck1UQXdNREF3V2pCQU1Rc3dDUVlEVlFRR0V3SkRUakVRTUE0R0ExVUVDQk1IUW1WcFNtbHVaekVRTUE0R0ExVUUKQnhNSFFtVnBTbWx1WnpFTk1Bc0dBMVVFQXhNRVpYUmpaRENDQVNJd0RRWUpLb1pJaHZjTkFRRUJCUUFEZ2dFUApBRENDQVFvQ2dnRUJBTGhTTDBsZXNCak9EWGc0M3lrVi9OMXhFbDhRa3c2Q0ZvcVZDQVhrdjY0QzZSTXpMWXU4Cjg4eWVLejQzcFNDSUgwTGtSdE1DRkxMTzZndlY0cEt4YVJvT2M4c0lDK25RM1RpZWhOYWdma0k1Snp2WVBJMDIKYzhPNnBkU2ZsUEN6SUNPekhBVW5wbUdlREFFVFlNNERxc0piMlJBTnJ2K0Y4cE93TGdWMTEva051aWZpazEwdwpjZHFSdmM5YjduMGk3Tko3M095WElkVG9pb0hiejZZeVNqZ2R5emFPMzYxdFltYzg1Q3kxU2ZRMXJ1ZUZpemZrCnpSWG53Zm1BRE8zaDdoamV4aU5pYnBsLzhTYk8rNzdkeGRiMVBseWZyRk9zSzNSZGZrLythNDNFKzFncjRaeFYKR1Q2alRqR0JVbUJudlR4M0dMWEZlZEFvRUZMK1M3L2RlYzBDQXdFQUFhT0JuVENCbWpBT0JnTlZIUThCQWY4RQpCQU1DQmFBd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUdDQ3NHQVFVRkJ3TUNNQXdHQTFVZEV3RUIvd1FDCk1BQXdIUVlEVlIwT0JCWUVGTHNKb2pPRUZGcGVEdEhFSTBZOEZIUjQvV0c4TUI4R0ExVWRJd1FZTUJhQUZEUlMKakhxMm0wVWVFM0JmSzZsNklRSk9TZXNyTUJzR0ExVWRFUVFVTUJLSEJNQ29BajJIQk1Db0FqNkhCTUNvQWo4dwpEUVlKS29aSWh2Y05BUUVMQlFBRGdnRUJBQzVOSlh6QTQvTStFRjFHNXBsc2luSC9sTjlWWDlqK1FHdU0wRWZrCjhmQnh3bmV1ZzNBM2l4OGxXQkhZTCtCZ0VySWNsc21ZVXpJWFJXd0h4ZklKV2x1Ukx5NEk3OHB4bDBaVTZWUTYKalFiQVI2YzhrK0FhbGxBTUJUTkphY3lTWkV4MVp2c3BVTUJUU0l3bmk5RFFDUDJIQStDNG5mdHEwMGRvckQwcgp5OXVDZ3dnSDFrOG42TkdSZ0lJbVl6dFlZZmZHbEQ3R3lybEM1N0plSkFFbElUaElEMks1Y090M2dUb2JiNk5oCk9pSWpNWVAwYzRKL1FTTHNMNjZZRTh5YnhvZ2M2L3JTYzBIblladkNZbXc5MFZxY05oU3hkT2liaWtPUy9SdDAKZHVRSnU3cmdkM3pldys3Y05CaTIwTFZrbzc3dDNRZWRZK0c3dUxVZ21qNWJudkU9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
```

将上述 `base64` 加密的字符串修改至文件中声明:`ca.pem` 对应 `etcd-ca`,`server-key.pem` 对应 `etcd-key`,`server.pem` 对应 `etcd-cert`; 修改 `etcd` 证书的位置; 修改 `etcd` 的连接地址 (与 api-server 中配置 /opt/kubernetes/cfg/kube-apiserver.conf 中相同)

```
# vim calico-etcd.yaml
...
apiVersion: v1
kind: Secret
type: Opaque
metadata:
  name: calico-etcd-secrets
  namespace: kube-system
data:
  # Populate the following with etcd TLS configuration if desired, but leave blank if
  # not using TLS for etcd.
  # The keys below should be uncommented and the values populated with the base64
  # encoded contents of each file that would be associated with the TLS data.
  # Example command for encoding a file contents: cat <file> | base64 -w 0
  etcd-key: 填写上面的加密字符串
  etcd-cert: 填写上面的加密字符串
  etcd-ca: 填写上面的加密字符串
...
kind: ConfigMap
apiVersion: v1
metadata:
  name: calico-config
  namespace: kube-system
data:
  # Configure this with the location of your etcd cluster.
  etcd_endpoints: "https://192.168.2.61:2379, https://192.168.2.62:2379, https://192.168.2.63:2379"
  # If you're using TLS enabled etcd uncomment the following.
  # You must also populate the Secret below with these files.
  etcd_ca: "/calico-secrets/etcd-ca"
  etcd_cert: "/calico-secrets/etcd-cert"
  etcd_key: "/calico-secrets/etcd-key"
```

根据实际网络规划修改 Pod CIDR(CALICO\_IPV4POOL\_CIDR), 与 controller-manager 配置 /opt/kubernetes/cfg/kube-controller-manager.conf 中相同

```
# vim calico-etcd.yaml
...
320             - name: CALICO_IPV4POOL_CIDR
321               value: "10.244.0.0/16"
...
```

选择工作模式 (CALICO\_IPV4POOL\_IPIP), 支持 BGP, IPIP, 此处先关闭 IPIP 模式

```
# vim calico-etcd.yaml
...
309             - name: CALICO_IPV4POOL_IPIP
310               value: "Never"
...
```

修改完后应用清单

```
# kubectl apply -f calico-etcd.yaml
secret/calico-etcd-secrets created
configmap/calico-config created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
# kubectl get pods -n kube-system
```

如果事先部署了 `fannel` 网络组件, 需要先卸载和删除 `flannel`, 在每个节点均需要操作

```
# kubectl delete -f kube-flannel.yaml
# ip link delete cni0
# ip link delete flannel.1
# ip route
default via 192.168.2.2 dev eth0
10.244.1.0/24 via 192.168.2.63 dev eth0
10.244.2.0/24 via 192.168.2.62 dev eth0
169.254.0.0/16 dev eth0 scope link metric 1002
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
192.168.2.0/24 dev eth0 proto kernel scope link src 192.168.2.61
# ip route del 10.244.1.0/24 via 192.168.2.63 dev eth0
# ip route del 10.244.2.0/24 via 192.168.2.62 dev eth0
# ip route
default via 192.168.2.2 dev eth0
169.254.0.0/16 dev eth0 scope link metric 1002
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
192.168.2.0/24 dev eth0 proto kernel scope link src 192.168.2.61
```

## 4, Calico 管理工具

下载工具: [https://github.com/projectcalico/calicoctl/releases](https://github.com/projectcalico/calicoctl/releases)

```
# wget -O /usr/local/bin/calicoctl https://github.com/projectcalico/calicoctl/releases/download/v3.11.1/calicoctl
# chmod + x /usr/local/bin/calicoctl
```

使用 `calicoctl` 查看服务状态

```
# ./calicoctl node status
Calico process is running.

IPv4 BGP status
+--------------+-------------------+-------+----------+-------------+
| PEER ADDRESS |     PEER TYPE     | STATE |  SINCE   |    INFO     |
+--------------+-------------------+-------+----------+-------------+
| 192.168.2.62 | node-to-node mesh | up    | 02:58:05 | Established |
| 192.168.2.63 | node-to-node mesh | up    | 03:08:46 | Established |
+--------------+-------------------+-------+----------+-------------+

IPv6 BGP status
No IPv6 peers found.
```

实际上, 使用 `calicoctl` 查看 `node` 状态就是调用系统查看的, 与 `netstat` 效果一样

```
# netstat -antp| grep bird
tcp        0      0 0.0.0.0:179             0.0.0.0:*               LISTEN      62709/bird
tcp        0      0 192.168.2.61:179        192.168.2.63:58963      ESTABLISHED 62709/bird
tcp        0      0 192.168.2.61:179        192.168.2.62:37390      ESTABLISHED 62709/bird
```

想要查看更多的信息, 需要指定配置查看 `etcd` 中的数据
创建配置文件

```
# mkdir /etc/calico
# vim /etc/calico/calicoctl.cfg
apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "etcdv3"
  etcdEndpoints: "https://192.168.2.61:2379, https://192.168.2.62:2379, https://192.168.2.63:2379"
  etcdKeyFile: "/opt/etcd/ssl/server-key.pem"
  etcdCertFile: "/opt/etcd/ssl/server.pem"
  etcdCACertFile: "/opt/etcd/ssl/ca.pem"
```

查看数据等操作

```
# calicoctl get node
NAME
k8s-master-01
k8s-node-01
k8s-node-02
```

查看 IPAM 的 IP 地址池:

```
# ./calicoctl get ippool
NAME                  CIDR            SELECTOR
default-ipv4-ippool   10.244.0.0/16   all()

# ./calicoctl get ippool -o wide
NAME                  CIDR            NAT    IPIPMODE   VXLANMODE   DISABLED   SELECTOR
default-ipv4-ippool   10.244.0.0/16   true   Never      Never       false      all()
```

## 5, Calico BGP 模式

! [](https://image.ssgeek.com/20200626-04.png)

`Pod 1` 访问 `Pod 2` 大致流程如下:

1.  数据包从容器 1 出到达 `Veth Pair` 另一端 (宿主机上, 以 `cali` 前缀开头);

2.  宿主机根据路由规则, 将数据包转发给下一跳 (网关);

3.  到达 `Node2`, 根据路由规则将数据包转发给 `cali` 设备, 从而到达容器 2.


路由表:

```
# node1
10.244.36.65 dev cali4f18ce2c9a1 scope link
10.244.169.128/26 via 192.168.31.63 dev ens33 proto bird
10.244.235.192/26 via 192.168.31.61 dev ens33 proto bird
# node2
10.244.169.129 dev calia4d5b2258bb scope link
10.244.36.64/26 via 192.168.31.62 dev ens33 proto bird
10.244.235.192/26 via 192.168.31.61 dev ens33 proto bird
```

其中, 这里最核心的 " 下一跳 " 路由规则, 就是由 `Calico` 的 `Felix` 进程负责维护的. 这些路由规则信息, 则是通过 `BGP Client` 也就是 `BIRD` 组件, 使用 `BGP` 协议传输而来的.

不难发现,`Calico` 项目实际上将集群里的所有节点, 都当作是边界路由器来处理, 它们一起组成了一个全连通的网络, 互相之间通过 `BGP` 协议交换路由规则. 这些节点, 我们称为 `BGP Peer`.

calico 相关文件

```
# ls /opt/cni/bin/calico-ipam
/opt/cni/bin/calico-ipam
# cat /etc/cni/net.d/
10-calico.conflist  calico-kubeconfig    calico-tls/
# cat /etc/cni/net.d/10-calico.conflist
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.1",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "etcd_endpoints": "https://192.168.2.61:2379, https://192.168.2.62:2379, https://192.168.2.63:2379",
      "etcd_key_file": "/etc/cni/net.d/calico-tls/etcd-key",
      "etcd_cert_file": "/etc/cni/net.d/calico-tls/etcd-cert",
      "etcd_ca_cert_file": "/etc/cni/net.d/calico-tls/etcd-ca",
      "mtu": 1440,
      "ipam": {
          "type": "calico-ipam"
      },
      "policy": {
          "type": "k8s"
      },
      "kubernetes": {
          "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "portmap",
      "snat": true,
      "capabilities": {"portMappings": true}
    }
  ]
}
```

## 6, Calico Route Reflector 模式 (RR)

[https://docs.projectcalico.org/master/networking/bgp](https://docs.projectcalico.org/master/networking/bgp)

`Calico` 维护的网络在默认是 (Node-to-Node Mesh) 全互联模式,`Calico` 集群中的节点之间都会相互建立连接, 用于路由交换. 但是随着集群规模的扩大,`mesh` 模式将形成一个巨大服务网格, 连接数成倍增加.

这时就需要使用 `Route Reflector`(路由器反射) 模式解决这个问题.

确定一个或多个 `Calico` 节点充当路由反射器 (一般配置两个以上), 让其他节点从这个 `RR` 节点获取路由信息.

具体步骤如下:

**1, 关闭 node-to-node BGP 网格**

默认 `node to node` 模式最好在 100 个节点以下
添加 `default BGP` 配置, 调整 `nodeToNodeMeshEnabled` 和 `asNumber`: bgp.yaml

```
# cat << EOF | calicoctl create -f -
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  logSeverityScreen: Info
  nodeToNodeMeshEnabled: false
  asNumber: 63400
EOF
# calicoctl apply -f bgp.yaml  # 一旦执行, 集群会立即断网
Successfully applied 1 'BGPConfiguration' resource(s)
# calicoctl get bgpconfig
NAME      LOGSEVERITY   MESHENABLED   ASNUMBER
default   Info          false         63400
# calicoctl node status
Calico process is running.

IPv4 BGP status
No IPv4 peers found.

IPv6 BGP status
No IPv6 peers found.
```

ASN 号可以通过获取

```
# calicoctl get nodes --output= wide
NAME            ASN       IPV4              IPV6
k8s-master-01   (63400)   192.168.2.61/24
k8s-node-01     (63400)   192.168.2.62/24
k8s-node-02     (63400)   192.168.2.63/24
```

**2, 配置指定节点充当路由反射器**

为方便让 `BGPPeer` 轻松选择节点, 通过标签选择器匹配.

给路由器反射器节点打标签:
增加第二个路由反射器时, 给新的 `node` 打标签并配置成反射器节点即可

```
# kubectl label node k8s-node-02 route-reflector= true
node/k8s-node-02 labeled
```

然后配置路由器反射器节点 `routeReflectorClusterID`:

```
# calicoctl get nodes k8s-node-02 -o yaml> node.yaml
# vim node.yaml
apiVersion: projectcalico.org/v3
kind: Node
metadata:
  annotations:
    projectcalico.org/kube-labels: '{"beta.kubernetes.io/arch":"amd64","beta.kubernetes.io/os":"linux","kubernetes.io/arch":"amd64","kubernetes.io/hostname":"k8s-node2","kubernetes.io/os":"linux"}'
  creationTimestamp: null
  labels:
    beta.kubernetes.io/arch: amd64
    beta.kubernetes.io/os: linux
    kubernetes.io/arch: amd64
    kubernetes.io/hostname: k8s-node2
    kubernetes.io/os: linux
  name: k8s-node2
spec:
  bgp:
    ipv4Address: 192.168.31.63/24
    routeReflectorClusterID: 244.0.0.1   # 增加集群 ID
  orchRefs:
  - nodeName: k8s-node2
    orchestrator: k8s
# ./calicoctl apply -f node.yaml
Successfully applied 1 'Node' resource(s)
```

现在, 很容易使用标签选择器将路由反射器节点与其他非路由反射器节点配置为对等:

表示所有节点都连接路由反射器节点

```
# vim peer-with-route-reflectors.yaml
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: peer-with-route-reflectors
spec:
  nodeSelector: all()
  peerSelector: route-reflector == 'true'
# calicoctl apply -f peer-with-route-reflectors.yaml
Successfully applied 1 'BGPPeer' resource(s)
# calicoctl get bgppeer
NAME                         PEERIP   NODE    ASN
peer-with-route-reflectors            all()   0
```

查看节点的 `BGP` 连接状态, 只有本节点与路由反射器节点的连接:

```
# calicoctl node status
Calico process is running.

IPv4 BGP status
+--------------+---------------+-------+----------+-------------+
| PEER ADDRESS |   PEER TYPE   | STATE |  SINCE   |    INFO     |
+--------------+---------------+-------+----------+-------------+
| 192.168.2.63 | node specific | up    | 04:17:14 | Established |
+--------------+---------------+-------+----------+-------------+

IPv6 BGP status
No IPv6 peers found.
```

## 7, Calico IPIP 模式

`Flannel host-gw` 模式最主要的限制, 就是要求集群宿主机之间是二层连通的. 而这个限制对于 `Calico` 来说, 也同样存在.

修改为 `IPIP` 模式:
也可以直接在部署 `calico` 的时候直接修改

```
# calicoctl get ipPool -o yaml > ipip.yaml
# vi ipip.yaml
apiVersion: projectcalico.org/v3
kind: IPPool
metadata:
  name: default-ipv4-ippool
spec:
  blockSize: 26
  cidr: 10.244.0.0/16
  ipipMode: Always  # 启动 ipip 模式
  natOutgoing: true

# calicoctl apply -f ipip.yaml
# calicoctl get ippool -o wide
NAME                  CIDR            NAT    IPIPMODE   VXLANMODE   DISABLED   SELECTOR
default-ipv4-ippool   10.244.0.0/16   true   Always     Never       false      all()
# ip route # 会增加 tunl0 网卡
default via 192.168.2.2 dev eth0
10.244.44.192/26 via 192.168.2.63 dev tunl0 proto bird onlink
blackhole 10.244.151.128/26 proto bird
10.244.154.192/26 via 192.168.2.62 dev tunl0 proto bird onlink
169.254.0.0/16 dev eth0 scope link metric 1002
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1
192.168.2.0/24 dev eth0 proto kernel scope link src 192.168.2.61
```

IPIP 示意图:

! [](https://image.ssgeek.com/20200626-05.png)

`Pod 1` 访问 `Pod 2` 大致流程如下:

1.  数据包从容器 1 出到达 `Veth Pair` 另一端 (宿主机上, 以 `cali` 前缀开头);
2.  进入 IP 隧道设备 (tunl0), 由 `Linux` 内核 `IPIP` 驱动封装在宿主机网络的 `IP` 包中 (新的 `IP` 包目的地之是原 `IP` 包的下一跳地址, 即 192.168.31.63), 这样, 就成了 `Node1` 到 `Node2` 的数据包;
3.  数据包经过路由器三层转发到 `Node2`;
4.  `Node2` 收到数据包后, 网络协议栈会使用 `IPIP` 驱动进行解包, 从中拿到原始 `IP` 包;
5.  然后根据路由规则, 根据路由规则将数据包转发给 `cali` 设备, 从而到达容器 2.

路由表:

```
# node1
10.244.36.65 dev cali4f18ce2c9a1 scope link
10.244.169.128/26 via 192.168.31.63 dev tunl0 proto bird onlink
# node2
10.244.169.129 dev calia4d5b2258bb scope link
10.244.36.64/26 via 192.168.31.62 dev tunl0 proto bird onlink
```

不难看到, 当 `Calico` 使用 `IPIP` 模式的时候, 集群的网络性能会因为额外的封包和解包工作而下降. 所以建议你将所有宿主机节点放在一个子网里, 避免使用 `IPIP`.

## 8, Calico 网络策略

部署完成 Calico 后, 就可以实现 k8s 中的网络策略 `NetworkPolicy`, 对于网络策略在前面的文章 [使用 flannel+ canal 实现 k8s 的 NetworkPolicy](https://www.ssgeek.com/post/shi-yong-flannelcanal-shi-xian-k8s-de-networkpolicy/) 有详细描述, 这里不再赘述.😊

文章参考来源: [https://docs.projectcalico.org/](https://docs.projectcalico.org/)

posted @ 2020-06-26 13:20  [SSgeek](https://www.cnblogs.com/ssgeek/)  阅读 (1768)  评论 (0)  [编辑](https://i.cnblogs.com/EditPosts.aspx? postid=13194687)  [收藏](javascript: void(0))

https://cloud.tencent.com/developer/article/1638845

# Calico 介绍, 原理与使用

2020-06-042020-06-04 16:23:51 阅读 1.2K0

## **什么是 Calico ?**

`Calico` 是一套开源的网络和网络安全方案, 用于容器, 虚拟机, 宿主机之前的网络连接, 可以用在 kubernetes, OpenShift, DockerEE, OpenStrack 等 PaaS 或 IaaS 平台上.

## **Calico 组件概述**

*   `Felix`: calico 的核心组件, 运行在每个节点上. 主要的功能有 ` 接口管理 `,` 路由规则 `,`ACL 规则 ` 和 ` 状态报告 `
    *   ` 接口管理 `: Felix 为内核编写一些接口信息, 以便让内核能正确的处理主机 endpoint 的流量. 特别是主机之间的 ARP 请求和处理 ip 转发.
    *   ` 路由规则 `: Felix 负责主机之间路由信息写到 linux 内核的 FIB(Forwarding Information Base) 转发信息库, 保证数据包可以在主机之间相互转发.
    *   `ACL 规则 `: Felix 负责将 ACL 策略写入到 linux 内核中, 保证主机 endpoint 的为有效流量不能绕过 calico 的安全措施.
    *   ` 状态报告 `: Felix 负责提供关于网络健康状况的数据. 特别是, 它报告配置主机时出现的错误和问题. 这些数据被写入 etcd, 使其对网络的其他组件和操作人员可见.
*   `Etcd`: 保证数据一致性的数据库, 存储集群中节点的所有路由信息. 为保证数据的可靠和容错建议至少三个以上 etcd 节点.
*   `Orchestrator plugin`: 协调器插件负责允许 kubernetes 或 OpenStack 等原生云平台方便管理 Calico, 可以通过各自的 API 来配置 Calico 网络实现无缝集成. 如 kubernetes 的 cni 网络插件.
*   `Bird`: BGP 客户端, Calico 在每个节点上的都会部署一个 BGP 客户端, 它的作用是将 Felix 的路由信息读入内核, 并通过 BGP 协议在集群中分发. 当 Felix 将路由插入到 Linux 内核 FIB 中时, BGP 客户端将获取这些路由并将它们分发到部署中的其他节点. 这可以确保在部署时有效地路由流量.
*   `BGP Router Reflector`: 大型网络仅仅使用 BGP client 形成 mesh 全网互联的方案就会导致规模限制, 所有节点需要 N^2 个连接, 为了解决这个规模问题, 可以采用 `BGP 的 Router Reflector` 的方法, 使所有 BGP Client 仅与特定 RR 节点互联并做路由同步, 从而大大减少连接数.
*   `Calicoctl`: calico 命令行管理工具.

## **Calico 网络模式**

*   `BGP 边界网关协议 (Border Gateway Protocol, BGP)`: 是互联网上一个核心的去中心化自治路由协议.BGP 不使用传统的内部网关协议 (IGP) 的指标.
    *   `Route Reflector 模式 (RR) (路由反射)`: Calico 维护的网络在默认是 (Node-to-Node Mesh) 全互联模式, Calico 集群中的节点之间都会相互建立连接, 用于路由交换. 但是随着集群规模的扩大, mesh 模式将形成一个巨大服务网格, 连接数成倍增加. 这时就需要使用 Route Reflector(路由器反射) 模式解决这个问题.
*   `IPIP 模式 `: 把 IP 层封装到 IP 层的一个 tunnel. 作用其实基本上就相当于一个基于 IP 层的网桥! 一般来说, 普通的网桥是基于 mac 层的, 根本不需 IP, 而这个 ipip 则是通过两端的路由做一个 tunnel, 把两个本来不通的网络通过点对点连接起来.

## **BGP 概述**

`BGP(border gateway protocol) 是外部路由协议 (边界网关路由协议)`, 用来在 AS 之间传递路由信息是一种增强的距离矢量路由协议 (应用场景), 基本功能是在自治系统间自动交换无环路的路由信息, 通过交换带有自治系统号序列属性的路径可达信息, 来构造自治系统的拓扑图, 从而消除路由环路并实施用户配置的路由策略.

(边界网关协议 (BGP), 提供自治系统之间无环路的路由信息交换 (无环路保证主要通过其 AS-PATH 实现), BGP 是基于策略的路由协议, 其策略通过丰富的路径属性 (attributes) 进行控制.BGP 工作在应用层, 在传输层采用可靠的 TCP 作为传输协议 (BGP 传输路由的邻居关系建立在可靠的 TCP 会话的基础之上). 在路径传输方式上, BGP 类似于距离矢量路由协议. 而 BGP 路由的好坏不是基于距离 (多数路由协议选路都是基于带宽的), 它的选路基于丰富的路径属性, 而这些属性在路由传输时携带, 所以我们可以把 BGP 称为路径矢量路由协议. 如果把自治系统浓缩成一个路由器来看待, BGP 作为路径矢量路由协议这一特征便不难理解了. 除此以外, BGP 又具备很多链路状态 (LS) 路由协议的特征, 比如触发式的增量更新机制, 宣告路由时携带掩码等.)

> 实际上, Calico 项目提供的 `BGP` 网络解决方案, 与 `Flannel` 的 `host-gw` 模式几乎一样. 也就是说, Calico 也是基于路由表实现容器数据包转发, 但不同于 Flannel 使用 flanneld 进程来维护路由信息的做法, 而 Calico 项目使用 BGP 协议来自动维护整个集群的路由信息.

## **BGP 两种模式**

### **全互联模式 (node-to-node mesh)**

` 全互联模式 ` 每一个 BGP Speaker 都需要和其他 BGP Speaker 建立 BGP 连接, 这样 BGP 连接总数就是 N^2, 如果数量过大会消耗大量连接. 如果集群数量超过 100 台官方不建议使用此种模式.

### **路由反射模式 Router Reflection(RR)**

`RR 模式 ` 中会指定一个或多个 BGP Speaker 为 RouterReflection, 它与网络中其他 Speaker 建立连接, 每个 Speaker 只要与 Router Reflection 建立 BGP 就可以获得全网的路由信息. 在 calico 中可以通过 `Global Peer` 实现 RR 模式.

## **Calico BGP 概述**

### **BGP 是怎么工作的?**

这个也是跨节点之间的通信, 与 flannel 类似, 其实这张图相比于 flannel, 通过一个路由器来路由, flannel.1 就相比于 vxlan 模式去掉, 所以会发现这里是没有网桥存在, 完全就是通过路由来实现, 这个数据包也是先从 veth 设备对另一口发出, 到达宿主机上的 cali 开头的虚拟网卡上, 到达这一头也就到达了宿主机上的网络协议栈, 另外就是当创建一个 pod 时帮你先起一个 infra containers 的容器, 调用 calico 的二进制帮你去配置容器的网络, 然后会根据路由表决定这个数据包到底发送到哪里去, 可以从 ip route 看到路由表信息, 这里显示是目的 cni 分配的子网络和目的宿主机的网络, 当进行跨主机通信的时候之间转发到下一跳地址走宿主机的 eth0 网卡出去, 也就是一个直接的静态路由, 这个下一跳就跟 host-gw 的形式一样, 和 host-gw 最大的区别是 calico 使用 BGP 路由交换, 而 host-gw 是使用自己的路由交换, BGP 这个方案比较成熟, 在大型网络中用的也比较多, 所以要比 flannel 的方式好, 而这些路由信息都是由 BGP client 传输.

### **为什么叫边界网关协议呢?**

和 flannel host-gw 工作模式基本上一样, BGP 是一个边界路由器, 主要是在每个自治系统的最边界与其他自治系统的传输规则, 而这些节点之间组成的 BGP 网络是一个全网通的网络, 这个网络就称为一个 `BGP Peer`.

启动文件放在 `/opt/cni/bin` 目录下,/etc/cni/net.d 目录下记录子网的相关配置信息.

```
$ cat /etc/cni/net.d/10-calico.conflist

{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.0",
  "plugins": \[
    {
      "type": "calico",
      "log\_level": "info",
      "etcd\_endpoints": "https://10.10.0.174:2379",
      "etcd\_key\_file": "/etc/cni/net.d/calico-tls/etcd-key",
      "etcd\_cert\_file": "/etc/cni/net.d/calico-tls/etcd-cert",
      "etcd\_ca\_cert\_file": "/etc/cni/net.d/calico-tls/etcd-ca",
      "mtu": 1440,
      "ipam": {
          "type": "calico-ipam"
      },
      "policy": {
          "type": "k8s"
      },
      "kubernetes": {
          "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "portmap",
      "snat": true,
      "capabilities": {"portMappings": true}
    }
  \]
}
```

### **Pod 1 访问 Pod 2 流程如下**

1, 数据包从 Pod1 出到达 Veth Pair 另一端 (宿主机上, 以 cali 前缀开头)

2, 宿主机根据路由规则, 将数据包转发给下一跳 (网关)

3, 到达 Node2, 根据路由规则将数据包转发给 cali 设备, 从而到达 Pod2.

其中, 这里最核心的 ` 下一跳 ` 路由规则, 就是由 `Calico` 的 `Felix` 进程负责维护的. 这些路由规则信息, 则是通过 BGP Client 中 BIRD 组件, 使用 BGP 协议来传输.

不难发现, Calico 项目实际上将集群里的所有节点, 都当作是边界路由器来处理, 它们一起组成了一个全连通的网络, 互相之间通过 BGP 协议交换路由规则. 这些节点, 我们称为 `BGP Peer`.

而 `Flannel host-gw` 和 `Calico` 的唯一不一样的地方就是当数据包下一跳到达 node2 节点容器时发生变化, 并且出数据包也发生变化, 知道它是从 veth 的设备流出, 容器里面的数据包到达宿主机上, 这个数据包到达 node2 之后, 它又根据一个特殊的路由规则, 这个会记录目的通信地址的 cni 网络, 然后通过 cali 设备进去容器, 这个就跟网线一样, 数据包通过这个网线发到容器中, 这也是一个 ` 二层的网络互通才能实现 `.

## **Route Reflector 模式 (RR) (路由反射)**

> 设置方法请参考官方链接 https://docs.projectcalico.org/master/networking/bgp

Calico 维护的网络在默认是 `(Node-to-Node Mesh) 全互联模式 `, Calico 集群中的节点之间都会相互建立连接, 用于路由交换. 但是随着集群规模的扩大, mesh 模式将形成一个巨大服务网格, 连接数成倍增加. 这时就需要使用 Route Reflector(路由器反射) 模式解决这个问题. 确定一个或多个 Calico 节点充当路由反射器, 让其他节点从这个 RR 节点获取路由信息.

在 BGP 中可以通过 calicoctl node status 看到启动是 node-to-node mesh 网格的形式, 这种形式是一个全互联的模式, 默认的 BGP 在 k8s 的每个节点担任了一个 BGP 的一个喇叭, 一直吆喝着扩散到其他节点, 随着集群节点的数量的增加, 那么上百台节点就要构建上百台链接, 就是全互联的方式, 都要来回建立连接来保证网络的互通性, 那么增加一个节点就要成倍的增加这种链接保证网络的互通性, 这样的话就会使用大量的网络消耗, 所以这时就需要使用 Route reflector, 也就是找几个大的节点, 让他们去这个大的节点建立连接, 也叫 RR, 也就是公司的员工没有微信群的时候, 找每个人沟通都很麻烦, 那么建个群, 里面的人都能收到, 所以要找节点或着多个节点充当路由反射器, 建议至少是 2 到 3 个, 一个做备用, 一个在维护的时候不影响其他的使用.

## **IPIP 模式概述**

`IPIP` 是 linux 内核的驱动程序, 可以对数据包进行隧道, 上图可以看到两个不同的网络 vlan1 和 vlan2. 基于现有的以太网将原始包中的原始 IP 进行一次封装, 通过 tunl0 解包, 这个 tunl0 类似于 ipip 模块, 和 Flannel vxlan 的 veth 很类似.

### **Pod1 访问 Pod2 流程如下:**

1, 数据包从 Pod1 出到达 Veth Pair 另一端 (宿主机上, 以 cali 前缀开头).

2, 进入 IP 隧道设备 (tunl0), 由 Linux 内核 IPIP 驱动封装, 把源容器 ip 换成源宿主机 ip, 目的容器 ip 换成目的主机 ip, 这样就封装成 Node1 到 Node2 的数据包.

```
此时包的类型:
  原始 IP 包:
  源 IP:10.244.1.10
  目的 IP:10.244.2.10

   TCP:
   源 IP: 192.168.31.62
   目的 iP:192.168.32.63
```

3, 数据包经过路由器三层转发到 Node2.

4, Node2 收到数据包后, 网络协议栈会使用 IPIP 驱动进行解包, 从中拿到原始 IP 包.

5, 然后根据路由规则, 将数据包转发给 cali 设备, 从而到达 Pod2.

### **实际查看路由情况**

跨宿主机之间容器访问:

环境:

|
Pod 名称



 |

Pod IP



 |

宿主机 IP



 |
| --- | --- | --- |
|

zipkin-dependencies-production



 |

10.20.169.155



 |

192.168.162.248



 |
|

zipkin-production



 |

10.20.36.85



 |

192.168.163.40



 |

zipkin-dependencies-production 访问 zipkin-production 过程如下:

登录 zipkin-dependencies-production 容器中, 查看路由信息, 可以看到 0.0.0.0 指向了 169.254.1.1(eth0), 查看容器 ip 地址.

`eth0@if1454` 为在宿主机上创建的虚拟网桥的一端, 另一端为 `calic775b4c8175`

去往容器 zipkin-production 的数据包, 由 zipkin-dependencies-production 容器里经过网桥设备转发到 zipkin-dependencies-production 所在的宿主机上, 在宿主机上查看路由表.

可以看到 zipkin-production (pod-ip 10.20.36.85) 的下一跳是 192.168.163.40, 也就是 zipkin-production 所在的宿主机 ip 地址, 网络接口是 tunl0

> tunl0 设备是一个 ip 隧道 (ip tunnel) 设备, 技术原理是 IPIP, ip 包进入 ip 隧道后会被 linux 内核的 IPIP 驱动接管并重新封装 (伪造) 成去宿主机网络的 ip 包, 然后发送出去. 这样原始 ip 包经过封装后下一跳地址就是 192.168.163.40

数据包到达 zipkin-production 的宿主机 192.168.163.40 后, 经过解包查找 10.20.36.85 的路由为网桥设备 cali0393db3e4a6

最终 10.20.36.85 经过网桥设备 cali0393db3e4a6 被转发到容器 zipkin-production 内部, 返回的数据包路径也是一样.

## **Calico 优势 与 劣势**

### **优势**

*   没有封包和解包过程, 完全基于两端宿主机的路由表进行转发
*   可以配合使用 `Network Policy` 做 pod 和 pod 之前的访问控制

### **劣势**

*   要求宿主机处于同一个 2 层网络下, 也就是连在一台交换机上
*   路由的数目与容器数目相同, 非常容易超过路由器, 三层交换, 甚至 node 的处理能力, 从而限制了整个网络的扩张.(可以使用大规模方式解决)
*   每个 node 上会设置大量 (海量) 的 iptables 规则, 路由, 运维, 排障难度大.
*   原理决定了它不可能支持 VPC, 容器只能从 calico 设置的网段中获取 ip.

## **Calico 管理工具**

calicoctl 工具安装

```
\# 下载工具: https://github.com/projectcalico/calicoctl/releases

$ wget -O /usr/local/bin/calicoctl https://github.com/projectcalico/calicoctl/releases/download/v3.13.3/calicoctl

$ chmod + x /usr/local/bin/calicoctl
``````
\# 查看集群节点状态

$ calicoctl node status
```

如果使用 `calicoctl get node`, 需要指定 calicoctl 配置, 默认使用 `/etc/calico/calicoctl.cfg`

```
\# 设置 calicoctl 配置文件
$ vim /etc/calico/calicoctl.cfg

apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "etcdv3"
  etcdEndpoints: https://10.10.0.174:2379
  etcdKeyFile: /opt/kubernetes/ssl/server-key.pem
  etcdCertFile: /opt/kubernetes/ssl/server.pem
  etcdCACertFile: /opt/kubernetes/ssl/ca.pem

# 查看 calico 节点
$ calicoctl get nodes

# 查看 IPAM 的 IP 地址池
$ calicoctl get ippool -o wide

# 查看 bgp 网络配置情况
$ calicoctl get bgpconfig

# 查看 ASN 号, 一个编号就是一个自治系统
$ calicoctl get nodes --output= wide

# 查看 bgp peer
$ calicoctl get bgppeer
```

## **CNI 网络方案优缺点及最终选择**

先考虑几个问题:

1, 需要细粒度网络访问控制?

这个 flannel 是不支持的, calico 支持, 所以做多租户网络方面的控制 ACL, 那么要选择 calico.

2, 追求网络性能?

选择 `flannel host-gw` 模式 和 `calico BGP` 模式.

3, 服务器之前是否可以跑 BGP 协议?

很多的公有云是不支持跑 BGP 协议, 那么使用 calico 的 BGP 模式自然是不行的.

4, 集群规模多大?

如果规模不大,100 以下节点可以使用 flannel, 优点是维护比较简单.

5, 是否有维护能力?

calico 的路由表很多, 而且走 BGP 协议, 一旦出现问题排查起来也比较困难, 上百台的, 路由表去排查也是很麻烦, 这个具体需求需要根据自己的情况而定.

## **参考链接**

*   https://kuaibao.qq.com/s/20200111AZOTQ200? refer= spider
*   https://blog.csdn.net/zhonglinzhang/article/details/97613927
*   https://blog.51cto.com/14143894/2463392
*   https://blog.csdn.net/x503809622/article/details/82629474

本文分享自微信公众号 - YP 小站 (ypxiaozhan), 作者: YP 小站

原文出处及转载信息见文内详细说明, 如有侵权, 请联系 yunjia\_community@tencent.com 删除.

原始发表时间:2020-05-12

本文参与 [腾讯云自媒体分享计划](/developer/support-plan), 欢迎正在阅读的你也加入, 一起分享.

[展开阅读全文](javascript:;)

[举报](javascript:;)

点赞 1 分享

### 我来说两句

0 条评论

[登录](javascript:;) 后参与评论

