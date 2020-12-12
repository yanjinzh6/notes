---
title: Docker 修改 Cgroup Driver 为 systemd
date: 2020-12-06 13:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-cgroup-systemd
photo:
---

## 简介

Cgroup Driver

<!-- more -->

## 问题

在运行 `kubeadm init` 时, 经常会有个警告: `detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd"`

虽然可以进行下去, 但是还是需要了解其中的原因

## 解决

出现该警告是 Docker 的 Cgroup Driver 和 kubelet 的 Cgroup Driver 不一致

虽然官方有说明文档提示

> 使用 docker 时，kubeadm 会自动为其检测 cgroup 驱动并在运行时对 /var/lib/kubelet/kubeadm-flags.env 文件进行配置

所以这里是建议 Docker 使用 systemd 这种 Cgroup Driver

修改文件 `/etc/docker/daemon.json`, 没有则创建

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

```sh
# 重启 Docker
systemctl daemon-reload
systemctl restart docker
```

修改 kubelet 的 Cgroup Driver, 修改 `/etc/systemd/system/kubelet.service.d/10-kubeadm.conf` 文件, 增加 `--cgroup-driver=cgroupfs`

```conf
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --cgroup-driver=cgroupfs"
```

```sh
# 重启 kubelet
systemctl daemon-reload
systemctl restart kubelet
```
