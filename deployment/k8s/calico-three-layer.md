---
title: calico-three-layer
date: 2021-03-14 18:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: calico-three-layer
photo:
---

https://jinrunsen.com/post/%E9%A1%B9%E7%9B%AE/2020-06/%E9%9D%9E%E9%9B%86%E7%BE%A4%E4%B8%BB%E6%9C%BA%E8%AE%BF%E9%97%AE%E9%9B%86%E7%BE%A4%E5%AE%B9%E5%99%A8%E7%9A%84%E7%BD%91%E7%BB%9C%E6%96%B9%E6%A1%88/

# 基于 Calico 的集群内外网络互通方案

2020-06-16

[网络](/categories/%E7%BD%91%E7%BB%9C/) [云原生](/categories/%E4%BA%91%E5%8E%9F%E7%94%9F/)

## 文章目录

*   [背景](#背景)
*   [Calico](#calico)
    *   [简介](#简介)
    *   [容器网络](#容器网络)
    *   [跨网段访问](#跨网段访问)
*   [主机不在集群内的方案](#主机不在集群内的方案)
    *   [实践](#实践)
    *   [校验与排查](#校验与排查)
*   [后续](#后续)

## 背景

在迁移服务到集群上时, 不会一次性将所有业务都迁移上去. 但集群内外的服务依然需要基于 IP 的网络连通访问. 这就需要集群外的主机能访问容器网络段.

经过调研与实践, 我们可以使用集群现在在使用的 `CNI` 插件 `Calico` 来实现这一需求.

## Calico

Calico 是一个基于 BGP 的纯三层的数据中心网络方案 (也支持 overlay 网络). 可以作为 Kubernetes CNI 插件安装在集群中来管理容器网络. 还支持在物理机上部署与其他平台通信. 基于此功能, 可以满足我们的需求, 在非集群主机中访问集群容器网络中的服务.

### 简介

Calico 作为容器网络方案与其它方案最大的不同是它没有采用 overlay 网络做报文的转发, 而是提供了纯 3 层的网络模型. 三层通信模型表示每个容器都通过 IP 直接通信, 中间通过路由转发找到对方. 在这个过程中, 容器所在的节点类似于传统的路由器, 提供了路由查找的功能. 要想路由能够正常工作, 每个容器所在的主机节点扮演了虚拟路由器 (vRouter) 的功能, 而且这些 vRouter 必须有某种方法, 能够知道整个集群的路由信息.

这个需求与传统网络的路由发现选择类似, Calico 采用了标准的 BGP 边界网关协议来做路由发现与选择. 由于采用的是标准协议, Calico 模拟路由器的路由表信息可以被传播到网络的其他路由设备中, 这样就实现了在三层网络上的高速跨节点网络.

现实中的网络并不总是支持 BGP 路由, 因此 Calico 也设计了一种 ipip 模式, 使用 overlay 的方式传输数据.ipip 的包头非常小, 而且是内置在内核中的, 因此它的速度理论上要比 VXLAN 快, 但是安全性更差.

除了网络连接, 网络策略是 Calico 最受追捧的功能之一. 使用 Calico 的策略语言, 可以实现对容器, 虚拟机工作负载和裸机主机各节点之间网络通信进行细粒度和动态的安全规则控制.Calico 基于 iptables 实现了 Kubernetes 的网络策略, 通过在各个节点上应用 ACL(访问控制列表) 提供工作负载的多租户隔离, 安全组及其他可达性限制等功能. 此外, Calico 还可以与服务网格 Istio 集成, 以便在服务网格层和网络基础架构层中解释和实施集群内工作负载的网络策略.

### 容器网络

Calico 会在主机上为每一个容器的 IP 配置它的访问路由, 再使用 BGP 协议, 将自己的路由信息传播出去, 其它主机收到后再将路由写入自己的路由表中. 主机与容器, 容器与容器可以只靠路由表就能互相访问了, 不需要任何的 overlay 网络.

! [20200612142355](http://gasxia.oss-cn-shanghai.aliyuncs.com/markdown/5006252f215bdb11fb1f9949117b1040.png)

20200612142355

#### 动态管理

K8S 集群会不断的创建, 删除容器, 节点也可能不断的加入, 退出.

每个容器的创建与删除需要在一台节点上做一次路由更改操作.

每添加一个节点需要通知所有节点添加一条路由, 该节点本身还要同步所有路由.

如此复杂的操作, 不适合手动配置, 需要在每个节点上启动 agent 来做操作.Calico 在主机上的 agent 叫做 Felix.

! [20200615112148](http://gasxia.oss-cn-shanghai.aliyuncs.com/markdown/b72e6ede18850ee387c5fa166ae4d7c1.png)

20200615112148

#### 路由广播

Calico 使用了 `BGP` 路由协议, 来自动地在整个集群中分发路由信息.

BGP 本身是一个 Linux 内核原生支持的, 专门用在大规模数据中心里维护不同的 " 自治系统 " 之间路由信息的, 无中心的路由协议.

### 跨网段访问

主机不在同一个网段, 中间就会经过路由器, 路由器上是不会有关于容器网络的路由表的. 所以得用其他方式来解决.

#### IPIP

在 2 个主机之间使用 IPIP 技术打通一个隧道, 隧道在两个主机上各有一个端点, 流量会在端点上进行封装, 将容器的 IP 作为乘客协议放在隧道里面, 而物理主机的 IP 放在外面作为承载协议.

! [20200612142229](http://gasxia.oss-cn-shanghai.aliyuncs.com/markdown/d343d1dab879913a16196e9a8d26f0a4.png)

20200612142229

! [20200612142339](http://gasxia.oss-cn-shanghai.aliyuncs.com/markdown/7ecb69d4821439a5d47a7b67100f5d7d.png)

20200612142339

#### 配置路由器

在自己可以控制路由器的情况下, 可以将路由器也加入 BGP 协议节点中, 与集群主机同步路由表. 这样就避免了上述方案中的封包带来的性能损耗.

! [20200612143254](http://gasxia.oss-cn-shanghai.aliyuncs.com/markdown/0a700ccdfa5b55802ed5528366728282.png)

20200612143254

## 主机不在集群内的方案

主机不在集群内时, 很大概率也不在一个网段内. 所以在处理这个问题时, 得先理解上述方案来处理跨网段问题.

Calico 是支持在集群外主机上来部署服务的, 而且其使用的 BGP 协议 本身就是标准通用的.

在集群外主机上手动部署好了之后, 配置集群的访问权限, 就可以与集群通信. 但是 Calico 的服务是通过监听 `Kubernetes Node` 资源来创建 BGP 的客户端节点的.

此时 `Kubernetes Node` 是没有这个集群外主机的, 也就不会接收来自集群外主机的 BGP 连接请求.

解决方案是, 手动在集群内创建非集群主机的 `node` 资源, 也不需要在主机上添加任何 k8s 组件. 只是作为一条数据, 供 calico 使用.

### 实践

#### 前提

*   集群已部署好
*   集群的网络插件是 Calico

#### 主机上安装 calico

[官方教程](https://docs.projectcalico.org/getting-started/bare-metal/about)

##### 创建环境变量文件

创建 `/etc/calico/calico.env` 环境变量文件:

<table class="lntable"> <tbody> <tr> <td class="lntd"> <pre class="chroma"> <code> <span class="lnt">1
</span> <span class="lnt">2
</span> <span class="lnt">3
</span> <span class="lnt">4
</span> <span class="lnt">5
</span> <span class="lnt">6
</span> <span class="lnt">7
</span> <span class="lnt">8
</span> </code> </pre> </td> <td class="lntd"> <pre class="chroma"> <code class="language-toml" data-lang="toml"> <span class="nx"> DATASTORE_TYPE</span> <span class="p">= </span> <span class="nx"> kubernetes</span>  <span class="err">//</span> <span class="nx"> 数据库类型 </span>
<span class="nx"> KUBECONFIG</span> <span class="p">= </span> <span class="err">/</span> <span class="nx"> vagrant</span> <span class="err">/</span> <span class="nx"> kube_config_cluster</span> <span class="p">.</span> <span class="nx"> yml</span> <span class="err">//</span> <span class="nx"> 目标集群的 kubeconfig 文件路径 </span>
<span class="nx"> CALICO_NODENAME</span> <span class="p">= </span> <span class="s2">"172.36.1.113"</span> <span class="err">//</span> <span class="nx"> 在集群里为这台主机注册的 nodename</span>
<span class="nx"> NO_DEFAULT_POOLS</span> <span class="p">= </span> <span class="s2">"true"</span> <span class="err">//</span> <span class="nx"> 集群已经配置过 </span> <span class="nx"> IPPOOL</span> <span class="nx"> 了 </span> <span class="err">, </span> <span class="nx"> 这里不需要再创建默认的了 </span>
<span class="nx"> CALICO_IP</span> <span class="p">= </span> <span class="s2">""</span>
<span class="nx"> CALICO_IP6</span> <span class="p">= </span> <span class="s2">""</span>
<span class="nx"> CALICO_AS</span> <span class="p">= </span> <span class="s2">""</span>
<span class="nx"> CALICO_NETWORKING_BACKEND</span> <span class="p">= </span> <span class="nx"> bird</span> <span class="err">//</span> <span class="err"></span> <span class="nx"> 网络模型选择 </span> <span class="p">, </span> <span class="nx"> bird</span> <span class="nx"> 就是 bgp 协议的客户端 </span>
</code> </pre> </td> </tr> </tbody> </table>

##### 配置 `systemd` 启动 calico docker 容器

<table class="lntable"> <tbody> <tr> <td class="lntd"> <pre class="chroma"> <code> <span class="lnt"> 1
</span> <span class="lnt"> 2
</span> <span class="lnt"> 3
</span> <span class="lnt"> 4
</span> <span class="lnt"> 5
</span> <span class="lnt"> 6
</span> <span class="lnt"> 7
</span> <span class="lnt"> 8
</span> <span class="lnt"> 9
</span> <span class="lnt">10
</span> <span class="lnt">11
</span> <span class="lnt">12
</span> <span class="lnt">13
</span> <span class="lnt">14
</span> <span class="lnt">15
</span> <span class="lnt">16
</span> <span class="lnt">17
</span> <span class="lnt">18
</span> <span class="lnt">19
</span> <span class="lnt">20
</span> <span class="lnt">21
</span> <span class="lnt">22
</span> <span class="lnt">23
</span> <span class="lnt">24
</span> <span class="lnt">25
</span> <span class="lnt">26
</span> <span class="lnt">27
</span> <span class="lnt">28
</span> <span class="lnt">29
</span> <span class="lnt">30
</span> <span class="lnt">31
</span> <span class="lnt">32
</span> <span class="lnt">33
</span> <span class="lnt">34
</span> <span class="lnt">35
</span> <span class="lnt">36
</span> <span class="lnt">37
</span> <span class="lnt">38
</span> <span class="lnt">39
</span> <span class="lnt">40
</span> <span class="lnt">41
</span> <span class="lnt">42
</span> </code> </pre> </td> <td class="lntd"> <pre class="chroma"> <code class="language-bash" data-lang="bash"> <span class="o"> [</span> Unit<span class="o">]</span>
<span class="nv"> Description</span> <span class="o">= </span> calico-node
<span class="nv"> After</span> <span class="o">= </span> docker.service
<span class="nv"> Requires</span> <span class="o">= </span> docker.service

<span class="o"> [</span> Service<span class="o">]</span>
<span class="nv"> EnvironmentFile</span> <span class="o">= </span>/etc/calico/calico.env
<span class="nv"> ExecStartPre</span> <span class="o">= </span>-/usr/bin/docker rm -f calico-node
<span class="nv"> ExecStart</span> <span class="o">= </span>/usr/bin/docker run --net<span class="o">= </span> host --privileged <span class="se">\
</span> <span class="se"> </span> --name<span class="o">= </span> calico-node <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> NODENAME</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> CALICO_NODENAME</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> IP</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> CALICO_IP</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> IP6</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> CALICO_IP6</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> CALICO_NETWORKING_BACKEND</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> CALICO_NETWORKING_BACKEND</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> AS</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> CALICO_AS</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> NO_DEFAULT_POOLS</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> NO_DEFAULT_POOLS</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> DATASTORE_TYPE</span> <span class="o">= </span> kubernetes <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> ETCD_ENDPOINTS</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> ETCD_ENDPOINTS</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> ETCD_CA_CERT_FILE</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> ETCD_CA_CERT_FILE</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> ETCD_CERT_FILE</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> ETCD_CERT_FILE</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> ETCD_KEY_FILE</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> ETCD_KEY_FILE</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> KUBECONFIG</span> <span class="o">= </span> <span class="si">${</span> <span class="nv"> KUBECONFIG</span> <span class="si">} </span> <span class="se">\
</span> <span class="se"> </span> -e <span class="nv"> CALICO_STARTUP_LOGLEVEL</span> <span class="o">= </span> debug <span class="se">\
</span> <span class="se"> </span> -v /var/log/calico:/var/log/calico <span class="se">\
</span> <span class="se"> </span> -v /var/run/calico:/var/run/calico <span class="se">\
</span> <span class="se"> </span> -v /var/lib/calico:/var/lib/calico <span class="se">\
</span> <span class="se"> </span> -v /run/docker/plugins:/run/docker/plugins <span class="se">\
</span> <span class="se"> </span> -v /lib/modules:/lib/modules <span class="se">\
</span> <span class="se"> </span> -v /etc/pki:/pki <span class="se">\
</span> <span class="se"> </span> <span class="c1"># 挂载 kubeconfig</span>
 -v /home/.kube/srv4-server.yaml:/vagrant/kube_config_cluster.yml <span class="se">\
</span> <span class="se"> </span> calico/node: v3.14.1


<span class="nv"> ExecStop</span> <span class="o">= </span>-/usr/bin/docker stop calico-node

<span class="nv"> Restart</span> <span class="o">= </span> on-failure
<span class="nv"> StartLimitBurst</span> <span class="o">= </span> <span class="m">3</span>
<span class="nv"> StartLimitInterval</span> <span class="o">= </span>60s

<span class="o"> [</span> Install<span class="o">]</span>
<span class="nv"> WantedBy</span> <span class="o">= </span> multi-user.target
</code> </pre> </td> </tr> </tbody> </table>

#### 添加 K8S node

创建 `K8S node` 资源对应当前的机器.

<table class="lntable"> <tbody> <tr> <td class="lntd"> <pre class="chroma"> <code> <span class="lnt">1
</span> <span class="lnt">2
</span> <span class="lnt">3
</span> <span class="lnt">4
</span> <span class="lnt">5
</span> <span class="lnt">6
</span> <span class="lnt">7
</span> </code> </pre> </td> <td class="lntd"> <pre class="chroma"> <code class="language-yaml" data-lang="yaml"> <span class="k"> kind</span> <span class="p">: </span> <span class="w"> </span> Node<span class="w">
</span> <span class="w"> </span> <span class="k"> metadata</span> <span class="p">: </span> <span class="w">
</span> <span class="w">  </span> <span class="k"> name</span> <span class="p">: </span> <span class="w"> </span> <span class="m">172.36.1.113</span> <span class="w"> </span>//<span class="w"> </span> nodeName<span class="w"> </span> 一般填机器 IP 就好 <span class="w">
</span> <span class="w"> </span> <span class="k"> spec</span> <span class="p">: </span> <span class="w">
</span> <span class="w">  </span> <span class="k"> podCIDR</span> <span class="p">: </span> <span class="w"> </span> <span class="m">10.42.0.0</span>/<span class="m">24</span> <span class="w">
</span> <span class="w">  </span> <span class="k"> podCIDRs</span> <span class="p">: </span> <span class="w">
</span> <span class="w">  </span>- <span class="m">10.42.0.0</span>/<span class="m">24</span> <span class="w">
</span> </code> </pre> </td> </tr> </tbody> </table>

### 校验与排查

#### calicoctl

**安装**

[官方安装教程](https://docs.projectcalico.org/getting-started/clis/calicoctl/install)

1.  进入 `PATH` 目录, 或者把当前目录加到 `PATH` 中
2.  下载 `calicoctl`

<table class="lntable"> <tbody> <tr> <td class="lntd"> <pre class="chroma"> <code> <span class="lnt">1
</span> </code> </pre> </td> <td class="lntd"> <pre class="chroma"> <code class="language-bash" data-lang="bash"> curl -O -L  https://github.com/projectcalico/calicoctl/releases/download/v3.14.1/calicoctl
</code> </pre> </td> </tr> </tbody> </table>

3.  改为可执行文件

<table class="lntable"> <tbody> <tr> <td class="lntd"> <pre class="chroma"> <code> <span class="lnt">1
</span> </code> </pre> </td> <td class="lntd"> <pre class="chroma"> <code class="language-bash" data-lang="bash"> chmod + x calicoctl
</code> </pre> </td> </tr> </tbody> </table>

4.  配置 calicoctl 以连接到 Kubernetes API
    [https://docs.projectcalico.org/getting-started/clis/calicoctl/configure/kdd](https://docs.projectcalico.org/getting-started/clis/calicoctl/configure/kdd)

5.  或者直接命令行


<table class="lntable"> <tbody> <tr> <td class="lntd"> <pre class="chroma"> <code> <span class="lnt">1
</span> </code> </pre> </td> <td class="lntd"> <pre class="chroma"> <code class="language-bash" data-lang="bash"> <span class="nv"> DATASTORE_TYPE</span> <span class="o">= </span> kubernetes <span class="nv"> KUBECONFIG</span> <span class="o">= </span>~/.kube/config calicoctl get nodes
</code> </pre> </td> </tr> </tbody> </table>

**查看 node 连接状态**

[https://docs.projectcalico.org/reference/calicoctl/node/status](https://docs.projectcalico.org/reference/calicoctl/node/status)

<table class="lntable"> <tbody> <tr> <td class="lntd"> <pre class="chroma"> <code> <span class="lnt"> 1
</span> <span class="lnt"> 2
</span> <span class="lnt"> 3
</span> <span class="lnt"> 4
</span> <span class="lnt"> 5
</span> <span class="lnt"> 6
</span> <span class="lnt"> 7
</span> <span class="lnt"> 8
</span> <span class="lnt"> 9
</span> <span class="lnt">10
</span> <span class="lnt">11
</span> <span class="lnt">12
</span> <span class="lnt">13
</span> <span class="lnt">14
</span> <span class="lnt">15
</span> </code> </pre> </td> <td class="lntd"> <pre class="chroma"> <code class="language-bash" data-lang="bash">$ calicoctl node status
Calico process is running.

IPv4 BGP status
+---------------+-------------------+-------+------------+-------------+
<span class="p"> |</span> PEER ADDRESS  <span class="p"> |</span>     PEER TYPE     <span class="p"> |</span> STATE <span class="p"> |</span>   SINCE    <span class="p"> |</span>    INFO     <span class="p"> |</span>
+---------------+-------------------+-------+------------+-------------+
<span class="p"> |</span> 172.23.30.100 <span class="p"> |</span> node-to-node mesh <span class="p"> |</span> up    <span class="p"> |</span> 2020-06-12 <span class="p"> |</span> Established <span class="p"> |</span>
<span class="p"> |</span> 172.24.0.85   <span class="p"> |</span> node-to-node mesh <span class="p"> |</span> up    <span class="p"> |</span> 2020-06-12 <span class="p"> |</span> Established <span class="p"> |</span>
<span class="p"> |</span> 172.24.0.86   <span class="p"> |</span> node-to-node mesh <span class="p"> |</span> up    <span class="p"> |</span> 2020-06-12 <span class="p"> |</span> Established <span class="p"> |</span>
<span class="p"> |</span> 172.24.28.3   <span class="p"> |</span> node-to-node mesh <span class="p"> |</span> up    <span class="p"> |</span> 2020-06-12 <span class="p"> |</span> Established <span class="p"> |</span>
+---------------+-------------------+-------+------------+-------------+

IPv6 BGP status
No IPv6 peers found.
</code> </pre> </td> </tr> </tbody> </table>

**确定配置**

<table class="lntable"> <tbody> <tr> <td class="lntd"> <pre class="chroma"> <code> <span class="lnt">1
</span> <span class="lnt">2
</span> <span class="lnt">3
</span> <span class="lnt">4
</span> </code> </pre> </td> <td class="lntd"> <pre class="chroma"> <code class="language-bash" data-lang="bash"> <span class="c1"># 查看 IPPOOL 配置 </span>
calicoctl get ippool -o yaml
<span class="c1"># 查看 felixconfig 配置 </span>
calicoctl get felixconfig -o yaml
</code> </pre> </td> </tr> </tbody> </table>

#### 观察日志

*   bird 日志 `cat /var/log/calico/bird/current`
*   felix 日志 `cat /var/log/calico/felix/current`
*   运行日志 `docker logs -f calico-node`

#### 观察路由表

`ip route` 应该有将容器 IP 段路由到其它主机的路由, 通网段路由到 `eth0` 网卡, 跨网段路由到 `tunl0 IPIP` 网卡.

## 后续

1.  了解 ippool 的分配容器 IP 的机制

文章作者 金润森

上次更新 2020-06-16

[Calico](/tags/calico/) [网络](/tags/%E7%BD%91%E7%BB%9C/) [云原生](/tags/%E4%BA%91%E5%8E%9F%E7%94%9F/)

[Linux 性能优化专栏笔记 网络性能优化 上一篇](/linux-preformance-network-note/) [Rancher 产生僵尸进程 下一篇](/rancher-zombie-process/)
