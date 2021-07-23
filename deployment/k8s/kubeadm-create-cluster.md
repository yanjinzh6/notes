---
title: kubeadm-create-cluster
date: 2021-01-30 22:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-create-cluster
photo:
---

https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/

# 使用 kubeadm 创建集群

! [](https://raw.githubusercontent.com/kubernetes/kubeadm/master/logos/stacked/color/kubeadm-stacked-color.png) 使用 `kubeadm`, 你 能创建一个符合最佳实践的最小化 Kubernetes 集群. 事实上, 你可以使用 `kubeadm` 配置一个通过 [Kubernetes 一致性测试](https://kubernetes.io/blog/2017/10/software-conformance-certification) 的集群. `kubeadm` 还支持其他集群生命周期功能, 例如 [](/zh/docs/reference/access-authn-authz/bootstrap-tokens) 启动引导令牌 和集群升级.

kubeadm 工具很棒, 如果你需要:

*   一个尝试 Kubernetes 的简单方法.
*   一个现有用户可以自动设置集群并测试其应用程序的途径.
*   其他具有更大范围的生态系统和 / 或安装工具中的构建模块.

你可以在各种机器上安装和使用 `kubeadm`: 笔记本电脑, 一组云服务器, Raspberry Pi 等. 无论是部署到云还是本地, 你都可以将 `kubeadm` 集成到预配置系统中, 例如 Ansible 或 Terraform.

## 准备开始

要遵循本指南, 你需要:

*   一台或多台运行兼容 deb/rpm 的 Linux 操作系统的计算机; 例如: Ubuntu 或 CentOS.
*   每台机器 2 GB 以上的内存, 内存不足时应用会受限制.
*   用作控制平面节点的计算机上至少有 2 个 CPU.
*   集群中所有计算机之间具有完全的网络连接. 你可以使用公共网络或专用网络.

你还需要使用可以在新集群中部署特定 Kubernetes 版本对应的 `kubeadm`.

[Kubernetes 版本及版本倾斜支持策略](/zh/docs/setup/release/version-skew-policy/#supported-versions) 适用于 `kubeadm` 以及整个 Kubernetes. 查阅该策略以了解支持哪些版本的 Kubernetes 和 `kubeadm`. 该页面是为 Kubernetes v1.20 编写的.

`kubeadm` 工具的整体功能状态为一般可用性 (GA). 一些子功能仍在积极开发中. 随着工具的发展, 创建集群的实现可能会略有变化, 但总体实现应相当稳定.

> **说明:** 根据定义, 在 `kubeadm alpha` 下的所有命令均在 alpha 级别上受支持.

## 目标

*   安装单个控制平面的 Kubernetes 集群
*   在集群上安装 Pod 网络, 以便你的 Pod 可以相互连通

## 操作指南

### 在你的主机上安装安装 kubeadm

查看 [](/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm)" 安装 kubeadm".

> **说明:**
>
> 如果你已经安装了 kubeadm, 执行 `apt-get update && apt-get upgrade` 或 `yum update` 以获取 kubeadm 的最新版本.
>
> 升级时, kubelet 每隔几秒钟重新启动一次, 在 crashloop 状态中等待 kubeadm 发布指令.crashloop 状态是正常现象. 初始化控制平面后, kubelet 将正常运行.

### 初始化控制平面节点

控制平面节点是运行控制平面组件的机器, 包括 [etcd](/zh/docs/tasks/administer-cluster/configure-upgrade-etcd/ "etcd 是兼具一致性和高可用性的键值数据库, 用作保存 Kubernetes 所有集群数据的后台数据库.") (集群数据库) 和 [API Server](/zh/docs/reference/command-line-tools-reference/kube-apiserver/ " 提供 Kubernetes API 服务的控制面组件.") (命令行工具 [kubectl](/docs/user-guide/kubectl-overview/ "kubectl 是用来和 Kubernetes API 服务器进行通信的命令行工具.") 与之通信).

1.  (推荐) 如果计划将单个控制平面 kubeadm 集群升级成高可用, 你应该指定 `--control-plane-endpoint` 为所有控制平面节点设置共享端点. 端点可以是负载均衡器的 DNS 名称或 IP 地址.
2.  选择一个 Pod 网络插件, 并验证是否需要为 `kubeadm init` 传递参数. 根据你选择的第三方网络插件, 你可能需要设置 `--pod-network-cidr` 的值. 请参阅 [安装 Pod 网络附加组件](#pod-network).
3.  (可选) 从版本 1.14 开始,`kubeadm` 尝试使用一系列众所周知的域套接字路径来检测 Linux 上的容器运行时. 要使用不同的容器运行时, 或者如果在预配置的节点上安装了多个容器, 请为 `kubeadm init` 指定 `--cri-socket` 参数. 请参阅 [安装运行时](/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime).
4.  (可选) 除非另有说明, 否则 `kubeadm` 使用与默认网关关联的网络接口来设置此控制平面节点 API server 的广播地址. 要使用其他网络接口, 请为 `kubeadm init` 设置 `--apiserver-advertise-address= <ip-address>` 参数. 要部署使用 IPv6 地址的 Kubernetes 集群, 必须指定一个 IPv6 地址, 例如 `--apiserver-advertise-address= fd00::101`
5.  (可选) 在 `kubeadm init` 之前运行 `kubeadm config images pull`, 以验证与 gcr.io 容器镜像仓库的连通性.

要初始化控制平面节点, 请运行:

```
kubeadm init <args>
```

### 关于 apiserver-advertise-address 和 ControlPlaneEndpoint 的注意事项

`--apiserver-advertise-address` 可用于为控制平面节点的 API server 设置广播地址, `--control-plane-endpoint` 可用于为所有控制平面节点设置共享端点.

`--control-plane-endpoint` 允许 IP 地址和可以映射到 IP 地址的 DNS 名称. 请与你的网络管理员联系, 以评估有关此类映射的可能解决方案.

这是一个示例映射:

```
192.168.0.102 cluster-endpoint
```

其中 `192.168.0.102` 是此节点的 IP 地址,`cluster-endpoint` 是映射到该 IP 的自定义 DNS 名称. 这将允许你将 `--control-plane-endpoint= cluster-endpoint` 传递给 `kubeadm init`, 并将相同的 DNS 名称传递给 `kubeadm join`. 稍后你可以修改 `cluster-endpoint` 以指向高可用性方案中的负载均衡器的地址.

kubeadm 不支持将没有 `--control-plane-endpoint` 参数的单个控制平面集群转换为高可用性集群.

### 更多信息

有关 `kubeadm init` 参数的更多信息, 请参见 [](/zh/docs/reference/setup-tools/kubeadm/kubeadm) kubeadm 参考指南.

要使用配置文件配置 `kubeadm init` 命令, 请参见 [带配置文件使用 kubeadm init](/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file).

要自定义控制平面组件, 包括可选的对控制平面组件和 etcd 服务器的活动探针提供 IPv6 支持, 请参阅 [](/zh/docs/setup/production-environment/tools/kubeadm/control-plane-flags) 自定义参数.

要再次运行 `kubeadm init`, 你必须首先 [卸载集群](#tear-down).

如果将具有不同架构的节点加入集群, 请确保已部署的 DaemonSet 对这种体系结构具有容器镜像支持.

`kubeadm init` 首先运行一系列预检查以确保机器 准备运行 Kubernetes. 这些预检查会显示警告并在错误时退出. 然后 `kubeadm init` 下载并安装集群控制平面组件. 这可能会需要几分钟. 完成之后你应该看到:

```
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a Pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join <control-plane-host>: <control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256: <hash>
```

要使非 root 用户可以运行 kubectl, 请运行以下命令, 它们也是 `kubeadm init` 输出的一部分:

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

或者, 如果你是 `root` 用户, 则可以运行:

```
export KUBECONFIG=/etc/kubernetes/admin.conf
```

记录 `kubeadm init` 输出的 `kubeadm join` 命令. 你需要此命令 [将节点加入集群](#join-nodes).

令牌用于控制平面节点和加入节点之间的相互身份验证. 这里包含的令牌是密钥. 确保它的安全, 因为拥有此令牌的任何人都可以将经过身份验证的节点添加到你的集群中. 可以使用 `kubeadm token` 命令列出, 创建和删除这些令牌. 请参阅 [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-token) kubeadm 参考指南.

### 安装 Pod 网络附加组件

> **注意:**
>
> 本节包含有关网络设置和部署顺序的重要信息. 在继续之前, 请仔细阅读所有建议.
>
> **你必须部署一个基于 Pod 网络插件的 [容器网络接口](/zh/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/#cni " 容器网络接口 (CNI) 插件是遵循 appc/CNI 协议的一类网络插件.") (CNI), 以便你的 Pod 可以相互通信. 在安装网络之前, 集群 DNS (CoreDNS) 将不会启动.**
>
> *   注意你的 Pod 网络不得与任何主机网络重叠: 如果有重叠, 你很可能会遇到问题. (如果你发现网络插件的首选 Pod 网络与某些主机网络之间存在冲突, 则应考虑使用一个合适的 CIDR 块来代替, 然后在执行 `kubeadm init` 时使用 `--pod-network-cidr` 参数并在你的网络插件的 YAML 中替换它).
>
> *   默认情况下,`kubeadm` 将集群设置为使用和强制使用 [](/zh/docs/reference/access-authn-authz/rbac) RBAC(基于角色的访问控制). 确保你的 Pod 网络插件支持 RBAC, 以及用于部署它的 manifests 也是如此.
>
> *   如果要为集群使用 IPv6(双协议栈或仅单协议栈 IPv6 网络), 请确保你的 Pod 网络插件支持 IPv6. IPv6 支持已在 CNI [v0.6.0](https://github.com/containernetworking/cni/releases/tag/v0.6.0) 版本中添加.

> **说明:** 目前 Calico 是 kubeadm 项目中执行 e2e 测试的唯一 CNI 插件. 如果你发现与 CNI 插件相关的问题, 应在其各自的问题跟踪器中记录而不是在 kubeadm 或 kubernetes 问题跟踪器中记录.

一些外部项目为 Kubernetes 提供使用 CNI 的 Pod 网络, 其中一些还支持 [](/zh/docs/concepts/services-networking/network-policies) 网络策略.

请参阅实现 [Kubernetes 网络模型](/zh/docs/concepts/cluster-administration/networking/#how-to-implement-the-kubernetes-networking-model) 的附加组件列表.

你可以使用以下命令在控制平面节点或具有 kubeconfig 凭据的节点上安装 Pod 网络附加组件:

```
kubectl apply -f <add-on.yaml>
```

每个集群只能安装一个 Pod 网络.

安装 Pod 网络后, 您可以通过在 `kubectl get pods --all-namespaces` 输出中检查 CoreDNS Pod 是否 `Running` 来确认其是否正常运行. 一旦 CoreDNS Pod 启用并运行, 你就可以继续加入节点.

如果您的网络无法正常工作或 CoreDNS 不在 " 运行中 " 状态, 请查看 `kubeadm` 的 [](/zh/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm) 故障排除指南.

### 控制平面节点隔离

默认情况下, 出于安全原因, 你的集群不会在控制平面节点上调度 Pod. 如果你希望能够在控制平面节点上调度 Pod, 例如用于开发的单机 Kubernetes 集群, 请运行:

```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

输出看起来像:

```
node "test-01" untainted
taint "node-role.kubernetes.io/master:" not found
taint "node-role.kubernetes.io/master:" not found
```

这将从任何拥有 `node-role.kubernetes.io/master` taint 标记的节点中移除该标记, 包括控制平面节点, 这意味着调度程序将能够在任何地方调度 Pods.

### 加入节点

节点是你的工作负载 (容器和 Pod 等) 运行的地方. 要将新节点添加到集群, 请对每台计算机执行以下操作:

*   SSH 到机器
*   成为 root (例如 `sudo su -`)
*   运行 `kubeadm init` 输出的命令. 例如:

```
kubeadm join --token <token> <control-plane-host>: <control-plane-port> --discovery-token-ca-cert-hash sha256: <hash>
```

如果没有令牌, 可以通过在控制平面节点上运行以下命令来获取令牌:

```
kubeadm token list
```

输出类似于以下内容:

```
TOKEN                    TTL  EXPIRES              USAGES           DESCRIPTION            EXTRA GROUPS
8ewj1p.9r9hcjoqgajrj4gi  23h  2018-06-12T02:51:28Z authentication,  The default bootstrap  system:
                                                   signing          token generated by     bootstrappers:
                                                                    'kubeadm init'.        kubeadm:
                                                                                           default-node-token
```

默认情况下, 令牌会在 24 小时后过期. 如果要在当前令牌过期后将节点加入集群, 则可以通过在控制平面节点上运行以下命令来创建新令牌:

```
kubeadm token create
```

输出类似于以下内容:

```
5didvk.d09sbcov8ph2amjw
```

如果你没有 `--discovery-token-ca-cert-hash` 的值, 则可以通过在控制平面节点上执行以下命令链来获取它:

```
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | \    openssl dgst -sha256 -hex | sed 's/^.* //'
```

输出类似于以下内容:

```
8cb2de97839780a412b93877f8507ad6c94f73add17d5d7058e91741c9d5ec78
```

> **说明:** 要为 `<control-plane-host>: <control-plane-port>` 指定 IPv6 元组, 必须将 IPv6 地址括在方括号中, 例如:`[fd00::101]:2073`

输出应类似于:

```
[preflight] Running pre-flight checks

... (log output of join workflow) ...

Node join complete:
* Certificate signing request sent to control-plane and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on control-plane to see this machine join.
```

几秒钟后, 当你在控制平面节点上执行 `kubectl get nodes`, 你会注意到该节点出现在输出中.

### (可选) 从控制平面节点以外的计算机控制集群

为了使 kubectl 在其他计算机 (例如笔记本电脑) 上与你的集群通信, 你需要将管理员 kubeconfig 文件从控制平面节点复制到工作站, 如下所示:

```
scp root@<control-plane-host>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf get nodes
```

> **说明:**
>
> 上面的示例假定为 root 用户启用了 SSH 访问. 如果不是这种情况, 你可以使用 `scp` 将 admin.conf 文件复制给其他允许访问的用户.
>
> admin.conf 文件为用户提供了对集群的超级用户特权. 该文件应谨慎使用. 对于普通用户, 建议生成一个你为其授予特权的唯一证书. 你可以使用 `kubeadm alpha kubeconfig user --client-name <CN>` 命令执行此操作. 该命令会将 KubeConfig 文件打印到 STDOUT, 你应该将其保存到文件并分发给用户. 之后, 使用 `kubectl create (cluster) rolebinding` 授予特权.

### (可选) 将 API 服务器代理到本地主机

如果要从集群外部连接到 API 服务器, 则可以使用 `kubectl proxy`:

```
scp root@<control-plane-host>:/etc/kubernetes/admin.conf .
kubectl --kubeconfig ./admin.conf proxy
```

你现在可以在本地访问 API 服务器 http://localhost:8001/api/v1

## 清理

如果你在集群中使用了一次性服务器进行测试, 则可以关闭这些服务器, 而无需进一步清理. 你可以使用 `kubectl config delete-cluster` 删除对集群的本地引用.

但是, 如果要更干净地取消配置群集, 则应首先 [清空节点](/docs/reference/generated/kubectl/kubectl-commands#drain) 并确保该节点为空, 然后取消配置该节点.

### 删除节点

使用适当的凭证与控制平面节点通信, 运行:

```
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
```

在删除节点之前, 请重置 `kubeadm` 安装的状态:

```
kubeadm reset
```

重置过程不会重置或清除 iptables 规则或 IPVS 表. 如果你希望重置 iptables, 则必须手动进行:

```
iptables -F && iptables -t nat -F && iptables -t mangle -F && iptables -X
```

如果要重置 IPVS 表, 则必须运行以下命令:

```
ipvsadm -C
```

现在删除节点:

```
kubectl delete node <node name>
```

如果你想重新开始, 只需运行 `kubeadm init` 或 `kubeadm join` 并加上适当的参数.

### 清理控制平面

你可以在控制平面主机上使用 `kubeadm reset` 来触发尽力而为的清理.

有关此子命令及其选项的更多信息, 请参见 [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-reset)`kubeadm reset` 参考文档.

## 下一步

*   使用 [Sonobuoy](https://github.com/heptio/sonobuoy) 验证集群是否正常运行
*   有关使用 kubeadm 升级集群的详细信息, 请参阅 [](/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade) 升级 kubeadm 集群.
*   在 [kubeadm 参考文档](/zh/docs/reference/setup-tools/kubeadm/kubeadm) 中了解有关高级 `kubeadm` 用法的信息
*   了解有关 Kubernetes[](/zh/docs/concepts) 概念和 [](/zh/docs/reference/kubectl/overview)`kubectl` 的更多信息.
*   有关 Pod 网络附加组件的更多列表, 请参见 [](/zh/docs/concepts/cluster-administration/networking) 集群网络页面.
*   请参阅 [](/zh/docs/concepts/cluster-administration/addons) 附加组件列表以探索其他附加组件, 包括用于 Kubernetes 集群的日志记录, 监视, 网络策略, 可视化和控制的工具.
*   配置集群如何处理集群事件的日志以及 在 Pods 中运行的应用程序. 有关所涉及内容的概述, 请参见 [](/zh/docs/concepts/cluster-administration/logging) 日志架构.

### 反馈

*   有关 bugs, 访问 [kubeadm GitHub issue tracker](https://github.com/kubernetes/kubeadm/issues)
*   有关支持, 访问 [](https://kubernetes.slack.com/messages/kubeadm)#kubeadm Slack 频道
*   General SIG 集群生命周期开发 Slack 频道: [](https://kubernetes.slack.com/messages/sig-cluster-lifecycle)#sig-cluster-lifecycle
*   SIG 集群生命周期 [SIG information](https://github.com/kubernetes/community/tree/master/sig-cluster-lifecycle#readme)
*   SIG 集群生命周期邮件列表: [kubernetes-sig-cluster-lifecycle](https://groups.google.com/forum/#! forum/kubernetes-sig-cluster-lifecycle)

## 版本倾斜政策

版本 v1.20 的 kubeadm 工具可以使用版本 v1.20 或 v1.19 的控制平面部署集群.kubeadm v1.20 还可以升级现有的 kubeadm 创建的 v1.19 版本的集群.

由于没有未来, kubeadm CLI v1.20 可能会或可能无法部署 v1.21 集群.

这些资源提供了有关 kubelet 与控制平面以及其他 Kubernetes 组件之间受支持的版本倾斜的更多信息:

*   Kubernetes [](/zh/docs/setup/release/version-skew-policy) 版本和版本偏斜政策
*   Kubeadm-specific [安装指南](/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl)

## 局限性

### 集群弹性

此处创建的集群具有单个控制平面节点, 运行单个 etcd 数据库. 这意味着如果控制平面节点发生故障, 你的集群可能会丢失数据并且可能需要从头开始重新创建.

解决方法:

*   定期 [备份 etcd](https://coreos.com/etcd/docs/latest/admin_guide.html). kubeadm 配置的 etcd 数据目录位于控制平面节点上的 `/var/lib/etcd` 中.

*   使用多个控制平面节点. 你可以阅读 [](/zh/docs/setup/production-environment/tools/kubeadm/ha-topology) 可选的高可用性拓扑 选择集群拓扑提供的 [](/zh/docs/setup/production-environment/tools/kubeadm/high-availability) 高可用性.

### 平台兼容性

kubeadm deb/rpm 软件包和二进制文件是为 amd64, arm (32-bit), arm64, ppc64le 和 s390x 构建的遵循 [多平台提案](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/multi-platform.md).

从 v1.12 开始还支持用于控制平面和附加组件的多平台容器镜像.

只有一些网络提供商为所有平台提供解决方案. 请查阅上方的 网络提供商清单或每个提供商的文档以确定提供商是否 支持你选择的平台.

## 故障排除

如果你在使用 kubeadm 时遇到困难, 请查阅我们的 [](/zh/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm) 故障排除文档.
