---
title: kubeadm-config
date: 2021-01-30 20:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-config
photo:
---

https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-config/

# kubeadm config

在 `kubeadm init` 执行期间, kubeadm 将 `ClusterConfiguration` 对象上传到你的集群的 `kube-system` 名字空间下 名为 `kubeadm-config` 的 ConfigMap 对象中. 然后在 `kubeadm join`,`kubeadm reset` 和 `kubeadm upgrade` 执行期间读取此配置. 要查看此 ConfigMap, 请调用 `kubeadm config view`.

你可以使用 `kubeadm config print` 命令打印默认配置, 并使用 `kubeadm config migrate` 命令将旧版本的配置转化成新版本. `kubeadm config images list` 和 `kubeadm config images pull` 命令可以用来列出并拉取 kubeadm 所需的镜像.

更多信息请浏览 [使用带配置文件的 kubeadm init](/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/#config-file) 或 [使用带配置文件的 kubeadm join](/zh/docs/reference/setup-tools/kubeadm/kubeadm-join/#config-file).

在 Kubernetes v1.13.0 及更高版本中, 要列出 / 拉取 kube-dns 镜像而不是 CoreDNS 镜像, 必须使用 [这里](/zh/docs/reference/setup-tools/kubeadm/kubeadm-init-phase/#cmd-phase-addon) 所描述的 `--config` 方法.

## kubeadm config upload from-file

## kubeadm config view

### 概要

使用此命令, 可以查看 kubeadm 配置的集群中的 ConfigMap. 该配置位于 "kube-system" 命名空间中的名为 "kubeadm-config" 的 ConfigMap 中.

```
kubeadm config view [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> view 操作的帮助命令 </td> </tr> </tbody> </table>

### 继承于父命令的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用于和集群通信的 KubeConfig 文件. 如果未设置, 那么 kubeadm 将会搜索一个已经存在于标准路径的 KubeConfig 文件.</td> </tr> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm config print init-defaults

### 概要

此命令打印对象, 例如用于 'kubeadm init' 的默认 init 配置对象.

请注意, Bootstrap Token 字段之类的敏感值已替换为 {"abcdef.0123456789abcdef" "" "nil" <nil> \[\] \[\]} 之类的占位符值以通过验证, 但不执行创建令牌的实际计算.

```
kubeadm config print init-defaults [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--component-configs stringSlice</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 组件配置 API 对象的逗号分隔列表, 打印其默认值. 可用值: [KubeProxyConfiguration KubeletConfiguration]. 如果未设置此参数, 则不会打印任何组件配置.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> init-defaults 操作的帮助命令 </td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 与集群通信时使用的 kubeconfig 文件. 如果未设置该参数, 则可以在一组标准位置中搜索现有的 kubeconfig 文件.</td> </tr> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm config print join-defaults

### 概要

此命令打印对象, 例如用于 'kubeadm join' 的默认 join 配置对象.

请注意, 诸如启动引导令牌字段之类的敏感值已替换为 {"abcdef.0123456789abcdef" "" "nil" <nil> \[\] \[\]} 之类的占位符值以通过验证, 但不执行创建令牌的实际计算.

```
kubeadm config print join-defaults [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--component-configs stringSlice</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 组件配置 API 对象的逗号分隔列表, 打印其默认值. 可用值: [KubeProxyConfiguration KubeletConfiguration]. 如果未设置此参数, 则不会打印任何组件配置.</td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> join-defaults 操作的帮助命令 </td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 与集群通信时使用的 kubeconfig 文件. 如果未设置该参数, 则可以在一组标准位置中搜索现有的 kubeconfig 文件.</td> </tr> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm config migrate

### 概要

此命令允许您在 CLI 工具中将本地旧版本的配置对象转换为最新支持的版本, 而无需变更集群中的任何内容. 在此版本的 kubeadm 中, 支持以下 API 版本:

*   kubeadm.k8s.io/v1beta2

因此, 无论您在此处传递 --old-config 参数的版本是什么, 当写入到 stdout 或 --new-config (如果已指定) 时, 都会读取, 反序列化, 默认, 转换, 验证和重新序列化 API 对象.

换句话说, 如果您将此文件传递给 "kubeadm init", 则该命令的输出就是 kubeadm 实际上在内部读取的内容.

```
kubeadm config migrate [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> migrate 操作的帮助信息 </td> </tr> <tr> <td colspan="2">--new-config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 使用新的 API 版本生成的 kubeadm 配置文件的路径. 这个路径是可选的. 如果没有指定, 输出将被写到 stdout.</td> </tr> <tr> <td colspan="2">--old-config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 使用旧 API 版本且应转换的 kubeadm 配置文件的路径. 此参数是必需的.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用于和集群通信的 kubeconfig 文件. 如果未设置, 那么 kubeadm 将会搜索一个已经存在于标准路径的 kubeconfig 文件.</td> </tr> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm config images list

### 概要

打印 kubeadm 要使用的镜像列表. 配置文件用于自定义任何镜像或镜像存储库.

```
kubeadm config images list [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--allow-missing-template-keys      默认值: true</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 如果设置为 true, 则在模板中缺少字段或哈希表的键时忽略模板中的任何错误. 仅适用于 golang 和 jsonpath 输出格式.</td> </tr> <tr> <td colspan="2">-o, --experimental-output string      默认值:"text"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 输出格式: text| json| yaml| go-template| go-template-file| template| templatefile| jsonpath| jsonpath-as-json| jsonpath-file 其中之一 </td> </tr> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--feature-gates string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一组键值对 (key= value), 用于描述各种特征. 选项是: <br> Auditing= true| false (ALPHA - 默认 = false) <br> CoreDNS= true| false (默认 = true) <br> DynamicKubeletConfig= true| false (BETA - 默认 = false) </td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> list 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择要从中拉取控制平面镜像的容器仓库 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择一个特定的 Kubernetes 版本 </td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用于和集群通信的 kubeconfig 文件. 如果它没有被设置, 那么 kubeadm 将会搜索一个已经存在于标准路径的 kubeconfig 文件.</td> </tr> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## kubeadm config images pull

### 概要

拉取 kubeadm 使用的镜像.

```
kubeadm config images pull [flags]
```

### 选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--config string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> kubeadm 配置文件的路径.</td> </tr> <tr> <td colspan="2">--cri-socket string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 要连接的 CRI 套接字的路径. 如果为空, 则 kubeadm 将尝试自动检测此值; 仅当安装了多个 CRI 或具有非标准 CRI 插槽时, 才使用此选项.</td> </tr> <tr> <td colspan="2">--feature-gates string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 一系列键值对 (key= value), 用于描述各种特征. 可选项是: <br> IPv6DualStack= true| false (ALPHA - 默认值 = false) </td> </tr> <tr> <td colspan="2">-h, --help</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> pull 操作的帮助命令 </td> </tr> <tr> <td colspan="2">--image-repository string      默认值:"k8s.gcr.io"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 选择用于拉取控制平面镜像的容器仓库 </td> </tr> <tr> <td colspan="2">--kubernetes-version string      默认值:"stable-1"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 为控制平面选择一个特定的 Kubernetes 版本.</td> </tr> </tbody> </table>

### 从父命令继承的选项

<table style="width:100%; table-layout: fixed"> <colgroup> <col span="1" style="width:10px"> <col span="1"> </colgroup> <tbody> <tr> <td colspan="2">--kubeconfig string      默认值:"/etc/kubernetes/admin.conf"</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> 用于和集群通信的 kubeconfig 文件. 如果它没有被设置, 那么 kubeadm 将会搜索一个已经存在于标准路径的 kubeconfig 文件.</td> </tr> <tr> <td colspan="2">--rootfs string</td> </tr> <tr> <td> </td> <td style="line-height:130%; word-wrap: break-word"> [实验] 到 ' 真实 ' 主机根文件系统的路径.</td> </tr> </tbody> </table>

## 接下来

*   [](/zh/docs/reference/setup-tools/kubeadm/kubeadm-upgrade) kubeadm upgrade 将 Kubernetes 集群升级到更新版本 \[kubeadm upgrade\]
