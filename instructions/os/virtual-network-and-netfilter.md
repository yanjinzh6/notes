---
title: virtual-network-and-netfilter
date: 2021-03-13 19:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: virtual-network-and-netfilter
---

[virtual-network-and-netfilter](https://zhuanlan.zhihu.com/p/223038075)

# 手撕 Linux 网络——Linux 虚拟网络设备与 netfilter 框架汇总

[! [中本大头蒜](https://pic1.zhimg.com/v2-73380c33fc9a671365052a69f8efc44e_xs.jpg? source=172ae18b)](//www.zhihu.com/people/yao-chao-51-30)

[中本大头蒜](//www.zhihu.com/people/yao-chao-51-30)

Full-stack, 致力于区块链与云计算

22 人赞同了该文章

这篇文章是我学习 linux 设备与 netfilter 框架的总结, 大多数是源于网上大佬们的总结, 加上我自己的理解. 里面我自己的内容不多, 很多细节部分的描述都通过引用的方式. 本文主要提供一个知识架构以及知识索引, 方便日后温习. 本文主要包括以下内容:

*   linux 虚拟网络设备介绍以及实践, 包括 TAP/TUN, veth, bridge 以及 vlan
*   netfilter 框架各组件介绍, 包括 ebtables/arptables/iptables, conntrack 等



### Linux 虚拟网络设备

### TAP/TUN

TAP/TUN 设备是一种让用户态程序向内核协议栈注入数据的设备, TAP 等同于一个以太网设备, 工作在二层; 而 TUN 则是一个虚拟点对点设备, 工作在三层.

> 实现 `tun/tap` 设备的内核模块为 `tun`, 其模块介绍为 `Universal TUN/TAP device driver`, 该模块提供了一个设备接口 `/dev/net/tun` 供用户层程序读写, 用户层程序通过读写 `/dev/net/tun` 来向主机内核协议栈注入数据或接收来自主机内核协议栈的数据,**可以把 tun/tap 看成数据管道, 它一端连接主机协议栈, 另一端连接用户程序.**
> 为了使用 `tun/tap` 设备, 用户层程序需要通过系统调用打开 `/dev/net/tun` 获得一个读写该设备的文件描述符 (FD), 并且调用 ioctl() 向内核注册一个 TUN 或 TAP 类型的虚拟网卡 (实例化一个 tun/tap 设备), 其名称可能是 `tap7b7ee9a9-c1/vnetXX/tunXX/tap0` 等. 此后, 用户程序可以通过该虚拟网卡与主机内核协议栈交互. 当用户层程序关闭后, 其注册的 TUN 或 TAP 虚拟网卡以及路由表相关条目 (使用 tun 可能会产生路由表条目, 比如 openvpn) 都会被内核释放.**可以把用户层程序看做是网络上另一台主机, 他们通过 tap/tun 虚拟网卡与主机相连**.
> (本段出自: [云计算底层技术 - 虚拟网络设备 (tun/tap, veth)](https://link.zhihu.com/? target= https%3A//opengers.github.io/openstack/openstack-base-virtual-network-devices-tuntap-veth/))

上面提到 TAP 与 TUN 的区别在于 TAP 工作在 2 层, TUN 工作在 3 层, 这意味着通过 TAP 连接的用户层程序发出的数据不用穿越主机协议栈的网络层, 直接到达链路层. 光这么说它们之间的区别还是很抽象, 举个例子. 假如该用户程序是一个虚拟机, 我们知道虚拟机和主机是两台机器, 他们都桥接在主机上的软件交换机上, 主机的路由与防火墙不应该影响虚拟机. 为了达到这个效果, 我们可以让虚拟机通过 TAP 网卡访问外网, 这样虚拟机发出的数据包不会经过主机协议栈的网络层, 也就不会受到主机的路由和 iptables 的影响, 因为 iptables 和路由都是工作在网络层的 (也有办法让主机的 iptables 在链路层起作用, 下文会介绍)), 整个虚拟机访问外网的过程中, 主机仅仅充当一个交换机的角色.

Linux 上可以使用 `ip taptun` 和 `tunctl` 来操作的 TAP/TUN 设备, 在 [Linux 网络工具详解之 ip tuntap 和 tunctl 创建 tap/tun 设备](https://link.zhihu.com/? target= https%3A//cloud.tencent.com/developer/article/1432446) 中有详细介绍. 更推荐使用 `ip taptun` 命令, 这里仅仅列举一些常用命令.

```
# 创建 tap/tun
ip tuntap add dev tap0 mod tap
ip tuntap add dev tun0 mod tun

# 删除 tap/tun
ip tuntap del dev tap0 mod tap
ip tuntap del dev tun0 mod tun
```

本文附录里面会提供 TAP/TUN 设备实践的例子, 为了不影响行文结构就不写在这里了.



### veth

veth 设备总是成对出现, 也叫 veth pair, 一端发送的数据会由另外一端接受. 如果 veth-a 和 veth-b 是一对 veth 设备, veth-a 收到的数据会从 veth-b 发出, 相反, veth-b 收到的数据会从 veth-a 发出.veth pair 常用于连接 2 个网络命名空间 (network namespace).

我们可以使用 `ip link` 命令新建一对 veth 设备 veth-a, veth-b.

```
ip link add veth-a type veth peer name veth-b
```

当要删除 veth pair 时, 只需要删除 veth-a 和 veth-b 其中一个即可, 因为 veth pair 总是成对出现, 一个被删除, 另一个也自动被删除.

```
ip link del veth-a
```

关于 veth 设备的实践可以参考我的另外一篇文章, 里面用网络命名空间模拟 2 台隔离的主机, 通过 veth pair, bridge 实现互相连通.

[中本大头蒜: 手撕 Docker 网络 (1) —— 从 0 搭建 Linux 虚拟网络​zhuanlan.zhihu.com! [图标](https://pic2.zhimg.com/v2-f5c85fd37b21a4d62ec015bc0520f13d_180x120.jpg)](https://zhuanlan.zhihu.com/p/199298498)

###

### bridge

bridge, 是 Linux 提供的一种虚拟网络设备之一. 其工作方式非常类似于物理的网络交换机设备.Linux Bridge 可以工作在二层, 也可以工作在三层, 默认工作在二层. 工作在二层时, 可以在同一网络的不同主机间转发以太网报文; 一旦你给一个 Linux Bridge 分配了 IP 地址, 也就开启了该 Bridge 的三层工作模式.

> Bridge 是 Linux 上工作在内核协议栈二层的虚拟交换机, 虽然是软件实现的, 但它与普通的二层物理交换机功能一样. 可以添加若干个网络设备 (em1, eth0, tap,..) 到 Bridge 上 (`brctl addif`) 作为其接口, 添加到 Bridge 上的设备被设置为只接受二层数据帧并且转发所有收到的数据包到 Bridge 中 (bridge 内核模块), 在 Bridge 中会进行一个类似物理交换机的查 MAC 端口映射表, 转发, 更新 MAC 端口映射表这样的处理逻辑, 从而数据包可以被转发到另一个接口 / 丢弃 / 广播 / 发往上层协议栈, 由此 Bridge 实现了数据转发的功能. 如果使用 `tcpdump` 在 Bridge 接口上抓包, 是可以抓到桥上所有接口进出的包
> 跟物理交换机不同的是, 运行 Bridge 的是一个 Linux 主机, Linux 主机本身也需要 IP 地址与其它设备通信. 但被添加到 Bridge 上的网卡是不能配置 IP 地址的, 他们工作在数据链路层, 对路由系统不可见. 不过 Bridge 本身可以设置 IP 地址, 可以认为当使用 `brctl addbr br0` 新建一个 `br0` 网桥时, 系统自动创建了一个同名的隐藏 `br0` 网络设备.`br0` 一旦设置 IP 地址, 就意味着 `br0` 可以作为路由接口设备, 参与 IP 层的路由选择 (可以使用 `route -n` 查看最后一列 `Iface`). 因此只有当 `br0` 设置 IP 地址时, Bridge 才有可能将数据包发往上层协议栈.
> (本段出自: [云计算底层技术 - 虚拟网络设备 (Bridge, VLAN)](https://link.zhihu.com/? target= https%3A//opengers.github.io/openstack/openstack-base-virtual-network-devices-bridge-and-vlan/%23vlan))

在 Linux 下, 你可以用 `ip link` 工具包或 `brctl` 命令对 Linux bridge 进行管理.

```
# 创建网桥 br1
brctl addbr br1
ip link add br1 type bridge

# 删除网桥 br1
brctl delbr br1
ip link del br1

# 将 eth0 端口加入网桥 br1
brctl addif br1 eth0
ip link set dev eth0 master br1

# 从网桥 br1 中删除 eth0
brctl delif br1 eth0
ip link set dev eth0 nomaster

# 查询网桥信息
brctl show
brctl show br1
bridge link
```

关于 bridge 设备的实践可以参考我的另外一篇文章, 里面用网络命名空间模拟 2 台隔离的主机, 通过 veth pair, bridge 实现互相连通.

[中本大头蒜: 手撕 Docker 网络 (1) —— 从 0 搭建 Linux 虚拟网络​zhuanlan.zhihu.com! [图标](https://pic2.zhimg.com/v2-f5c85fd37b21a4d62ec015bc0520f13d_180x120.jpg)](https://zhuanlan.zhihu.com/p/199298498)

### vlan

在说 Linux 的 vlan 之前, 我们得了解现实的 vlan 是什么样的? 这里我推荐一篇知乎文章:

[行道科技: VLAN 基础知识​zhuanlan.zhihu.com! [图标](https://pic4.zhimg.com/v2-8e43f602281b3d397e0cccff209ce207_ipico.jpg)](https://zhuanlan.zhihu.com/p/35616289)

读完上面这篇文章相信大家对 vlan 和它的原理已经很了解了.VLAN 的基本原理是在二层协议里插入额外的 VLAN 协议数据 (例如, VLAN Tag) 对物理上相互连接的设备进行逻辑网络切割, 同时保持和传统二层设备的兼容性.Linux 里的 VLAN 设备是对 [IEEE 802.1.q 协议](https://link.zhihu.com/? target= https%3A//support.huawei.com/enterprise/zh/doc/EDOC1100088136) 的一种内部软件实现, 模拟现实世界中的 802.1.q 交换机.

[RedHat 的这篇文章](https://link.zhihu.com/? target= https%3A//access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/networking_guide/ch-configure_802_1q_vlan_tagging) 介绍了很多在 Linux 上配置 VLAN 的方法. 这里列出一些简单的命令:

```
# 以太网接口 eth0 中创建名为 VLAN8, ID 为 8 的 802.1Q VLAN 接口
ip link add link eth0 name eth0.8 type vlan id 8
# 删除 vlan
ip link delete eth0.8
```

vlan 本身用于隔离 2 层网络, 设备 A 和 B 即使连接在同一台交换机上, 但处于不同 vlan, 在不借助路由器或者三层交换机的前提下, 它们之间也是无法连通的. 就拿 A ping B 这个场景来说, A 处于 vlan1, B 处于 vlan2, 当 A ping B 时, A 不知道 B 的 mac 地址, 所以 A 会发起 ARP 广播询问 b 的 mac 地址.A 的 arp 数据包到达 802.1.q 交换机时, 交换机会把 A 的包递给同样连接 vlan1 的设备, 而不是像普通交换机一样广播给所有设备. 这样一来 A 永远不知道 B 的 mac 地址, 即使物理上讲 A 和 B 连接在同一个交换机上. 关于实践操作, 我推荐一篇文章:

[Linux 系统下实践 VLAN​ctimbai.github.io](https://link.zhihu.com/? target= https%3A//ctimbai.github.io/2019/01/07/tech/Linux%25E7%25B3%25BB%25E7%25BB%259F%25E4%25B8%258B%25E5%25AE%259E%25E8%25B7%25B5VLAN/)

* * *

### Netfilter 框架分析

由于我对 netfilter 了解甚少, 这一部分算是我学习中的总结, 本段大量借鉴下文, 非常推荐大家去读读这篇文章:

[云计算底层技术 -netfilter 框架研究​opengers.github.io](https://link.zhihu.com/? target= https%3A//opengers.github.io/openstack/openstack-base-netfilter-framework-overview/%23bridge%25E4%25B8%258Enetfilter)

netfilter 是 linux 内核中的一个数据包处理框架, 它的功能包括数据包过滤, 修改, SNAT/DNAT 等.netfilter 在内核协议栈的不同位置实现了 5 个 hook 点, 其它内核模块 (比如 ip\_tables) 可以向这些 hook 点注册处理函数, 这样当数据包经过这些 hook 点时, 其上注册的处理函数就被依次调用, 用户层工具像 iptables 一般都需要相应内核模块 ip\_tables 配合以完成与 netfilter 的交互.netfilter hooks, ip{6}\_tables, connection tracking, 和 NAT 子系统一起构成了 netfilter 框架的主要部分.

下图展示了 netfilter 框架的组件, 原图来自 [维基百科](https://link.zhihu.com/? target= https%3A//upload.wikimedia.org/wikipedia/commons/d/dd/Netfilter-components.svg). 图中, 蓝色部分是用户空间的工具, 我们可以使用这些工具往内核的 hook 点注册函数, 比如 `iptables` 命令. 灰色部分代表了其他网络组件, 其他颜色的部分代表了内核组件, 这些组件会在数据包经过 hook 点时执行我们的注册函数.

<img src="https://pic4.zhimg.com/v2-7c55f30961a8c1bcb3abc135f14e094f\_b.jpg" data-size="normal" data-rawwidth="710" data-rawheight="400" class="origin\_image zh-lightbox-thumb" width="710" data-original="https://pic4.zhimg.com/v2-7c55f30961a8c1bcb3abc135f14e094f\_r.jpg"/>

! [](data: image/svg+ xml; utf8, <svg xmlns='http://www.w3.org/2000/svg' width='710' height='400'> </svg>)

netfilter 框架组件图

下面这张图也是来自 [维基百科](https://link.zhihu.com/? target= https%3A//upload.wikimedia.org/wikipedia/commons/3/37/Netfilter-packet-flow.svg), 它展示了 netfilter 框架在协议栈的位置. 通过它我们可以很清楚地看到 netfilter 框架是如何处理通过不同协议栈路径上的数据包. 可以看到一个数据包从到达主机网卡到被主机某个进程接受或者被转发的整个过程中会经过很多 hook 点, 这些 hook 点的注册函数将会确定数据包的去留.netfilter 框架负责的区域主要是图中的有背景色的部分.

<img src="https://pic2.zhimg.com/v2-53dd4c877c819a92532c29ff89ef4681\_b.jpg" data-size="normal" data-rawwidth="1650" data-rawheight="475" class="origin\_image zh-lightbox-thumb" width="1650" data-original="https://pic2.zhimg.com/v2-53dd4c877c819a92532c29ff89ef4681\_r.jpg"/>

! [](data: image/svg+ xml; utf8, <svg xmlns='http://www.w3.org/2000/svg' width='1650' height='475'> </svg>)

netfilter 在协议栈中的位置

现在我们顺着数据包, 从左到右依次看看这张图里面涉及到的组件

### XDP/eBPF

BPF([Berkeley Packet Filter](https://link.zhihu.com/? target= https%3A//en.wikipedia.org/wiki/Berkeley_Packet_Filter)) 是一套完整的计算机体系结构, 和 X86, ARM 一样, 它规范了自己的指令集与运行时逻辑, eBPF(extended BPF) 是 BPF 的复杂版, 在 BPF 的基础上增加了更多指令和更多可调用函数.Linux 内核在网络处理路径上预置了很多 eBPF 的挂载点, 例如 xdp, qdisc, tcp-bpf, socket 等. 用户可以把程序编程成 eBPF 指令, 然后通过系统调用将 eBPF 指令加载到特点挂载点, 由特定事件来触发 eBPF 指令的执行. 其核心思想是 "_**与其把数据包复制到用户空间执行用户态程序过滤, 不如把过滤程序灌进内核去.**_"

XDP(eXpress Data Path) 就是一个 eBPF 挂载点. 它位于网络协议栈的最底层, 网卡层面, 这里做数据包过滤非常高效, 并且可以依托硬件特性将数据包过滤行为 offload 到硬件中. 从上面那张协议栈的图可以看出, 数据包第一个就经过 XDP 调用点, 在这里注入 eBPF 代码可以非常高效地处理数据包. 为什么快? 有 2 个原因:1. 数据包处理位置非常底层, 避开了很多内核 skb 处理开销.2. 可以将很多处理逻辑 Offload 到网卡硬件.

实践中, 有利用 XDP 技术对抗 DDOS 攻击, 提高丢包速率, 网上有比较数据, 利用 XDP 技术的丢包速率要比 iptables 高 4 倍左右. 可以参考文章:

[eBPF 技术实践: 高性能 ACL-InfoQ​www.infoq.cn! [图标](https://pic1.zhimg.com/v2-efafa9c513d447a7fae02430be447ca4_180x120.jpg)](https://link.zhihu.com/? target= https%3A//www.infoq.cn/article/Tc5Bugo5vBAkyaRb5CCU)

此外, 还有利用 eBPF 优化 IPVS 提高 k8s 的 service 性能, 可以参考文章:

[人类身份验证 - SegmentFault​segmentfault.com](https://link.zhihu.com/? target= https%3A//segmentfault.com/a/1190000023012440)

更多关于 BPF 的可以参考 CSDN 上大佬 [dog250 的博客](https://link.zhihu.com/? target= https%3A//me.csdn.net/dog250).

### alloc\_skb

如果数据包通过了 `XDP eBPF` 的检查 (XDP\_PASS), 网卡将通过中断告知 CPU 自己接收到一个数据包, CPU 根据网卡注册的中断处理函数, 先为数据包分配一个 skb([socket\_buffer](https://link.zhihu.com/? target= https%3A//www.cnblogs.com/hustcat/archive/2009/09/19/1569859.html)) 结构用以保存一个报文, 然后将网卡设备收到的数据复制到这个 skb 结构对应的缓冲区中, 进行后续处理.

### qdisc

qdisc 是 linux 流控模块 (tracffic control) 的一部分, 主要负责将数据包缓存起来, 用来控制网络收发速度, 每个网卡都有一个关联的 qdisc. 一般说到 qdisc 都是指 egress qdisc. 每块网卡实际上还可以添加一个 ingress qdisc . 详情参考: [Components of Linux Traffic Control](https://link.zhihu.com/? target= https%3A//tldp.org/HOWTO/Traffic-Control-HOWTO/components.html).

### bridge check

`bridge check` 会检查数据包的源网卡是不是属于某个 bridge 设备. 在 linux 网络设备中介绍了 bridge 设备, 它就是一个虚拟的交换机.

*   当收到这个数据包的网卡是属于某个 bridge 的话, 数据包将进入上图中蓝背景色的链路层 (Link layer). 在里面会经过一些链路层的 hook 点 (蓝色的小方框), 然后会进行 `bridge decision`, 即二层交换机的查表转发功能, 根据数据包目的 MAC 地址判断此数据包是转发还是交给上层处理
*   如果数据包的源网卡不属于某 bridge, 否则直接进入绿背景色的网络层 (Network Layer). 在里面会经过一些网络层的 hook 点 (绿色的小方框), 然后会进行 `route decision`, 即路由选择, 根据系统路由表 (`route` 命令查看), 决定数据包是 forward, 还是交给本地处理.

值得注意的是, 我们在图中看到链路层里面也有网络层的 hook 点 (绿色的小方框). 这是因为 `bridge_nf` 代码的作用 (从 2.6 kernel 开始),`bridge_nf` 的引入是为了解决在链路层 Bridge 中处理 IP 数据包的问题 (需要通过内核参数开启), 那为什么要在链路层 Bridge 中处理 IP 数据包, 而不等数据包通过网络层时候再处理呢? 这是因为不是所有的数据包都一定会通过网络层, 比如在上文讲 TAP 设备的时候提到, 虚拟机的流量走 TAP 设备可以不经过主机的网络层, 不受主机 iptables 的影响. 如果启用 `bridge_nf` 我们可以让主机的 iptables 也对虚拟机适用, 从而在主机层面控制进出虚拟机的流量, 这也是 openstack 中实现安全组功能的基础.

### ebtables/arptables/iptables

`ebtables` 即是以太网桥 (ether brigde) 防火墙, 以太网桥工作在数据链路层, ebtables 来过滤数据链路层数据包;`arptables` 可以当作是 linux 下的 ARP 防火墙, 可以防止 APR 欺诈, 还可以用来解决 LVS 中的 ARP 问题;`iptables` 则是网络层防火墙, 可以配置数据包过滤, NAT 等.`ebtables` 和 `arptables` 的 hook 点位于图中蓝色小方块处,`iptables` 的 hook 点位于图中绿色小方块处, 通过用户程序 ebtables/arptables/iptables 可以在这些 hook 点处设置规则对数据包进行操作. 关于这 3 个 table 介绍的文章有很多, 本文引用几篇不再赘述:

[ebtables 与 iptables 的区别 (ebtables 的简单应用)\_u013485792 的专栏 -CSDN 博客​blog.csdn.net! [图标](https://pic1.zhimg.com/v2-2a5027b5bff83f50a189c6146b4f7548_ipico.jpg)](https://link.zhihu.com/? target= https%3A//blog.csdn.net/u013485792/article/details/76522551) [arptables 详解 - 刺猬的温驯 - 博客园​www.cnblogs.com](https://link.zhihu.com/? target= https%3A//www.cnblogs.com/chenying99/articles/3181001.html) [iptables 详解: 图文并茂理解 iptables​www.zsythink.net! [图标](https://pic4.zhimg.com/v2-73e11e98881f07db72ae98728a30a137_ipico.jpg)](https://link.zhihu.com/? target= https%3A//www.zsythink.net/archives/1199)

### conntrack

`conntrack`, 即 connection tracking, 这是 netfilter 提供的连接跟踪机制, 此机制允许内核 " 审查 " 通过此处的所有网络数据包, 并能识别出此数据包属于哪个网络连接 (比如数据包 a 属于 `IP1:8888-> IP2:80` 这个 tcp 连接, 数据包 b 属于 `ip3:9999-> IP4:53` 这个 udp 连接). 因此, 连接跟踪机制使内核能够跟踪并记录通过此处的所有网络连接及其状态. 图中可以清楚看到连接跟踪代码所处的网络栈位置, 如果不想让某些数据包被跟踪 (`NOTRACK`), 那就要找位于椭圆形方框 `conntrack` 之前的表和链来设置规则, 比如 iptables 的 raw 表的 PREROUTING 和 OUTPUT 链处处.**conntrack 机制是 iptables 实现状态匹配 (`-m state`) 以及 NAT 的基础**, 它由单独的内核模块 `nf_conntrack` 实现.

每个通过 `conntrack` 的数据包, 内核都为其生成一个 conntrack 条目用以跟踪此连接, 对于后续通过的数据包, 内核会判断若此数据包属于一个已有的连接, 则更新所对应的 conntrack 条目的状态. 所有的 conntrack 条目都存放在一张表里, 称为连接跟踪表 (通过命令 `conntrack` 查看), 更多关于 `conntrack` 的细节可以参考文章:

[云计算底层技术 -netfilter 框架研究​opengers.github.io](https://link.zhihu.com/? target= https%3A//opengers.github.io/openstack/openstack-base-netfilter-framework-overview/%23bridge%25E4%25B8%258Enetfilter)

conntrack 有什么用呢? 通过跟踪表, 内核可以得知数据包的状态, 从图中可以看出数据包经过灰色椭圆标记的 conntrack 之后, 就可以查询得知它的状态了, 所以在后续的 hook 点中都可以使用数据包的状态进行状态匹配. 同样根据 conntrack 还可以得知 NAT 前后的源 IP 与目的 IP, 例如内网客户端通过 NAT 上网, 发起的请求数据包发送到外网服务器前经过 SNAT 源 IP 被修改了, 当服务器收到请求数据包返回响应数据包时, 响应数据包的目的 IP 就是请求数据包的源 IP, 这个数据包到达主机后, 我们不需要对目的 IP 做任何操作它都会神奇地找到内网客户端, 这就是 conntrack 的功劳.

但得到这些功能也是有代价的. 连接跟踪表中能够存放的 conntrack 条目的最大值, 即系统允许的最大连接跟踪数记作 CONNTRACK\_MAX . 当连接特别高频, 比如淘宝双 11 时, 连接数量超过 CONNTRACK\_MAX 将出现丢包现象, 在 k8s 生产环境中业界有关掉 conntrack 的做法. 关于这点下面这篇文章有详细阐述:

[When Linux conntrack is no longer your friend​www.projectcalico.org](https://link.zhihu.com/? target= https%3A//www.projectcalico.org/when-linux-conntrack-is-no-longer-your-friend/)

* * *

### 附录

### TAP/TUN 设备实践

接下来我们来实际用下 TAP/TUN 设备.TAP/TUN 是连接用户程序与内核协议栈的设备, 我们首先得有用户程序. 这里我使用的是 Golang 的 [songgao/water](https://link.zhihu.com/? target= https%3A//github.com/songgao/water) 包的例子, 这个包封装了对 TAP/TUN 设备操作的接口. 示例代码如下:

```
package main

import (
	"log"
	"github.com/songgao/packets/ethernet"
	"github.com/songgao/water"
)

func main() {
	config := water.Config{
		DeviceType: water.TAP,
	}
	// 指定 TAP 网卡的名称为 tap0
	config.Name = "tap0"
	// 创建新的 TAP 网卡
	ifce, err := water.New(config)
	if err != nil {
		log.Fatal(err)
	}
	log.Println("listening from interface: ", config.Name)
	var frame ethernet.Frame
        // 从 tap0 中读取数据包, 并在控制台输出出来数据包的目的 MAC, 源 MAC, 类型以及内容
	for {
		frame.Resize(1500)
		n, err := ifce.Read([]byte(frame))
		if err != nil {
			log.Fatal(err)
		}
		frame = frame[: n]
		log.Printf("Dst: %s Src: %s Ethertype: %x Payload: % x\n",
			frame.Destination(), frame.Source(), frame.Ethertype(), frame.Payload())
	}
}
```

在运行这个程序之前, 我们先看看我们的 Linux 支不支持 TAP 装置:

```
$ modinfo tap
filename:       /lib/modules/5.3.0-1033-aws/kernel/drivers/net/tap.ko
license:        GPL
author:         Sainath Grandhi <sainath.grandhi@intel.com>
author:         Arnd Bergmann <arnd@arndb.de>
srcversion:     7D231ADCC4540DF0A58F0EB
depends:
retpoline:      Y
intree:         Y
name:           tap
vermagic:       5.3.0-1033-aws SMP mod_unload
signat:         PKCS#7
signer:
sig_key:
sig_hashalgo:   md4
```

如果你得到类似的输出, 说明该系统支持 TAP 装置. 因为示例代码会在内核中创建一个名为 tap0 的 TAP 装置, 所以需要用 root 权限运行这段代码, 在运行前记得下载代码的依赖包.

```
$ go get github.com/songgao/packets/ethernet
$ go get github.com/songgao/water
$ sudo go run main.go
2020/09/08 06:31:52 listening from interface:  tap0
```

启动代码, 我们得到一条 log 告诉我们代码已经成功启动, 此时代码正在持续监听 tap0 网卡, 如果此时打断代码的执行, tap0 网卡将被删除, 所以我们再开一个终端, 查看 TAP/TUN 装置:

```
$ ip addr show type tun
14: tap0: <BROADCAST, MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 86:36:52:3e: bf:98 brd ff: ff: ff: ff: ff: ff
```

可以看到示例代码创造出来的 tap0 网卡, 但目前该网卡还没有 IP 地址, 且状态为 DOWN, 为了能给 tap0 传输网络包, 我们给它赋予一个 IP 地址 10.1.0.10/24 并且启动它:

```
$ ip addr add 10.1.0.10/24 dev tap0
$ ip link set dev tap0 up
```

现在当我们在 tap0 的网段发送 ping 广播时, 我们的程序就可以通过 tap0 收到 ping 的数据包, 并打印出来里面的内容:

```
$ ping -c 1 -b 10.1.0.255
WARNING: pinging broadcast address
PING 10.1.0.255 (10.1.0.255) 56(84) bytes of data.

# main.go 输出的日志
2020/09/08 07:13:56 Dst: ff: ff: ff: ff: ff: ff Src: 86: c3:5b: e4: a4:74 Ethertype: 0800 Payload: 45 00 00 54 00 00 40 00 40 01 25 9f 0a 01 00 0a 0a 01 00 ff 08 00 0c 90 1b 37 00 01 34 2f 57 5f 00 00 00 00 77 d6 0e 00 00 00 00 00 10 11 12 13 14 15 16 17 18 19 1a 1b 1c 1d 1e 1f 20 21 22 23 24 25 26 27 28 29 2a 2b 2c 2d 2e 2f 30 31 32 33 34 35 36 37 我们来分析下, 这个 ping 包是怎么到达我们的示例程序的:
```

通过日志我们看到目的 MAC 是 ff: ff: ff: ff: ff: ff, 正是 MAC 广播地址.

首先, 当我们输入 ping -c 1 -b 10.1.0.255 时, 内核根据目标地址和路由表判断数据包应该发向哪里, 我们来看看路由表:

```
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         172.31.32.1     0.0.0.0         UG    100    0        0 eth0
10.1.0.0        0.0.0.0         255.255.255.0   U     0      0        0 tap0
172.31.32.0     0.0.0.0         255.255.240.0   U     0      0        0 eth0
172.31.32.1     0.0.0.0         255.255.255.255 UH    100    0        0 eth0
```

还记得我们刚才给 tap0 赋予 IP 地址并且启用它吗? 在那一步执行完后, 会在路由表里面自动添加一条记录, 表示发往 10.1.0.0/24 的数据包全部经过 tap0 发送, 因此根据这条路由记录, 我们的 ping 包会发向 tap0. 我们知道 tap0 一端连接着主机链路层, 另一端连接着用户空间中的示例程序, 所以示例程序能够通过 tap0 读到链路层的 ping 的数据包.

TUN 装置的实践可以参照 [songgao/water](https://link.zhihu.com/? target= https%3A//github.com/songgao/water) 包的 TUN on MAC 的例子, 基本上与 TAP 的示例类似, 不同的是, 直接 ping TUN 装置的 IP, 示例程序可以数据包, 而直接 ping TAP 装置的 IP, 示例程序收不到数据包.

> 在 TAP 的示例中, 我们使用的是 ping -c 1 -b 10.1.0.255, 采用的广播模式, 并没直接 ping tap0 的 IP 地址 10.1.0.10. 当我执行 ping 10.1.0.10 发现示例程序并不输出 log, 查询发现 superuser 上有个类似的提问: [Sending Packets to tap0 interface](https://link.zhihu.com/? target= https%3A//superuser.com/questions/382008/sending-packets-to-tap0-interface). 有人提到 TAP 装置的输入和输出都必须是完整的链路层数据包, 而 TUN 装置的输入和输出必须是完整的 IP 包.TAP 装置经常和 bridge 一起使用, 借助 bridge 交换机模式将链路层流量 " 交换 " 到 TAP, 而不是通过路由表将 IP 层流量 " 路由 " 到 TAP. 如果需要使用路由表和 IP 包, 可以直接 TUN 装置. (关于这里, 我不是很确定, 查询了很多资料都没有确定的回答, 如果有错误希望大家指出, 在此感谢!)

编辑于 2020-09-09

[

Linux



](//www.zhihu.com/topic/19554300)

[

network



](//www.zhihu.com/topic/20182839)

[

iptables



](//www.zhihu.com/topic/19630401)

​赞同 22​

​3 条评论

​分享

​喜欢​收藏​申请转载
