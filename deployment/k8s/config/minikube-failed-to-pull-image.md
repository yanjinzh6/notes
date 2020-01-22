---
title: Minikube 无法拉取镜像
date: 2020-01-22 11:30:00
tags: '配置'
categories:
  - ['部署', 'k8s']
permalink: minikube-failed-to-pull-image
photo:
---

基于位置因素, 经常出现一些奇奇怪怪的问题, 今天测试个 jupyter 的配置后发现 minikube 一直创建不了 pod, 日志如下

```sh
kbc get pod jupyter-notebook-588b68dd65-5zbvf
NAME                                READY   STATUS             RESTARTS   AGE
jupyter-notebook-588b68dd65-5zbvf   0/1     ImagePullBackOff   0          28s
```

很明显又是访问不了 dockerhub 了, 但是又想起来, 本地已经有了容器了, 而且还是最新的版本了, 应该不需要再去拉了, 试了下 docker 命令, 应该是每次都要去比较一下 sha256 是不是最新的, 所以这里应该是 minikube 访问不了 dockerhub, 直接使用 docker 命令是可以的

<!-- more -->

```sh
docker pull jupyter/base-notebook 
Using default tag: latest
latest: Pulling from jupyter/base-notebook
Digest: sha256:7f696fe2cb4ee581262c8c053831188efbe048751bd8e7f0e2011cae41ec555c
Status: Image is up to date for jupyter/base-notebook:latest
```

```sh
kbc describe pod jupyter-notebook-588b68dd65-5zbvf
Name:         jupyter-notebook-588b68dd65-5zbvf
Namespace:    default
Priority:     0
Node:         ubuntu/172.16.0.2
Start Time:   Wed, 22 Jan 2020 10:47:39 +0800
Labels:       app=jupyter-notebook
              pod-template-hash=588b68dd65
Annotations:  <none>
Status:       Pending
IP:           10.1.1.165
IPs:
  IP:           10.1.1.165
Controlled By:  ReplicaSet/jupyter-notebook-588b68dd65
Containers:
  scipy-notebook:
    Container ID:  
    Image:         jupyter/scipy-notebook:latest
    Image ID:      
    Port:          8888/TCP
    Host Port:     0/TCP
    Command:
      start-notebook.sh
    Args:
      --NotebookApp.token=''
    State:          Waiting
      Reason:       ErrImagePull
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-8hq8q (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             False 
  ContainersReady   False 
  PodScheduled      True 
Volumes:
  default-token-8hq8q:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-8hq8q
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type     Reason     Age                From               Message
  ----     ------     ----               ----               -------
  Normal   Scheduled  <unknown>          default-scheduler  Successfully assigned default/jupyter-notebook-588b68dd65-5zbvf to ubuntu
  Normal   Pulling    22s (x2 over 36s)  kubelet, ubuntu    Pulling image "jupyter/scipy-notebook:latest"
  Warning  Failed     22s (x2 over 36s)  kubelet, ubuntu    Failed to pull image "jupyter/scipy-notebook:latest": rpc error: code = Unknown desc = failed to resolve image "docker.io/jupyter/scipy-notebook:latest": no available registry endpoint: failed to do request: Head https://registry-1.docker.io/v2/jupyter/scipy-notebook/manifests/latest: dial tcp 52.2.186.244:443: connect: connection refused
  Warning  Failed     22s (x2 over 36s)  kubelet, ubuntu    Error: ErrImagePull
  Normal   BackOff    8s (x2 over 35s)   kubelet, ubuntu    Back-off pulling image "jupyter/scipy-notebook:latest"
  Warning  Failed     8s (x2 over 35s)   kubelet, ubuntu    Error: ImagePullBackOff
```

当前版本的 minikube 是和 containerd 集成了, 所以需要修改一下 containerd 的配置文件

```sh
# /var/snap/microk8s/current/args/containerd-template.toml
    [plugins.cri.registry]
      [plugins.cri.registry.mirrors]
        [plugins.cri.registry.mirrors."docker.io"]
          endpoint = ["https://registry-1.docker.io"]
        [plugins.cri.registry.mirrors."localhost:32000"]
          endpoint = ["http://127.0.0.1:32000"]
        [plugins.cri.registry.mirrors."127.0.0.1:32000"]
          endpoint = ["http://127.0.0.1:32000"]
```

需要修改 docker.io 的 endpoint 为可以访问的地址, 这里配置了私有的注册表, 所以就偷懒把 docker 拉取的镜像推送到私有的注册表后使用就可以了
