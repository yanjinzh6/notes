---
title: Pod 异常状态排查方法
date: 2020-05-05 10:00:00
tags: 'Jenkins'
categories:
  - ['部署', 'CI']
permalink: pod-abnormal-status
photo:
---

## Pending

Pod 一直处于 Pending 状态说明 Pod 还没有调度到某个 Node 上面

可能的原因和解决方案

- 节点资源不足
- Pod 使用 HostNetwork, 对应的端口 (HostPort) 在节点上已被占用
- 节点没有打标签或打的标签值不匹配: 如 Pod 指定了 `nodeSelector`, `nodeAffinity`, `podAffinity` 或 `AntiAffinity` 等标签选择器, 但没有节点打对应的标签或打的标值不匹配
- 集群的 kube-scheduler 服务都挂掉了: 可以在各节点运行 `ps -elf | grep kube-scheduler` 命令来验证
- 创建 Pod 基础容器 (sanbox) 失败

## Waiting 或 ContainerCreating

- 创建 Pod 的基础容器 (sandbox) 失败, 例如节点本地没有 `pod-infrastructure` 镜像, 但是从 kubelet 拉取失败 (如认证问题)
- Pod yaml 定义中请求的 CPU, Memory 太小或者单位不对, 不足以成功运行 Sandbox
- 拉取镜像失败
  - 配置了错误的镜像
  - Kubelet 无法访问镜像仓库 (国内环境访问 `gcr.io` 需要特殊处理)
  - 拉取私有镜像的 imagePullSecret 没有配置或配置有误
  - 镜像太大, 拉取超时 (可以适当调整 kubelet 的 `--image-pull-progress-deadline` 和 `--runtime-request-timeout` 选项

## ImagePullBackOff

- http 类型的 registry, 但是没有添加到 dockerd 的 `insecure-registry=172.27.129.211:35000` 配置参数中
- https 类型的 registry, 但是使用自签名的 ca 证书, dockerd 不识别
- registry 需要认证, 但是 Pod 没有配置 imagePullSecret, 配置的 Secret 不存在或有误
- 镜像文件损坏, 需要重新 push 镜像文件
- kubelet 的 `--registry-qps`, `--registry-burst` 值太小 (默认分别为 `5`, `10`), 并发拉取镜像时被限制, `kubectl describe pod` 报错

## ImageInspectError

- 现象
  - 启动 Pod 失败, `kubectl get pods` 显示的 STATUS 为 `ImageInspectError`
  - `kubectl describe pod` 显示, `ImageInspectError` 的原因为 `readlink /mnt/disk0/docker/data/overlay2: invalid argument`
  - 打开 dockerd 的 debug 日志级别, 查看对应的日志显示 在 `readlink /mnt/disk0/docker/data/overlay2` 出现了 `os.PathError` 错误
- 原因
  - 节点上 docker 镜像文件损坏, 当使用它启动容器后, 容器文件系统错误, 进而导致系统调用 readlink() 返回 `os.PathError` 错误
  - dockerd 不能正确处理这个 [Error](https://github.com/allencloud/docker/blob/master/api/server/httputils/errors.go#L65), 提示 `FIXME: Got an API for which error does not match any expected type!!!`
  - image 文件损坏可能与重启服务器导致的文件系统不完整有关, 可以使用 `fsck` 命令修复文件系统

## CrashLoopBackOff

- CrashLoopBackOff 状态说明容器曾经启动了, 但又异常退出了。此时 Pod 的 RestartCounts 通常是大于 `0` 的, 从 `describe pod` 的 `State`, `Last State` 里, 以及容器日志可以发现一些容器退出的原因, 比如
  - 容器进程退出, 如域名解析失败, 连接数据库失败；
  - 健康检查失败退出
  - OOMKilled
  - 镜像文件损坏

## Error

通常处于 Error 状态说明 Pod 启动过程中发生了错误。常见的原因包括:

- 依赖的 ConfigMap, Secret 或者 PV 等不存在
- 请求的资源超过了管理员设置的限制, 比如超过了 LimitRange 等
- 违反集群的安全策略, 比如违反了 PodSecurityPolicy 等
- 容器无权操作集群内的资源, 比如开启 RBAC 后, 需要为 ServiceAccount 配置角色绑定

## Terminating 或 Unknown

正常情况下, 如果删除了 Pod, 经过一个 grace period (默认 30s) 后, 如果 Pod 还在 Running, 则 kublet 会向 docker 发送 kill 命令, 进而 docker 向 Pod 中的所有进程发送 SIGKILL 信号, 强行删除 Pod。所以, 如果节点工作正常, 一般一个 grace period 后, Pod 会被清除

如果节点失联 NotReady, 默认 5min 后, node controller 开始驱逐它上面的 Pods, 即将该 Node 上的 Pod 标记为 Terminating 状态, 然后在其它节点上再起 Pod。从 v1.5 开始, node controller 不再从 etcd 中强行删除 (force delete) 失联 Node 上的 Pod 信息, 而是等待节点恢复连接后, 确认驱逐的 Pod 都已经 Terminating 后才删除这些 Pods。所以, 这一段时间内, Pod 可能有多副本运行的情况。想要删除 NotReady 节点上的 Terminating 或 Unknown 状态 Pod 的方法

- 从集群中删除该 Node: `kubectl delete node`
- Node 恢复正常。Kubelet 会重新跟 kube-apiserver 通信确认这些 Pod 的期待状态, 进而再删除这些 Pod
- 用户强制删除。用户可以执行 `kubectl delete pods --grace-period=0 --force` 强制删除 Pod。除非明确知道 Pod 的确处于停止状态 (比如 Node 所在 VM 或物理机已经关机) , 否则不建议使用该方法。特别是 StatefulSet 管理的 Pod, 强制删除容易导致脑裂或者数据丢失等问题

## 强制删除

- 通过 `--force` 参数来强制删除
- 通过 `--now` 参数来立马生效
- 如果提示 `--grace-period` 未到期, 可以将 `--grace-period` 设为 0 等待删除

## 参考

- [排错指南 - Pod](https://github.com/opsnull/kubernetes-troubleshooting-book/blob/master/%E6%8E%92%E9%94%99%E6%8C%87%E5%8D%97-Pod.md)
