---
title: kubeadm-some-vpc-cluster
date: 2021-03-14 13:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-some-vpc-cluster
photo:
---

[kubeadm-some-vpc-cluster](https://zhangguanzhang.github.io/2018/09/24/k8s-some-vpc-cluster/)

        去年开始各大云厂商搞活动 120 元一年的云主机, 当时还不会 k8s, 但是了解到搭建集群需要好几台, 于是当时攒了好几台来做准备: tx 一台, 阿里两个别人账号买了两台, 再加上很久之前几何云送了一台. 当时还有人怼说买这么多干啥, 答曰: 以后学 k8s, 此人曰: 呵呵

        后面刚开始接触的时候不太了解网络, 用的 calico, 跨 vpc 根本不行, 有大佬说只能 vxlan 能跨 SDN 下的 vpc 之间
但是云厂商的主机都是做的 ip 的 nat, 网卡 ip 都是内网 ip, kubelet 上报的时候和 flannel 都会用这个 ip

*   calico 的 BGP 不能跨子网 (好像需要人为干预配置才能跨) 跨三层只能用 ipip 模式, 另外个人网络知识不够, 不懂 BGP
*   flannel 的 host-gateway 需要宿主机在同一个子网也就是说更不能跨 vpc, 另外 host-gateway 我们这的 SDN 会对包进行过滤, 如果机器的 ip 和 mac 绑定 (默认) 下发的包和发出去的网卡的 ip 和 mac 对不上会被 drop 掉, 而且 host-gateway 需要每个主机的 docker0 不同网段, 似乎搭建的话有点麻烦.flannel 的 vxlan 慢于 calico 的 ipip 模式, calico 的 ipip 不了解 (另外 ipip 安全性不如 vxlan)

        最开始 k8s 对网络设计提出一下要求

*   所有容器能够在没有 NAT 的情況下与其他容器通信.
*   所有节点能夠在没有 NAT 情況下与所有容器通信 (反之亦然).
*   容器看到的 IP 与其他人看到的 IP 是一样的.

        于是每个 pod 都需要一个 ip, 然后通过网络模型实现 pod, node, 非本机 pod 之前通信, 于是

*   高耦合的容器到容器通信 (即一个 pod 内容器): 通过 Pods 内 localhost 来通信.
*   Pod 到 Pod 的通信: 通过实现网络模型来解决.
*   Pod 到 Service 通信: 由 Service objects 结合 kube-proxy 解決.
*   外部到 Service 通信: 一样由 Service objects 结合 kube-proxy 解決.
    上面可以得出 pod 得有一个 ip, 但是又不得影响宿主机的原有网络, 于是 svc 和 pod 的 ip 都是集群内的宿主机自嗨内网 (人为干预可以让非集群机器访问到 clusterip)

        最首先的, master 肯定是单个了, 宣告的 apiserver, 也就是选项 `--advertise-address` 必须配置为 master 的公网 IP, kubeadm 的话是配置 `controlPlaneEndpoint`, 或者把一个指定域名加到 certSAN 里, 用 hosts 代替

        在 flannel 的 vxlan 模式里 `flannel.1` 充当了 vxlan 里的 `vtep` 身份, 垮主机的 pod 通信流程可以简化到如下
! [network](https://github.com/zhangguanzhang/Image-Hosting/blob/master/k8s/5B7A@13437XC28~112U%5BZTP.png? raw= true)
flannel 查找到目标 pod 所在节点是通过查询 etcd 的 node 信息维护一张 fdb 表, 可以通过下面命令查看. 安装 iproute

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> <span class="line">2</span> <br> <span class="line">3</span> <br> </pre> </td> <td class="code"> <pre> <span class="line">/usr/sbin/bridge fdb show dev flannel.1</span> <br> <span class="line"> </span> <br> <span class="line"> ip neigh show dev flannel.1</span> <br> </pre> </td> </tr> </tbody> </table>

kubelet 上报的 ip 是主机的 eth0 的网卡 ip, 由于公有云 ecs 都是做的 ip 的 nat, 上报的 ip 是内网 ip, 各 vpc 里的主机向这个内网 ip 发包的话会在出公网后被运营商路由器 drop 掉 (源目 ip 为内网 ip 不允许上公网)

        最开始我从 calico 换成 flannel 的时候没注意目录 `/etc/cni/net.d/` 残留有 calico 的配置文件, 导致 kubelet 还认为是跑的 calico, 但是找不到 calico 一直报错, 后面一气之下全部重装了操作系统
        后面又搭建起来后发现 (只有同一个 node 上的 pod 能互相访问) 跨节点的 pod 之间无法通信, 然后云上开了 flannel 的 udp 端口 `8472` 后依然不通.

测试 udp 端口通不通可以在 flannel 跑起来后安装 nc 命令测, 发现几何云 udp 包会被 drop 掉, 阿里, 腾讯和百度的不会

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> </pre> </td> <td class="code"> <pre> <span class="line"> echo -n foo | nc -4u -w1 < dest_public_ip> 8472</span> <br> </pre> </td> </tr> </tbody> </table>

抓包则安装 tcpdump 抓

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> </pre> </td> <td class="code"> <pre> <span class="line"> tcpdump -nn port 8472</span> <br> </pre> </td> </tr> </tbody> </table>

或者 nc 测试

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> <span class="line">2</span> <br> <span class="line">3</span> <br> <span class="line">4</span> <br> <span class="line">5</span> <br> </pre> </td> <td class="code"> <pre> <span class="line"># server</span> <br> <span class="line"> nc -l -u 8472</span> <br> <span class="line"> </span> <br> <span class="line"># client</span> <br> <span class="line"> nc -u < dest_public_ip> 8472</span> <br> </pre> </td> </tr> </tbody> </table>

最后找 flannel 的选项发现如下字段可以指定其他的 vtep 与自己通信的时候应该使用这个 ip 来通信

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> <span class="line">2</span> <br> <span class="line">3</span> <br> </pre> </td> <td class="code"> <pre> <span class="line"> [root@k8s-m1 ~]# docker run --rm quay.io/coreos/flannel: v0.10.0-amd64 --help |& grep -A1 public</span> <br> <span class="line">  -public-ip string</span> <br> <span class="line">    	IP accessible by other nodes for inter-host communication</span> <br> </pre> </td> </tr> </tbody> </table>

        但是 flannel 的 pod 是 ds 跑的, 如果每台机器都要配置各自的 public ip, 可以通过改写 initContainers 的逻辑, 通过类似 `curl ip.sb` 查询自己主机的公网出口 ip 来注入到一个变量里, 然后工作容器的 ars 加上 -public-ip 即可
另一种是 flannel 官方 yaml 里有 RBAC 能够读取 node 的状态,`kubectl describe node nodeName` 信息发现有 `Annotations` 如下

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> <span class="line">2</span> <br> <span class="line">3</span> <br> <span class="line">4</span> <br> <span class="line">5</span> <br> <span class="line">6</span> <br> </pre> </td> <td class="code"> <pre> <span class="line"> Annotations:        flannel.alpha.coreos.com/backend-data: {"VtepMAC":"xxxx"} </span> <br> <span class="line">                    flannel.alpha.coreos.com/backend-type: vxlan</span> <br> <span class="line">                    flannel.alpha.coreos.com/kube-subnet-manager: true</span> <br> <span class="line">                    flannel.alpha.coreos.com/public-ip: xxxxxxx</span> <br> <span class="line">                    node.alpha.kubernetes.io/ttl: 0</span> <br> <span class="line">                    volumes.kubernetes.io/controller-managed-attach-detach: true</span> <br> </pre> </td> </tr> </tbody> </table>

看到这个命名 `public-ip` 可以十分肯定的知道改这里也能行了, 于是如下命令修改完每个 node 信息为对应 node 的公网 IP 即可通信

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> </pre> </td> <td class="code"> <pre> <span class="line"> kubectl edit node < nodename> </span> <br> </pre> </td> </tr> </tbody> </table>

然后去对应 node 的 `8472` 端口抓包发现能通
    后面发现 `kubectl exec` 和 `log` 命令是走的 node 的 ip(也就是 kubelet 上报的各个主机的 eth0 的内网 ip) 会导致这俩命令超时
于是添加 DNAT 规则解决

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> </pre> </td> <td class="code"> <pre> <span class="line"> iptables -t nat -I OUTPUT -d 节点内网 ip -j DNAT --to 节点的公网 IP</span> <br> </pre> </td> </tr> </tbody> </table>

    这里发现很小几率会出现做完上面的 DNAT 也不通和超时, 我的某个 node 就出现这样情况, 需要配合增加下面命令解决

<table> <tbody> <tr> <td class="gutter"> <pre> <span class="line">1</span> <br> </pre> </td> <td class="code"> <pre> <span class="line"> iptables -t nat -I POSTROUTING -o eth0 -s < 目标节点内网 ip> -j MASQUERADE</span> <br> </pre> </td> </tr> </tbody> </table>

然后测试 pod 的 ip 访问和 svc 的 ip 访问即可通信

# [](#一些 flannel 网络文章 " 一些 flannel 网络文章 ") 一些 flannel 网络文章

[http://yangjunsss.github.io/2018-07-21/%E5%AE%B9%E5%99%A8%E7%BD%91%E7%BB%9C-Flannel-%E4%B8%BB%E8%A6%81-Backend-%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86%E5%92%8C%E9%AA%8C%E8%AF%81/](http://yangjunsss.github.io/2018-07-21/%E5%AE%B9%E5%99%A8%E7%BD%91%E7%BB%9C-Flannel-%E4%B8%BB%E8%A6%81-Backend-%E5%9F%BA%E6%9C%AC%E5%8E%9F%E7%90%86%E5%92%8C%E9%AA%8C%E8%AF%81/)
[https://blog.51cto.com/14146751/2334560? source= dra](https://blog.51cto.com/14146751/2334560? source= dra)
[https://ieevee.com/tech/2017/08/12/k8s-flannel-src.html](https://ieevee.com/tech/2017/08/12/k8s-flannel-src.html)
[https://programmer.ink/think/5da939768e5cb.html](https://programmer.ink/think/5da939768e5cb.html)

原文作者: [Zhangguanzhang](http://zhangguanzhang.github.io)

原文链接: [http://zhangguanzhang.github.io/2018/09/24/k8s-some-vpc-cluster/](http://zhangguanzhang.github.io/2018/09/24/k8s-some-vpc-cluster/)

发表日期: [September 24th 2018, 8:28:26 pm](http://zhangguanzhang.github.io/2018/09/24/k8s-some-vpc-cluster/)

更新日期: [September 24th 2018, 8:28:26 pm](http://zhangguanzhang.github.io/2018/09/24/k8s-some-vpc-cluster/)

版权声明: 本文采用 [知识共享署名 - 非商业性使用 4.0 国际许可协议](http://creativecommons.org/licenses/by-nc/4.0/) 进行许可

*   Next Post

    [

    IngressController 使用和它的高可用落地

    ](/2018/10/06/IngressController/ "IngressController 使用和它的高可用落地 ")
*   Previous Post

    [

    二进制部署 Kubernetes v1.11.x(1.12.x) HA 可选

    ](/2018/09/18/kubernetes-1-11-x-bin/ " 二进制部署 Kubernetes v1.11.x(1.12.x) HA 可选 ")

为正常使用来必力评论功能请激活 JavaScript