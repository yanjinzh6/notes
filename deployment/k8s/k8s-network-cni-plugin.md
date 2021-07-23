---
title: calico-bgp-config
date: 2021-03-20 11:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: calico-bgp-config
photo:
---

[k8s-network-cni-plugin](https://network.51cto.com/art/201908/601109.htm)

# 第一次, 如此清晰脱熟的直解 K8S 网络

2019-08-09 17:16 来源: [51cto 博客.](//www.sohu.com)

原标题: 第一次, 如此清晰脱熟的直解 K8S 网络

"

K8S 网络设计与实现是在学习 K8S 网络过程中总结的内容. 本文按照 K8S 网络设计原则, Pod 内部网络, Pod 之间网络等几个步骤讲解 K8S 复杂的网络架构.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/48cd88d8808f407a8d6c74e31bbe1600.jpeg)

图片出自: <你女儿也能看懂的插画版 Kubernetes 指南>

K8S 网络设计原则

K8S 网络设计原则如下:

*   每个 Pod 都拥有一个独立 IP 地址, Pod 内所有容器共享该 IP 地址.
*   集群内所有 Pod 都在一个直接连通的扁平网络中, 可通过 IP 直接访问. 即: 所有容器之间无需 NAT 就可以直接互相访问; 所有 Node 和所有容器之间无需 NAT 就可以直接互相访问; 容器自己看到的 IP 跟其他容器看到的一样.

K8S 网络规范

CNI 是由 CoreOS 提出的一个容器网络规范. 已采纳规范的包括 Apache Mesos, Cloud Foundry, Kubernetes, Kurma 和 RKT.

另外 Contiv Networking, Project Calico 和 Weave 这些项目也为 CNI 提供插件.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/ed1de82d47614ca4ae135e1c7af786b8.jpeg)

CNI 规定了一个容器 Runtime 和网络插件之间的简单的契约. 这个契约通过 JSON 的语法定义了 CNI 插件所需要提供的输入和输出.

CNI 插件提供两个功能:

*   一个用来将网络接口加入到指定网络.
*   另一个用来将其移除.

这两个接口分别在容器被创建和销毁的时候被调用. 容器 Runtime 首先需要获得一个网络命名空间以及一个容器 ID, 然后连同一些 CNI 配置参数传给网络驱动.

接着网络驱动会将该容器连接到网络并将分配的 IP 地址以 JSON 的格式返回给容器 Runtime.

K8S 网络插件要求

K8S 对网络插件的要求总的来讲主要有两个最基本的, 分别是:

*   要能够为每一个 Node 上的 Pod 分配互相不冲突的 IP 地址.
*   要所有 Pod 之间能够互相访问.

K8S 网络实现方案

K8S 网络实现方案有如下几种:

**隧道方案**

隧道方案在 IaaS 层的网络中应用也比较多, 将 Pod 分布在一个大二层的网络规模下. 网络拓扑简单, 但随着节点规模的增长复杂度会提升.

代表方案:

*   Weave: UDP 广播, 本机建立新的 BR, 通过 PCAP 互通.
*   Open vSwitch: 基于 VxLan 和 GRE 协议, 但是性能方面损失比较严重.
*   Flannel: UDP 广播, VxLan.
*   Racher: IPsec.

**路由方案**

路由方案一般是从 3 层或者 2 层实现隔离和跨主机容器互通的, 出了问题也很容易排查.

代表方案:

*   Calico: 基于 BGP 协议的路由方案, 支持很细致的 ACL 控制, 对混合云亲和度比较高.
*   Macvlan: 从逻辑和 Kernel 层来看隔离性和性能优的方案, 基于二层隔离, 所以需要二层路由器支持, 大多数云服务商不支持, 所以混合云上比较难以实现.

K8S Pod 的网络创建流程

K8S Pod 的网络创建流程如下:

*   每个 Pod 除了创建时指定的容器外, 都有一个 Kubelet 启动时指定的基础容器.
*   Kubelet 创建基础容器, 生成 Network Namespace.
*   Kubelet 调用网络 CNIdriver, 根据配置调用具体的 CNI 插件.
*   CNI 插件给基础容器配置网络.
*   Pod 中其他的容器共享基础容器的网络.

Pod 中的网络

Pod 是 K8S 的最小工作单元. 每个 Pod 包含一个或多个容器.K8S 管理的也是 Pod 而不是直接管理容器.Pod 中的容器会作为一个整体被 Master 调度到一个 Node 上运行.

Pod 的设计理念是支持多个容器在一个 Pod 中共享网络地址和文件系统, 可以通过进程间通信和文件共享这种简单高效的方式组合完成服务.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/54d637b6dbc5424fb05da145d32dfb41.jpeg)

一个 Pod 中可以包含多个容器, 而一个 Pod 只有一个 IP 地址. 那么多个容器之间互相访问和访问外网是如何使用这一个 IP 地址呢?

