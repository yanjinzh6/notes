---
title: kubeadm-init-phase
date: 2021-01-30 21:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-init-phase
photo:
---

https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init-phase/

# kubeadm init phase

`kubeadm init phase` 能确保调用引导过程的原子步骤. 因此, 如果希望自定义应用, 则可以让 kubeadm 做一些工作, 然后填补空白.

`kubeadm init phase` 与 [kubeadm init 工作流](/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/#init-workflow) 一致, 后台都使用相同的代码.

## kubeadm init phase preflight

使用此命令可以在控制平面节点上执行启动前检查.

*   [preflight](#tab-preflight-0)

### 概要

运行 kubeadm init 前的启动检查.

```
kubeadm init phase preflight [flags]
```

### 案例

```
# 使用配置文件对 kubeadm init 进行启动检查.
kubeadm init phase preflight --config kubeadm-config.yml
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> preflight 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--ignore-preflight-errors stringSlice</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 错误将显示为警告的检查列表: 例如:'IsPrivilegedUser, Swap'. 取值为 'all' 时将忽略检查中的所有错误.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase kubelet-start

此阶段将检查 kubelet 配置文件和环境文件, 然后启动 kubelet.

*   [kubelet-start](#tab-kubelet-start-0)

### 概要

使用 kubelet 配置文件编写一个文件, 并使用特定节点的 kubelet 设置编写一个环境文件, 然后 (重新) 启动 kubelet.

```
kubeadm init phase kubelet-start [flags]
```

### 示例

```
# 从 InitConfiguration 文件中写入带有 kubelet 参数的动态环境文件.
kubeadm init phase kubelet-start --config config.yaml
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--cri-socket string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 连接到 CRI 套接字的路径. 如果为空, 则 kubeadm 将尝试自动检测该值; 仅当安装了多个 CRI 或具有非标准 CRI 套接字时, 才使用此选项.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubelet-start 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--node-name string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定节点名称.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase certs

该阶段可用于创建 kubeadm 所需的所有证书.

*   [certs](#tab-certs-0)
*   [all](#tab-certs-1)
*   [ca](#tab-certs-2)
*   [apiserver](#tab-certs-3)
*   [apiserver-kubelet-client](#tab-certs-4)
*   [front-proxy-ca](#tab-certs-5)
*   [front-proxy-client](#tab-certs-6)
*   [etcd-ca](#tab-certs-7)
*   [etcd-server](#tab-certs-8)
*   [etcd-peer](#tab-certs-9)
*   [healthcheck-client](#tab-certs-10)
*   [apiserver-etcd-client](#tab-certs-11)
*   [sa](#tab-certs-12)

### 概要

此命令并非设计用来单独运行. 请参阅可用子命令列表.

```
kubeadm init phase certs [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> certs 操作的帮助命令 </td> </tr> </tbody> </table>

### 从父指令中继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成所有证书

```
kubeadm init phase certs all [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 将使用默认网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-cert-extra-sans stringSlice</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用于 API 服务器服务证书的可选额外替代名称 (SAN). 可以同时使用 IP 地址和 DNS 名称.</td> </tr> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 证书的存储路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> all 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--service-cidr string      默认值:"10.96.0.0/12"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> VIP 服务使用其它的 IP 地址范围.</td> </tr> <tr> <td colspan="2">--service-dns-domain string      默认值:"cluster.local"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 服务使用其它的域名, 例如:"myorg.internal".</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成自签名的 Kubernetes CA 以提供其他 Kubernetes 组件的身份, 并将其保存到 ca.cert 和 ca.key 文件中.

如果两个文件都已存在, 则 kubeadm 将跳过生成步骤, 使用现有文件.

Alpha 免责声明: 此命令当前为 Alpha 功能.

```
kubeadm init phase certs ca [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 证书的存储路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> ca 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成用于服务 Kubernetes API 的证书, 并将其保存到 apiserver.cert 和 apiserver.key 文件中.

默认 SAN 是 kubernetes, kubernetes.default, kubernetes.default.svc, kubernetes.default.svc.cluster.local,10.96.0.1,127.0.0.1.

如果两个文件都已存在, 则 kubeadm 将跳过生成步骤, 使用现有文件.

Alpha 免责声明: 此命令当前为 Alpha 功能.

```
kubeadm init phase certs apiserver [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 则使用默认的网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-cert-extra-sans stringSlice</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用于 API Server 服务证书的可选附加主体备用名称 (SAN). 可以是 IP 地址和 DNS 名称.</td> </tr> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 证书的存储路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> apiserver 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定特定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--service-cidr string      默认值:"10.96.0.0/12"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定服务 VIP 可使用的其他 IP 地址段.</td> </tr> <tr> <td colspan="2">--service-dns-domain string      默认值:"cluster.local"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为服务使用其他域名, 例如 "myorg.internal".</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成供 API 服务器连接 kubelet 的证书, 并将其保存到 apiserver-kubelet-client.cert 和 apiserver-kubelet-client.key 文件中.

如果两个文件都已存在, 则 kubeadm 将跳过生成步骤, 使用现有文件.

Alpha 免责声明: 此命令当前为 Alpha 功能.

```
kubeadm init phase certs apiserver-kubelet-client [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> apiserver-kubelet-client 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 指向宿主机上的 ' 实际 ' 根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成自签名 CA 来提供前端代理的身份, 并将其保存到 front-proxy-ca.cert 和 front-proxy-ca.key 文件中.

如果两个文件都已存在, kubeadm 将跳过生成步骤并将使用现有文件.

Alpha 免责声明: 此命令目前是 alpha 阶段.

```
kubeadm init phase certs front-proxy-ca [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> front-proxy-ca 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

为前端代理客户端生成证书, 并将其保存到 front-proxy-client.cert 和 front-proxy-client.key 文件中. 如果两个文件都已存在, kubeadm 将跳过生成步骤并将使用现有文件. Alpha 免责声明: 此命令目前是 alpha 阶段.

```
kubeadm init phase certs front-proxy-client [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> front-proxy-client 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成用于为 etcd 设置身份的自签名 CA, 并将其保存到 etcd/ca.cert 和 etcd/ca.key 文件中.

如果两个文件都已存在, 则 kubeadm 将跳过生成步骤, 使用现有文件.

Alpha 免责声明: 此命令当前为 Alpha 功能.

```
kubeadm init phase certs etcd-ca [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 证书的存储路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> etcd-ca 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成用于提供 etcd 服务的证书, 并将其保存到 etcd/server.cert 和 etcd/server.key 文件中.

默认 SAN 为 localhost,127.0.0.1,127.0.0.1,:: 1

如果两个文件都已存在, 则 kubeadm 将跳过生成步骤, 使用现有文件.

Alpha 免责声明: 此命令当前为 alpha 功能.

```
kubeadm init phase certs etcd-server [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> etcd-server 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成 etcd 节点相互通信的证书, 并将其保存到 etcd/peer.cert 和 etcd/peer.key 文件中.

默认 SAN 为 localhost,127.0.0.1,127.0.0.1,:: 1

如果两个文件都已存在, 则 kubeadm 将跳过生成步骤, 使用现有文件.

Alpha 免责声明: 此命令当前为 alpha 功能.

```
kubeadm init phase certs etcd-peer [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> etcd-peer 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成用于 etcd 健康检查的活跃性探针的证书, 并将其保存到 healthcheck-client.cert 和 etcd/healthcheck-client.key 文件中.

如果两个文件都已存在, 则 kubeadm 将跳过生成步骤, 使用现有文件.

Alpha 免责声明: 此命令当前为 alpha 功能.

```
kubeadm init phase certs etcd-healthcheck-client [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 证书存储的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> etcd-healthcheck-client 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成 apiserver 用于访问 etcd 的证书, 并将其保存到 apiserver-etcd-client.cert 和 apiserver-etcd-client.key 文件中.

如果两个文件都已存在, 则 kubeadm 将跳过生成步骤, 使用现有文件.

Alpha 免责声明: 此命令当前为 Alpha 功能.

```
kubeadm init phase certs apiserver-etcd-client [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 证书的存储路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> apiserver-etcd-client 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成用于签名 service account 令牌的私钥及其公钥, 并将其保存到 sa.key 和 sa.pub 文件中. 如果两个文件都已存在, 则 kubeadm 会跳过生成步骤, 而将使用现有文件.

Alpha 免责声明: 此命令当前为 alpha 阶段.

```
kubeadm init phase certs sa [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> sa 操作的帮助命令 </td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase kubeconfig

可以通过调用 `all` 子命令来创建所有必需的 kubeconfig 文件, 或者分别调用它们.

*   [kubeconfig](#tab-kubeconfig-0)
*   [all](#tab-kubeconfig-1)
*   [admin](#tab-kubeconfig-2)
*   [kubelet](#tab-kubeconfig-3)
*   [controller-manager](#tab-kubeconfig-4)
*   [scheduler](#tab-kubeconfig-5)

### 概要

此命令并非设计用来单独运行. 请阅读可用子命令列表.

```
kubeadm init phase kubeconfig [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeconfig 操作的帮助命令 </td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成所有 kubeconfig 文件

```
kubeadm init phase kubeconfig all [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果没有设置, 将使用默认的网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-bind-port int32      默认值:6443</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 要绑定到 API 服务器的端口.</td> </tr> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> all 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubeconfig-dir string      默认值:"/etc/kubernetes"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeconfig 文件的保存路径.</td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定特定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--node-name string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定节点名称.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

为管理员和 kubeadm 本身生成 kubeconfig 文件, 并将其保存到 admin.conf 文件中.

```
kubeadm init phase kubeconfig admin [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 则使用默认的网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-bind-port int32      默认值:6443</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 要绑定到 API 服务器的端口.</td> </tr> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> admin 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubeconfig-dir string      默认值:"/etc/kubernetes"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeconfig 文件的保存路径.</td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成 kubelet 要使用的 kubeconfig 文件, 并将其保存到 kubelet.conf 文件.

请注意, 该操作目的是_仅_应用于引导集群. 在控制平面启动之后, 应该从 CSR API 请求所有 kubelet 凭据.

```
kubeadm init phase kubeconfig kubelet [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 则使用默认的网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-bind-port int32      默认值:6443</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 要绑定到 API 服务器的端口.</td> </tr> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubelet 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubeconfig-dir string      默认值:"/etc/kubernetes"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeconfig 文件的保存路径.</td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--node-name string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定节点的名称.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成控制器管理器要使用的 kubeconfig 文件, 并保存到 controller-manager.conf 文件中.

```
kubeadm init phase kubeconfig controller-manager [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 则使用默认的网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-bind-port int32      默认值:6443</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 要绑定到 API 服务器的端口.</td> </tr> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> controller-manager 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubeconfig-dir string      默认值:"/etc/kubernetes"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeconfig 文件的保存路径.</td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs 字符串 </td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成调度器 (scheduler) 要使用的 kubeconfig 文件, 并保存到 scheduler.conf 文件中.

```
kubeadm init phase kubeconfig scheduler [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 则使用默认的网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-bind-port int32      默认值:6443</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 要绑定到 API 服务器的端口.</td> </tr> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> scheduler 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubeconfig-dir string      默认值:"/etc/kubernetes"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeconfig 文件的保存路径.</td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase control-plane

使用此阶段, 可以为控制平面组件创建所有必需的静态 Pod 文件.

*   [control-plane](#tab-control-plane-0)
*   [all](#tab-control-plane-1)
*   [apiserver](#tab-control-plane-2)
*   [controller-manager](#tab-control-plane-3)
*   [scheduler](#tab-control-plane-4)

### 概要

此命令并非设计用来单独运行. 请参阅可用子命令列表.

```
kubeadm init phase control-plane [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> control-plane 操作的帮助命令 </td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成所有的静态 Pod 清单文件

```
kubeadm init phase control-plane all [flags]
```

### 示例

```
# 为控制平面组件生成静态 Pod 清单文件, 其功能等效于 kubeadm init 生成的文件.
kubeadm init phase control-plane all

# 使用从某配置文件中读取的选项为生成静态 Pod 清单文件.
kubeadm init phase control-plane all --config config.yaml
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 将使用默认的网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-bind-port int32      默认值:6443</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器要绑定的端口.</td> </tr> <tr> <td colspan="2">--apiserver-extra-args mapStringString</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 形式为 < flagname>=< value> 的一组额外参数, 用来传递给 API 服务器, 或者覆盖其默认配置值 </td> </tr> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择一个稳定的 IP 地址或者 DNS 名称.</td> </tr> <tr> <td colspan="2">--controller-manager-extra-args mapStringString</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组形式为 < flagname>=< value> 的额外参数, 用来传递给控制管理器 (Controller Manager) 或覆盖其默认设置值 </td> </tr> <tr> <td colspan="2">--experimental-patches string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 包含名为 "target[suffix][+ patchtype].extension" 的文件的目录. 例如,"kube-apiserver0+ merge.yaml" 或者 "etcd.json". "patchtype" 可以是 "strategic","merge" 或 "json" 之一, 分别与 kubectl 所支持的 patch 格式相匹配. 默认的 "patchtype" 是 "strategic". "extension" 必须是 "json" 或 "yaml". "suffix" 是一个可选的字符串, 用来确定按字母顺序排序时首先应用哪些 patch.</td> </tr> <tr> <td colspan="2">--feature-gates string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组用来描述各种特性门控的键值 (key= value) 对. 选项是: <br> IPv6DualStack= true| false (ALPHA - 默认 = false) <br> PublicKeysECDSA= true| false (ALPHA - 默认 = false) </td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> all 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择用于拉取控制平面镜像的容器仓库 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择指定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--pod-network-cidr string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定 Pod 网络的 IP 地址范围. 如果设置了此标志, 控制平面将自动地为每个节点分配 CIDR.</td> </tr> <tr> <td colspan="2">--scheduler-extra-args mapStringString</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组形式为 < flagname>=< value> 的额外参数, 用来传递给调度器 (Scheduler) 或覆盖其默认设置值 <p> 传递给调度器 (scheduler) 一组额外的参数或者以 < flagname>=< value> 形式覆盖其默认值.</p> </td> </tr> <tr> <td colspan="2">--service-cidr string      默认值:"10.96.0.0/12"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为服务 VIP 选择 IP 地址范围.</td> </tr> </tbody> </table>

### 从父指令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 指向 ' 真实 ' 宿主机的根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成 kube-apiserver 静态 Pod 清单

```
kubeadm init phase control-plane apiserver [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 将使用默认网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-bind-port int32      默认值: 6443</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 要绑定到 API 服务器的端口.</td> </tr> <tr> <td colspan="2">--apiserver-extra-args mapStringString</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组 < flagname>=< value> 形式的额外参数, 用来传递给 API 服务器 或者覆盖其默认参数配置 </td> </tr> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 包含名为 "target[suffix][+ patchtype].extension" 的文件的目录. 例如,"kube-apiserver0+ merge.yaml" 或者 "etcd.json". "patchtype" 可以是 "strategic","merge" 或 "json" 之一, 分别与 kubectl 所支持的 patch 格式相匹配. 默认的 "patchtype" 是 "strategic". "extension" 必须是 "json" 或 "yaml". "suffix" 是一个可选的字符串, 用来确定按字母顺序排序时首先应用哪些 patch.</td> </tr> <tr> <td colspan="2">--feature-gates string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组键值对, 用于描述各种功能特性的特性门控. 选项是: <br> IPv6DualStack= true| false (ALPHA - 默认 = false) <br> PublicKeysECDSA= true| false (ALPHA - 默认 = false) </td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> apiserver 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择要从中拉取控制平面镜像的容器仓库 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本 </td> </tr> <tr> <td colspan="2">--service-cidr string      默认值:"10.96.0.0/12"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定服务 VIP 使用 IP 地址的其他范围.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统路径.</td> </tr> </tbody> </table>

### 概要

生成 kube-controller-manager 静态 Pod 清单

```
kubeadm init phase control-plane controller-manager [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--controller-manager-extra-args mapStringString</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组 < flagname>=< 形式的额外参数, 传递给控制器管理器 (Controller Manager) 或者覆盖其默认配置值 </td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 包含名为 "target[suffix][+ patchtype].extension" 的文件的目录. 例如,"kube-apiserver0+ merge.yaml" 或者 "etcd.json". "patchtype" 可以是 "strategic","merge" 或 "json" 之一, 分别与 kubectl 所支持的 patch 格式相匹配. 默认的 "patchtype" 是 "strategic". "extension" 必须是 "json" 或 "yaml". "suffix" 是一个可选的字符串, 用来确定按字母顺序排序时首先应用哪些 patch.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> controller-manager 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择要从中拉取控制平面镜像的容器仓库 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--pod-network-cidr string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定 Pod 网络的 IP 地址范围. 如果设置, 控制平面将自动为每个节点分配 CIDR.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

生成 kube-scheduler 静态 Pod 清单

```
kubeadm init phase control-plane scheduler [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--experimental-patches string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 包含名为 "target[suffix][+ patchtype].extension" 的文件的目录. 例如,"kube-apiserver0+ merge.yaml" 或者 "etcd.json". "patchtype" 可以是 "strategic","merge" 或 "json" 之一, 分别与 kubectl 所支持的 patch 格式相匹配. 默认的 "patchtype" 是 "strategic". "extension" 必须是 "json" 或 "yaml". "suffix" 是一个可选的字符串, 用来确定按字母顺序排序时首先应用哪些 patch.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> scheduler 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择要从中拉取控制平面镜像的容器仓库 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--scheduler-extra-args mapStringString</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组 < flagname>=< value> 形式的额外参数, 用来传递给调度器 或者覆盖其默认参数配置 </td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase etcd

根据静态 Pod 文件, 使用以下阶段创建本地 etcd 实例.

*   [etcd](#tab-etcd-0)
*   [local](#tab-etcd-1)

### 概要

此命令并非设计用来单独运行. 请参阅可用子命令列表.

```
kubeadm init phase etcd [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> etcd 操作的帮助命令 </td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

为本地单节点 etcd 实例生成静态 Pod 清单文件

```
kubeadm init phase etcd local [flags]
```

### 示例

```
# 为 etcd 生成静态 Pod 清单文件, 其功能等效于 kubeadm init 生成的文件.
kubeadm init phase etcd local

# 使用从配置文件读取的选项为 etcd 生成静态 Pod 清单文件.
kubeadm init phase etcd local --config config.yaml
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--experimental-patches string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 包含名为 "target[suffix][+ patchtype].extension" 的文件的目录的路径. 例如,"kube-apiserver0+ merge.yaml" 或仅仅是 "etcd.json". "patchtype" 可以是 "strategic","merge" 或 "json" 之一, 并且它们与 kubectl 支持的补丁格式匹配. 默认的 "patchtype" 为 "strategic". "extension" 必须为 "json" 或 "yaml". "suffix" 是一个可选字符串, 可用于确定首先按字母顺序应用哪些补丁.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> local 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择要从中拉取控制平面镜像的容器仓库 </td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase upload-config

可以使用此命令将 kubeadm 配置文件上传到集群. 或者使用 [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-config) kubeadm config.

*   [upload-config](#upload-config-0)
*   [all](#upload-config-1)
*   [kubeadm](#upload-config-2)
*   [kubelet](#upload-config-3)

### 概要

此命令并非设计用来单独运行. 请参阅可用的子命令列表.

```
kubeadm init phase upload-config [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> upload-config 操作的帮助命令 </td> </tr> </tbody> </table>

### 从父命令中继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

将所有配置上传到 ConfigMap

```
kubeadm init phase upload-config all [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> all 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 与集群通信时使用的 kubeconfig 文件. 如果未设置该参数, 则可以在一组标准位置中搜索现有的 kubeconfig 文件.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

将 kubeadm ClusterConfiguration 上传到 kube-system 命名空间中名为 kubeadm-config 的 ConfigMap 中. 这样就可以正确配置系统组件, 并在升级时提供无缝的用户体验.

另外, 可以使用 kubeadm 配置.

```
kubeadm init phase upload-config kubeadm [flags]
```

### 示例

```
# 上传集群配置
kubeadm init phase upload-config --config= myConfig.yaml
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 与集群通信时使用的 kubeconfig 文件. 如果未设置该参数, 则可以在一组标准位置中搜索现有的 kubeconfig 文件.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

将从 kubeadm InitConfiguration 对象提取的 kubelet 配置上传到集群中 kubelet-config-1.X 形式的 ConfigMap, 其中 X 是当前 (API 服务器) Kubernetes 版本的次要版本.

```
kubeadm init phase upload-config kubelet [flags]
```

### 示例

```
# 将 kubelet 配置从 kubeadm 配置文件上传到集群中的 ConfigMap.
kubeadm init phase upload-config kubelet --config kubeadm.yaml
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 到 kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubelet 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 与集群通信时使用的 kubeconfig 文件. 如果未设置该标签, 则可以通过一组标准路径来寻找已有的 kubeconfig 文件.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase upload-certs

使用以下阶段将控制平面证书上传到集群. 默认情况下, 证书和加密密钥会在两个小时后过期.

*   [upload-certs](#tab-upload-certs-0)

### 概要

此命令并非设计用来单独运行. 请参阅可用子命令列表.

```
kubeadm init phase upload-certs [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--certificate-key string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用于加密 kubeadm-certs Secret 中的控制平面证书的密钥.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> upload-certs 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用来与集群通信的 kubeconfig 文件. 如果此标志未设置, 则可以在一组标准的位置搜索现有的 kubeconfig 文件.</td> </tr> <tr> <td colspan="2">--skip-certificate-key-print</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 不要打印输出用于加密控制平面证书的密钥.</td> </tr> <tr> <td colspan="2">--upload-certs</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 将控制平面证书上传到 kubeadm-certs Secret.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase mark-control-plane

使用以下阶段来给具有 `node-role.kubernetes.io/master=""` 键值对的节点打标签 (label) 和记录污点 (taint).

*   [mark-control-plane](#tab-mark-control-plane-0)

### 概要

标记 Node 节点为控制平面节点

```
kubeadm init phase mark-control-plane [flags]
```

### 示例

```
# 将控制平面标签和污点应用于当前节点, 其功能等效于 kubeadm init 执行的操作.
kubeadm init phase mark-control-plane --config config.yml

# 将控制平面标签和污点应用于特定节点
kubeadm init phase mark-control-plane --node-name myNode
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> mark-control-plane 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--node-name string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定节点名称.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs 字符串 </td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase bootstrap-token

使用以下阶段来配置引导令牌.

*   [bootstrap-token](#tab-bootstrap-token-0)

### 概要

启动引导令牌 (bootstrap token) 用于在即将加入集群的节点和控制平面节点之间建立双向信任.

该命令使启动引导令牌 (bootstrap token) 所需的所有配置生效, 然后创建初始令牌.

```
kubeadm init phase bootstrap-token [flags]
```

### 示例

```
# 进行所有引导令牌配置, 并创建一个初始令牌, 功能上与 kubeadm init 生成的令牌等效.
kubeadm init phase bootstrap-token
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> bootstrap-token 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用于和集群通信的 kubeconfig 文件. 如果它没有被设置, 那么 kubeadm 将会搜索一个已经存在于标准路径的 kubeconfig 文件.</td> </tr> <tr> <td colspan="2">--skip-token-print</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 跳过打印 'kubeadm init' 生成的默认引导令牌.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase kubelet-finialize

使用以下阶段在 TLS 引导后更新与 kubelet 相关的设置. 你可以使用 `all` 子命令来运行所有 `kubelet-finalize` 阶段.

*   [kublet-finalize](#tab-kubelet-finalize-0)
*   [kublet-finalize-all](#tab-kubelet-finalize-1)
*   [kublet-finalize-cert-rotation](#tab-kubelet-finalize-2)

TLS 引导后更新与 kubelet 相关的设置

```
kubeadm init phase kubelet-finalize [flags]
```

### 示例

```
 # 在 TLS 引导后更新与 kubelet 相关的设置
  kubeadm init phase kubelet-finalize all --config
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubelet-finalize 操作的帮助命令 </td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

运行所有 kubelet-finalize 阶段

```
kubeadm init phase kubelet-finalize all [flags]
```

### 示例

```
 # 在 TLS 引导后更新与 kubelet 相关的设置
  kubeadm init phase kubelet-finalize all --config
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      默认值: "/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> all 操作的帮助命令 </td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

启用 kubelet 客户端证书轮换

```
kubeadm init phase kubelet-finalize experimental-cert-rotation [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--cert-dir string      Default: "/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> experimental-cert-rotation 操作的帮助命令 </td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm init phase addon

可以使用 `all` 子命令安装所有可用的插件, 或者有选择性地安装它们.

*   [addon](#tab-addon-0)
*   [all](#tab-addon-1)
*   [coredns](#tab-addon-2)
*   [kube-proxy](#tab-addon-3)

### 概要

此命令并非设计用来单独运行. 请参阅可用子命令列表.

```
kubeadm init phase addon [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> addon 操作的帮助命令 </td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

安装所有插件 (addon)

```
kubeadm init phase addon all [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 则将使用默认网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-bind-port int32      默认值:6443</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器绑定的端口.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td colspan="2">--feature-gates string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组键值对 (key= value), 描述了各种特征. 选项包括: <br> IPv6DualStack= true| false (ALPHA - 默认值 = false) </td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择用于拉取控制平面镜像的容器仓库 </td> </tr> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 与集群通信时使用的 kubeconfig 文件. 如果未设置该参数, 则可以在一组标准位置中搜索现有的 kubeconfig 文件.</td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--pod-network-cidr string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定 Pod 网络的 IP 地址范围. 如果已设置, 控制平面将自动为每个节点分配 CIDR.</td> </tr> <tr> <td colspan="2">--service-cidr string      默认值:"10.96.0.0/12"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为服务 VIP 使用 IP 地址的其他范围.</td> </tr> <tr> <td colspan="2">--service-dns-domain string      默认值:"cluster.local"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为服务使用其他域名, 例如 "myorg.internal".</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

通过 API 服务器安装 CoreDNS 附加组件. 请注意, 即使 DNS 服务器已部署, 在安装 CNI 之前 DNS 服务器不会被调度执行.

```
kubeadm init phase addon coredns [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--feature-gates string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组用来描述各种功能特性的键值 (key= value) 对. 选项是: <br> IPv6DualStack= true| false (ALPHA - 默认值 = false) </td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> coredns 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择用于拉取控制平面镜像的容器仓库 </td> </tr> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 与集群通信时使用的 kubeconfig 文件. 如果未设置该参数, 则可以在一组标准位置中搜索现有的 kubeconfig 文件.</td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--service-cidr string      默认值:"10.96.0.0/12"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为服务 VIP 选择 IP 地址范围.</td> </tr> <tr> <td colspan="2">--service-dns-domain string      默认值:"cluster.local"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 服务使用其它的域名, 例如:"myorg.internal".</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### 概要

通过 API 服务器安装 kube-proxy 附加组件.

```
kubeadm init phase addon kube-proxy [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 则将使用默认网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-bind-port int32      默认值: 6443</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器绑定的端口.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kube-proxy 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择用于拉取控制平面镜像的容器仓库 </td> </tr> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 与集群通信时使用的 kubeconfig 文件. 如果未设置该参数, 则可以在一组标准位置中搜索现有的 kubeconfig 文件.</td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择特定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--pod-network-cidr string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定 Pod 网络的 IP 地址范围. 如果已设置, 控制平面将自动为每个节点分配 CIDR.</td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

要使用 kube-dns 代替 CoreDNS, 必须传递一个配置文件:

```
# 仅用于安装 DNS 插件
kubeadm init phase addon coredns --config= someconfig.yaml
# 用于创建完整的控制平面节点
kubeadm init --config= someconfig.yaml
# 用于列出或者拉取镜像
kubeadm config images list/pull --config= someconfig.yaml
# 升级
kubeadm upgrade apply --config= someconfig.yaml
```

该文件必须在 [`ClusterConfiguration`](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2#ClusterConfiguration) 中包含一个 [`DNS`](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2#DNS) 字段, 以及包含一个插件的类型 - `kube-dns`(默认值为 `CoreDNS`).

```
apiVersion: kubeadm.k8s.io/v1beta2 kind: ClusterConfiguration dns:  type: "kube-dns"
```

有关 `v1beta2` 配置中每个字段的更多详细信息, 可以访问 [API](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2).

## 接下来

*   [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-init) kubeadm init 引导 Kubernetes 控制平面节点
*   [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-join) kubeadm join 将节点连接到集群
*   [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-reset) kubeadm reset 恢复通过 `kubeadm init` 或 `kubeadm join` 操作对主机所做的任何更改
*   [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-alpha) kubeadm alpha 尝试实验性功能
