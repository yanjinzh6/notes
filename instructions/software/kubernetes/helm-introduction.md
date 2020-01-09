---
title: helm 使用
date: 2020-1-8 12:00:00
tags: 'kubernetes'
categories:
  - ['使用说明', '软件']
permalink: helm-introduction
photo:
---

# 简介

Helm 是 Kubernetes 的一个包管理工具, 用来简化 Kubernetes 应用的部署和管理. 可以把 Helm 比作 CentOS 中的 yum 工具. Helm 具有以下几个基本概念:

* Chart: 是 Helm 管理的安装包, 里面包含需要部署的安装包资源. 可以把 Chart 比作 CentOS yum 使用的 rpm 文件. 每个 Chart 包含下面两部分:
  * 包的基本描述文件 Chart.yaml
  * 放在 templates 目录中的一个或多个 Kubernetes manifest 文件模板
* Release: 是 chart 的部署实例, 一个 chart 在一个 Kubernetes 集群上可以有多个 release, 即这个 chart 可以被安装多次
* Repository: chart 的仓库, 用于发布和存储 chart

<!-- more -->

## 作用

* 管理 Kubernetes manifest files
* 管理 Helm 安装包 charts
* 基于 chart 的 Kubernetes 应用分发

## 组成

* tiller 运行在 Kubernetes 集群上, 管理 chart 安装的 release
* helm 是一个命令行工具, 可在本地运行, 一般运行在 CI/CD Server 上.

# 安装

详细安装步骤请参考 [Helm 官方文档](https://helm.sh/docs/using_helm/#installing-helm)

## 安装客户端

### 二进制文件

1. 下载相应版本 [desired version](https://github.com/helm/helm/releases)
2. 解压 (<code>tar -zxvf helm-v2.0.0-linux-amd64.tgz</code>)
3. 找到 helm 二进制文件移动到 <code>/usr/local/bin/helm</code> (<code>mv linux-amd64/helm /usr/local/bin/helm</code>)
4. 执行命令 <code>helm help</code>

### Snap

The Snap package for Helm is maintained by [Snapcrafters](https://github.com/snapcrafters/helm).

```sh
$ sudo snap install helm --classic
```

### Homebrew

```sh
brew install kubernetes-helm
```

### Windows

```sh
choco install kubernetes-helm
# or
scoop install helm
```

### 使用脚本

```sh
$ curl https://raw.githubusercontent.com/helm/helm/master/scripts/get > get_helm.sh
$ chmod 700 get_helm.sh
$ ./get_helm.sh
```

# 验证
## 初始化 Helm

```sh
helm init --history-max 200
```

<code>--history-max</code> 参数设置最大历史记录, 如果没有设置将无限期保留历史记录

- 使用 <code>kubectl config current-context</code> 将 Tiller 安装到集群中
- 使用 <code>--kube-context</code> 参数指定不同集群
- 使用 <code>helm init --upgrade</code> 命令升级 Tiller
- 使用 <code>helm version</code> 检查安装状态

```sh
$ helm version
Client: &version.Version{SemVer:"v2.13.0", GitCommit:"79d07943b03aea2b76c12644b4b54733bc5958d6", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.13.0", GitCommit:"79d07943b03aea2b76c12644b4b54733bc5958d6", GitTreeState:"clean"}
```

默认情况下安装的 Tiller 未启用身份验证

# 命令

# 卸载