**答案是:**多个容器共享同一个底层的网络命名空间 Net(网络设备, 网络栈, 端口等).

下面以一个小例子说明, 创建一个 Pod 包含两个容器, yaml 文件如下:

apiVersion: apps/v1beta1

kind: Deployment

metadata:

name: Pod-two-container

spec:

replicas: 1

template:

metadata:

labels:

app: nginx

spec:

containers:

\- name: busybox

image: busybox

command:

\- "/bin/sh"

\- "-c"

\- "while true; do echo hello; sleep 1; done"

\- name: nginx

image: nginx

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/336d0921a7d9405dbd12b64f58eb2185.png)

创建 1 个 Pod 中包含 2 个 Container, 实际会创建 3 个 Container. 多出的一个是 "Pause" 容器.

该 Container 是 Pod 的基础容器, 为其他容器提供网络功能.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/ade838834e834e50a9a0b1f4e0f9713b.png)

查看 Pause 容器的基础信息:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/f700c782bfb348beac47367be58cbe67.png)

使用命令 docker inspect 容器 \_ID 查看 Nginx 详细信息, 其网络命令空间使用了 Pause 容器的命名空间, 同样还有进程间通信的命名空间.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/ef9034b10c2946188944f1edde4bab80.png)

再查看 Busybox, 可以发现其网络命令空间使用了 Pause 容器的命名空间, 进程通信的命名空间也是 Pause 容器的命名空间.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/bba02107f32343b496b9a0a8c039b5a5.png)

**实现方式:**Nginx 和 Busybox 之所以能够和 Pause 的命名空间连通是因为 Docker 有一个特性: 能够在创建时使用指定 Docker 的网络命名空间.

在 Docker 的官网上有一段描述:

https: //docs.docker.com /engine/reference/run/

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/7354a6891f134883bff8f0b897aeb01f.png)

所以如果要手动完成一个上面的 Pod, 可以先创建 Pause, 再创建 Nginx 和 Busybox, 同时将网络指定为 Pause 的网络命名空间即可.

docker run \--name pause mirrorgooglecontainers/pause-amd64:3.1

docker run \--name= nginx --network= container: pause nginx

docker run \--name= busybox --network= container: pause busybox

上述步骤由 K8S 帮助我们完成, 所以 Pod 命名空间应该是这样的:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/a4362f8e6c00409ab4b37b466831ab65.png)

讲完了 Pod 内部的网络实现, 我们主要来看 Pod 如何获取 IP 地址, 以及 Pod 与 Pod 之间的通信是怎么样的过程?

Flannel 网络

**Flannel 简介**

Flannel 是 CoreOS 团队针对 Kubernetes 设计的一个网络规划服务, 简单来说, 它的功能是让集群中的不同节点主机创建的 Docker 容器都具有全集群唯一的虚拟 IP 地址.

在默认的 Docker 配置中, 每个节点上的 Docker 服务会分别负责所在节点容器的 IP 分配. 这样导致的一个问题是, 不同节点上容器可能获得相同的 IP 地址.

Flannel 的设计目的就是为集群中的所有节点重新规划 IP 地址的使用规则, 从而使得不同节点上的容器能够获得 " 同属一个内网 " 且 " 不重复的 "IP 地址, 并让属于不同节点上的容器能够直接通过内网 IP 通信.

Flannel 实质上是一种 " 覆盖网络 (overlay network)", 也就是将 TCP 数据包装在另一种网络包里面进行路由转发和通信.

目前已经支持 UDP, Vxlan, Host-gw, Aws-vpc, Gce 和 Alloc 路由等数据转发方式, 默认的节点间数据通信方式是 UDP 转发.

**适用场景:**不需要隔离 Pod, 集群规模小.

**设计思想:**为每一个 Node 节点分配 IP 网段, 使 Node 之间 IP 不重复, Pod 之间直接使用 IP 访问.

**设计优势:**网络模型简单, 安装配置相对容易, 成熟度高, 适合大多数用例的环境.

**Flannel 对网络要求提出的解决办法**

互相不冲突的 IP:

*   Flannel 利用 Kubernetes API 或者 etcd 用于存储整个集群的网络配置, 根据配置记录集群使用的网段.
*   Flannel 在每个主机中运行 Flanneld 作为 Agent, 它会为所在主机从集群的网络地址空间中, 获取一个小的网段 Subnet, 本主机内所有容器的 IP 地址都将从中分配.

如测试环境中 IP 分配:

①Master 节点

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/1d5e77a21a4a46eca7f9f7333662144d.png)

②Node1

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/9d8f7cb909714d42a2c8ba2b4a91ff12.png)

③Node2

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/87c2a48de6804c0b9782876160e8d5e6.png)

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/f44c37b36f6b4e239c3198175447ef29.png)

在 Flannel Network 中, 每个 Pod 都会被分配唯一的 IP 地址, 且每个 K8S Node 的 Subnet 各不重叠, 没有交集.

