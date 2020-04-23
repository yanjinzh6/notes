---
title: Microk8s 配置 Helm3
date: 2020-02-25 11:30:00
tags: '配置'
categories:
  - ['部署', 'k8s']
permalink: microk8s-enabled-helm3
photo:
---

> Helm3 是新版本的 Kubernetes 的应用管理工具, 可以通过使用应用模版 (chart) , 添加自定义配置或使用默认参数将应用部署到 Kubernetes 中, 后期就可以通过 Helm 来进行升级, 回滚, 删除等等操作的管理.
> 以前版本的 Helm 是 C/S 架构, 分为客户端 Helm 和 服务端 Tiller, Tiller 主要用于在 Kubernetes 集群中管理各种应用发布的版本, 在 Helm 3 中移除了 Tiller, 版本相关的数据直接存储在了 Kubernetes 中.

- 使用 microk8s 开启 helm3, 只需要简单输入命令, 等待下载完成即可, `microk8s.enable helm3`
- 新安装的 helm3 没有自带 repo, 需要手动添加, 这里配置了一个基础的 repo, `microk8s.helm3 repo add stable https://kubernetes-charts.storage.googleapis.com`
- 更多的 repo 列表可以通过 [helm hub GitHub](https://github.com/helm/hub/blob/master/config/repo-values.yaml) 查看, 可以通过 [hub.helm.sh](https://hub.helm.sh/) 搜索需要的应用
- 添加 repo 后就可以直接通过 install 命令部署应用了, `microk8s.helm3 install my-rabbitmq stable/rabbitmq`, 这里通过使用默认参数部署 rabbitmq, 注意 helm3 install 需要 name 参数, 关于如何自定义参数配置后面详细的应用部署再进行说明

<!-- more -->

```sh
microk8s.helm3 install my-rabbitmq stable/rabbitmq
NAME: my-rabbitmq
LAST DEPLOYED: Tue Feb 25 11:30:06 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

Credentials:

    Username      : user
    echo "Password      : $(kubectl get secret --namespace default my-rabbitmq -o jsonpath="{.data.rabbitmq-password}" | base64 --decode)"
    echo "ErLang Cookie : $(kubectl get secret --namespace default my-rabbitmq -o jsonpath="{.data.rabbitmq-erlang-cookie}" | base64 --decode)"

RabbitMQ can be accessed within the cluster on port  at my-rabbitmq.default.svc.cluster.local

To access for outside the cluster, perform the following steps:

To Access the RabbitMQ AMQP port:

    kubectl port-forward --namespace default svc/my-rabbitmq 5672:5672
    echo "URL : amqp://127.0.0.1:5672/"

To Access the RabbitMQ Management interface:

    kubectl port-forward --namespace default svc/my-rabbitmq 15672:15672
    echo "URL : http://127.0.0.1:15672/"
```
