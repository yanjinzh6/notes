---
title: kubeadm-cluster-cross-vpc-external-etcd
date: 2021-03-20 13:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-cluster-cross-vpc-external-etcd
photo:
---

[kubeadm-cluster-cross-vpc-external-etcd](https://blog.csdn.net/chen645800876/article/details/108316772)

# 云服务器 - 异地部署集群服务 -Kubernetes(K8S)-Kubeadm- 外部 ETCD 服务 - 多 master 节点 - 安装篇

! [](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[chen645800876](https://blog.csdn.net/chen645800876) 2020-08-31 10:44:28 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 816 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollectionActive.png) 收藏  1

分类专栏: [k8s](https://blog.csdn.net/chen645800876/category_10346129.html) 文章标签: [kubernetes](https://www.csdn.net/tags/MtjaAgxsMDAzMzItYmxvZwO0O0OO0O0O.html) [docker](https://www.csdn.net/tags/Ntjakg4sNzAwMC1ibG9n.html) [etcd](https://www.csdn.net/tags/MtTaEg0sMzQ0MDQtYmxvZwO0O0OO0O0O.html) [云服务器](https://www.csdn.net/tags/MtTaEg0sNDk3MzAtYmxvZwO0O0OO0O0O.html)

版权声明: 本文为博主原创文章, 遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议, 转载请附上原文出处链接和本声明.

本文链接: [https://blog.csdn.net/chen645800876/article/details/108316772](https://blog.csdn.net/chen645800876/article/details/108316772)

版权

### 文章目录

*   *   *   *   [一, 前言](#_2)
            *   *   [1.1 问题](#11__4)
                *   [1.2 解决方案](#12__16)
                *   [1.3 服务器配置](#13__20)
                *   [1.4 准备工作 (所有节点操作)](#14__28)
            *   [二, 使用 \`kubeadm\` 部署 \`etcd\` 集群](#kubeadmetcd_130)
            *   *   [2.1 准备配置文件](#21__132)
                *   *   [\`master01\` 节点配置文件](#master01_134)
                    *   [\`master02\` 节点配置](#master02_200)
                    *   [\`master03\` 节点配置](#master03_247)
                *   [2.2 搭建 \`etcd\`](#22_etcd_294)
                *   *   [2.2.1 \`master01\` 节点上操作](#221_master01_296)
                    *   [2.2.2 复制四个 \`ca\` 证书到其它 \`master\` 节点](#222_camaster_321)
                    *   [2.2.3 \`master02\`,\`master03\` 节点执行以下操作](#223_master02master03_336)
                    *   [2.2.4 检查 \`etcd\` 集群是否部署成功 (任意节点)](#224_etcd_355)
            *   [三, 部署集群](#_375)
            *   *   [3.1 初始化主节点](#31__377)
                *   *   [3.1.1 初始化 \`master01\` 节点](#311_master01_379)
                    *   [3.1.2 \`master02\`,\`master03\` 加入集群](#312_master02master03_390)
                *   [3.2 部署 \`flannel\` 网络 (\`master01\` 节点)](#32_flannelmaster01_401)
                *   *   [3.2.12 下载 \`flannel.yaml\` 文件](#3212flannelyaml_462)
                    *   [3.2.3 修改 flannel 配置](#323_flannel_468)
                    *   [3.2.4 部署 flannel](#324_flannel_580)
                *   [3.3 修改 \`master02\`,\`master03\` 节点的一些配置](#33_master02master03_586)
                *   *   [3.3.1 修改 \`kube-apiserver\` 配置](#331_kubeapiserver_588)
                    *   [3.3.2 修改 \`admin.conf\` 配置](#332_adminconf_619)
                *   [3.4 部署成功后状态](#34__638)
            *   [四, 易出问题](#_644)
            *   *   [4.1 etcd 集群无法建立](#41_etcd_646)
                *   [4.2 网络不通](#42__650)
                *   *   [4.2.1 flannel 配置问题](#421_flannel_652)
                    *   [4.2.2 使用的 \`kubelet\`,\`kubeadm\` 版本不一致](#422_kubeletkubeadm_666)
                    *   [4.2.3 没有把上次安装的东西清理干净](#423__672)

#### 一, 前言

##### 1.1 问题

之前已经写过一篇关于云服务器异地部署 `k8s` 的文章:

[云服务器 - 异地部署集群服务 -Kubernetes(K8S)-Kubeadm 安装方式 - 完整篇](https://blog.csdn.net/chen645800876/article/details/105833835)

然后有小伙伴私信我, 说多 `master` 节点无法正常部署.

于是和他研究一番, 发现是由于 `etcd` 集群无法正常扩展导致的.

目前不知道怎么解决扩展 `etcd` 的问题, 但是找到了一个还算比较简单的方式, 可以完成多 `master` 节点的部署

##### 1.2 解决方案

利用 `kubeadm` 的分段部署方案, 先把 `etcd` 集群部署好, 然后以外部 `etcd` 的方式部署集群

##### 1.3 服务器配置

| 节点 | 内网 ip | 公网 ip | 配置 |
| --- | --- | --- | --- |
| master01(香港) | 172.31.191.81 | 47.242.37.143 | 2C2G |
| master02(新加坡) | 172.21.221.64 | 47.241.14.2 | 2C2G |
| master03(上海) | 172.26.42.80 | 47.100.165.85 | 2C2G |

##### 1.4 准备工作 (所有节点操作)

不再赘述, 详细请参考上篇文章:

[云服务器异地部署 - 准备阶段](https://blog.csdn.net/chen645800876/article/details/105833835#__39)

主要是

*   建立虚拟 `IP`

    ```
    cat > /etc/sysconfig/network-scripts/ifcfg-eth0:1 <<EOF
    BOOTPROTO= static
    DEVICE= eth0:1
    IPADDR=47.242.37.143
    PREFIX=32
    TYPE= Ethernet
    USERCTL= no
    ONBOOT= yes
    EOF

    cat > /etc/sysconfig/network-scripts/ifcfg-eth0:1 <<EOF
    BOOTPROTO= static
    DEVICE= eth0:1
    IPADDR=47.241.14.2
    PREFIX=32
    TYPE= Ethernet
    USERCTL= no
    ONBOOT= yes
    EOF

    cat > /etc/sysconfig/network-scripts/ifcfg-eth0:1 <<EOF
    BOOTPROTO= static
    DEVICE= eth0:1
    IPADDR=47.100.165.85
    PREFIX=32
    TYPE= Ethernet
    USERCTL= no
    ONBOOT= yes
    EOF

    systemctl daemon-reload && systemctl restart network
    ```

    效果如下

    ```
    [root@master01 kubeadm]# ip addr
    1: lo: <LOOPBACK, UP, LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
    2: eth0: <BROADCAST, MULTICAST, UP, LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 00:16:3e:00: c8:86 brd ff: ff: ff: ff: ff: ff
        inet 172.31.191.81/20 brd 172.31.191.255 scope global dynamic eth0
           valid_lft 315260578sec preferred_lft 315260578sec
        inet 47.242.37.143/32 brd 47.242.37.143 scope global eth0:1
    ```
*   关闭 `swap` 分区

*   安装 `docker`,`kubelet`,`kubeadm`,`kubectl`

*   打开相应端口

*   [给 kubelet 启动项, 添加 `--node-ip`\= 公网 IP 参数](https://blog.csdn.net/chen645800876/article/details/105833835#32_kubelet_232)

    ```
    # 修改启动文件末尾添加 --node-ip 参数, 所有节点都要做
    [root@master01 kubeadm]# vim /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf

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
    ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS --node-ip=47.242.37.143
    ```
*   使用 `kubeadm config images list` 命令查看所有需要下载的镜像, 服务器无法下载镜像时, 请手动导入相关镜像

    ```
    [root@master01 kubeadm]# kubeadm config images list
    I0828 23:55:46.054281   13938 version.go:252] remote version is much newer: v1.19.0; falling back to: stable-1.18
    W0828 23:55:46.623163   13938 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
    k8s.gcr.io/kube-apiserver: v1.18.8
    k8s.gcr.io/kube-controller-manager: v1.18.8
    k8s.gcr.io/kube-scheduler: v1.18.8
    k8s.gcr.io/kube-proxy: v1.18.8
    k8s.gcr.io/pause:3.2
    k8s.gcr.io/etcd:3.4.3-0
    k8s.gcr.io/coredns:1.6.7
    ```

#### 二, 使用 `kubeadm` 部署 `etcd` 集群

##### 2.1 准备配置文件

###### `master01` 节点配置文件

`kubeadm-config.yaml`

```
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.0
apiServer:
  certSANs:    #填写所有 kube-apiserver 节点的 hostname, IP, VIP
  - master    #请替换为 hostname
  - master01
  - master02
  - master03
  - 47.242.37.143   #请替换为公网
  - 172.31.191.81  #请替换为私网
  - 47.241.14.2
  - 172.21.221.64
  - 47.100.165.85
  - 172.26.42.80
  - 10.96.0.1   #不要替换, 此 IP 是 API 的集群地址, 部分服务会用到
  extraArgs:
    advertise-address: 47.242.37.143
etcd:
  external:
        endpoints:
        - https://47.242.37.143:2379
        - https://47.241.14.2:2379
        - https://47.100.165.85:2379
        caFile: /etc/kubernetes/pki/etcd/ca.crt
        certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
        keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
controlPlaneEndpoint: 47.242.37.143:6443 #替换为公网 IP
networking:
  podSubnet: 10.244.0.0/16
  serviceSubnet: 10.96.0.0/12
--- 将默认调度方式改为 ipvs
apiVersion: kubeproxy-config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs
```

`kubeadm-etcd.yaml`

```
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.0
etcd:
  local:
      serverCertSANs:
      - 47.242.37.143   #请替换为公网
      peerCertSANs:
      - 47.242.37.143   #请替换为公网
      extraArgs:
            initial-cluster: infra0= https://47.242.37.143:2380, infra1= https://47.241.14.2:2380, infra2= https://47.100.165.85:2380
            initial-cluster-state: new
            name: infra0
            listen-peer-urls: https://0.0.0.0:2380
            listen-client-urls: https://0.0.0.0:2379
            advertise-client-urls: https://127.0.0.1:2379, https://47.242.37.143:2379
            initial-advertise-peer-urls: https://47.242.37.143:2380
```

###### `master02` 节点配置

`kubeadm-config.yaml`

```
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.0
apiServer:
  certSANs:    #填写所有 kube-apiserver 节点的 hostname, IP, VIP
  - master    #请替换为 hostname
  - master01
  - master02
  - master03
  - 47.242.37.143   #请替换为公网
  - 172.31.191.81  #请替换为私网
  - 47.241.14.2
  - 172.21.221.64
  - 47.100.165.85
  - 172.26.42.80
  - 10.96.0.1   #不要替换
  extraArgs:
    advertise-address: 47.241.14.2
controlPlaneEndpoint: 47.241.14.2:6443 #替换为公网 IP
```

`kubeadm-etcd.yaml`

```
apiVersion: "kubeadm.k8s.io/v1beta2"
kind: ClusterConfiguration
etcd:
    local:
        serverCertSANs:
        - 47.241.14.2
        peerCertSANs:
        - 47.241.14.2
        extraArgs:
            initial-cluster: infra0= https://47.242.37.143:2380, infra1= https://47.241.14.2:2380, infra2= https://47.100.165.85:2380
            initial-cluster-state: new
            name: infra1
            listen-peer-urls: https://0.0.0.0:2380
            listen-client-urls: https://0.0.0.0:2379
            advertise-client-urls: https://127.0.0.1:2379, https://47.241.14.2:2379
            initial-advertise-peer-urls: https://47.241.14.2:2380
```

###### `master03` 节点配置

`kubeadm-config.yaml`

```
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.18.0
apiServer:
  certSANs:    #填写所有 kube-apiserver 节点的 hostname, IP, VIP
  - master    #请替换为 hostname
  - master01
  - master02
  - master03
  - 47.242.37.143   #请替换为公网
  - 172.31.191.81  #请替换为私网
  - 47.241.14.2
  - 172.21.221.64
  - 47.100.165.85
  - 172.26.42.80
  - 10.96.0.1   #不要替换, 此 IP 是 API 的集群地址, 部分服务会用到
  extraArgs:
    advertise-address: 47.100.165.85
controlPlaneEndpoint: 47.100.165.85:6443 #替换为公网 IP
```

`kubeadm-etcd.yaml`

```
apiVersion: "kubeadm.k8s.io/v1beta2"
kind: ClusterConfiguration
etcd:
    local:
        serverCertSANs:
        - 47.100.165.85
        peerCertSANs:
        - 47.100.165.85
        extraArgs:
            initial-cluster: infra0= https://47.242.37.143:2380, infra1= https://47.241.14.2:2380, infra2= https://47.100.165.85:2380
            initial-cluster-state: new
            name: infra2
            listen-peer-urls: https://0.0.0.0:2380
            listen-client-urls: https://0.0.0.0:2379
            advertise-client-urls: https://127.0.0.1:2379, https://47.100.165.85:2379
            initial-advertise-peer-urls: https://47.100.165.85:2380
```

##### 2.2 搭建 `etcd`

###### 2.2.1 `master01` 节点上操作

```
# 生成相关证书, 其中 ca 证书要复制到其它主节点中
kubeadm init phase certs ca # 将在 /etc/kubenertes/pki 路径下生成 ca.crt 和 ca.key 文件, 请复制这两个文件到其它节点相同位置上
kubeadm init phase certs etcd-ca  # 将在 /etc/kubenertes/pki/etcd 路径下生成 ca.crt 和 ca.key 文件, 请复制这两个文件到其它节点相同位置上

# 生成 etcd 相关证书
kubeadm init phase certs etcd-server --config= kubeadm-etcd.yaml
kubeadm init phase certs etcd-peer  --config= kubeadm-etcd.yaml
kubeadm init phase certs etcd-healthcheck-client  --config= kubeadm-etcd.yaml
kubeadm init phase certs apiserver-etcd-client  --config= kubeadm-etcd.yaml
kubeadm init phase certs sa

# 启动 kubelet
kubeadm init phase kubelet-start --config= kubeadm-config.yaml
# 生成配置文件
kubeadm init phase kubeconfig kubelet --config= kubeadm-config.yaml
# 启动 etcd
kubeadm init phase etcd local --config= kubeadm-etcd.yaml
```

用 `docker ps` 命令查看 `etcd` 是否启动

###### 2.2.2 复制四个 `ca` 证书到其它 `master` 节点

将 `master01` 节点生成的 `/etc/kubenertes/pki` 路径中 `ca.key`,`ca.crt` 和 `/etc/kubernetes/pki/etcd` 路径中 `ca.key`,`ca.crt` 复制到 `master02` 和 `master03` 节点相同位置

复制这两个证书到其它节点相同位置

```
/etc/kubernetes/pki/
├── ca.crt
├── ca.key
├── etcd
│   ├── ca.crt
│   ├── ca.key
```

###### 2.2.3 `master02`,`master03` 节点执行以下操作

```
# 保证 ca 根证书已经复制过来

kubeadm init phase certs etcd-server --config= kubeadm-etcd.yaml
kubeadm init phase certs etcd-peer  --config= kubeadm-etcd.yaml
kubeadm init phase certs etcd-healthcheck-client  --config= kubeadm-etcd.yaml
kubeadm init phase certs apiserver-etcd-client  --config= kubeadm-etcd.yaml
kubeadm init phase certs sa  # 不用加配置

# 启动 kubelet
kubeadm init phase kubelet-start --config= kubeadm-config.yaml
kubeadm init phase kubeconfig kubelet --config= kubeadm-config.yaml
# etcd 集群
kubeadm init phase etcd local --config= kubeadm-etcd.yaml
```

###### 2.2.4 检查 `etcd` 集群是否部署成功 (任意节点)

```
[root@master1 ~]# docker run --rm -it \
--net host \
-v /etc/kubernetes:/etc/kubernetes k8s.gcr.io/etcd:3.4.3-0 etcdctl \
--cert /etc/kubernetes/pki/etcd/peer.crt \
--key /etc/kubernetes/pki/etcd/peer.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--endpoints  https://47.242.37.143:2379, https://47.100.165.85:2379, https://47.241.14.2:2379 endpoint health --cluster

# 结果
https://127.0.0.1:2379 is healthy: successfully committed proposal: took = 144.788934ms
https://127.0.0.1:2379 is healthy: successfully committed proposal: took = 282.870507ms
https://127.0.0.1:2379 is healthy: successfully committed proposal: took = 282.819733ms
https://47.100.165.85:2379 is healthy: successfully committed proposal: took = 284.760742ms
https://47.242.37.143:2379 is healthy: successfully committed proposal: took = 516.849874ms
https://47.241.14.2:2379 is healthy: successfully committed proposal: took = 531.566588ms
```

#### 三, 部署集群

##### 3.1 初始化主节点

###### 3.1.1 初始化 `master01` 节点

```
# 初始化第一个 master 节点
# --skip-phases= preflight 跳过检查
kubeadm init --skip-phases= preflight --config= kubeadm-config.yaml

# 此步骤成功后, 将会打印下面信息
kubeadm join 47.242.37.143:6443 --token pazu6d.xdfsdfg8ztaziohevb     --discovery-token-ca-cert-hash sha256: b69171665f63f9f52fb1ea09054e717672ac9b0de4a323369df176b3d2ec6b     --control-plane
```

###### 3.1.2 `master02`,`master03` 加入集群

```
# 加入集群
# --skip-phases= preflight 是为了跳过检查, 由于已经配置了 etcd, kubelet 也已经启动了

kubeadm join --skip-phases= preflight 47.242.37.143:6443 --token pazu6d.xdbcg8ztaziohevb \
    --discovery-token-ca-cert-hash sha256: b69171665f63f9f52fb1ea09054e717672ac9b0de4a323369bd69176b3d2ec6b \
    --control-plane
```

##### 3.2 部署 `flannel` 网络 (`master01` 节点)

3.2.1 手动开启 `ipvs` 转发模式

```
# 前面都成功了, 但是有时候默认并不会启用 `IPVS` 模式, 那就手动修改一下, 只修改一处
# 注意: 修改后, 请删除所有 kube-proxy 的 pod, 它会自动重新创建, 然后使用 ipvsadm -Ln 命令, 查看是否生效
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

###### 3.2.12 下载 `flannel.yaml` 文件

```
wget https://raw.githubusercontent.com/coreos/flannel/2140ac876ef134e0ed5af15c65e414cf26827915/Documentation/kube-flannel.yml
```

###### 3.2.3 修改 flannel 配置

注意, 一般改 `amd64` 架构相关配置, 如果你机器有其它架构, 那一并修改

总共修改**两处**, 目的是让 `flannel` 绑定本地接口, 同时不使用本地接口的 `IP`, 改用声明的公网 `IP` 地址

```
……略

---
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
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: PUBLIC_IP     #添加环境变量
          valueFrom:          #
            fieldRef:          #
              fieldPath: status.podIP #
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run/flannel
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg
……略
```

###### 3.2.4 部署 flannel

```
kubectl apply -f kube-flannel.yaml
```

##### 3.3 修改 `master02`,`master03` 节点的一些配置

###### 3.3.1 修改 `kube-apiserver` 配置

```
# vim /etc/kubernetes/manifests/kube-apiserver.yaml

apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 172.26.42.80:6443
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
    - --advertise-address=47.242.37.143   # 分别改为 master02 和 master03 节点的公网 ip 地址
    - --allow-privileged= true
    - --authorization-mode= Node, RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins= NodeRestriction
    - --enable-bootstrap-token-auth= true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt

……
```

###### 3.3.2 修改 `admin.conf` 配置

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUN5RENDQWJDZ0F3SUJBZ0lCQURBTkJna3Foa2lHOXcwQkFRc0ZBREFWTVJNd0VRWURWUVFERXdwcmRXSmwKY201bGRHVnpNQjRYRFRJd01EZ3pNVEF4TkRBME5Wb1hEVE13TURneU9UQXhOREEwTlZvd0ZURVRNQkVHQTFVRQpBeE1LYTNWaVpYSnVaWFJsY3pDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBdfdsfuTEVkdm9kWWtJcEZlWE9FTWhUdHNwcmVFMmlkUzRZaEhWWmhmUERDQklDczR5eVR0enlibG0KZ3V6RGNxcHFHTVdzdS9UOElTakJodi9sejZuTXZsZXc4KzJyMUd3TXNnWVo3dUJQZnltMlBJL2VTUTNoMWorYQpXcmlYSkhpSlhmaG1YR2JtZGlrQ0F3RUFBYU1qTUNFd0RnWURWUjBQQVFIL0JBUURBZ0trTUE4R0ExVWRFd0VCCi93UUZNQU1CQWY4d0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dFQkFMNlRQSE9uYTlGVTZIeVBMSGVtazRUaDgvZzEKK1A2Wmo1U0hGN213eU1sME9MR1dyTVpLL2ZxNEpKY3hHQkFzQ0NvV2NXMi82cEYxM2hZVlMzckd6cUlNRFV6NwpXL0FBRWtRYXBveGVpNXM2dEE0d250T0x2SDJKeG1hQ2dIN1l1L2x1cnlOY3E3Z2VXc1lueisvNGZvbjRPN3lXClQ2cDBGZzlJclJDSytQNFd3aitjY09vYjMwQlI1dVg3Sno0RWZjNWZQWjlDMkVaSlBnNlV3cVpFcFhSaGFnbVAKQmQrN1N0aEJ2RHFPZzdlTk50Zmo4enEyc3lyMVpJcFhpeWhCMERmdDM5NDBPTTNkZkpXeGsrK0draXcwSWEyNwpjSVRrck5YYTBJMWF1WklIbVdPamQyejhwdjRCN2tic1ZWVkNGeTBpWE9VMjFxNmFhTVdmVzh0cjJyQT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo=
    server: https://47.242.37.143:6443 #分别改为 master01 和 master02 的公网 ip 地址
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
```

##### 3.4 部署成功后状态

! [node 节点信息](https://img-blog.csdnimg.cn/20200831103607880.png#pic_center)
! [各 pod 信息](https://img-blog.csdnimg.cn/20200831103627271.png? x-oss-process= image/watermark, type_ZmFuZ3poZW5naGVpdGk, shadow_10, text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2NoZW42NDU4MDA4NzY=, size_16, color_FFFFFF, t_70#pic_center)

#### 四, 易出问题

##### 4.1 etcd 集群无法建立

使用 `docker logs` 查看 `etcd` 的日志, 如果是因为访问其它节点超时, 请移除虚拟网卡后重试, 建立成功后, 可以去除虚拟 `IP`

##### 4.2 网络不通

###### 4.2.1 flannel 配置问题

先查看 flannel 的日志

```
# kubectl logs -n kube-system kube-flannel-ds-amd64-qkqxx
Using interface with name eth0 and address 172.26.42.80
Using 47.100.165.85 as external address
```

正常情况, flannel 绑定的接口地址是内网的和外部地址是公网

如果都绑定了内网的, 请检查 `/usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf` 文件中末尾是否添加了 `--node-ip= 公网 IP` 参数检查, 检查虚拟 `IP` 是否建立

###### 4.2.2 使用的 `kubelet`,`kubeadm` 版本不一致

可能 错误是 `Unable to update cni config: no networks found in /etc/cni/net.d`

我在搭建过程中遇到的问题, 一直没找到原因, 因为两台完全一样的操作, 为什么一台正常, 一台又不能用. 幸亏朋友提醒了一下才发现, 不然坑死了

###### 4.2.3 没有把上次安装的东西清理干净

```
rm -rf /var/lib/cni/flannel/* && rm -rf /var/lib/cni/networks/cbr0/* && ip link delete cni0 && rm -rf /var/lib/cni/networks/cni0/* && systemctl stop kubelet && systemctl stop docker && iptables --flush && iptables -tnat --flush && systemctl start kubelet && systemctl start docker && ipvsadm --clear && rm -rf $HOME/.kube/config
```