Pod 之间互相访问:

*   Flanneld 将本主机获取的 Subnet 以及用于主机间通信的 Public IP 通过 etcd 存储起来, 需要时发送给相应模块.
*   Flannel 利用各种 Backend 机制, 例如 UDP, Vxlan 等等, 跨主机转发容器间的网络流量, 完成容器间的跨主机通信.

**Flannel 架构原理**

Flannel 架构图:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/eb31544f4cd240b4a067a2cd1641559a.png)

各个组件的解释如下:

**Cni0:**网桥设备, 每创建一个 Pod 都会创建一对 Veth Pair. 其中一端是 Pod 中的 eth0, 另一端是 cni0 网桥中的端口 (网卡).

Pod 中从网卡 eth0 发出的流量都会发送到 cni0 网桥设备的端口 (网卡) 上.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/a6ca4ccf4dc9476c8baf51ce25bfb9cb.png)

**注:**cni0 设备获得的 IP 地址是该节点分配到的网段的第一个地址.

**Flannel.1:**Overlay 网络的设备, 用来进行 Vxlan 报文的处理 (封包和解包). 不同 Node 之间的 Pod 数据流量都从 Overlay 设备以隧道的形式发送到对端.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/9221c1d4588846b9ab76af824ec8d2e7.png)

**Flanneld:**Flannel 在每个主机中运行 Flanneld 作为 Agent, 它会为所在主机从集群的网络地址空间中, 获取一个小的网段 Subnet, 本主机内所有容器的 IP 地址都将从中分配.

同时 Flanneld 监听 K8S 集群数据库, 为 Flannel.1 设备提供封装数据时必要的 Mac, IP 等网络数据信息.

不同 Node 上的 Pod 的通信流程:

*   Pod 中产生数据, 根据 Pod 的路由信息, 将数据发送到 cni0.
*   cni0 根据节点的路由表, 将数据发送到隧道设备 Flannel.1.
*   Flannel.1 查看数据包的目的 IP, 从 Flanneld 获得对端隧道设备的必要信息, 封装数据包.
*   Flannel.1 将数据包发送到对端设备. 节点的网卡接收到数据包, 发现数据包为 Overlay 数据包, 解开外层封装, 并发送内层封装到 Flannel.1 设备.
*   Flannel.1 设备查看数据包, 根据路由表匹配, 将数据发送给 cni0 设备.
*   cni0 匹配路由表, 发送数据给网桥上对应的端口.

**通信流程**

**①Pod1 中的容器到 cni0**

Pod1 与 Pod3 能够互相 Ping 通:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/9ceb96f9c2c24aeca7c96849865fba35.png)

Ping 包的 Dst IP 为 192.20.1.43, 根据路由匹配到最后一条路由表项, 去往 192.20.0.0/12 的包都转发给 192.20.0.1.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/2e7e6c504f49451e86a9372a26463ebf.png)

192.20.0.1 为 cni0 的 IP 地址.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/1a281e7cec7942de8ae05ec915d97487.png)

**②cni0 到 Flannel1.1**

当 Icmp 包达到 cni0 之后, cni0 发现 Dst 为 192.20.1.43, CNI 根据主机路由表来查找匹配项.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/f7b8c31ff5024d88b0ac4ed159b0efb9.png)

根据最小匹配原则, 匹配到图上的一条路由表项. 去往 192.20.1.0/24 网段的包, 发送 192.20.1.0 网关, 网关设备是 Flannel.1.

**③Flannel.1**

**内层封装:**Flannel.1 为 Vxlan 隧道端点, 当数据包来到 Flannel.1 时, 需要将数据包封装起来. 此时:

*   源 IP src ip 为 192.20.0.51.
*   目的 IP dst ip 为 192.20.1.43.

数据包继续封装需要知道目的 IP 192.20.1.43, IP 地址对应的 Mac 地址. 此时, Flannel.1 不会发送 ARP 请求去获得目的 IP 的 Mac 地址, 而是将请求发送到用户空间的 Flanned 程序.

Flanned 程序收到内核的请求事件之后, 从 etcd 查找能够匹配该地址的子网的 Flannel.1 设备的 Mac 地址, 即发往的 Pod 所在 Host 中 Flannel.1 设备的 Mac 地址.

Flannel 在为 Node 节点分配 IP 网段时记录了所有的网段和 Mac 等信息. 该过程交互流程如下图所示:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/96b4e7f2319140afb7612f9124a85e26.png)

Flanned 将查询到的信息放入 Master 节点的 ARP Cache 表中:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/ca791cb437a54cd6a00d029259f4216b.png)

到这里, Vxlan 的内层数据包就完成了封装. 格式是这样的:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/b3cf8b900b3545c2aca3b8f76e0cc06b.png)

简单总结这个流程:

*   数据包到达 Flannel.1, 通过查找路由表, 知道数据包要通过 Flannel.1 发往 192.20.1.0.
*   通过 ARP Cache 表, 知道了目的 IP 192.20.1.0 的 Mac 地址.

