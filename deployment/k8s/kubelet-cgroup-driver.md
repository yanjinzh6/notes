---
title: kubectl-cgroup-driver
date: 2021-02-21 10:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubectl-cgroup-driver
photo:
---

使用旧版本的 (18) 的 docker 和 (20) 版本的 kubeadm 在检测阶段不会提示 docker cgroup driver 不匹配的警告, 但是会一直出现如下错误

```sh
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp [::1]:10248: connect: connection refused.
timed out waiting for the condition
error uploading crisocket
k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/join.runKubeletStartJoinPhase
        /workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/join/kubelet.go:190
k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow.(*Runner).Run.func1
        /workspace/src/k8s.io/kubernetes/_output/dockerized/go/src/k8s.io/kubernetes/cmd/kubeadm/app/cmd/phases/workflow/runner.go:234
```

排查后发现是 Kubelet 无法启动, 使用 `journalctl -xeu kubelet` 查看日志发现如下报错

```sh
 10:34:30.679034   19173 server.go:269] failed to run Kubelet: misconfiguration: kubelet cgroup driver: "systemd" is different from docker cgroup driver: "cgroupfs"
tine 1 [running]:
o/kubernetes/vendor/k8s.io/klog/v2.stacks(0xc000010001, 0xc000874a00, 0xaa, 0xfc)
```

修改 docker 服务配置 `/etc/docker/daemon.json`, 重启 docker 后正常

```json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

```sh
systemctl daemon-reload
systemctl restart docker
```
