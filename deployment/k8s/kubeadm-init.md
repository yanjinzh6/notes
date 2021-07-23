---
title: kubeadm-init
date: 2021-01-30 13:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-init
photo:
---

https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/

# kubeadm init

此命令初始化一个 Kubernetes 控制平面节点.

### 概要

运行此命令来搭建 Kubernetes 控制平面节点.

"init" 命令执行以下阶段:

```
preflight                    Run pre-flight checks
certs                        Certificate generation
  /ca                          Generate the self-signed Kubernetes CA to provision identities for other Kubernetes components
  /apiserver                   Generate the certificate for serving the Kubernetes API
  /apiserver-kubelet-client    Generate the certificate for the API server to connect to kubelet
  /front-proxy-ca              Generate the self-signed CA to provision identities for front proxy
  /front-proxy-client          Generate the certificate for the front proxy client
  /etcd-ca                     Generate the self-signed CA to provision identities for etcd
  /etcd-server                 Generate the certificate for serving etcd
  /etcd-peer                   Generate the certificate for etcd nodes to communicate with each other
  /etcd-healthcheck-client     Generate the certificate for liveness probes to healthcheck etcd
  /apiserver-etcd-client       Generate the certificate the apiserver uses to access etcd
  /sa                          Generate a private key for signing service account tokens along with its public key
kubeconfig                   Generate all kubeconfig files necessary to establish the control plane and the admin kubeconfig file
  /admin                       Generate a kubeconfig file for the admin to use and for kubeadm itself
  /kubelet                     Generate a kubeconfig file for the kubelet to use *only* for cluster bootstrapping purposes
  /controller-manager          Generate a kubeconfig file for the controller manager to use
  /scheduler                   Generate a kubeconfig file for the scheduler to use
kubelet-start                Write kubelet settings and (re) start the kubelet
control-plane                Generate all static Pod manifest files necessary to establish the control plane
  /apiserver                   Generates the kube-apiserver static Pod manifest
  /controller-manager          Generates the kube-controller-manager static Pod manifest
  /scheduler                   Generates the kube-scheduler static Pod manifest
etcd                         Generate static Pod manifest file for local etcd
  /local                       Generate the static Pod manifest file for a local, single-node local etcd instance
upload-config                Upload the kubeadm and kubelet configuration to a ConfigMap
  /kubeadm                     Upload the kubeadm ClusterConfiguration to a ConfigMap
  /kubelet                     Upload the kubelet component config to a ConfigMap
upload-certs                 Upload certificates to kubeadm-certs
mark-control-plane           Mark a node as a control-plane
bootstrap-token              Generates bootstrap tokens used to join a node to a cluster
kubelet-finalize             Updates settings relevant to the kubelet after TLS bootstrap
  /experimental-cert-rotation  Enable kubelet client certificate rotation
addon                        Install required addons for passing Conformance tests
  /coredns                     Install the CoreDNS addon to a Kubernetes cluster
  /kube-proxy                  Install the kube-proxy addon to a Kubernetes cluster
``````
kubeadm init [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--apiserver-advertise-address string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器所公布的其正在监听的 IP 地址. 如果未设置, 则使用默认网络接口.</td> </tr> <tr> <td colspan="2">--apiserver-bind-port int32      默认值:6443</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> API 服务器绑定的端口.</td> </tr> <tr> <td colspan="2">--apiserver-cert-extra-sans stringSlice</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用于 API Server 服务证书的可选附加主题备用名称 (SAN). 可以是 IP 地址和 DNS 名称.</td> </tr> <tr> <td colspan="2">--cert-dir string      默认值:"/etc/kubernetes/pki"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 保存和存储证书的路径.</td> </tr> <tr> <td colspan="2">--certificate-key string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用于加密 kubeadm-certs Secret 中的控制平面证书的密钥.</td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--control-plane-endpoint string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面指定一个稳定的 IP 地址或 DNS 名称.</td> </tr> <tr> <td colspan="2">--cri-socket string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 要连接的 CRI 套接字的路径. 如果为空, 则 kubeadm 将尝试自动检测此值; 仅当安装了多个 CRI 或具有非标准 CRI 插槽时, 才使用此选项.</td> </tr> <tr> <td colspan="2">--dry-run</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 不要应用任何更改; 只是输出将要执行的操作.</td> </tr> <tr> <td colspan="2">--experimental-patches string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 包含名为 "target[suffix][+ patchtype].extension" 的文件的目录路径. 例如,"kube-apiserver0+ merge.yaml" 或仅仅是 "etcd.json". "patchtype" 可以是 "strategic","merge" 或 "json" 之一, 并且它们与 kubectl 支持的补丁格式匹配. 默认的 "patchtype" 为 "strategic". "extension" 必须为 "json" 或 "yaml". "suffix" 是一个可选字符串, 可用于确定首先按字母顺序应用哪些补丁.</td> </tr> <tr> <td colspan="2">--feature-gates string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组用来描述各种功能特性的键值 (key= value) 对. 选项是: <br> IPv6DualStack= true| false (ALPHA - default= false) </td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> init 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--ignore-preflight-errors stringSlice</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 错误将显示为警告的检查列表; 例如:'IsPrivilegedUser, Swap'. 取值为 'all' 时将忽略检查中的所有错误.</td> </tr> <tr> <td colspan="2">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择用于拉取控制平面镜像的容器仓库 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择一个特定的 Kubernetes 版本.</td> </tr> <tr> <td colspan="2">--node-name string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指定节点的名称.</td> </tr> <tr> <td colspan="2">--pod-network-cidr string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 指明 pod 网络可以使用的 IP 地址段. 如果设置了这个参数, 控制平面将会为每一个节点自动分配 CIDRs.</td> </tr> <tr> <td colspan="2">--service-cidr string      默认值:"10.96.0.0/12"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为服务的虚拟 IP 地址另外指定 IP 地址段 </td> </tr> <tr> <td colspan="2">--service-dns-domain string      默认值:"cluster.local"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为服务另外指定域名, 例如:"myorg.internal".</td> </tr> <tr> <td colspan="2">--skip-certificate-key-print</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 不要打印用于加密控制平面证书的密钥.</td> </tr> <tr> <td colspan="2">--skip-phases stringSlice</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 要跳过的阶段列表 </td> </tr> <tr> <td colspan="2">--skip-token-print</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 跳过打印 'kubeadm init' 生成的默认引导令牌.</td> </tr> <tr> <td colspan="2">--token string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 这个令牌用于建立控制平面节点与工作节点间的双向通信. 格式为 [a-z0-9]{6}\.[a-z0-9]{16} - 示例: abcdef.0123456789abcdef</td> </tr> <tr> <td colspan="2">--token-ttl duration      默认值:24h0m0s</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 令牌被自动删除之前的持续时间 (例如 1 s,2 m,3 h). 如果设置为 '0', 则令牌将永不过期 </td> </tr> <tr> <td colspan="2">--upload-certs</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 将控制平面证书上传到 kubeadm-certs Secret.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

### Init 命令的工作流程

`kubeadm init` 命令通过执行下列步骤来启动一个 Kubernetes 控制平面节点.

1.  在做出变更前运行一系列的预检项来验证系统状态. 一些检查项目仅仅触发警告, 其它的则会被视为错误并且退出 kubeadm, 除非问题得到解决或者用户指定了 `--ignore-preflight-errors= <错误列表>` 参数.

2.  生成一个自签名的 CA 证书来为集群中的每一个组件建立身份标识. 用户可以通过将其放入 `--cert-dir` 配置的证书目录中 (默认为 `/etc/kubernetes/pki`) 来提供他们自己的 CA 证书以及 / 或者密钥. APIServer 证书将为任何 `--apiserver-cert-extra-sans` 参数值提供附加的 SAN 条目, 必要时将其小写.

3.  将 kubeconfig 文件写入 `/etc/kubernetes/` 目录以便 kubelet, 控制器管理器和调度器用来连接到 API 服务器, 它们每一个都有自己的身份标识, 同时生成一个名为 `admin.conf` 的独立的 kubeconfig 文件, 用于管理操作.

4.  为 API 服务器, 控制器管理器和调度器生成静态 Pod 的清单文件. 假使没有提供一个外部的 etcd 服务的话, 也会为 etcd 生成一份额外的静态 Pod 清单文件.

    静态 Pod 的清单文件被写入到 `/etc/kubernetes/manifests` 目录; kubelet 会监视这个目录以便在系统启动的时候创建 Pod.

    一旦控制平面的 Pod 都运行起来, `kubeadm init` 的工作流程就继续往下执行.


5.  对控制平面节点应用标签和污点标记以便不会在它上面运行其它的工作负载.

6.  生成令牌, 将来其他节点可使用该令牌向控制平面注册自己. 如 [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-token) kubeadm token 文档所述, 用户可以选择通过 `--token` 提供令牌.

7.  为了使得节点能够遵照 [](/zh/docs/reference/access-authn-authz/bootstrap-tokens) 启动引导令牌 和 [](/zh/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping) TLS 启动引导 这两份文档中描述的机制加入到集群中, kubeadm 会执行所有的必要配置:

    *   创建一个 ConfigMap 提供添加集群节点所需的信息, 并为该 ConfigMap 设置相关的 RBAC 访问规则.

    *   允许启动引导令牌访问 CSR 签名 API.

    *   配置自动签发新的 CSR 请求.


    更多相关信息, 请查看 [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-join) kubeadm join.


8.  通过 API 服务器安装一个 DNS 服务器 (CoreDNS) 和 kube-proxy 附加组件. 在 Kubernetes 版本 1.11 和更高版本中, CoreDNS 是默认的 DNS 服务器. 要安装 kube-dns 而不是 CoreDNS, 必须在 kubeadm `ClusterConfiguration` 中配置 DNS 插件. 有关配置的更多信息, 请参见下面的 " 带配置文件使用 kubeadm init" 一节. 请注意, 尽管已部署 DNS 服务器, 但直到安装 CNI 时才调度它.

    > **警告:** 从 v1.18 开始, 在 kubeadm 中使用 kube-dns 已废弃, 并将在以后的版本中将其删除.


### 在 kubeadm 中使用 init phases

Kubeadm 允许你使用 `kubeadm init phase` 命令分阶段创建控制平面节点.

要查看阶段和子阶段的有序列表, 可以调用 `kubeadm init --help`. 该列表将位于帮助屏幕的顶部, 每个阶段旁边都有一个描述. 注意, 通过调用 `kubeadm init`, 所有阶段和子阶段都将按照此确切顺序执行.

某些阶段具有唯一的标志, 因此, 如果要查看可用选项的列表, 请添加 `--help`, 例如:

```
sudo kubeadm init phase control-plane controller-manager --help
```

你也可以使用 `--help` 查看特定父阶段的子阶段列表:

```
sudo kubeadm init phase control-plane --help
```

`kubeadm init` 还公开了一个名为 `--skip-phases` 的参数, 该参数可用于跳过某些阶段. 参数接受阶段名称列表, 并且这些名称可以从上面的有序列表中获取.

例如:

```
sudo kubeadm init phase control-plane all --config= configfile.yaml
sudo kubeadm init phase etcd local --config= configfile.yaml
# 你现在可以修改控制平面和 etcd 清单文件
sudo kubeadm init --skip-phases= control-plane, etcd --config= configfile.yaml
```

该示例将执行的操作是基于 `configfile.yaml` 中的配置在 `/etc/kubernetes/manifests` 中写入控制平面和 etcd 的清单文件. 这允许你修改文件, 然后使用 `--skip-phases` 跳过这些阶段. 通过调用最后一个命令, 你将使用自定义清单文件创建一个控制平面节点.

### 结合一份配置文件来使用 kubeadm init

> **注意:** 配置文件的功能仍然处于 alpha 状态并且在将来的版本中可能会改变.

通过一份配置文件而不是使用命令行参数来配置 `kubeadm init` 命令是可能的, 但是一些更加高级的功能只能够通过配置文件设定. 这份配置文件通过 `--config` 选项参数指定的, 它必须包含 `ClusterConfiguration` 结构, 并可能包含更多由 `---\n` 分隔的结构. 在某些情况下, 可能不允许将 `--config` 与其他标志混合使用.

可以使用 [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-config) kubeadm config print 命令打印出默认配置.

如果你的配置没有使用最新版本, **推荐**使用 [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-config) kubeadm config migrate 命令进行迁移.

有关配置的字段和用法的更多信息, 你可以访问 API 参考页面并从 [列表](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm#pkg-subdirectories) 中选择一个版本.

### 添加 kube-proxy 参数

kubeadm 配置中有关 kube-proxy 的说明请查看:

*   [kube-proxy](https://godoc.org/k8s.io/kubernetes/pkg/proxy/apis/config#KubeProxyConfiguration)

使用 kubeadm 启用 IPVS 模式的说明请查看:

*   [IPVS](https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/ipvs/README.md)

### 向控制平面组件传递自定义的命令行参数

有关向控制平面组件传递命令行参数的说明请查看: [](/zh/docs/setup/production-environment/tools/kubeadm/control-plane-flags) 控制平面命令行参数

### 使用自定义的镜像

默认情况下, kubeadm 会从 `k8s.gcr.io` 仓库拉取镜像. 如果请求的 Kubernetes 版本是 CI 标签 (例如 `ci/latest`), 则使用 `gcr.io/kubernetes-ci-images`.

你可以通过使用 [带有配置文件的 kubeadm](#config-file) 来重写此操作.

允许的自定义功能有:

*   使用其他的 `imageRepository` 来代替 `k8s.gcr.io`.
*   将 `useHyperKubeImage` 设置为 `true`, 使用 HyperKube 镜像.
*   为 etcd 或 DNS 附件提供特定的 `imageRepository` 和 `imageTag`.

请注意配置文件中的配置项 `kubernetesVersion` 或者命令行参数 `--kubernetes-version` 会影响到镜像的版本.

### 将控制平面证书上传到集群

通过将参数 `--upload-certs` 添加到 `kubeadm init`, 你可以将控制平面证书临时上传到集群中的 Secret. 请注意, 此 Secret 将在 2 小时后自动过期. 证书使用 32 字节密钥加密, 可以使用 `--certificate-key` 指定. 通过将 `--control-plane` 和 `--certificate-key` 传递给 `kubeadm join`, 可以在添加其他控制平面节点时使用相同的密钥下载证书.

以下阶段命令可用于证书到期后重新上传证书:

```
kubeadm init phase upload-certs --upload-certs --certificate-key= SOME_VALUE --config= SOME_YAML_FILE
```

如果未将参数 `--certificate-key` 传递给 `kubeadm init` 和 `kubeadm init phase upload-certs`, 则会自动生成一个新密钥.

以下命令可用于按需生成新密钥:

```
kubeadm certs certificate-key
```

### 使用 kubeadm 管理证书

有关使用 kubeadm 进行证书管理的详细信息, 请参阅 [](/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-certs) 使用 kubeadm 进行证书管理. 该文档包括有关使用外部 CA, 自定义证书和证书更新的信息.

### 管理 kubeadm 为 kubelet 提供的 systemd 配置文件

`kubeadm` 包自带了关于 `systemd` 如何运行 `kubelet` 的配置文件. 请注意 `kubeadm` 客户端命令行工具永远不会修改这份 `systemd` 配置文件. 这份 `systemd` 配置文件属于 kubeadm DEB/RPM 包.

有关更多信息, 请阅读 [管理 systemd 的 kubeadm 内嵌文件](/zh/docs/setup/production-environment/tools/kubeadm/kubelet-integration/#the-kubelet-drop-in-file-for-systemd).

### 结合 CRI 运行时使用 kubeadm

默认情况下, kubeadm 尝试检测你的容器运行环境. 有关此检测的更多详细信息, 请参见 [kubeadm CRI 安装指南](/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-runtime).

### 设置节点的名称

默认情况下, `kubeadm` 基于机器的主机地址分配一个节点名称. 你可以使用 `--node-name` 参数覆盖此设置. 此标识将合适的 [`--hostname-override`](/zh/docs/reference/command-line-tools-reference/kubelet/#options) 值传递给 kubelet.

### 在没有互联网连接的情况下运行 kubeadm

要在没有互联网连接的情况下运行 kubeadm, 你必须提前拉取所需的控制平面镜像.

你可以使用 `kubeadm config images` 子命令列出并拉取镜像:

```
kubeadm config images list
kubeadm config images pull
```

kubeadm 需要的所有镜像, 例如 `k8s.gcr.io/kube-*`,`k8s.gcr.io/etcd` 和 `k8s.gcr.io/pause` 都支持多种架构.

### kubeadm 自动化

除了像文档 [](/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm) kubeadm 基础教程 中所描述的那样, 将从 `kubeadm init` 取得的令牌复制到每个节点, 你还可以并行地分发令牌以实现简单自动化. 要实现自动化, 你必须知道控制平面节点启动后将拥有的 IP 地址, 或使用 DNS 名称或负载均衡器的地址.

1.  生成一个令牌. 这个令牌必须具有以下格式:`< 6 个字符的字符串>.< 16 个字符的字符串>`. 更加正式的说法是, 它必须符合以下正则表达式:`[a-z0-9]{6}\.[a-z0-9]{16}`.

    kubeadm 可以为你生成一个令牌:

    ```
    kubeadm token generate
    ```


2.  使用这个令牌同时启动控制平面节点和工作节点. 它们一旦运行起来应该就会互相寻找对方并且建立集群. 同样的 `--token` 参数可以同时用于 `kubeadm init` 和 `kubeadm join` 命令.

3.  当加入其他控制平面节点时, 可以对 `--certificate-key` 执行类似的操作. 可以使用以下方式生成密钥:

    ```
    kubeadm certs certificate-key
    ```


一旦集群启动起来, 你就可以从控制平面节点的 `/etc/kubernetes/admin.conf` 文件获取管理凭证, 并使用这个凭证同集群通信.

注意这种搭建集群的方式在安全保证上会有一些宽松, 因为这种方式不允许使用 `--discovery-token-ca-cert-hash` 来验证根 CA 的哈希值 (因为当配置节点的时候, 它还没有被生成). 更多信息请参阅 [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-join) kubeadm join 文档.

## 接下来

*   进一步阅读了解 [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-init-phase) kubeadm init phase
*   [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-join) kubeadm join 启动一个 Kubernetes 工作节点并且将其加入到集群
*   [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-upgrade) kubeadm upgrade 将 Kubernetes 集群升级到新版本
*   [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-reset) kubeadm reset 恢复 `kubeadm init` 或 `kubeadm join` 命令对节点所作的变更