**外层封装:**此时内层封装已经准备好, 需要找到 Vxlan 的外层封装.Kernel 需要查看 Node 上的 FDB(forwarding database) 以获得内层封包中目的 Vtep 设备所在的 Node 地址.

因为已经从 ARP Table 中查到目的设备 Mac 地址为 52:77:71: e6:4f:58, 同时在 FDB 中存在该 Mac 地址对应的 Node 节点的 IP 地址.

如果 FDB 中没有这个信息, 那么 Kernel 会向用户空间的 Flanned 程序发起 "L2 MISS" 事件.Flanneld 收到该事件后, 会查询 etcd, 获取该 Vtep 设备对应的 Node 的 "Public IP", 并将信息注册到 FDB 中.

当内核获得了发往机器的 IP 地址后, ARP 得到 Mac 地址, 之后就能完成 Vxlan 的外层封装.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/ebe9918300a84e4980eefcd051026130.png)

**④对端 Flannel.1**

Node 节点的 eth0 网卡接收到 Vxlan 设备包, Kernal 将识别出这是一个 Vxlan 包, 将包拆开之后转给节点上的 Flannel.1 设备.

这样数据包就从发送节点到达目的节点, Flannel.1 设备将接收到一个如下的数据包:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/cf97533747b04d0ba38bbae7d5d5ac4c.png)

目的地址为 192.20.1.43, Flannel.1 查找自己的路由表, 根据路由表完成转发.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/205ea6554ede4f4494ee01e00a42d072.png)

根据匹配原则, Flannel.1 将去往 192.20.1.0/24 的流量转发到 cni0 上去.

**⑤cnio 到 Pod**

cni0 是一个网桥设备. 当 cni0 拿到数据包之后, 通过 Veth Pair, 将数据包发送给 Pod. 查看 Node 节点中的网桥.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/d95c479160d9432faac4b8067e3d9fc4.png)

在 Node 节点上通过 ARP 解析可以开出,192.20.1.43 的 Mac 地址为 66:57:8e:3d:00:85:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/642143cb833d4f80ae0703f5dbe07525.png)

该地址为 Pod 的网卡 eth0 的地址.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/0d75490c9c8e476c9ece939dc33e41ad.png)

同时通过 Veth Pair 的配对关系可以看出, Pod 中的 eth0 是 Veth Pair 的一端, 另一端在 Node 节点行上, 对应的网卡是 vethd356ffc1@if3:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/980fc3dfad3a42e38be0057ffc87cdff.png)

所以, 在 cni0 网桥上挂载的 Pod 的 Veth Pair 为 vethd356ffc1, 即:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/034f3e20324f4e088e028034aab4f221.png)

eth0@if50 和 vethd356ffc1@if3 组成的一对 Veth Pair. 其效果相当于将 Pod 中的 eth0 直接插在到 cni0 上.

简单总结 cni0 转发流量的原理:

*   首先通过 ARP 查找出 IP 地址对应的 Mac 地址.
*   将流量转发给 Mac 地址所在 eth0 网的对应的 Veth Pair 端口.
*   Veth Pair 端口接收到流量, 直接将流量注入到 Pod 的 eth0 网卡上.

总结 Flannel 的特点:

*   使集群中的不同 Node 主机创建的 Docker 容器都具有全集群唯一的虚拟 IP 地址.
*   建立一个覆盖网络 (overlay network), 通过这个覆盖网络, 将数据包原封不动的传递到目标容器.
*   创建一个新的虚拟网卡 Flannel0 接收 Docker 网桥的数据, 通过维护路由表, 对接收到的数据进行封包和转发.
*   etcd 保证了所有 Node 上 Flanned 所看到的配置是一致的. 同时每个 Node 上的 Flanned 监听 etcd 上的数据变化, 实时感知集群中 Node 的变化.

**不同后端的封装**

Flannel 可以指定不同的转发后端网络, 常用的有 Hostgw, UDP, Vxlan 等, 上述使用的就是 Vxlan 网络.

**①Hostgw:**是简单的 Backend, 它的原理非常简单, 直接添加路由, 将目的主机当做网关, 直接路由原始封包.

例如, 我们从 etcd 中监听到一个 EventAdded 事件 Subnet 为 10.1.15.0/24 被分配给主机 Public IP 192.168.0.100.

Hostgw 要做的工作就是在本主机上添加一条目的地址为 10.1.15.0/24, 网关地址为 192.168.0.100, 输出设备为上文中选择的集群间交互的网卡即可.

**优点:**简单, 直接, 效率高.

**缺点:**要求所有的 Pod 都在一个子网中, 如果跨网段就无法通信.

**②UDP:**如何应对 Pod 不在一个子网里的场景呢? 将 Pod 的网络包作为一个应用层的数据包, 使用 UDP 封装之后在集群里传输, 即 Overlay.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/d624c9766e2d423ab94433938e7ec97e.png)

