---
title: calico-bgp-config
date: 2021-03-14 19:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: calico-bgp-config
photo:
---

[calico-bgp-config](https://www.yxingxing.net/archives/calico-20200312-bgp)

# calico 二, 配置 BGP 对等体

大番茄 2020 年 03 月 12 日 875 次浏览

Calico v3.11.2
Kubernetes v1.16.7

## 一, 安装 calicoctl

[https://docs.projectcalico.org/v3.11/getting-started/calicoctl/install](https://docs.projectcalico.org/v3.11/getting-started/calicoctl/install)

calicoctl 是管理 calico 对象的工具.
git 下载地址:
[https://github.com/projectcalico/calicoctl/releases/](https://github.com/projectcalico/calicoctl/releases/)

下载, 添加执行权限并放到 PATH 路径下:
[https://github.com/projectcalico/calicoctl/releases/download/v3.11.2/calicoctl](https://github.com/projectcalico/calicoctl/releases/download/v3.11.2/calicoctl)

配置:
[https://docs.projectcalico.org/v3.11/getting-started/calicoctl/configure/](https://docs.projectcalico.org/v3.11/getting-started/calicoctl/configure/)
我这里是使用 etcd 作为 calico 的存储, 所以需要配置 calicoctl 连接 etcd.
calicoctl 默认使用 `/etc/calico/calicoctl.cfg` 作为配置文件.
官方网站中有很多例子, 根据例子写就行.
比如我这里:

```
apiVersion: projectcalico.org/v3
kind: CalicoAPIConfig
metadata:
spec:
  datastoreType: "etcdv3"
  etcdEndpoints: "https://172.100.102.71:2379, https://172.100.102.72:2379, https://172.100.102.73:2379"
  etcdKeyFile: "/etc/calico/calico-etcd.key"
  etcdCertFile: "/etc/calico/calico-etcd.crt"
  etcdCACertFile: "/etc/calico/cacert.pem"
```

需要连接 etcd 的证书, 证书也可以内置到文件里, 网站中有介绍.

测试一下:

```
[root@k8s-master calico]# calicoctl get nodes
NAME
k8s-node1
k8s-node2
k8s-node3
```

查看其他信息:
查看地址池

```
[root@k8s-master ~]# calicoctl get ipPool
NAME                  CIDR          SELECTOR
default-ipv4-ippool   10.6.0.0/16   all()
```

ipam(IP Address Management) 可分配的地址信息.

```
[root@k8s-master ~]# calicoctl ipam show
+----------+-------------+-----------+------------+--------------+
| GROUPING |    CIDR     | IPS TOTAL | IPS IN USE |   IPS FREE   |
+----------+-------------+-----------+------------+--------------+
| IP Pool  | 10.6.0.0/16 |     65536 | 5 (0%)     | 65531 (100%) |
+----------+-------------+-----------+------------+--------------+
```

有一些查询当前节点信息的, calicoctl 会连接 calico 的 socket, 所以需要 calicoctl 在运行 calico 的节点执行.
如:

```
[root@k8s-master ~]# calicoctl node status
Calico process is not running.
[root@k8s-master ~]#
```

在 calico 节点就可以了:

```
[root@k8s-node1 ~]# calicoctl node status
Calico process is running.

IPv4 BGP status
+----------------+-------------------+-------+------------+-------------+
|  PEER ADDRESS  |     PEER TYPE     | STATE |   SINCE    |    INFO     |
+----------------+-------------------+-------+------------+-------------+
| 172.100.102.72 | node-to-node mesh | up    | 11:21:07   | Established |
| 172.100.102.73 | node-to-node mesh | up    | 2020-03-10 | Established |
+----------------+-------------------+-------+------------+-------------+

IPv6 BGP status
No IPv6 peers found.
```

node-to-node mesh 是 BGP 的全互连模式.

## 二, 配置 BGP

[https://docs.projectcalico.org/v3.11/networking/bgp](https://docs.projectcalico.org/v3.11/networking/bgp)

### 说明

BGP(边界网关协议) 是运行于 TCP 上的一种自治系统的路由协议. 是用于在网络中的两个路由器之间交换路由信息的标准协议. 每个运行 BGP 的路由器都有一个或多个 `BGP 对等体 `.
BGP 对等体: 通过 BGP 与其通信的其他路由器.
路由器与其他路由器建立对等关系, 其实就是建立了 TCP 长连接, 然后用 BGP 协议相互通信. 对方路由器就是本路由器的 BGP 对等体, 反过来也是一样, 本路由器也是对方路由器的对等体. 通俗讲对等关系就是邻居关系, 对等体就是邻居.

### BGP 模式

Calico 维护的网络默认是 (Node-to-Node Mesh) 全互联模式, Calico 集群中的所有节点之间都会相互建立连接, 用于路由交换.

```
[root@k8s-node1 ~]# ss -tan | grep 179
LISTEN     0      8            *:179                      *:*
ESTAB      0      0      172.100.102.71:179                172.100.102.72:49409
ESTAB      0      0      172.100.102.71:179                172.100.102.73:55701
```

可以之间看到有两个连接.
还有上面 `node status` 显示的也一样. 而其他节点也是同样.
这种情况下节点数量越多, 网络中的连接数就会成倍增加, 而且在 100 个节点左右会遇到性能瓶颈.

解决办法就是把其中的几个 calico 节点当做路由反射器 (route reflector), 然后其他节点只需要把这几个节点当做对等体建立连接就可以.
路由器反射器会把传递过来的路由, 在传递给其他节点, 来实现路由交换.

在这之前先来看看对等体怎么配置. 路由反射器也是需要单独配置对等体的, 所以这一步是必须的.

* * *

首先需要关闭全互连模式, 但是现在的 BGP 没有配置资源, 都是默认的配置.
所以还需要创建一个 BGP 配置资源, 并且配置为关闭全互连模式.
[https://docs.projectcalico.org/v3.11/reference/resources/bgpconfig](https://docs.projectcalico.org/v3.11/reference/resources/bgpconfig)
如:

```
apiVersion: projectcalico.org/v3
kind: BGPConfiguration
metadata:
  name: default
spec:
  logSeverityScreen: Info
  nodeToNodeMeshEnabled: false
  asNumber: 64520
```

`BGPConfiguration`: BGPConfiguration 是全局的配置资源.
`nodeToNodeMeshEnabled`: 就是是否开启全互连模式.
`asNumber`: as 表示自治系统, asN 自治系统编号. 默认是 64512. 这里必须提供, 不然 asNumber 好像就变成空了.
64512-65535 也是私有的 ASN, 不能出现在公网.ASN 在外网是唯一的, 由 IANA 地址授权委员会统一分配, 不过在内网就无所谓了. 官网的例子是 63400, 同样没问题.
使用 `calicoctl get nodes -o wide` 可以查询节点的 ASN.

```
[root@k8s-master ~]# calicoctl get nodes -o wide
NAME        ASN       IPV4                IPV6
k8s-node1   (64512)   172.100.102.71/24
k8s-node2   (64512)   172.100.102.72/24
k8s-node3   (64512)   172.100.102.73/24
```

注意: 全互连模式关闭以后, k8s 集群网络就中断了.
应用文件, 与 k8s 的方式一样:

```
[root@k8s-master ~]# calicoctl apply -f bgpconfig.yaml
Successfully applied 1 'BGPConfiguration' resource(s)
```

应用以后, bgp 之间的连接就断了, 然后路由也没了, 网络也断了.

```
[root@k8s-node1 ~]# calicoctl node status
Calico process is running.

IPv4 BGP status
No IPv4 peers found.

IPv6 BGP status
No IPv6 peers found.
```

如果把 `nodeToNodeMeshEnabled` 改为 `true` 再次应用, 就会再次启用全互连模式, 网络也会恢复.

### 配置对等体

[https://docs.projectcalico.org/v3.11/reference/resources/bgppeer](https://docs.projectcalico.org/v3.11/reference/resources/bgppeer)
有时候需要把指定的设备或节点添加为对等体, 比如网络中的物理路由器, 支持 BGP 的三层交换机, 还有路由反射器等.

先来看一下我这里的节点信息:

```
[root@k8s-master ~]# calicoctl get nodes -o wide
NAME        ASN       IPV4                IPV6
k8s-node1   (64520)   172.100.102.71/24
k8s-node2   (64520)   172.100.102.72/24
k8s-node3   (64520)   172.100.102.73/24
```

对等体资源对象叫做 `BGPPeer`.

#### 全局对等体

我打算把 k8s-node3 作为全局的对等体, 让 k8s-node1, k8s-node2 去连.
全局对等体的意思是, 全局生效, 所有节点都会去把目标当做对等体去连.

```
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: my-global-peer
spec:
  peerIP: 172.100.102.73
  asNumber: 64520
```

`peerIP`: 目标的地址, 其他节点回去连接这个地址.
`asNumber`: 目标设置的 asNumber.
应用:

```
[root@k8s-master ~]# calicoctl apply -f bgppeer.yaml
Successfully applied 1 'BGPPeer' resource(s)
```

查看:

```
[root@k8s-master ~]# calicoctl get bgppeer
NAME             PEERIP           NODE       ASN
my-global-peer   172.100.102.73   (global)   64520
```

在 k8s-node1, k8s-node2, k8s-node3 上看一下效果.

* * *

k8s-node1:

```
[root@k8s-node1 ~]# calicoctl node status
Calico process is running.

IPv4 BGP status
+----------------+-----------+-------+----------+-------------+
|  PEER ADDRESS  | PEER TYPE | STATE |  SINCE   |    INFO     |
+----------------+-----------+-------+----------+-------------+
| 172.100.102.73 | global    | up    | 09:42:22 | Established |
+----------------+-----------+-------+----------+-------------+
```

可以看到之与 k8s-node3(172.100.102.73) 建立了连接.

! [image.png](https://www.yxingxing.net/upload/2020/3/image-6c07ac8995374a949cff73b2b3aa72dd.png)
路由里也只有 k8s-node3 的路由信息.

* * *

k8s-node2:

```
[root@k8s-node2 ~]# calicoctl node status
Calico process is running.

IPv4 BGP status
+----------------+-----------+-------+----------+-------------+
|  PEER ADDRESS  | PEER TYPE | STATE |  SINCE   |    INFO     |
+----------------+-----------+-------+----------+-------------+
| 172.100.102.73 | global    | up    | 09:42:21 | Established |
+----------------+-----------+-------+----------+-------------+
```

! [image.png](https://www.yxingxing.net/upload/2020/3/image-e7fe79f9166f4f629a7414f13b7502a4.png)
这里也是只有 k8s-node3 的路由.

* * *

k8s-node3:

k8s-node1 与 k8s-node2 都与 k8s-node3 建立了对等关系.

```
[root@k8s-node3 ~]# calicoctl node status
Calico process is running.

IPv4 BGP status
+----------------+---------------+-------+----------+-------------+
|  PEER ADDRESS  |   PEER TYPE   | STATE |  SINCE   |    INFO     |
+----------------+---------------+-------+----------+-------------+
| 172.100.102.71 | node specific | up    | 09:42:22 | Established |
| 172.100.102.72 | node specific | up    | 09:42:21 | Established |
+----------------+---------------+-------+----------+-------------+

IPv6 BGP status
```

! [image.png](https://www.yxingxing.net/upload/2020/3/image-59fc597d39f04d82a028609c6607903e.png)
因为 k8s-node3 与另外两个节点都有对等关系, 那两个节点也是 k8s-node3 的对等体. 所以上面有两个节点的路由.

* * *

这里可能会有点疑惑, 既然 k8s-node3 有两个节点的路由, 为什么没有发给其他两个节点, 这就是 BGP 的内部安全机制了. 只有把 k8s-node3 作为路由反射器才行.

* * *

#### 配置指定节点可以使用的对等体

上面是全局对等体 (所有节点的对等体), 这里是只作为某些节点的对等体.
跟上面的配置只多了 `nodeSelector`.
先把全局的删除了:

```
[root@k8s-master ~]# calicoctl delete bgppeer my-global-peer
Successfully deleted 1 'BGPPeer' resource(s)
```

calico 可以读取 k8s 的节点 label 信息, 也可以自己创建 label.
可以使用 `calicoctl get nodes k8s-node1 -o yaml` 查看节点信息.

所以这里直接在 k8s 里添加 label.

```
[root@k8s-master ~]# kubectl label node/k8s-node1 rack= rack-3
node/k8s-node1 labeled
```

创建 bgppeer:

```
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: rack3-tor
spec:
  peerIP: 172.100.102.73
  asNumber: 64520
  nodeSelector: rack == "rack-3"
```

`nodeSelector`: 节点标签选择器, 选择有 `rack: rack-3` 标签的节点使用这个对等体.
应用:

```
[root@k8s-master ~]# calicoctl apply -f bgppeer.yaml
Successfully applied 1 'BGPPeer' resource(s)
```

结果就是只有 k8s-node1 去连接了. k8s-node3 上也只有 k8s-node1 的连接.

也可以使用 `node` 参数来直接限制某个节点才可以连接对等体.

* * *

### 路由反射器

分两步:
1, 选择节点创建为路由反射器.
2, 创建 BGPPeer 指定路由反射器为对等体.

#### 创建路由反射器

我这里选择 k8s-node3 作为路由反射器.

需要修改节点配置:
[https://docs.projectcalico.org/v3.11/reference/resources/node](https://docs.projectcalico.org/v3.11/reference/resources/node)
添加 `routeReflectorClusterID` 配置就可以了.

首先, 导出 k8s-node3 的节点配置:

```
[root@k8s-master ~]# calicoctl get node k8s-node3 -o yaml > k8s-node3.yaml
```

文件里的 `annotations` 与 `labels` 信息是从 k8s 里拿到的, 所以可以删除. 当然, 不删也可以, 正常情况下应该都是不删的. 我这里只是为了突出重点, 就把没用的删了.

```
apiVersion: projectcalico.org/v3
kind: Node
metadata:
  name: k8s-node3
spec:
  bgp:
    ipv4Address: 172.100.102.73/24
    routeReflectorClusterID: 244.0.0.1
  orchRefs:
  - nodeName: k8s-node3
    orchestrator: k8s
```

主要就是 `routeReflectorClusterID`: 路由反射器集群 ID, 不知道为什么使用 ip 地址作为 ID. 官网例子使用的是 `244.0.0.1`, 我这里也用这个了.
`orchRefs` 关联协调器, 应该是与 k8s 集群关联的. 如果删了, 那些从 k8s 继承过来的 label 与 annotations 之类的配置就都没了.
应用:

```
[root@k8s-master ~]# calicoctl apply -f k8s-node3.yaml
Successfully applied 1 'Node' resource(s)
```

现在路由反射器已经创建好了. 现在 k8s-node1 与 k8s-node2 再把 k8s-node3 作为对等体, 路由里就有其他节点的路由了.

如果再添加其他的节点为路由器反射器, 对应节点添加 `routeReflectorClusterID: 244.0.0.1` 配置就可以了.

路由反射器一般也都是多个节点高可用, 防止一个节点挂了, 网络就挂了.
这时再用上面直接指定对等体 ip 的方式就不够好使了.

#### 选择路由反射器为对等体

还是创建 `BGPPeer`, 使用 `peerSelector` 标签选择器. 设置此项后, peerIPand 与 asNumber 字段必须为空.

防止混乱删除之前的 BGPPeer.

```
calicoctl delete bgppeer rack3-tor
```

为路由反射器节点添加标签:

```
[root@k8s-master ~]# kubectl label node k8s-node3 route-reflector= true
node/k8s-node3 labeled
```

添加新的 BGPPeer:

```
apiVersion: projectcalico.org/v3
kind: BGPPeer
metadata:
  name: peer-with-route-reflectors
spec:
  nodeSelector: all()
  peerSelector: route-reflector == 'true'
```

这里的 nodeSelector 加不加都无所谓, 不加就是 `global` 类型的, 加了就是 `node specific` 的, 只是这里的也是表示所有节点. 官网的例子是加上的.

```
[root@k8s-master ~]# calicoctl apply -f bgppeer.yaml
Successfully applied 1 'BGPPeer' resource(s)
```

这样就可以了.
其他路由反射器, 只要添加 `route-reflector` 标签就可以了.

```
[root@k8s-node1 ~]# calicoctl node status
Calico process is running.

IPv4 BGP status
+----------------+---------------+-------+----------+-------------+
|  PEER ADDRESS  |   PEER TYPE   | STATE |  SINCE   |    INFO     |
+----------------+---------------+-------+----------+-------------+
| 172.100.102.73 | node specific | up    | 12:52:07 | Established |
| 172.100.102.72 | node specific | up    | 12:53:54 | Established |
+----------------+---------------+-------+----------+-------------+
```

* * *

### 用到的资源对象

`BGPConfiguration`: BGP 全局默认配置.
`Node`: BGP 中单节点配置
`BGPPeer`: BGP 中的对等配置.

[上一篇: calico 三, IPPool 与封装](https://www.yxingxing.net/archives/calico-20200313-tunnel) [下一篇: 关于 calico 的一个测试以及 arp\_proxy](https://www.yxingxing.net/archives/calico-20200311-repeate)
