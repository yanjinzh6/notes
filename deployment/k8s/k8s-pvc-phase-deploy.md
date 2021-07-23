---
title: k8s-pvc-phase-deploy
date: 2021-01-30 16:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: k8s-pvc-phase-deploy
photo:
---

https://zhuanlan.zhihu.com/p/74134318

# k8s——部署基于公网的 k8s 集群

[! [xiaoyangmai](https://pic2.zhimg.com/v2-2c8edddb7161a8a3148d53bf5fcfe7e7_xs.jpg? source=172ae18b)](//www.zhihu.com/people/mai-xiao-yang-39)

[xiaoyangmai](//www.zhihu.com/people/mai-xiao-yang-39)

5 人赞同了该文章

折腾了两周 Kubernetes, 终于部署到了公网, 可以把不同类型的阿里云学生机都利用起来了.

最复杂的地方是 Kubernetes+ flannel 网络架构, 对我这种没太多网络工程基础的自学者有点不友好, 各种网络相关的参数, iptables, 防火墙, 虚拟网卡, 网关, 路由 ... 建议先用 kubeadm 通过内网搭一次, 熟悉下各个组件的功能和排查日志的方法. 后面的搭建过程也是基于 kubeadm 搭建的最小可用集群.

## 通过 iptables 在防火墙阶段转发流量

[ubuntu 16.04 部署 kubernetes 集群 [详细教程]​cloud.tencent.com](https://link.zhihu.com/? target= https%3A//cloud.tencent.com/developer/article/1378483)

参考这位老哥的方法, 在最后 kubeadm join 扩展从节点的过程, 从节点 flannel 总是起不来, 把 pod 的日志打出来

```
10.244.0.1:6443 no route to host
```

10.244.0.1 是 flannel 分配用与集群内部通讯的网段, 这个 host 代表 master 节点, 这个网段 kubeadm init 的时候要和 flannel 通过 --pod-network-cidr=10.244.0.0/16 参数约定好,6443 是 apiserver 服务的端口.

路由不到这个 host, 比较有可能的是 iptables 的规则乱掉了, k8s 和 docker 本身会在 iptables 层设置很多过滤转发的规则, 干涉 iptables 层容易出问题.

[iptables 详解: 图文并茂理解 iptables​www.zsythink.net! [图标](https://pic4.zhimg.com/v2-73e11e98881f07db72ae98728a30a137_ipico.jpg)](https://link.zhihu.com/? target= http%3A//www.zsythink.net/archives/1199/)

iptables debug 方法:

```
iptables -t raw -A OUTPUT -p icmp -j TRACE
iptables -t raw -A PREROUTING -p icmp -j TRACE
modprobe ipt_LOG
tail /var/log/messages
```

这个日志其实我看不出 iptables 出了什么问题, 明明转发规则是有的, 但是就是没有命中, 没有把访问集群内 IP 10.244.0.0/16 的流量转发到集群外的 IP. 这里折腾了很久都不行, 只能换个方向折腾.

参考上面的老哥的方法, 设置 kubeadm init --apiserver-advertise-address 参数, 按参数定义是可以构建公网集群的. 但是实际 etcd 会启动不了, init 会卡在启动 etcd 步骤, 卡到超时. 我们分析一下原因.

```
root@kube0:/workdir# journalctl -xeu kubelet
dial tcp 12.12.12.12:6443: connect: connection refused
```

公网 IP 的 6443 端口, 是在参数里声明的 apiserver 端口. 显然 apiserver 没用正确启动.

```
root@kube0:/workdir# kc get pods -A
The connection to the server 39.108.140.239:6443 was refused - did you specify the right host or port?
```

既然 apiserver 没有启动, 当然不能通过它查日志. 直接看下 docker 容器.

```
root@kube0:/workdir# d ps -a
CONTAINER ID        IMAGE                                               COMMAND                  CREATED             STATUS                       ...
b048e3273495        68c3eb07bfc3                                        "kube-apiserver --ad…"   2 minutes ago       Exited (255) 2 minutes ago  ...
8ee4225bf2a8        2c4adeb21b4f                                        "etcd --advertise-cl…"   4 minutes ago       Exited (1) 4 minutes ago   ...
0ffe6b3cacc2        d75082f1d121                                        "kube-controller-man…"   10 minutes ago      Up 10 minutes              ...
ad559b892126        b0b3c4c404da                                        "kube-scheduler --bi…"   10 minutes ago      Up 10 minutes              ...
f9792064243e        registry.aliyuncs.com/google_containers/pause:3.1   "/pause"                 10 minutes ago      Up 10 minutes               ...
f61a692e6d0a        registry.aliyuncs.com/google_containers/pause:3.1   "/pause"                 10 minutes ago      Up 10 minutes               ...
43fdaee28d58        registry.aliyuncs.com/google_containers/pause:3.1   "/pause"                 10 minutes ago      Up 10 minutes               ...
2aaaf966f859        registry.aliyuncs.com/google_containers/pause:3.1   "/pause"                 10 minutes ago      Up 10 minutes               ...
```

apiserver 和 etcd 都是失败状态, apiserver 是依赖 etcd 的, 直接看 etcd.

```
root@kube0:/workdir# d logs 2dcefe1c7b44
2019-07-30 07:19:22.771764 I | etcdmain: etcd Version: 3.3.10
2019-07-30 07:19:22.771853 I | etcdmain: Git SHA: 27fc7e2
2019-07-30 07:19:22.771858 I | etcdmain: Go Version: go1.10.4
2019-07-30 07:19:22.771863 I | etcdmain: Go OS/Arch: linux/amd64
2019-07-30 07:19:22.771868 I | etcdmain: setting maximum number of CPUs to 1, total number of available CPUs is 1
2019-07-30 07:19:22.771944 I | embed: peerTLS: cert = /etc/kubernetes/pki/etcd/peer.crt, key = /etc/kubernetes/pki/etcd/peer.key, \
ca = , trusted-ca = /etc/kubernetes/pki/etcd/ca.crt, client-cert-auth = true, crl-file =
2019-07-30 07:19:22.772074 C | etcdmain: listen tcp 12.12.12.12:2380: bind: cannot assign requested address
```

显然, 监听公网 IP 的 2380 端口失败, 导致 etcd 启动不了. 通过 ifconfig 查看网卡设备可以看到, 阿里云的公网 IP 是不绑定在网卡的, 听说以前经典网络是可以绑定的, 现在都改成 VPC 网了, 公网 IP 绑定的网卡估计在外层网关上, 机器本地肯定监听不到了.

有两个方向可以尝试,1: 虚拟网卡, 在本地虚拟一个绑定公网 IP 的网卡, 把内网的流量复制过来.2: 看能不能设置 etcd 监听的 IP.

[XiaoYang: Kubernetes——etcd 集群启动参数说明及注意事项​zhuanlan.zhihu.com! [图标](https://zhstatic.zhihu.com/assets/zhihu/editor/zhihu-card-default.svg)](https://zhuanlan.zhihu.com/p/75834420)

etcd 通过调整启动参数是可以调整节点监听的 IP 地址的, 但是 kubeadm 没有暴露这部分参数给我们设置, 需要看下 kubeadm 是否允许在 init 过程中调整 etcd 的参数.

[kubeadm init phase​kubernetes.io! [图标](https://pic3.zhimg.com/v2-e41528f5257d89d58028ff39c3c2f712_180x120.jpg)](https://link.zhihu.com/? target= https%3A//kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm-init-phase/)

在 k8s v1.8.0 及之后版本, kubeadm 允许一种分段 (phase) 的构建方式, 构建 etcd 是其中一个 phase. 用这个方式给了使用者更大的操作空间. 调整 etcd 的 yaml 文件里面的参数可以改变监听的目标.

## 公网 k8s 集群的搭建流程

修正 hostname

```
vi /etc/hostname
vi /etc/hosts
reboot
```

安装 docker

```
apt install docker.io
```

插入 apt-key

```
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add
```

新增软件源

```
echo "deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
apt update
```

安装 kubeadm, kubectl, kubelet

```
apt install -V kubeadm kubectl kubelet
```

锁定版本

```
apt-mark hold kubelet kubeadm kubectl
```

检验版本

```
kubeadm version
```

在~/.bashrc 导入环境

```
export KUBECONFIG=/etc/kubernetes/admin.conf
alias kc= kubectl
source <(kubectl completion bash | sed s/kubectl/kc/g)
swapoff –a
```

初始化集群

```
#在 kubeadm-config.yml 设置好镜像源, 版本, 集群网段,--apiserver-advertise-address 为外网 ip 等
kubeadm init phase preflight --config kubeadm-config.yml --ignore-preflight-errors NumCPU
kubeadm init phase certs all --config kubeadm-config.yml
kubeadm init phase kubeconfig all --config kubeadm-config.yml
kubeadm init phase kubelet-start --config kubeadm-config.yml
kubeadm init phase control-plane all --config kubeadm-config.yml
kubeadm init phase etcd local --config kubeadm-config.yml
vi /etc/kubernetes/manifests/etcd.yaml
#把 --listen-client-urls 和 --listen-peer-urls 都改成 0.0.0.0: xxx
#kc apply -f /etc/kubernetes/manifests/etcd.yaml
kubeadm init --skip-phases= preflight, certs, kubeconfig, kubelet-start, control-plane, etcd --config kubeadm-config.yml
```

确认 Pods 启动

```
kc get pods -A
```

集群初始化如果遇到问题, 可以使用下面的命令进行清理:

```
kubeadm reset
ifconfig cni0 down
ip link delete cni0
ifconfig flannel.1 down
ip link delete flannel.1
rm -rf /var/lib/cni/
rm -rf /var/lib/etcd
```

出于安全考虑, 默认配置下 Kubernetes 不会将 Pod 调度到 Master 节点. 如果希望将 k8s-master 也当作 Node 使用, 可以执行如下命令:

```
kubectl taint node kube0 node-role.kubernetes.io/master-
```

其中 k8s-master 是主机节点 hostname 如果要恢复 Master Only 状态, 执行如下命令:

```
kubectl taint node k8s-master node-role.kubernetes.io/master=""
```

或者在任务 yaml 上添加容忍段

```
tolerations:
    - key: node-role.kubernetes.io/master
      operator: Exists
      effect: NoSchedule
```

传递桥接的 IPv4 流量到 iptables chains

```
sysctl net.bridge.bridge-nf-call-iptables=1
```

申请应用 cni 网络

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/bc79dd1505b0c8681ece4de4c0d86c5cd2643275/Documentation/kube-flannel.yml
```

确认 coredns 正常运行

```
root@kube0:/workdir# kc get pods -A
NAMESPACE     NAME                            READY   STATUS    RESTARTS   AGE
kube-system   coredns-bccdc95cf-jfh72         0/1     Running   0          43s
kube-system   coredns-bccdc95cf-vf6hf         0/1     Running   0          43s
kube-system   etcd-kube0                      1/1     Running   0          79s
kube-system   kube-apiserver-kube0            1/1     Running   2          61s
kube-system   kube-controller-manager-kube0   1/1     Running   0          2m1s
kube-system   kube-flannel-ds-amd64-mh7ns     1/1     Running   0          10s
kube-system   kube-proxy-5mk8w                1/1     Running   0          43s
kube-system   kube-scheduler-kube0            1/1     Running   0          114s
```

添加从节点

```
#在从节点把上面流程走到初始化集群之前
#在 master 查看 join 命令
kubeadm token create --print-join-command
#设置 --apiserver-advertise-address 为外网 IP, 并在从节点执行
kubeadm join ...
```

在 master 确保 worker 已正确加入集群

```
kc get nodes -o wide
```



[XiaoYang: Kubernetes——集群网关运维​zhuanlan.zhihu.com! [图标](https://zhstatic.zhihu.com/assets/zhihu/editor/zhihu-card-default.svg)](https://zhuanlan.zhihu.com/p/74175616)

通过这个方法测试下连通情况, 如果集群有问题, 查看日志

```
journalctl -f -u kubelet
```

## 可能的问题

1.coredns 不断 CrashLoopBackOff, 重启 iptables 和集群

```
systemctl stop kubelet
systemctl stop docker
iptables --flush
iptables -tnat --flush
systemctl start kubelet
systemctl start docker
```

2.etcd 不断 CrashLoopBackOff