上图来自 Flannel 官方, 其中右边 Packer 的封装格式就是使用 UDP 完成 Overlay 的格式.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/226d387673fe43aca4573a757d120964.png)

当容器 10.1.15.2/24 要和容器 10.1.20.2/24 通信时:

*   因为该封包的目的地不在本主机 Subnet 内, 因此封包会首先通过网桥转发到主机中.
*   在主机上经过路由匹配, 进入网卡 Flannel.1.(需要注意的是 Flannel.1 是一个 Tun 设备, 它是一种工作在三层的虚拟网络设备, 而 Flanneld 是一个 Proxy, 它会监听 Flannel.1 并转发流量.)
*   当封包进入 Flannel.1 时, Flanneld 就可以从 Flanne.1 中将封包读出, 由于 Flanne.1 是三层设备, 所以读出的封包仅仅包含 IP 层的报头及其负载.
*   最后 Flanneld 会将获取的封包作为负载数据, 通过 UDP Socket 发往目的主机.
*   在目的主机的 Flanneld 会监听 Public IP 所在的设备, 从中读取 UDP 封包的负载, 并将其放入 Flannel.1 设备内.
*   容器网络封包到达目的主机, 之后就可以通过网桥转发到目的容器了.

**优点:**Pod 能够跨网段访问.

**缺点:**隔离性不够, UDP 不能隔离两个网段.

**③Vxlan:**和上文提到的 udpbackend 的封包结构是非常类似的, 不同之处是多了一个 vxlanheader, 以及原始报文中多了个二层的报头.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/a471bf54fe4c43d59cf07c3ef7a61165.png)

当初始化集群里, Vxlan 网络的初始化工作: 主机 B 加入 Flannel 网络时, 它会将自己的三个信息写入 etcd 中, 分别是: Subnet10.1.16.0/24, Public IP 192.168.0.101, Vtep 设备 Flannel.1 的 Mac 地址 MAC B.

之后, 主机 A 会得到 EventAdded 事件, 并从中获取上文中 B 添加至 etcd 的各种信息.

这个时候, 它会在本机上添加三条信息:

*   路由信息: 所有通往目的地址 10.1.16.0/24 的封包都通过 Vtep 设备 Flannel.1 设备发出, 发往的网关地址为 10.1.16.0, 即主机 B 中的 Flannel.1 设备.
*   FDB 信息: MAC 地址为 MAC B 的封包, 都将通过 Vxlan 发往目的地址 192.168.0.101, 即主机 B.
*   ARP 信息: 网关地址 10.1.16.0 的地址为 MAC B.

事实上, Flannel 只使用了 Vxlan 的部分功能, 由于 VNI 被固定为 1, 本质上工作方式和 UDP Backend 是类似的, 区别无非是将 UDP 的 Proxy 换成了内核中的 Vxlan 处理模块.

而原始负载由三层扩展到了二层, 但是这对三层网络方案 Flannel 是没有意义的, 这么做也仅仅只是为了适配 Vxlan 的模型.

**存在问题**

存在的问题如下:

*   不支持 Pod 之间的网络隔离.Flannel 设计思想是将所有的 Pod 都放在一个大的二层网络中, 所以 Pod 之间没有隔离策略.
*   **设备复杂, 效率不高.**Flannel 模型下有三种设备, 数量经过多种设备的封装, 解析, 势必会造成传输效率的下降.


! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/b26ce3c94b874239bd7bf742245fbd11.jpeg)

Calico 网络

**Calico 简介**

Calico 是一种开源网络和网络安全解决方案, 适用于容器, 虚拟机和基于主机的本机工作负载.

Calico 支持广泛的平台, 包括 Kubernetes, OpenShift, Docker EE, OpenStack 和裸机服务.

在虚拟化平台中, 比如 OpenStack, Docker 等都需要实现 Workloads 之间互连, 但同时也需要对容器做隔离控制, 就像在 Internet 中的服务仅开放 80 端口, 公有云的多租户一样, 提供隔离和管控机制.

而在多数的虚拟化平台实现中, 通常都使用二层隔离技术来实现容器的网络, 这些二层的技术有一些弊端, 比如需要依赖 VLAN, Bridge 和隧道等技术.

其中 Bridge 带来了复杂性, Vlan 隔离和 Tunnel 隧道则消耗更多的资源并对物理环境有要求, 随着网络规模的增大, 整体会变得越加复杂.

Calico 是一个纯三层的方案, 把每个 Node 节点认为是一个路由器, 然后把所有的容器认为是连在这个路由器上的网络终端, 在路由器之间跑标准的路由协议——BGP 的协议, 然后让它们自己去学习这个网络拓扑该如何转发.

Calico 为每个主机分配了一段子网作为容器可分配的 IP 范围, 这样就可以根据子网的 CIDR 为每个主机生成比较固定的路由规则, Pod 发送另一个主机的流量能够根据路由规则匹配而被转发.

