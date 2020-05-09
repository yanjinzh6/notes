---
title: 在资源耗尽情况下卸载 release 出现的问题
date: 2020-05-04 20:00:00
tags: 'Jenkins'
categories:
  - ['部署', 'CI']
permalink: helm3-uninstall-error
photo:
---

接上一回 [Jenkins jnlp job 失败导致 pod 错误卡死](./jenkins-jnlp-error)

想着清理一些暂时不需要的部署, 结果发现还没缓过来, 因为还部署着一个 MongoDB 所以将一些其他的应用关闭一下, 检查应用的日志发现 MongoDB 还在正常读取着

<!-- more -->

```sh
microk8s.helm3 list
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
grpc-envoy      default         26              2020-03-04 16:28:19.49929059 +0800 CST  deployed        envoy-1.9.0     1.11.2
my-mongo        default         2               2020-03-06 17:27:41.612907523 +0800 CST deployed        mongodb-7.8.5   4.2.3
```

```sh
microk8s.helm3 uninstall grpc-envoy
Error: uninstallation completed with 1 error(s): Delete https://127.0.0.1:16443/api/v1/namespaces/default/services/envoy: stream error: stream ID 79; INTERNAL_ERROR
microk8s.helm3 uninstall grpc-envoy
Error: uninstall: Release not loaded: grpc-envoy: release: not found
microk8s.helm3 list
Error: Kubernetes cluster unreachable
```

关掉其他应用后就可以正常访问了, 但是 release 已经在上次的错误中被删除了

```sh
microk8s.helm3 uninstall grpc-envoy
Error: uninstall: Release not loaded: grpc-envoy: release: not found
```
