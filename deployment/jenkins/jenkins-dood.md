---
title: Jenkins 使用 Docker 容器方法
date: 2020-04-14 12:00:00
tags: 'Jenkins'
categories:
  - ['部署', 'CI']
permalink: jenkins-dood
photo:
---

- DinD (Docker-in-Docker) 在 Docker 容器中部署一套 Docker 容器, 与宿主隔离
- DooD (Docker-outside-of-Docker) 主要是通过容器里面运行当前的 docker 命令

Docker 采取的是 Client/Server 架构, 我们常用的 docker xxx 命令工具, 只是 docker 的 client, 我们通过该命令行执行命令时, 实际上是在通过 client 与 docker engine 通信.

DinD 是相对简单, 直接在创建镜像的时候安装好 Docker 就可以了

```Dockerfile
RUN echo "deb http://mirrors.aliyun.com/debian/ stretch main non-free contrib" > /etc/apt/sources.list \
  && echo "deb-src http://mirrors.aliyun.com/debian/ stretch main non-free contrib" >> /etc/apt/sources.list \
  && echo "deb http://mirrors.aliyun.com/debian-security stretch/updates main" >> /etc/apt/sources.list \
  && echo "deb-src http://mirrors.aliyun.com/debian-security stretch/updates main" >> /etc/apt/sources.list \
  && echo "deb http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib" >> /etc/apt/sources.list \
  && echo "deb-src http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib" >> /etc/apt/sources.list \
  && echo "deb http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib" >> /etc/apt/sources.list \
  && echo "deb-src http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib" >> /etc/apt/sources.list \
  && apt-get update \
  && apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common \
  && curl -fsSL https://get.docker.com | bash -s docker-cli --mirror Aliyun \
  && usermod -aG 999 jenkins \
  && apt-get purge && apt-get autoremove&& apt-get clean \
  && apt-get remove apt-transport-https ca-certificates curl gnupg-agent software-properties-common -y \
  && rm -rf /var/lib/apt/lists/* && rm -rf /tmp/*
```

这里在 Jenkins 中安装 Docker,  并将用户 jenkins 添加到 docker 用户组中

默认情况下, Docker守护进程会生成一个 socket (`/var/run/docker.sock`) 文件来进行本地进程通信, 因此只能在本地使用 docker 客户端或者使用 Docker API 进行操作.
sock 文件是 UNIX 域套接字, 它可以通过文件系统 (而非网络地址) 进行寻址和访问.

DooD 是通过数据卷的形式将 docker 客户端和 `/var/run/docker.sock` socket 套接字挂载到容器内部, 实现容器内使用 docker 命令的方法

```yml
volumes:
  - /var/run/docker.sock:/var/run/docker.sock
  # dood
  - /usr/local/bin/docker:/usr/bin/docker
```

但是很多情况下宿主系统和容器基本系统不一致, 就还是需要在容器内安装 docker 客户端, 并使用 socket 套接字与宿主机器通信

宿主机器也可以开启远程端口, 让容器内的客户端可以使用.

关于用户权限这块, 一般是把用户加入到 docker 用户组, 默认是 999, `usermod -aG 999 jenkins`

参考

- [Jenkins 里运行 Docker](http://www.dockone.io/article/431)
- [Docker in Docker](https://www.cnblogs.com/kirito-c/p/11357522.html)