**适用场景:**集群规模较大, 环境中的 Pod 之间需要隔离.

**设计思想:**Calico 不使用隧道或 NAT 来实现转发, 而是巧妙的把所有二三层流量转换成三层流量, 并通过 Host 上路由配置完成跨 Host 转发.

设计优势如下:

**①更优的资源利用:**二层网络通讯需要依赖广播消息机制, 广播消息的开销与 Host 的数量呈指数级增长, Calico 使用的三层路由方法, 则完全抑制了二层广播, 减少了资源开销.

另外, 二层网络使用 VLAN 隔离技术, 天生有 4096 个规格限制, 即便可以使用 Vxlan 解决, 但 Vxlan 又带来了隧道开销的新问题. 而 Calico 不使用 Vlan 或 Vxlan 技术, 使资源利用率更高.

**②可扩展性:**Calico 使用与 Internet 类似的方案, Internet 的网络比任何数据中心都大, Calico 同样天然具有可扩展性.

**③简单而更容易 Debug:**因为没有隧道, 意味着 Workloads 之间路径更短更简单, 配置更少, 在 Host 上更容易进行 Debug 调试.

**④更少的依赖:**Calico 仅依赖三层路由可达.

**⑤可适配性:**Calico 较少的依赖性使它能适配所有 VM, Container, 白盒或者混合环境场景.

**Calico 架构原理**

架构图如下:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/adf74d675fb14541b23d4944e4d6593c.png)

Calico 网络模型主要工作组件:

*   Felix: 运行在每一台 Host 的 Agent 进程, 主要负责网络接口管理和监听, 路由, ARP 管理, ACL 管理和同步, 状态上报等.
*   etcd: 分布式键值存储, 主要负责网络元数据一致性, 确保 Calico 网络状态的准确性, 可以与 Kubernetes 共用.
*   BGP Client(BIRD): Calico 为每一台 Host 部署一个 BGP Client, 使用 BIRD 实现, BIRD 是一个单独的持续发展的项目, 实现了众多动态路由协议比如 BGP, OSPF, RIP 等. 在 Calico 的角色是监听 Host 上由 Felix 注入的路由信息, 然后通过 BGP 协议广播告诉剩余 Host 节点, 从而实现网络互通.
*   BGP Route Reflector(BIRD): 在大型网络规模中, 如果仅仅使用 BGP Client 形成 Mesh 全网互联的方案就会导致规模限制, 因为所有节点之间俩俩互联, 需要 N^2 个连接. 为了解决这个规模问题, 可以采用 BGP 的 RouterReflector 的方法, 使所有 BGPClient 仅与特定 RR 节点互联并做路由同步, 从而大大减少连接数.

**Felix:**会监听 ectd 中心的存储, 从它获取事件, 比如说用户在这台机器上加了一个 IP, 或者是创建了一个容器等.

用户创建 Pod 后, Felix 负责将其网卡, IP, MAC 都设置好, 然后在内核的路由表里面写一条, 注明这个 IP 应该到这张网卡.

同样如果用户制定了隔离策略, Felix 同样会将该策略创建到 ACL 中, 以实现隔离.

**BIRD:**是一个标准的路由程序, 它会从内核里面获取哪一些 IP 的路由发生了变化, 然后通过标准 BGP 的路由协议扩散到整个其他的宿主机上, 让其他 Node 节点都知道这个 IP 在这里, 生成路由条目就比较方便.

由于 Calico 是一种纯三层的实现, 因此可以避免与二层方案相关的数据包封装的操作, 中间没有任何的 NAT, 没有任何的 Overlay.

所以它的转发效率可能是所有方案中最高的, 因为它的包直接走原生 TCP/IP 的协议栈, 它的隔离也因为这个栈而变得好做.

因为 TCP/IP 的协议栈提供了一整套的防火墙的规则, 所以它可以通过 IPTABLES 的规则达到比较复杂的隔离逻辑.

**C****alico 网络 Node 之间两种网络**

**IPIP:**是把一个 IP 数据包放在在一个 IP 包里, 即把 IP 层封装到 IP 层的一个 Tunnel.

它的作用基本上就相当于一个基于 IP 的网桥! 一般来说, 普通的网桥是基于 Mac 的, 根本不需 IP.

而 IPIP 则是通过两端的路由做一个 Tunnel, 把两个本来不通的网络通过点对点连接起来.

**BGP:**边界网关协议 (Border Gateway Protocol, BGP) 是互联网上一个核心的去中心化自治路由协议.

它通过维护 IP 路由表或 ' 前缀 ' 表来实现自治系统 (AS) 之间的可达性, 属于矢量路由协议.

BGP 不使用传统的内部网关协议 (IGP) 的指标, 而使用基于路径, 网络策略或规则集来决定路由.

**IPIP 工作模式**

**①测试环境**

