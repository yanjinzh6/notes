---
title: docker-nginx-https
date: 2021-02-01 19:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: docker-nginx-https
photo:
---

https://blog.csdn.net/qq_17054989/article/details/88563060

修改
/lib/systemd/system/docker.service

在里面的 EXECStart 的后面增加后如下:

```
ExecStart=/usr/bin/dockerd --graph /home/docker
```

添加 nfs 容器支持

修改 /etc/docker/daemon.json, 选择使用 vfs, 然后重启 docker
{
"storage-driver": "vfs"
}

```
systemctl daemon-reload
systemctl docekr restart
```
