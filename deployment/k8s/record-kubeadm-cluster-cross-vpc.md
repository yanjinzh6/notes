---
title: Kubeadm 跨 vpc 搭建集群踩过的坑
date: 2021-03-20 14:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: record-kubeadm-cluster-cross-vpc
photo:
---

## 简介

vpc

## 环境

Debian 10 4.19.0-13-amd64 docker://20.10.3 v1.20.2

## 使用公网 ip 初始化 Master 导致无法 api service 无法启动

### 分段配置 Master

修改 etcd 监听 `https://0.0.0.0:2378`

在初始化过程中手动修改 etcd 配置文件

或者使用 --config 选项进行配置

## Worker 节点使用内网 ip 注册导致查看存在节点上的资源时就会超时

通过 iptables 转发, 在 Master 上将相应节点的内网地址转发到公网地址上

最简单的方法, 而且影响也比较小, 一开始没有选择比较遗憾

或者修改 Worker 节点的 kubelet 配置, 将 node-ip 设置为机器的公网 ip, 通过配置文件 /etc/default/kubelet 设置 KUBELET_EXTRA_ARGS=--node-ip=$PRIVATE_IP

在重启 kubelet 服务后会自动将 INTERNAL-IP 修改为已经更新的 ip

kube-proxy 使用 node-ip 进行监听导致无法监听

flannel 还是使用内网的 ip, 因为是判断网卡的 ip

尝试创建虚拟网卡来设置公网 ip

通过配置 iptables 将网卡流量转到公网 ip, 结果是等待非常久, 而且不小心删除了虚拟网卡导致了所有访问机器的流量都转到了一个无法到达的 ip, 结果就是 ssh 也断了, 只能通过 VNC 进行重置 iptables 操作

flannel 通过 public-ip 来暴露通信的 ip, 通过 public-ip-overwrite 注解可以强制覆盖设置, 所以设置后查看 flannel 实例的日志已经使用上新的 ip, 问题就是一重启就会导致 public-ip-overwrite 注解丢失

部署 cert manager 发现 pod 不能与 service 通信, 通过进入 pod 中发现 10.244.0.x 网段的 pod 无法与新建的 10.244.1.x 网段的 pod 通信, 是 flannel 的问题

查看 flannel 日志发现有大量的 iptables 规则配置错误的问题, 这个其实是因为没有开启 udp:8472 端口导致的, 但是当时没有详细阅读使用文档导致掉进一个大坑

现象是 flannel 实例报告无法配置 iptables role, 但是进入 pod 中操作是正常的, 以为是兼容性问题, 因为之前安装 kubeadm 时会有个建议使用旧版本的 iptables, 但是报错不一样, 手动配置完 iptables 后依旧无法使用

处理过程中多次的重置 kubeadm 而没有清理创建 flannel 创建的网卡和 cni 网络时会偶尔出现 flannel 配置的 cni 网络和网卡不一致, 例如 flannel 是 10.244.0.x 然后 cni 使用的是 10.244.1.x, 这里需要 down 一下网卡并删掉, 还有清理 /var/lib/cni/ 文件夹下面的 flannel 配置文件

这一过程也有改用了 calico 插件, 但是通过说明只能支持三层交换机内使用, 使用中会出现无法建立对等连接的问题, 而且使用的也是内网 ip, 虽然可以通过 ip 规则强制配置为公网 ip, 但是日志中还是会使用内网 ip, 这样就是配置为 ipip 模式也没办法正常解析, 这个需要进一步的参考使用文档

开放接口后, 重新配置了下 flannel 正常, 测试了不同节点的 pod 通信正常

由于设置了每个节点的 kubelet 为各自的公网 ip, 但确没有相应的网卡, 导致 kube-proxy 在监听的时候用的是公网 ip, 所有的转发都被拒绝, 只能重置为内网 ip, 并且在 Master 中配置 iptables 转发来解决 kuebctl 等命令超时的问题

但是像 node-exporter 这样的应用好像因为内网 ip 而无法获取到数据, 这个需要进一步排查

prometheus server 无法找到内网机器的 node-exporter 地址, 所以 node 一直是下线的状态, 在 prometheus server 所在的节点配置了 nat 转发还是不行

采用添加一个虚拟 tun 设备来配置公网的网卡, 并尝试通过添加 nat 转发来将物理网卡的流量转发到虚拟网卡的方案失败, 没有配置网桥 bridge 来桥接两个网卡, 尽管使用了转发和路由, 但还是无法在虚拟网卡中获取到数据包, ping 可以正常响应, 但是 curl 去获取就会卡住, 估计是找不到下一个路由, 但是使用网桥需求清理掉网卡的 ip, 由于是 vpc 设备, 远程桌面比较麻烦使用, 所以没有使用这个方案

还是回去最初的方法, 通过为网卡添加一个公网地址, 通过自定义 kubelet 的 node-ip 额外参数使节点用公网 ip 注册进集群, 然后节点上面的 kube-proxy 实例都会使用到 node-ip 来作为监听的 ip, 这里查询了很多资料都不清楚该修改节点的 kube-proxy 实例的哪个字段好使用其他 ip 进行监听

既然无法修改监听的 ip, 也没办法通过防火墙转发, 路由等方式转发物理网卡的流量, 那就使用最初的方法, 通过程序进行转发, 这里简单的使用 nginx 监听物理网卡的 80, 443 端口, 并将流量转发到虚拟 ip 的 80, 443 端口

尝试了下高可用集群, 一直无法配置 Master 的高可用, VIP 无法建立连接, 应该只能在内网中部署