一个 Msater 节点, IP 172.171.5.95, 一个 Node 节点 IP 172.171.5.96:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/e64b708762414795bd9aaec43aba0141.png)

创建一个 Daemonset 的应用, Pod1 落在 Master 节点上 IP 地址为 192.168.236.3, Pod2 落在 Node 节点上 IP 地址为 192.168.190.203:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/6bbc63d97b07463899d53aa871c5103f.png)

Pod1 Ping Pod2:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/0068b9c7efd74d19b9a4ce0d491d262d.jpeg)

**②Ping 包旅程**

Pod1 上的路由信息:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/626d4dd2f92f4b0bb592f3a06cff5122.png)

根据路由信息, Ping 192.168.190.203, 会匹配到第一条. 第一条路由的意思是: 去往任何网段的数据包都发往网管 169.254.1.1, 然后从 eth0 网卡发送出去.

路由表中 Flags 标志的含义:

*   U: UP 表示当前为启动状态.
*   H: Host 表示该路由为一个主机, 多为达到数据包的路由.
*   G: Gateway 表示该路由是一个网关, 如果没有说明目的地是直连的.
*   D: Dynamicaly 表示该路由是重定向报文修改.
*   M: 表示该路由已被重定向报文修改.

Master 节点上的路由信息:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/578dc2d68ca443b583f8b8e74993ee3b.png)

当 Ping 包来到 Master 节点上, 会匹配到路由 tunl0. 该路由的意思是: 去往 192.169.190.192/26 的网段的数据包都发往网关 172.171.5.96.

因为 Pod1 在 5.95, Pod2 在 5.96. 所以数据包就通过该路由发往到 Node 节点上.

Node 节点上路由信息:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/d5abf9e93bb24df78ff95411cb97ec05.png)

当 Node 节点网卡收到数据包之后, 发现发往的目的 IP 为 192.168.190.203, 于是匹配到红线的路由 (绿线是 Node 结点对应的发往 Master 的路由).

**该路由的意思是:**192.168.190.203 是本机直连设备, 去往设备的数据包发往 caliadce112d250. 这个设备就是 Pod2 的 Veth Pair 的一端.

在创建 Pod2 时 Calico 会给 Pod2 创建一个 Veth Pair 设备. 一端是 Pod2 的网卡, 另一端就是我们看到的 caliadce112d250.

可以通过如下方式验证: 在 Pod2 中安装 ethtool 工具, 然后使用 ethtool-S eth0, 查看 Veth Pair 另一端的设备号.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/30c1e3cf3eca4674a3bf97342f5c827e.png)

Pod2 网卡另一端的设备编号是 18, 在 Node 上查看编号为 18 的网络设备, 可以发现该网络设备就是 caliadce112d250.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/b55710ea4aee4de4bd1e08db8dc3b7f0.png)

所以, Node 上的路由, 发送 caliadce112d250 的数据其实就是发送到 Pod2 的网卡中.Ping 包的旅行到这里就到了目的地.

查看一下 Pod2 中的路由信息, 发现该路由信息和 Pod1 中是一样的.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/7c0996317ccd44ffbdfa459e01ffa708.png)

顾名思义, IPIP 网络就是将 IP 网络封装在 IP 网络里.IPIP 网络的特点是所有 Pod 的数据流量都从隧道 tunl0 发送, 并且在 tunl0 这增加了一层传输层的封包.

在 Master 网卡上抓包分析该过程:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/e9cbbb2caf744a3899681343f34e43b3.png)

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/1e71878b9eab41ff92495bc1366ee17a.jpeg)

打开 ICMP 285, Pod1 Ping Pod2 的数据包, 能够看到该数据包一共 5 层, 其中 IP 所在的网络层有两个, 分别是 Pod 之间的网络和主机之间的网络封装.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/f0b4a09bc78646919d150d74b5331e25.png)

根据数据包的封装顺序, 应该是在 Pod1 Ping Pod2 的 ICMP 包外面多封装了一层主机之间的数据包.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/f9d0b0a258f747a587c7a15d2f6821d1.png)

之所以要这样做是因为 tunl0 是一个隧道端点设备, 在数据到达时要加上一层封装, 便于跨网段访问.

两层 IP 封装的具体内容:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/f0be7fcb8d494203ba0bfe34deecf0de.jpeg)

IPIP 的连接方式:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/002a9038e0d94f3b89288273cde0244f.png)

**BGP 工作模式**

**①修改配置**

在安装 Calico 网络时, 默认安装是 IPIP 网络.calico.yaml 文件中, 将 CALICO\_IPV4POOL\_IPIP 的值修改成 "off", 就能够替换成 BGP 网络.

**②对比**

BGP 网络相比较 IPIP 网络, 最大的不同之处就是没有了隧道设备 tunl0, 且不通过隧道设备发送流量.

前面介绍过 IPIP 网络 Pod 之间的流量发送 tunl0, 然后 tunl0 发送对端设备.BGP 网络中, Pod 之间的流量直接从网卡发送目的地, 减少了 tunl0 这个环境.

