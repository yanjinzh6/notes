---
title: k8s-port-iptables
date: 2021-03-14 15:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: k8s-port-iptables
photo:
---

[k8s-port-iptables](https://blog.csdn.net/weixin_43405242/article/details/108872333)

# Kubernetes 开启 iptables 后 规则添加以及验证

! [](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

[weixin\_43405242](https://blog.csdn.net/weixin_43405242) 2020-10-10 12:38:47 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png) 375 ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png) ! [](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollectionActive.png) 收藏

分类专栏: [Kubernetes](https://blog.csdn.net/weixin_43405242/category_10439490.html)

版权声明: 本文为博主原创文章, 遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议, 转载请附上原文出处链接和本声明.

本文链接: [https://blog.csdn.net/weixin\_43405242/article/details/108872333](https://blog.csdn.net/weixin_43405242/article/details/108872333)

版权

## K8s 开启 iptables 后规则设置和验证

## 1.Master node 需要配置的 iptables 规则

### 1.1 INPUT 链

iptables 的表, 链, 端口对应的服务地址

| Protocol | iptables table | ipables chain | Port Range | Purpose | Used by |
| --- | --- | --- | --- | --- | --- |
| UDP | filter | INPUT | 8472 | flannel | network |
| TCP | filter | INPUT | 6443 | Kubernetes API server | ALL node |
| TCP | filter | INPUT | 2379-2380 | etcd server client API | kube-apiserver, etcd |
| TCP | filter | INPUT | 10250 | kubelet API | Control plane, Self , kubectl exec |
| TCP | filter | INPUT | 10251 | kube-scheduler | self |
| TCP | filter | INPUT | 10252 | kube-controller-manager | self |

#### 1.1.1 pod 通讯通过 flannel 的 udp 端口 8472 通讯

##### 1.1.1.1 包封装图

! [在这里插入图片描述](https://img-blog.csdnimg.cn/img_convert/6425bc8d92d80d077f97a32759697d29.png#pic_center)

#### 1.1.1.2 不通的情况

启动 node2 的 iptables, 确定 8472udp 端口不通的情况, 在 node1 的 pod 上测试 node2 上的 pod 是否能 ping 通

```
# kubectl get pod -o wide
NAME                        READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
busybox                     1/1     Running   0          162m    10.244.1.48   xen11-196   <none>           <none>
my-nginx-75897978cd-64zj2   1/1     Running   0          4h15m   10.244.1.39   xen11-196   <none>           <none>
my-nginx-75897978cd-bbrtm   1/1     Running   0          23d     10.244.1.29   xen11-196   <none>           <none>
my-nginx-75897978cd-t7s6n   1/1     Running   0          142m    10.244.2.2    xen11-197   <none>           <none>
```

Pod IP 列表

| pod | node |
| --- | --- |
| 10.244.2.2 | xen11-197 |
| 10.244.1.48 | xen11-196 |

启动 iptables

```
systemctl start iptables
```

确认 UDP 端口不通

```
 #nc -z -v -u 172.25.11.197 8472
Ncat: Version 7.50 ( https://nmap.org/ncat )
Ncat: Connected to 172.25.11.197:8472.
Ncat: No route to host.
```

启动 busybox 测试, 并测试

```
 kubectl run -i --tty busybox --image= busybox --restart= Never
 在前端执行 ping
```

在后端执行 ping

```
 kubectl exec busybox -- ping 10.244.0.6
```

测试结果是无法 ping 通.

#### 1.1.1.3 通的情况

在 node2 上添加 udp 规则

```
iptables -I INPUT -p udp --dport 8472 -j ACCEPT
``````
# flannel 端口用于封装 10.244.0.0/16 网段的 IP 包, 协议 UDP, 端口 8472
iptables -I INPUT -p udp --dport 8472 -j ACCEPT
```

### 1.1.2 开通 kubectl exec 端口权限

```
 # 开通 kubectl exec 端口权限 10250
# 如果不开通就会报如下错
 kubectl exec -it my-nginx-75897978cd-t7s6n sh
Error from server: error dialing backend: dial tcp 172.25.11.197:10250: connect: no route to host

iptables -I INPUT -p tcp --dport 10250 -j ACCEPT

# 开通后就能执行 kubectl exec
# 示例修改 nginx 的首页内容, 便于测试
# kubectl exec -it my-nginx-75897978cd-t7s6n -- bash -c "echo 10.244.2.2 >> /usr/share/nginx/html/index.html"
#
```

### 1.2 Forward 链

具体看包封装图

由于 cni0 是 pod 网桥 IP 地址, flannel.1 是路由网关地址, flannel.1 是包封装网卡地址, 数据包需要 Forward 才能转发, 打开 iptables forward
注意同时在 /etc/sysctl.conf 里面打开 net.ipv4.ip\_forward = 1

```
iptables -I  FORWARD -s 10.244.0.0/16 -j ACCEPT
iptables -I  FORWARD -d 10.244.0.0/16 -j ACCEPT
```

这样 pod 之间跨主机才能互相 ping 通, 同时宿主机才能 ping 通到跨主机的 pod

## 1.3 Output 链

iptables 默认没开启 OUTPUT 限制, 如果客户严格限制, 需要加上 OUTPUT 过滤, 需要开通 10.244.0.0(pod) 网段

```
iptables -I  OUTPUT -s 10.244.0.0/16 -j ACCEPT
```

## 2.Worker node 需要配置的 iptables 规则

### 2.1 Input 规则

| Protocol | iptables table | ipables chain | Port Range | Purpose | Used by |
| --- | --- | --- | --- | --- | --- |
| UDP | filter | INPUT | 8472 | flannel | network |
| TCP | filter | INPUT | 10250 | kubelet API | Control plane, Self , kubectl exec |
| TCP | filter | INPUT | 30000-3\*\*\*\*\* | NodePortService | All |

### 2.2 Forward 链

具体看包封装图

由于 cni0 是 pod 网桥 IP 地址, flannel.1 是路由网关地址, flannel.1 是包封装网卡地址, 数据包需要 Forward 才能转发, 打开 iptables forward
注意同时在 /etc/sysctl.conf 里面打开 net.ipv4.ip\_forward = 1

```
iptables -I  FORWARD -s 10.244.0.0/16 -j ACCEPT
iptables -I  FORWARD -d 10.244.0.0/16 -j ACCEPT
```

这样 pod 之间跨主机才能互相 ping 通, 同时宿主机才能 ping 通到跨主机的 pod

## 2.3 Output 链

iptables 默认没开启 OUTPUT 限制, 如果客户严格限制, 需要加上 OUTPUT 过滤, 需要开通 10.244.0.0(pod) 网段
同时需要加上连接 Master node 的 OUTPUT 过滤

```
iptables -I  OUTPUT -s 10.244.0.0/16 -j ACCEPT
iptables -I  OUTPUT -d MasterIP/32 -j ACCEPT
#如果是集群, 还需要加上集群地址和其他 Masternode 地址
```

## 3.kubernetes NAT 规则查看

kubernetes iptables NAT 规则举例, 这部分有 kublet-proxy 设置, 无需自己设置

```
iptables -S -t nat| grep 8088
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.103.228.172/32 -p tcp -m comment --comment "default/nginx-8080: cluster IP" -m tcp --dport 8088 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.103.228.172/32 -p tcp -m comment --comment "default/nginx-8080: cluster IP" -m tcp --dport 8088 -j KUBE-SVC-LTTOZPFESA6UOUYL
iptabes -S -t nat  KUBE-MARK-MASQ
```

KUBE-SERVICES 为 kubernets 自定义链:

```
[root@xen11-195 ~]# iptables -S  KUBE-SERVICES -t nat
-N KUBE-SERVICES
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.100.43.239/32 -p tcp -m comment --comment "kube-system/traefik: http-redirect cluster IP" -m tcp --dport 1080 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.100.43.239/32 -p tcp -m comment --comment "kube-system/traefik: http-redirect cluster IP" -m tcp --dport 1080 -j KUBE-SVC-DP4KP26T3K75I2A2
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.97.140.25/32 -p tcp -m comment --comment "monitoring/alertmanager-main: web cluster IP" -m tcp --dport 9093 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.97.140.25/32 -p tcp -m comment --comment "monitoring/alertmanager-main: web cluster IP" -m tcp --dport 9093 -j KUBE-SVC-NAZP4SD6XLP35COK
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.109.140.226/32 -p tcp -m comment --comment "monitoring/prometheus-k8s: web cluster IP" -m tcp --dport 9090 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.109.140.226/32 -p tcp -m comment --comment "monitoring/prometheus-k8s: web cluster IP" -m tcp --dport 9090 -j KUBE-SVC-IFO32E4YIRUTZPGJ
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns: dns-tcp cluster IP" -m tcp --dport 53 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.96.0.10/32 -p tcp -m comment --comment "kube-system/kube-dns: dns-tcp cluster IP" -m tcp --dport 53 -j KUBE-SVC-ERIFXISQEP7F7OF4
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.97.96.87/32 -p tcp -m comment --comment "monitoring/prometheus-adapter: https cluster IP" -m tcp --dport 443 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.97.96.87/32 -p tcp -m comment --comment "monitoring/prometheus-adapter: https cluster IP" -m tcp --dport 443 -j KUBE-SVC-GRVIJZ6QHJZF73YT
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.108.234.116/32 -p tcp -m comment --comment "default/my-nginx: cluster IP" -m tcp --dport 80 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.108.234.116/32 -p tcp -m comment --comment "default/my-nginx: cluster IP" -m tcp --dport 80 -j KUBE-SVC-BEPXDJBUHFCSYIC3
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.96.0.1/32 -p tcp -m
```

iptables 实现负载均衡:

```
[root@xen11-195 ~]# iptables -S KUBE-SVC-LTTOZPFESA6UOUYL -t nat
-N KUBE-SVC-LTTOZPFESA6UOUYL
-A KUBE-SVC-LTTOZPFESA6UOUYL -m statistic --mode random --probability 0.33332999982 -j KUBE-SEP-2YRI2HTTSM2JTT2G
-A KUBE-SVC-LTTOZPFESA6UOUYL -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-7TB3S7FLLR7UHYH4
-A KUBE-SVC-LTTOZPFESA6UOUYL -j KUBE-SEP-3RIUDCBRFGYQJL4F
```

最后追踪到 pod IP 地址

```
[root@xen11-195 ~]# iptables -S KUBE-SEP-2YRI2HTTSM2JTT2G -t nat
-N KUBE-SEP-2YRI2HTTSM2JTT2G
-A KUBE-SEP-2YRI2HTTSM2JTT2G -s 10.244.2.13/32 -j KUBE-MARK-MASQ
-A KUBE-SEP-2YRI2HTTSM2JTT2G -p tcp -m tcp -j DNAT --to-destination 10.244.2.13:80
[root@xen11-195 ~]# iptables -S  KUBE-SEP-7TB3S7FLLR7UHYH4 -t nat
-N KUBE-SEP-7TB3S7FLLR7UHYH4
-A KUBE-SEP-7TB3S7FLLR7UHYH4 -s 10.244.2.14/32 -j KUBE-MARK-MASQ
-A KUBE-SEP-7TB3S7FLLR7UHYH4 -p tcp -m tcp -j DNAT --to-destination 10.244.2.14:80
```

## 4\. Iptables 数据流供参考

! [在这里插入图片描述](https://img-blog.csdnimg.cn/20201010123823853.png? x-oss-process= image/watermark, type_ZmFuZ3poZW5naGVpdGk, shadow_10, text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MzQwNTI0Mg==, size_16, color_FFFFFF, t_70#pic_center)
