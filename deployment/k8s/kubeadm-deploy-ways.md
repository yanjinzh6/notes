---
title: kubeadm-deploy-ways
date: 2021-01-31 11:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-deploy-ways
photo:
---

https://segmentfault.com/a/1190000018741112

# [Kubernetes 的几种主流部署方式 02-kubeadm 部署高可用集群](/a/1190000018741112)

[! [](https://avatar-static.segmentfault.com/147/441/1474411814-5c24a266c097b_big64)**寞月**](/u/lonelymoon) 发布于 2019-04-02

在上篇文章 [minikube 部署](https://segmentfault.com/a/1190000018607114) 中, 有提到 Minikube 部署 Kubernetes 的核心就是 Kubeadm, 这篇文章来详细说明下 Kubeadm 原理及部署步骤. 写这篇文章的时候, Kubernetes1.14 刚刚发布, 所以部署步骤以 1.14 版为主.

# Kubeadm 原理简述

Kubeadm 工具的出发点很简单, 就是尽可能简单的部署一个生产可用的 Kubernetes 集群. 实际也确实很简单, 只需要两条命令:

```
# 创建一个 Master 节点
$ kubeadm init

# 将一个 Node 节点加入到当前集群中
$ kubeadm join <Master 节点的 IP 和端口 >
```

kubeadm 做了这些事
执行 kubeadm init 时:

*   自动化的集群机器合规检查
*   自动化生成集群运行所需的各类证书及各类配置, 并将 Master 节点信息保存在名为 cluster-info 的 ConfigMap 中.
*   通过 static Pod 方式, 运行 API server, controller manager , scheduler 及 etcd 组件.
*   生成 Token 以便其他节点加入集群

执行 kubeadm join 时:

*   节点通过 token 访问 kube-apiserver, 获取 cluster-info 中信息, 主要是 apiserver 的授权信息 (节点信任集群).
*   通过授权信息, kubelet 可执行 TLS bootstrapping, 与 apiserver 真正建立互信任关系 (集群信任节点).

**简单来说, kubeadm 做的事就是把大部分组件都容器化, 通过 StaticPod 方式运行, 并自动化了大部分的集群配置及认证等工作, 简单几步即可搭建一个可用 Kubernetes 的集群.**

这里有个问题, 为什么不把 kubelet 组件也容器化呢, 是因为, kubelet 在配置容器网络, 管理容器数据卷时, 都需要直接操作宿主机, 而如果现在 kubelet 本身就运行在一个容器里, 那么直接操作宿主机就会变得很麻烦. 比如, 容器内要做 NFS 的挂载, 需要 kubelet 先在宿主机执行 mount 挂载 NFS. 如果 kubelet 运行在容器中问题来了, 如果 kubectl 运行在容器中, 要操作宿主机的 Mount Namespace 是非常复杂的. 所以, kubeadm 选择把 kubelet 运行直接运行在宿主机中, 使用容器部署其他 Kubernetes 组件. 所以, Kubeadm 部署要安装的组件有 Kubeadm, kubelet, kubectl 三个.

上面说的是 kubeadm 部署方式的一般步骤, kubeadm 部署是可以自由定制的, 包括要容器化哪些组件, 所用的镜像, 是否用外部 etcd, 是否使用用户证书认证等以及集群的配置等等, 都是可以灵活定制的, 这也是 kubeadm 能够快速部署一个高可用的集群的基础. 详细的说明可以参考 [官方 Reference](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/). 但是,**kubeadm 最重要的作用还是解决集群部署问题, 而不是集群配置管理的问题, 官方也建议把 Kubeadm 作为一个基础工具, 在其上层再去量身定制适合自己的集群的管理工具**(例如 minikube).

# Kubeadm 部署一个高可用集群

## Kubernetes 的高可用

Kubernetes 的高可用主要指的是控制平面的高可用, 简单说, 就是有多套 Master 节点组件和 Etcd 组件, 工作节点通过负载均衡连接到各 Master.HA 有两种做法, 一种是将 etcd 与 Master 节点组件混布在一起:

另外一种方式是, 使用独立的 Etcd 集群, 不与 Master 节点混布:

两种方式的相同之处在于都提供了控制平面的冗余, 实现了集群高可以用, 区别在于:
Etcd 混布方式:

*   所需机器资源少
*   部署简单, 利于管理
*   容易进行横向扩展
*   风险大, 一台宿主机挂了, master 和 etcd 就都少了一套, 集群冗余度受到的影响比较大.

Etcd 独立部署方式:

*   所需机器资源多 (按照 Etcd 集群的奇数原则, 这种拓扑的集群关控制平面最少就要 6 台宿主机了).
*   部署相对复杂, 要独立管理 etcd 集群和和 master 集群.
*   解耦了控制平面和 Etcd, 集群风险小健壮性强, 单独挂了一台 master 或 etcd 对集群的影响很小.

## 部署环境

**由于机器资源不足, 下面的部署测试, 只会以混布的方式部署一个 1\*haproxy,2\*master,2\*node, 共 5 台机器的集群, 实际上由于 etcd 选举要过半数, 至少要 3 台 master 节点才能构成高可用, 在生产环境, 还是要根据实际情况, 尽量选择风险低的拓扑结构.**

*   机器:

master-1:192.168.41.230 (控制平面节点 1)
master-2:192.168.41.231 (控制平面节点 2)
node-1:172.16.201.108 (工作节点 1)
node-2:172.16.201.109 (工作节点 2)
haproxy:192.168.41.231 (haproxy)

*   系统内核版本:

```
# cat /etc/redhat-release
CentOS Linux release 7.6.1810 (Core)
# uname -r
5.0.5-1.el7.elrepo.x86_64
```

*   集群版本

kubeadm:1.14.0
Kubernetes:1.14.0
Docker: Community 18.09.4
haproxy: 1.5.18

## 部署步骤

### 机器准备

> 在所有节点上操作:

*   关闭 selinux, firewall

```
setenforce  0
sed -i 's/SELINUX= enforcing/SELINUX= permissive/' /etc/selinux/config
systemctl stop firewalld
systemctl disable firewalld
```

*   关闭 swap, (1.8 版本后的要求, 目的应该是不想让 swap 干扰 pod 可使用的内存 limit)

```
swapoff -a
```

*   修改下面内核参数, 否则请求数据经过 iptables 的路由可能有问题

```
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

### 安装 kubeadm, docker

> 在除了 haproxy 以外所有节点上操作

*   将 Kubernetes 安装源改为阿里云, 方便国内网络环境安装

```
cat << EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name= Kubernetes
baseurl= https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey= https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

*   安装 docker-ce

```
wget -O /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum install -y docker-ce
```

*   安装 kubelet kubeadm kubectl

```
 yum install -y  kubelet kubeadm kubectl
```

### 安装配置负载均衡

> 在 haproxy 节点操作:

```
# 安装 haproxy
yum install haproxy -y

# 修改 haproxy 配置
cat << EOF > /etc/haproxy/haproxy.cfg
global
    log         127.0.0.1 local2
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

defaults
    mode                    tcp
    log                     global
    retries                 3
    timeout connect         10s
    timeout client          1m
    timeout server          1m

frontend kube-apiserver
    bind *:6443 # 指定前端端口
    mode tcp
    default_backend master

backend master # 指定后端机器及端口, 负载方式为轮询
    balance roundrobin
    server master-1  192.168.41.230:6443 check maxconn 2000
    server master-2  192.168.41.231:6443 check maxconn 2000
EOF

# 开机默认启动 haproxy, 开启服务
systemctl enable haproxy
systemctl start haproxy

# 检查服务端口情况:
# netstat -lntup | grep 6443
tcp        0      0 0.0.0.0:6443            0.0.0.0:*               LISTEN      3110/haproxy
```

### 部署 Kubernetes

> 在 master-1 节点操作:

*   准备集群配置文件, 目前用的 api 版本为 v1beta1, 具体配置可以参考 [官方 reference](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta1)

```
cat << EOF > /root/kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: v1.14.0 # 指定 1.14 版本
controlPlaneEndpoint: 192.168.41.232:6443 # haproxy 地址及端口
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers # 指定镜像源为阿里源
networking:
  podSubnet: 10.244.0.0/16 # 计划使用 flannel 网络插件, 指定 pod 网段及掩码
EOF
```

*   执行节点初始化

```
systemctl enable kubelet
systemctl start kubelet
kubeadm  config images pull  --config kubeadm-config.yaml  # 通过阿里源预先拉镜像
kubeadm init --config= kubeadm-config.yaml --experimental-upload-certs
```

安装成功, 可以看到输出

```
You can now join any number of the control-plane node running the following command on each as root:
# master 节点用以下命令加入集群:

  kubeadm join 192.168.41.232:6443 --token ocb5tz.pv252zn76rl4l3f6 \
    --discovery-token-ca-cert-hash sha256:141bbeb79bf58d81d551f33ace207c7b19bee1cfd7790112ce26a6a300eee5a2 \
    --experimental-control-plane --certificate-key 20366c9cdbfdc1435a6f6d616d988d027f2785e34e2df9383f784cf61bab9826

Then you can join any number of worker nodes by running the following on each as root:
# 工作节点用以下命令加入集群:
kubeadm join 192.168.41.232:6443 --token ocb5tz.pv252zn76rl4l3f6 \
    --discovery-token-ca-cert-hash sha256:141bbeb79bf58d81d551f33ace207c7b19bee1cfd7790112ce26a6a300eee5a2
```

原来的 kubeadm 版本, join 命令只用于工作节点的加入, 而新版本加入了 --experimental-contaol-plane 参数后, 控制平面 (master) 节点也可以通过 kubeadm join 命令加入集群了.

*   加入另外一个 master 节点

> 在 master-2 操作:

```
kubeadm join 192.168.41.232:6443 --token ocb5tz.pv252zn76rl4l3f6 \
--discovery-token-ca-cert-hash sha256:141bbeb79bf58d81d551f33ace207c7b19bee1cfd7790112ce26a6a300eee5a2 \
--experimental-control-plane --certificate-key 20366c9cdbfdc1435a6f6d616d988d027f2785e34e2df9383f784cf61bab9826

mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

现在, 在任何一个 master 节点, 执行 kubectl get no, 可以看到, 集群中已经有 2 台 master 节点了

```
# kubectl get no
NAME       STATUS     ROLES    AGE     VERSION
master-1   NotReady   master   34m     v1.14.0
master-2   NotReady   master   4m52s   v1.14.0
```

*   加入两个工作节点

> 分别在两个 node 节点操作:

```
kubeadm join 192.168.41.232:6443 --token ocb5tz.pv252zn76rl4l3f6 \
    --discovery-token-ca-cert-hash sha256:141bbeb79bf58d81d551f33ace207c7b19bee1cfd7790112ce26a6a300eee5a2
```

再次执行 kubectl get no

```
# kubectl  get no
NAME       STATUS     ROLES    AGE     VERSION
master-1   NotReady   master   45m     v1.14.0
master-2   NotReady   master   15m     v1.14.0
node-1     NotReady   <none>   6m19s   v1.14.0
node-2     NotReady   <none>   4m59s   v1.14.0
```

可以看到两个 node 节点都加入集群了. 可是, 各个节点状态为什么都是 NotReady 呢. 通过执行 kubectl describe master-1, 可以看到这样的提示:

```
runtime network not ready: NetworkReady= false reason: NetworkPluginNotReady message: docker: network plugin is not ready: cni config uninitialized
```

原来是因为网络插件没有就绪导致的. 所以 , 我们来安装一波

```
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/a70459be0084506e4ec919aa1c114638878db11b/Documentation/kube-flannel.yml
```

再次查看节点状态, 可以看到所有节点都已经 ready 了.

```
# kubectl  get no
NAME       STATUS   ROLES    AGE    VERSION
master-1   Ready    master   134m   v1.14.0
master-2   Ready    master   104m   v1.14.0
node-1     Ready    <none>   94m    v1.14.0
node-2     Ready    <none>   93m    v1.14.0
```

至此, 一个 2 主节点 2 工作节点的 k8s 集群已经搭建完毕. 如果要加入更多的 master 或 node 节点, 只要多次执行 kubeadm join 命令加入集群就好, 不需要额外配置, 非常方便.

### token 过期问题

使用 kubeadm join 命令新增节点, 需要 2 个参数,--token 与 --discovery-token-ca-cert-hash. 其中, token 有限期一般是 24 小时, 如果超过时间要新增节点, 就需要重新生成 token.

```
# 重新创建 token, 创建完也可以通过 kubeadm token list 命令查看 token 列表
$ kubeadm token create
s058gw.c5x6eeze28****

# 通过以下命令查看 sha256 格式的证书 hash
$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
9592464b295699696ce35e5d1dd155580ee29d9bd0884b*****

# 在新节点执行 join
$  kubeadm join api-serverip: port --token s058gw.c5x6eeze28**** --discovery-token-ca-cert-hash 9592464b295699696ce35e5d1dd155580ee29d9bd0884b*****
```

### 集群测试

跟上篇文章 [minikube 部署](https://segmentfault.com/a/1190000018607114) 一样, 这里部署一个简单的 goweb 服务来测试集群, 运行时暴露 8000 端口, 同时访问 /info 路径会显示容器的主机名.

*   准备 deployment 和 svc 的 yaml:

```
# deployment-goweb.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: goweb
spec:
  selector:
    matchLabels:
      app: goweb
  replicas: 4
  template:
    metadata:
      labels:
        app: goweb
    spec:
      containers:
      - image: lingtony/goweb
        name: goweb
        ports:
        - containerPort: 8000
``````
# svc-goweb.yaml
apiVersion: v1
kind: Service
metadata:
  name: gowebsvc
spec:
  selector:
    app: goweb
  ports:
  - name: default
    protocol: TCP
    port: 80
    targetPort: 8000
```

*   部署服务

```
kubectl apply -f deployment-goweb.yaml
kubectl  apply -y svc-goweb.yaml
```

*   查看 pod 及服务

```
[root@master-1 ~]# kubectl  get po -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP           NODE     NOMINATED NODE   READINESS GATES
goweb-6c569f884-67z89   1/1     Running   0          25m   10.244.1.2   node-1   <none>           <none>
goweb-6c569f884-bt4p6   1/1     Running   0          25m   10.244.1.3   node-1   <none>           <none>
goweb-6c569f884-dltww   1/1     Running   0          25m   10.244.1.4   node-1   <none>           <none>
goweb-6c569f884-vshkm   1/1     Running   0          25m   10.244.3.4   node-2   <none>           <none>
# 可以看到,4 个 pod 分布在不同的 node 上
[root@master-1 ~]# kubectl  get svc
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
gowebsvc     ClusterIP   10.106.202.0   <none>        80/TCP    11m
kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP   21h
# 暴露 80 端口
```

*   测试访问

```
[root@master-1 ~]# curl http://10.106.202.0/info
Hostname: goweb-6c569f884-bt4p6
[root@master-1 ~]# curl http://10.106.202.0/info
Hostname: goweb-6c569f884-67z89
[root@master-1 ~]# curl http://10.106.202.0/info
Hostname: goweb-6c569f884-vshkm
#可以看到, 对 SVC 的请求会在 pod 间负载均衡.
```

# 小结

本文简单介绍了 kubeadm 工具原理, 以及如何用它部署一个高可用的 kubernetes 集群. 需要注意的是, kubeadm 工具总体已经 GA, 可以在生产环境使用了. 但是文中通过 "kubeadm join -experimental-contaol-plane" 参数增加主节点的方式, 还是在 alpha 阶段, 实际在生产环境还是用 init 方式来增加主节点比较稳定.kubeadm 更多详细配置可以参考 [官方文档](https://kubernetes.io/docs/reference/setup-tools/kubeadm/kubeadm/)

[docker](/t/docker) [kubernetes](/t/kubernetes) [容器技术](/t/%E5%AE%B9%E5%99%A8%E6%8A%80%E6%9C%AF)

阅读 14.9k 更新于 2019-07-08

赞 9 收藏 6

[分享](#)

本作品系原创, [采用 <署名 - 非商业性使用 - 禁止演绎 4.0 国际> 许可协议](https://creativecommons.org/licenses/by-nc-nd/4.0/)

* * *

! [](https://image-static.segmentfault.com/383/303/3833030929-5ffd15fc110ab)

[

##### Kubernetes, Istio 点滴

](/blog/lonely_moon)

GO, Kubernetes, Istio 点滴以及记录一些杂七杂八的东西

关注专栏