Master 节点上路由信息. 从路由信息来看, 没有 tunl0 设备.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/ab72342e174a47348d80137083367157.png)

同样创建一个 Daemonset, Pod1 在 Master 节点上, Pod2 在 Node 节点上.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/6adae440e7ee4ca787f8ffd479105bbb.png)

**③Ping 包旅程**

Pod1 Ping Pod2:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/36f2921541d048758ca5b5b7e49b5f35.png)

根据 Pod1 中的路由信息, Ping 包通过 eth0 网卡发送到 Master 节点上.

Master 节点上路由信息. 根据匹配到的 192.168.190.192 路由, 该路由的意思是: 去往网段 192.168.190.192/26 的数据包, 发送网段 172.171.5.96.

而 5.96 就是 Node 节点. 所以, 该数据包直接发送了 5.96 机器:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/d4baba4f65f546dd8a5607bb6b839d62.png)

Node 节点上的路由信息. 根据匹配到的 192.168.190.192 的路由, 数据将发送给 cali6fcd7d1702e 设备, 该设备和上面分析的是一样, 为 Pod2 的 Veth Pair 的一端. 数据就直接发送给 Pod2 的网卡.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/46c8eac79a9b49b8b97a410cc2c2f1f5.png)

当 Pod2 对 Ping 包做出回应之后, 数据到达 Node 节点上, 匹配到 192.168.236.0 的路由.

该路由说的是: 去往网段 192.168.236.0/26 的数据, 发送给网关 172.171.5.95. 数据包就直接通过网卡 ens160, 发送到 Master 节点上.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/ceb8617b6afb425c8b8d97abe7f1663a.png)

通过在 Master 节点上抓包, 查看经过的流量, 筛选出 ICMP, 找到 Pod1 Ping Pod2 的数据包.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/c44d58c140a847f08e1a624b47ecfcf3.png)

可以看到 BGP 网络下, 没有使用 IPIP 模式, 数据包是正常的封装.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/b125e711019b4728be7af4ac89a07ed8.png)

值得注意的是 Mac 地址的封装.192.168.236.0 是 Pod1 的 IP,192.168.190.198 是 Pod2 的 IP.

而源 Mac 地址是 Master 节点网卡的 Mac, 目的 Mac 是 Node 节点的网卡的 Mac.

这说明, 在 Master 节点的路由接收到数据, 重新构建数据包时, 使用 ARP 请求, 将 Node 节点的 Mac 拿到, 然后封装到数据链路层.

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/cbc69643db3b4cce9ce9a7da5654368f.png)

BGP 的连接方式如下图:

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/da612af703f4495593f9fef034a6facb.png)

**网络对比**

IPIP 网络:

*   流量: tunlo 设备封装数据, 形成隧道, 承载流量.
*   适用网络类型: 适用于互相访问的 Pod 不在同一个网段中, 跨网段访问的场景. 外层封装的 IP 能够解决跨网段的路由问题.
*   效率: 流量需要 tunl0 设备封装, 效率略低.

BGP 网络:

*   流量: 使用路由信息导向流量.
*   适用网络类型: 适用于互相访问的 Pod 在同一个网段, 跨网段需要上行交换机或路由器支持. 适用于大型网络.
*   效率: 原生 HostGW, 效率高.

**存在问题**

**①租户隔离问题**

Calico 的三层方案是直接在 Host 上进行路由寻址, 那么对于多租户如果使用同一个 CIDR 网络就面临着地址冲突的问题.

**②路由规模问题**

通过路由规则可以看出, 路由规模和 Pod 分布有关, 如果 Pod 离散分布在 Host 集群中, 势必会产生较多的路由项.

**③IPtables 规则规模问题**

1 台 Host 上可能虚拟化十几或几十个容器实例, 过多的 IPtables 规则造成复杂性和不可调试性, 同时也存在性能损耗.

**④跨子网时的网关路由问题**

当对端网络不为二层可达时, 需要通过三层路由时, 需要网关支持自定义路由配置, 即 Pod 的目的地址为本网段的网关地址, 再由网关进行跨三层转发.

K8S 网络方案对比

以下对比为摘录内容:

http: //www.sohu.com/a/256113338\_764649

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/86f6fd5827b641a6b292cbdb20fddd63.jpeg)

! [](http://5b0988e595225.cdn.sohucs.com/images/20190810/68a74985f3db4cac847944643422cd52.jpeg)

作者: 李金葵, 云计算开发工程师

编辑: 陶家龙, 孙淑娟

征稿: 有投稿, 寻求报道意向技术人请联络 editor@51cto.com[返回搜狐, 查看更多](//www.sohu.com/? strategyid=00001  " 点击进入搜狐首页 ")

责任编辑:

声明: 该文观点仅代表作者本人, 搜狐号系信息发布平台, 搜狐仅提供信息存储空间服务.

阅读 ()
