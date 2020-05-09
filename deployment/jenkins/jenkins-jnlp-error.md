---
title: Jenkins jnlp job 失败导致 pod 错误卡死
date: 2020-05-04 19:00:00
tags: 'Jenkins'
categories:
  - ['部署', 'CI']
permalink: jenkins-jnlp-error
photo:
---

## 简介

大环境说明一下, 得说 `2G` 的机器太难了

```sh
top - 16:33:35 up 101 days, 20:55,  0 users,  load average: 13.11, 12.22, 8.01
Tasks: 178 total,   7 running, 137 sleeping,   0 stopped,   2 zombie
%Cpu(s):  5.9 us, 92.4 sy,  0.0 ni,  0.0 id,  1.3 wa,  0.0 hi,  0.3 si,  0.0 st
KiB Mem :  2041368 total,    62824 free,  1588424 used,   390120 buff/cache
KiB Swap:  2097148 total,       56 free,  2097092 used.   293296 avail Mem
```

由于测试的机器配置很糟糕, 当出现 Jenkins job 失败情况后很容易就导致了 jnlp pod Error 状态, 如何就一直卡着

```sh
jenkins-ops          jnlp-slave-3dh8s                          0/1     Error     0          50m
```

<!-- more -->

这时候一般就手动进行删除操作 `kbc delete pod jnlp-slave-3dh8s -n jenkins-ops`

当然也很容易就出现 k8s 服务器连接不上了

```sh
Unable to connect to the server: net/http: TLS handshake timeout
```

查询后发现 Error 状态的容器并没有记录下什么关键信息的事件

```sh
Name:         jnlp-slave-xmbbf
Namespace:    jenkins-ops
Priority:     0
Node:         ubuntu/172.16.0.2
Start Time:   Sat, 09 May 2020 16:22:35 +0800
Labels:       jenkins=slave
              jenkins/label=slave-agent
Annotations:  <none>
Status:       Failed
IP:           10.1.1.106
IPs:
  IP:  10.1.1.106
Containers:
  jnlp:
    Container ID:   containerd://7c4c636b3b1d176f96d1a0e5d96c3561c242a64e88772df8599aaf20a3991f9f
    Image:          127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo:1
    Image ID:       127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo@sha256:e6f508efbd0bf226a6f6c12b3ff1e3c40d5583c00695edec84f520f59a6401c3
    Port:           <none>
    Host Port:      <none>
    State:          Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Sat, 09 May 2020 16:22:59 +0800
      Finished:     Sat, 09 May 2020 16:23:56 +0800
    Ready:          False
    Restart Count:  0
    Environment:
      JENKINS_SECRET:         a9c282433692c376789affcb21888a7e719d27054987048ebc6ff3296cfa2af5
      JENKINS_AGENT_NAME:     jnlp-slave-xmbbf
      JENKINS_NAME:           jnlp-slave-xmbbf
      JENKINS_AGENT_WORKDIR:  /home/jenkins/agent
      JENKINS_URL:            http://jenkins.jenkins-ops.svc.cluster.local/
    Mounts:
      /home/jenkins/agent from workspace-volume (rw)
      /home/jenkins/agent/workspace from volume-1 (rw)
      /var/run/docker.sock from volume-0 (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from jenkins-token-5lpfl (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  volume-0:
    Type:          HostPath (bare host directory volume)
    Path:          /var/run/docker.sock
    HostPathType:
  volume-1:
    Type:          HostPath (bare host directory volume)
    Path:          /data/jenkins_home/jnlp-hexo/
    HostPathType:
  workspace-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  jenkins-token-5lpfl:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  jenkins-token-5lpfl
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  beta.kubernetes.io/os=linux
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned jenkins-ops/jnlp-slave-xmbbf to ubuntu
  Normal  Pulling    13m        kubelet, ubuntu    Pulling image "127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo:1"
  Normal  Pulled     13m        kubelet, ubuntu    Successfully pulled image "127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo:1"
  Normal  Created    13m        kubelet, ubuntu    Created container jnlp
  Normal  Started    13m        kubelet, ubuntu    Started container jnlp`
```

## 小结

这次没有打印 logs 就把 pod 删除掉了, 操作过程非常不流畅, 感觉整个程序都已经失去响应了, 后面再发现情况需要试试能否打印容器日志

大概问题如下

- 由于多个 pod Error 状态导致没有资源再运行了
- 配置过低导致过渡使用交换区造成没有响应, 看上去 `kswapd0` 进程一直在消耗 CPU
- 能否自动删除 Error 状态的 pod
- [Kubernetes plugin](https://j.yzer.club/configureClouds/) 插件配置更新限制数量, 超时时间参数看看能否优化
