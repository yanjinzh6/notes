---
title: Istio 使用
date: 2020-1-8 13:00:00
tags: 'kubernetes'
categories:
  - ['使用说明', '软件']
permalink: Istio-introduction
photo:
---

## 简介

[Istio](https://cloud.google.com/istio/?hl=zh-cn) 是一个开源的独立服务网格，可为您成功运行分布式微服务架构提供所需的基础。随着各组织越来越多地采用云平台，开发者必须使用微服务设计架构以实现可移植性，而运营者必须管理包含混合和多云部署的大型分布式部署。Istio 采用一种一致的方式来保护, 连接和监控微服务，降低了管理微服务部署的复杂性。

<!-- more -->

## 安装

Istio 安装在自己的 istio-system 命名空间中, 可以管理来自所有其他命名空间的服务

### 下载

转到 [Istio 版本页面](https://github.com/istio/istio/releases)下载
或者使用命令下载

```sh
curl -L https://git.io/getLatestIstio | ISTIO_VERSION=1.1.0 sh -
```

移至 Istio 包目录

```sh
cd istio-1.1.0
```

安装目录包含:

- 安装 Kubernetes 的 YAML 文件 install/
- 示例应用程序 samples/
- 目录中的 istioctl 客户端二进制 bin/ 文件. istioctl 在将 Envoy 手动注入边车代理时使用.
- 该 istio.VERSION 配置文件

将 bin 目录添加到系统 \$PATH 中

```sh
export PATH=$PWD/bin:$PATH
```

### 部署

[进入 Istio 包目录](https://istio.io/docs/setup/kubernetes/install/helm/)

#### 如果您的群集未部署 Tiller 且您不想安装它

```sh
## 为 istio 组件创建名称空间
$ kubectl create namespace istio-system
## 使用 kubectl apply 安装所有 Istio 自定义资源定义 CRD
helm template install/kubernetes/helm/istio-init --name istio-init --namespace istio-system | kubectl apply -f -
## 使用命令验证是否已将所有 Istio CRD 提交到 Kubernetes api-server
$ kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l
53
## 选择配置文件(https://istio.io/docs/setup/kubernetes/additional-setup/config-profiles/), 然后渲染并应用与您选择的配置文件对应的Istio核心组件, 建议生产部署使用默认配置文件
$ helm template install/kubernetes/helm/istio --name istio --namespace istio-system | kubectl apply -f -
```

#### 使用 Helm 和 Tiller 安装

```sh
## 为 Tiller 定义角色
$ kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
## 使用 service-account 安装Tiller
$ helm init --service-account tiller
## 安装 istio chart 以引导所有 Istio 的 CRD
$ helm install install/kubernetes/helm/istio-init --name istio-init --namespace istio-system
## 使用命令验证是否已将所有 Istio CRD 提交到 Kubernetes api-server
$ kubectl get crds | grep 'istio.io\|certmanager.k8s.io' | wc -l
53
## 选择配置文件(https://istio.io/docs/setup/kubernetes/additional-setup/config-profiles/), 然后渲染并应用与您选择的配置文件对应的Istio核心组件, 建议生产部署使用默认配置文件
$ helm template install/kubernetes/helm/istio --name istio --namespace istio-system | kubectl apply -f -
```

## 验证安装

```sh
## 参考配置文件中的组件表, 验证是否已部署与所选配置文件对应的 Kubernetes 服务
$ kubectl get svc -n istio-system
$ kubectl get pods -n istio-system
```

## 卸载

```sh
## without Tiller
$ helm template install/kubernetes/helm/istio --name istio --namespace istio-system | kubectl delete -f -
$ kubectl delete namespace istio-system
## with Tiller
$ helm delete --purge istio
$ helm delete --purge istio-init
```

### 删除 CRD 和 Istio 配置

可以使用 kubectl 简单地删除 CRD

```sh
## 永久删除 Istio 的 CR D和整个 Istio 配置
$ kubectl delete -f install/kubernetes/helm/istio-init/files
```

## 使用

### 前提

要成为 Istio 服务网格的一部分, Kubernetes 集群中的 pod 和服务必须满足以下要求:

- 需要给端口正确命名: 服务端口必须进行命名. 端口名称只允许是 <code>name: <协议>[-<后缀>-]</code>. 其中<code><协议></code>部分可选择范围包括:

  - grpc
  - http
  - http2
  - https
  - mongo
  - redis
  - tcp
  - tls
  - udp
    Istio 可以通过对这些协议的支持来提供路由能力
    例如 name: http2-foo 和 name: http 都是有效的端口名, 但 name: http2foo 就是无效的. 如果没有给端口进行命名, 或者命名没有使用指定前缀, 那么这一端口的流量就会被视为普通 TCP 流量 (除非显式的用 Protocol: UDP 声明该端口是 UDP 端口) .

- Pod 端口: Pod 必须包含每个容器将监听的明确端口列表. 在每个端口的容器规范中使用 containerPort. 任何未列出的端口都将绕过 Istio Proxy.

- 关联服务: Pod 不论是否公开端口, 都必须关联到至少一个 Kubernetes 服务上, 如果一个 Pod 属于多个服务, 这些服务不能在同一端口上使用不同协议, 例如 HTTP 和 TCP.

- Deployment 应带有 app 以及 version 标签: 在使用 Kubernetes Deployment 进行 Pod 部署的时候, 建议显式的为 Deployment 加上 app 以及 version 标签. 每个 Deployment 都应该有一个有意义的 app 标签和一个用于标识 Deployment 版本的 version 标签. app 标签在分布式追踪的过程中会被用来加入上下文信息. Istio 还会用 app 和 version 标签来给遥测指标数据加入上下文信息.

- Application UID: 不要使用 ID (UID) 值为 1337 的用户来运行应用.

- NET_ADMIN 功能: 如果您的群集中实施了 Pod 安全策略, 除非您使用 Istio CNI 插件, 您的 pod 必须具有 NET_ADMIN 功能. 请参阅必需的 Pod 功能.

### 使用 Helm 更改 Istio 设置

Helm 默认安装插件配置

```yaml
##
## Gateways Configuration, refer to the charts/gateways/values.yaml
## for detailed configuration
##
gateways:
  enabled: true

##
## sidecar-injector webhook configuration, refer to the
## charts/sidecarInjectorWebhook/values.yaml for detailed configuration
##
sidecarInjectorWebhook:
  enabled: true

##
## galley configuration, refer to charts/galley/values.yaml
## for detailed configuration
##
galley:
  enabled: true

##
## mixer configuration
##
## @see charts/mixer/values.yaml, it takes precedence
mixer:
  enabled: true
  policy:
    # if policy is enabled the global.disablePolicyChecks has affect.
    enabled: true

  telemetry:
    enabled: true
##
## pilot configuration
##
## @see charts/pilot/values.yaml
pilot:
  enabled: true

##
## security configuration
##
security:
  enabled: true

##
## nodeagent configuration
##
nodeagent:
  enabled: false

##
## addon grafana configuration
##
grafana:
  enabled: false

##
## addon prometheus configuration
##
prometheus:
  enabled: true

##
## addon servicegraph configuration
##
servicegraph:
  enabled: false

##
## addon jaeger tracing configuration
##
tracing:
  enabled: false

##
## addon kiali tracing configuration
##
kiali:
  enabled: false

##
## Istio CNI plugin enabled
##   This must be enabled to use the CNI plugin in Istio.  The CNI plugin is installed separately.
##   If true, the privileged initContainer istio-init is not needed to perform the traffic redirect
##   settings for the istio-proxy.
##
istio_cni:
  enabled: false

## addon Istio CoreDNS configuration
##
istiocoredns:
  enabled: false
```

可以通过添加一个或多个 <code>--set <key>=<value></code> 来自定义安装选项

```sh
## 通过 --set grafana.enabled=true 选项启用 Grafana 组件
helm install install/kubernetes/helm/istio --name istio --namespace istio-system --set grafana.enabled=true
```
