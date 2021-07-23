---
title: kubeadm-cluster-cross-vpc
date: 2021-03-20 12:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-cluster-cross-vpc
photo:
---

[kubeadm-cluster-cross-vpc](https://blog.csdn.net/chen645800876/article/details/105833835)

# 云服务器 - 异地部署集群服务 -Kubernetes(K8S)-Kubeadm 安装方式 - 完整篇

! [](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[chen645800876](https://blog.csdn.net/chen645800876) 2020-04-29 10:26:56 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 1728  ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollectionActive.png) 收藏  8   ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/planImg.png) 原力计划

分类专栏: [k8s](https://blog.csdn.net/chen645800876/category_10346129.html) 文章标签: [kubernetes](https://www.csdn.net/tags/MtjaAgxsMDAzMzItYmxvZwO0O0OO0O0O.html) [linux](https://www.csdn.net/tags/MtjaQg5sMDY0MC1ibG9n.html) [docker](https://www.csdn.net/tags/Ntjakg4sNzAwMC1ibG9n.html) [网络](https://www.csdn.net/tags/MtzaIgwsNDIwOTMtYmxvZwO0O0OO0O0O.html) [centos](https://www.csdn.net/tags/MtTaEg0sMzk5NjctYmxvZwO0O0OO0O0O.html)

版权声明: 本文为博主原创文章, 遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议, 转载请附上原文出处链接和本声明.

本文链接: [https://blog.csdn.net/chen645800876/article/details/105833835](https://blog.csdn.net/chen645800876/article/details/105833835)

版权

### 文章目录

*   *   *   *   [一, 前言](#_2)
            *   *   [此方法安装集群后, 无法再添加控制节点,\*\*不推荐使用 \*\*.](#_3)
                *   [1.0 问题](#10__7)
                *   [1.1 绑定弹性网卡](#11__15)
                *   [1.2 二进制安装](#12__23)
                *   [1.3 虚拟网卡 +\`kubeadm\` 安装](#13_kubeadm_31)
            *   [二, 准备 (所有节点)](#__43)
            *   *   [2.1 调整内核参数](#21__51)
                *   [2.2 调整系统时区](#22__67)
                *   [2.3 关闭系统不需要的服务](#23__79)
                *   [2.4 设置 \`rsyslogd\` 和 \`systemd journald\`](#24_rsyslogdsystemd_journald_86)
                *   [2.5 \`ipvs\` 前置条件准备](#25_ipvs_122)
                *   [2.6 \`Docker\` 安装](#26_Docker_144)
                *   [2.7 关闭 swap 分区](#27_swap_175)
                *   [2.8 \`Kubeadm\`,\`Kubelet\`,\`Kubectl\` 安装](#28_KubeadmKubeletKubectl_183)
                *   [2.9 云服务器控制面板打开相应端口](#29__207)
            *   [三, 安装](#_215)
            *   *   [3.1 建立虚拟网卡 (所有节点)](#31__217)
                *   [3.2 修改 \`kubelet\` 启动参数 (重点, 所有节点都要操作)](#32_kubelet_236)
                *   [3.3 使用 \`kubeadm\` 初始化主节点 (主节点)](#33_kubeadm_261)
                *   [3.4 修改 \`kube-apiserver\` 参数 (主节点)](#34_kubeapiserver_324)
                *   [3.5 工作节点加入集群 (工作节点)](#35__415)
                *   [3.6 检查是否加入集群 (主节点)](#36__438)
                *   [3.7 修改 \`flannel\` 文件并安装 (主节点)](#37_flannel_445)
                *   [3.8 检查网络是否连通 (主节点)](#38__557)
                *   [3.9 手动开启配置, 开启 \`ipvs\` 转发模式 (主节点)](#39_ipvs_576)
                *   [3.10 移除虚拟网卡](#310__635)

#### 一, 前言

##### 此方法安装集群后, 无法再添加控制节点,**不推荐使用**.

请看下篇文章:
[云服务器 - 异地部署集群服务 -Kubernetes(K8S)-Kubeadm- 外部 ETCD 服务 - 多 master 节点 - 安装篇](https://blog.csdn.net/chen645800876/article/details/108316772)

##### 1.0 问题

云服务器, 在跨区域安装 Kubernetes(K8S) 集群服务时, 会将内网 IP 注册进集群, 导致节点间网络不通
有三种方案解决:

*   **绑定弹性网卡**
*   **二进制安装**
*   **虚拟网卡 +`kubeadm` 安装**

##### 1.1 绑定弹性网卡

**方案:** 服务器绑定弹性网卡, 网卡绑弹性 IP, 集群时, 直接绑定此 IP, 然后再注册集群即可

这种方式部分主机不支持安装, 比如我买的抢占式实例.

如果你的云主机支持的话, 建议直接上弹性网卡, 省得麻烦

##### 1.2 二进制安装

**方案:** 配置中指定公网和私网 IP

这种方式非常繁琐, 需要配置各种证书, 需要的朋友可以看我第一篇博客

[云服务器 - 异地部署集群服务 -Kubernetes(K8S)- 网络篇](https://blog.csdn.net/chen645800876/article/details/105279648)

##### 1.3 虚拟网卡 +`kubeadm` 安装

**方案:** 虚拟一张网卡, IP 用当前节点的公网 IP, 然后使用此 IP 注册进集群

这种方案比二进制安装简单很多, 不过仍然需要修改一些配置

**本文讲第 3 种安装方式**

#### 二, 准备 (所有节点)

此步骤参考尚硅谷的视频

[尚硅谷 Kubernetes 教程 (17h 深入掌握 k8s)](https://www.bilibili.com/video/BV1w4411y7Go? p=12)

以系统版本 `centos7.6` 版本为例

##### 2.1 调整内核参数

```
cat > k8s.conf <<EOF
#开启网桥模式
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
#开启转发
net.ipv4.ip_forward = 1
##关闭 ipv6
net.ipv6.conf.all.disable_ipv6=1
EOF
cp k8s.conf /etc/sysctl.d/k8s.conf
sysctl -p /etc/sysctl.d/k8s.conf
```

##### 2.2 调整系统时区

```
# 设置系统时区为 中国 / 上海
timedatectl set-timezone Asia/Shanghai
# 将当前的 UTC 时间写入硬件时钟
timedatectl set-local-rtc 0
# 重启依赖于系统时间的服务
systemctl restart rsyslog
systemctl restart crond
```

##### 2.3 关闭系统不需要的服务

```
#关闭邮件服务
systemctl stop postfix && systemctl disable postfix
```

##### 2.4 设置 `rsyslogd` 和 `systemd journald`

默认有两个日志服务, 使用 `journald` 关闭 `rsyslogd`

```
mkdir /var/log/journal # 持久化保存日志的目录
mkdir /etc/systemd/journald.conf.d
cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
[Journal]
# 持久化
Storage= persistent

# 压缩历史日志
Compress= yes

SysnIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000

# 最大占用空间 10G
SystemMaxUse=10G

# 单日志文件最大 200M
SystemMaxFileSize=200M

# 日志保存时间 2 周
MaxRetentionSec=2week

# 不将日志转发到 syslog
ForwardToSyslog= no

EOF

systemctl restart systemd-journald
```

##### 2.5 `ipvs` 前置条件准备

`ipvs` 转发效率比 `iptables` 更高, 看上去也比 `iptables` 舒服

```
# step1
modprobe br_netfilter

# step2
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

# step3
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

##### 2.6 `Docker` 安装

```
# step 1: 安装必要的一些系统工具
yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 更新并安装 Docker-CE
yum makecache fast
yum -y install docker-ce
# Step 4: 开启 Docker 服务
systemctl start docker

# 创建 `/etc/docker` 目录
mkdir -p /etc/docker

# 配置 `daemon`
cat > /etc/docker/daemon.json << EOF
{
  "exec-opts": ["native.cgroupdriver= systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  }
}
EOF

# 启动 docker
systemctl daemon-reload && systemctl restart docker && systemctl enable docker
```

##### 2.7 关闭 swap 分区

```
swapoff -a
```

##### 2.8 `Kubeadm`,`Kubelet`,`Kubectl` 安装

```
# 添加源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name= Kubernetes
baseurl= https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey= https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 关闭 selinux
setenforce 0

# 安装 kubelet, kubeadm, kubectl
yum install -y kubelet kubeadm kubectl

# 设置为开机自启
systemctl enable kubelet
```

##### 2.9 云服务器控制面板打开相应端口

*   `10250/10260` `TCP` 端口: 给 `kube-schedule`,`kube-controll`,`kube-proxy`,`kubelet` 等使用

*   `6443` `TCP` 端口: 给 `kube-apiserver` 使用

*   `2379` `2380` `2381` `TCP` 商品:`ETCD` 使用

*   `8472` `UDP` 端口:`vxlan` 使用端口


#### 三, 安装

##### 3.1 建立虚拟网卡 (所有节点)

```
# step1 , 注意替换你的公网 IP 进去
cat > /etc/sysconfig/network-scripts/ifcfg-eth0:1 <<EOF
BOOTPROTO= static
DEVICE= eth0:1
IPADDR= 你的公网 IP
PREFIX=32
TYPE= Ethernet
USERCTL= no
ONBOOT= yes
EOF
# step2 如果是 centos8, 需要重启
systemctl restart network
# step3 查看新建的 IP 是否进去
ip addr
```

##### 3.2 修改 `kubelet` 启动参数 (重点, 所有节点都要操作)

```
# 此文件安装 kubeadm 后就存在了
vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

# 注意, 这步很重要, 如果不做, 节点仍然会使用内网 IP 注册进集群
# 在末尾添加参数 --node-ip= 公网 IP

# Note: This dropin only works with kubeadm and kubelet v1.11+
[Service]
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf"
Environment="KUBELET_CONFIG_ARGS=--config=/var/lib/kubelet/config.yaml"
# This is a file that "kubeadm init" and "kubeadm join" generates at runtime, populating the KUBELET_KUBEADM_ARGS variable dynamically
EnvironmentFile=-/var/lib/kubelet/kubeadm-flags.env
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/sysconfig/kubelet
ExecStart=
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS --node-ip=47.74.22.13
```

##### 3.3 使用 `kubeadm` 初始化主节点 (主节点)

**如果网络问题, 不能下载相关镜像**

**请先手动导入!!!**

```
# 主节点相关镜像
[root@master images]# docker images
REPOSITORY                           TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy                v1.18.0             43940c34f24f        4 weeks ago         117MB
k8s.gcr.io/kube-controller-manager   v1.18.0             d3e55153f52f        4 weeks ago         162MB
k8s.gcr.io/kube-scheduler            v1.18.0             a31f78c7c8ce        4 weeks ago         95.3MB
k8s.gcr.io/kube-apiserver            v1.18.0             74060cea7f70        4 weeks ago         173MB
k8s.gcr.io/pause                     3.2                 80d28bedfe5d        2 months ago        683kB
k8s.gcr.io/coredns                   1.6.7               67da37a9a360        3 months ago        43.8MB
k8s.gcr.io/etcd                      3.4.3-0             303ce5db0e90        6 months ago        288MB
quay.io/coreos/flannel               v0.11.0-amd64       ff281650a721        15 months ago       52.6MB
``````
# step1 添加配置文件, 注意替换下面的 IP
cat > kubeadm-config.yaml <<EOF
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.0
apiServer:
  certSANs:    #填写所有 kube-apiserver 节点的 hostname, IP, VIP
  - master    #请替换为 hostname
  - 47.74.22.13   #请替换为公网
  - 175.24.19.12  #请替换为私网
  - 10.96.0.1   #不要替换, 此 IP 是 API 的集群地址, 部分服务会用到
controlPlaneEndpoint: 47.74.22.13:6443 #替换为公网 IP
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
--- 将默认调度方式改为 ipvs
apiVersion: kubeproxy-config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs
EOF

# step2 如果是 1 核心或者 1G 内存的请在末尾添加参数 (--ignore-preflight-errors= all), 否则会初始化失败
# 同时注意, 此步骤成功后, 会打印, 两个重要信息
kubeadm init --config= kubeadm-config.yaml

# 信息 1 上面初始化成功后, 将会生成 kubeconfig 文件, 用于请求 api 服务器, 请执行下面操作
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 信息 2 此信息用于后面工作节点加入主节点使用
kubeadm join 47.74.22.13:6443 --token sdfs.dsfsdfsdfijdth \
    --discovery-token-ca-cert-hash sha256: sdfsdfsdfsdfsdfsdfsdfsdfg9a460f44b118050091245c1d
```

##### 3.4 修改 `kube-apiserver` 参数 (主节点)

```
# 修改两个信息, 添加 --bind-address 和修改 --advertise-address
vim /etc/kubernetes/manifests/kube-apiserver.yaml

apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 47.74.22.13:6443
  creationTimestamp: null
  labels:
    component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    - --advertise-address=47.74.22.13  #修改为公网 IP
    - --bind-address=0.0.0.0 #添加此参数
    - --allow-privileged= true
    - --authorization-mode= Node, RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins= NodeRestriction
    - --enable-bootstrap-token-auth= true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers= https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types= InternalIP, ExternalIP, Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names= front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix= X-Remote-Extra-
    - --requestheader-group-headers= X-Remote-Group
    - --requestheader-username-headers= X-Remote-User
    - --secure-port=6443
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
    image: k8s.gcr.io/kube-apiserver: v1.18.0
    imagePullPolicy: IfNotPresent
    livenessProbe:
      failureThreshold: 8
      httpGet:
        host: 175.24.19.12
        path: /healthz
        port: 6443
        scheme: HTTPS
      initialDelaySeconds: 15
      timeoutSeconds: 15
    name: kube-apiserver
    resources:
      requests:
        cpu: 250m
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/pki
      name: etc-pki
      readOnly: true
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
  hostNetwork: true
  priorityClassName: system-cluster-critical
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/pki
      type: DirectoryOrCreate
    name: etc-pki
  - hostPath:
      path: /etc/kubernetes/pki
      type: DirectoryOrCreate
    name: k8s-certs
status: {}
```

##### 3.5 工作节点加入集群 (工作节点)

**如果网络问题, 不能下载相关镜像**

**请先手动导入!!!**

```
## 工作节点相关镜像
k8s.gcr.io/kube-proxy    v1.18.0             43940c34f24f        4 weeks ago         117MB
k8s.gcr.io/pause         3.2                 80d28bedfe5d        2 months ago        683kB
k8s.gcr.io/coredns       1.6.7               67da37a9a360        3 months ago        43.8MB
quay.io/coreos/flannel   v0.11.0-amd64       ff281650a721        15 months ago       52.6MB
``````
# 需要虚拟 IP 和 kubelet 启动参数都改成功后, 再执行
kubeadm join 47.74.22.13:6443 --token sdfs.dsfsdfsdfijdth \
    --discovery-token-ca-cert-hash sha256: sdfsdfsdfsdfsdfsdfsdfsdfg9a460f44b118050091245c1d
```

##### 3.6 检查是否加入集群 (主节点)

```
# 成功后, INTERNAL-IP 均显示公网 IP
[root@master ~]# kubectl get nodes -o wide
```

##### 3.7 修改 `flannel` 文件并安装 (主节点)

```
# 下载 flannel 的 yaml 配置文件

# 共修改两个地方, 一个是 args 下, 添加
 args:
 - --public-ip=$(PUBLIC_IP) # 添加此参数, 申明公网 IP
 - --iface= eth0             # 添加此参数, 绑定网卡


 # 然后是 env 下
 env:
 - name: PUBLIC_IP     #添加环境变量
   valueFrom:
     fieldRef:
       fieldPath: status.podIP
``````
... 略

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds-amd64
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
              - matchExpressions:
                  - key: beta.kubernetes.io/os
                    operator: In
                    values:
                      - linux
                  - key: beta.kubernetes.io/arch
                    operator: In
                    values:
                      - amd64
      hostNetwork: true
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: quay.io/coreos/flannel: v0.11.0-amd64
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel: v0.11.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --public-ip=$(PUBLIC_IP) # 添加此参数, 申明公网 IP
        - --iface= eth0             # 添加此参数, 绑定网卡
        - --kube-subnet-mgr
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
             add: ["NET_ADMIN"]
        env:
        - name: PUBLIC_IP     #添加环境变量
          valueFrom:          #
            fieldRef:          #
              fieldPath: status.podIP #
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
... 略
``````
#创建 flannel
kubectl apply -f flannel.yaml
```

##### 3.8 检查网络是否连通 (主节点)

```
# 检查 pod 是否都是 ready 状态
kubectl get pods -o wide --all-namespaces
...

# 手动创建一个 pod
kubectl create deployment nginx --image= nginx

# 查看 pod 的 ip
kubectl get pods -o wide

# 主节点或其它节点, ping 一下此 ip, 看看是否能 ping 通

# 没有的话, 查看 2.9 章节中说明的端口是否打开
```

##### 3.9 手动开启配置, 开启 `ipvs` 转发模式 (主节点)

```
# 前面都成功了, 但是有时候默认并不会启用 `IPVS` 模式, 那就手动修改一下, 只修改一处
# 修改后, 如果没有及时生效, 请删除 kube-proxy, 会自动重新创建, 然后使用 ipvsadm -Ln 命令, 查看是否生效
# ipvsadm 没有安装的, 使用 yum install ipvsadm 安装
kubectl edit configmaps -n kube-system kube-proxy

---
apiVersion: v1
data:
  config.conf: |-
    apiVersion: kubeproxy.config.k8s.io/v1alpha1
    bindAddress: 0.0.0.0
    clientConnection:
      acceptContentTypes: ""
      burst: 0
      contentType: ""
      kubeconfig: /var/lib/kube-proxy/kubeconfig.conf
      qps: 0
    clusterCIDR: 10.244.0.0/16
    configSyncPeriod: 0s
    conntrack:
      maxPerCore: null
      min: null
      tcpCloseWaitTimeout: null
      tcpEstablishedTimeout: null
    detectLocalMode: ""
    enableProfiling: false
    healthzBindAddress: ""
    hostnameOverride: ""
    iptables:
      masqueradeAll: false
      masqueradeBit: null
      minSyncPeriod: 0s
      syncPeriod: 0s
    ipvs:
      excludeCIDRs: null
      minSyncPeriod: 0s
      scheduler: ""
      strictARP: false
      syncPeriod: 0s
      tcpFinTimeout: 0s
      tcpTimeout: 0s
      udpTimeout: 0s
    kind: KubeProxyConfiguration
    metricsBindAddress: ""
    mode: "ipvs"  # 如果为空, 请填入 `ipvs`
    nodePortAddresses: null
    oomScoreAdj: null
    portRange: ""
    showHiddenMetricsForVersion: ""
    udpIdleTimeout: 0s
    winkernel:
      enableDSR: false
      networkName: ""
...
```

##### 3.10 移除虚拟网卡

**注意:** 此步骤, 可以不执行, 因为实测不移除, 节点间的网络也通了, 如果是 `centos8` 网络不通, 再执行下面的操作, 移除虚拟虚拟网卡

```
 # 移除虚拟网卡
 mv /etc/sysconfig/network-scripts/ifcfg-eth0\:1 /root/
 # 重启
 reboot
```
