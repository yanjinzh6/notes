---
title: 所有 Pod 重启问题
date: 2020-06-14 11:30:00
tags: '故障排查'
categories:
  - ['部署', 'k8s']
permalink: all-pods-restarts
photo:
---

## 现状

最近发现连接 pod 的 app 经常报错连不上, 检查了 pod 状态都是正常的, 以为是网络问题, 但是想想内网是没那么频繁的断网的, 查看了以下 `kubectl get pod -A`, 发现一个奇怪的情况 `RESTARTS` 项好像都增加了, 于是检查一下 pod 的详情, 发现有如下日志

```sh
Normal  SandboxChanged  52m (x2 over 52m)  kubelet, ubuntu  Pod sandbox changed, it will be killed and re-created.
Normal  Pulled          52m                kubelet, ubuntu  Container image "docker.io/bitnami/mongodb:4.2.3-debian-10-r10" already present on machine
Normal  Created         52m                kubelet, ubuntu  Created container my-mongo-mongodb
Normal  Started         52m                kubelet, ubuntu  Started container my-mongo-mongodb
```

<!-- more -->

查了下事件列表 `kubectl get event -A` , 所以 pod 都有以上事件

## 排查

所有 pod 都一直 `re-created`, 重新看一下 pod 详情

```sh
State:          Running
  Started:      Wed, 17 Jun 2020 15:39:43 +0800
Last State:     Terminated
  Reason:       Unknown
  Exit Code:    255
  Started:      Wed, 17 Jun 2020 08:34:33 +0800
  Finished:     Wed, 17 Jun 2020 15:39:28 +0800
```

发现所有的 pod 都是被停止

查询系统日志 `journalctl --since "2020-06-17 15:30:00" --until "2020-06-17 15:40:00" --no-pager > kubelet.log`

```sh
Jun 17 15:38:17 ubuntu microk8s.daemon-apiserver[4933]: W0617 15:38:17.381328    4933 clientconn.go:1208] grpc: addrConn.createTransport failed to connect to {https://127.0.0.1:12379  <nil> 0 <nil>}. Err :connection error: desc = "transport: Error while dialing dial tcp 127.0.0.1:12379: connect: connection refused". Reconnecting...

Jun 17 15:38:17 ubuntu etcd[4846]: skipped leadership transfer for single voting member cluster
Jun 17 15:38:17 ubuntu systemd[1]: Stopped Service for snap application microk8s.daemon-etcd.

Jun 17 15:38:17 ubuntu microk8s.daemon-flanneld[4830]: E0617 15:38:17.882451    4830 watch.go:171] Subnet watch failed: client: etcd cluster is unavailable or misconfigured; error #0: unexpected EOF
Jun 17 15:38:17 ubuntu microk8s.daemon-flanneld[4830]: E0617 15:38:17.862852    4830 watch.go:43] Watch subnets: client: etcd cluster is unavailable or misconfigured; error #0: unexpected EOF
Jun 17 15:38:18 ubuntu systemd[1]: Stopping Service for snap application microk8s.daemon-apiserver-kicker...
Jun 17 15:38:18 ubuntu systemd[1]: Stopped Service for snap application microk8s.daemon-apiserver-kicker.
Jun 17 15:38:18 ubuntu systemd[1]: Stopping Service for snap application microk8s.daemon-containerd...

Jun 17 15:38:18 ubuntu microk8s.daemon-containerd[4975]: time="2020-06-17T15:38:18+08:00" level=fatal msg="containerd-shim: ttrpc server failure" error="ttrpc: server closed"

Jun 17 15:38:19 ubuntu microk8s.daemon-flanneld[4830]: E0617 15:38:19.056250    4830 watch.go:43] Watch subnets: client: etcd cluster is unavailable or misconfigured; error #0: dial tcp 127.0.0.1:12379: connect: connection refused
Jun 17 15:38:19 ubuntu microk8s.daemon-flanneld[4830]: E0617 15:38:19.057133    4830 watch.go:171] Subnet watch failed: client: etcd cluster is unavailable or misconfigured; error #0: dial tcp 127.0.0.1:12379: connect: connection refused

Jun 17 15:38:19 ubuntu microk8s.daemon-apiserver[4933]: W0617 15:38:19.522980    4933 clientconn.go:1208] grpc: addrConn.createTransport failed to connect to {https://127.0.0.1:12379  <nil> 0 <nil>}. Err :connection error: desc = "transport: Error while dialing dial tcp 127.0.0.1:12379: connect: connection refused". Reconnecting...

Jun 17 15:38:21 ubuntu microk8s.daemon-controller-manager[4817]: I0617 15:38:21.534184    4817 pv_controller_base.go:311] Shutting down persistent volume controller
Jun 17 15:38:21 ubuntu microk8s.daemon-controller-manager[4817]: I0617 15:38:21.825643    4817 pv_controller_base.go:505] claim worker queue shutting down
Jun 17 15:38:21 ubuntu microk8s.daemon-apiserver[4933]: E0617 15:38:21.844239    4933 status.go:71] apiserver received an error that is not an metav1.Status: &errors.errorString{s:"context canceled"}

Jun 17 15:38:21 ubuntu systemd[1]: snap.microk8s.daemon-controller-manager.service: Main process exited, code=exited, status=255/n/a
Jun 17 15:38:21 ubuntu systemd[1]: snap.microk8s.daemon-controller-manager.service: Failed with result 'exit-code'.
Jun 17 15:38:21 ubuntu systemd[1]: snap.microk8s.daemon-controller-manager.service: Service hold-off time over, scheduling restart.
Jun 17 15:38:21 ubuntu systemd[1]: snap.microk8s.daemon-controller-manager.service: Scheduled restart job, restart counter is at 1.
Jun 17 15:38:21 ubuntu systemd[1]: Stopped Service for snap application microk8s.daemon-controller-manager.
Jun 17 15:38:21 ubuntu systemd[1]: Started Service for snap application microk8s.daemon-controller-manager.
```

看着还真像是网络问题导致 microk8s 重启
