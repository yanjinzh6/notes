---
title: Docker 网络模式
date: 2020-07-05 18:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-network-mode
photo:
---

## 简介

Docker 4 种网络模式:

- Bridge: 桥接式网络模式, 为每一个容器分配, 设置 IP 等, 并将容器连接到一个 docker0 虚拟网桥, 通过 docker0 网桥以及 Iptables nat 表配置与宿主机通信
- Host: 开放式网络模式, 使用宿主机的 IP 和端口
- Container: 联合挂载式网络模式, 是 host 网络模式的延伸, 与指定的容器共享 IP, 端口范围
- None: 封闭式网络模式, 关闭了容器的网络功能
- 自定义网络: 需要进行配置

<!-- more -->

## 详细介绍

通过 `docker network` 命令可以查看到当前所有的网络

```sh
docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
8a1e789d0af0        bridge              bridge              local
d79a76bae1fc        dev-net             bridge              local
440ab1d633ba        dev_default         bridge              local
cd9075436b0c        host                host                local
8dd5f2161981        none                null                local
```

### Bridge 网络模式

- Docker 宿主机创建一个 docker0 网卡, 随机分配一个本地未占用的私有网段, 默认为 172.17.0.1/16
- Docker 容器会增加一个 eth0 的网卡, 随机分配同一网段: 从 172.17.0.0/16 中分配一个 ip
- 当 Docker 创建一个容器时, 同时会创建了一对 veth pair 接口 (当数据包发送到一个接口时, 另外一个接口也可以收到相同的数据包) , 这对接口一端在容器内, 即 eth0, 另一端在本地并被挂载到 docker0 网桥, 名称以 veth 开头 (例如 vethAQI2QT) , 通过这种方式, 主机可以跟容器通信, 容器之间也可以相互通信, Docker 就创建了在主机和所有容器之间一个虚拟共享网络

可以通过以下命令查看当前网卡

```sh
ifconfig | grep docker -A 8
ifconfig | grep eth0 -A 7
```

在该模式下的容器从 docker0 子网中分配一个 IP, 并使用 docker0 的 IP 地址作为默认网关

Bridge 模式是 docker 容器的默认网络模式, 不带 `--network` 参数, 就是 bridge 模式, 使用 `docker run -p` 时, docker 实际是在 iptables 做了 DNAT 规则, 实现端口转发功能, 可以使用 `iptables -t nat -vnL` 查看

由于 Docker 在 Mac 中是通过在 Mac 中的虚拟机实现的, 一种是基于 HyperKit (Docker Desktop for Mac), 另一种是基于 Virtual Box (Docker Toolbox) , 两者的区别会导致宿主机和容器直接网络访问方式不同, Docker Desktop for Mac 是基于 `/var/run/docker.sock` 文件, Docker Toolbox 是基于虚拟网卡, 还有就是管理容器的方式不一样, Docker Desktop for Mac 是使用 HyperKit,  基于 Hypervisor 的轻量级虚拟化机, Docker Toolbox 则是使用 Virtual Box 创建虚拟机的方式来创建容器

Docker 在 Mac 中的限制

- 没有 docker0 bridge 网卡
- 宿主机不能 ping 通容器的ip
- 宿主机不能从主机 Mac 访问 Linux bridge

Mac 宿主机 和 容器互通 的解决方案

- 容器内访问宿主机, 直接使用宿主机的 IP, 在 Docker 18.03 过后推荐使用 特殊的 DNS 记录 host.docker.internal 访问宿主机, 但是注意, 这个只是在 Docker Desktop for Mac 中作为开发时有效,  网关的 DNS 记录: gateway.docker.internal
- 宿主机访问容器, 使用本机 localhost 端口映射功能, 使用 –publish (单个端口) , -p (单个端口) , -P (所有端口)  将本机的端口和容器的端口映射

### Host 网络模式

使用 Host 模式, 容器将和宿主机共用一个 Network Namespace, 容器使用宿主机的 IP 和端口, 其他方面, 如文件系统, 进程列表等还是和宿主机隔离, `docker run --network host` 使用该参数指定容器运行在 Host 网络模式下

```sh
# 运行容器
docker run --name=nginx_host --rm --net=host -p 80:80 -d nginx
WARNING: Published ports are discarded when using host network mode
cc8a66f2ecb77c3a3d711140d56dee211d13527a2ed0101fff37cf83eff60ce2

# 查询容器
docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                               NAMES
cc8a66f2ecb7        nginx               "nginx -g 'daemon of…"   About a minute ago   Up About a minute                                       nginx_host

# 查询 IP 端口绑定
netstat -nplt | grep nginx
```

Docker 17.06+ 支持, 仅 Linux 版本

### Container 网络模式

使用 Container 模式, 指定新创建的容器和已经存在的一个容器共享一个 Network Namespace, 和一个指定的容器共享 IP, 端口范围等, 两个容器除了网络方面, 其他的如文件系统, 进程列表等还是隔离的, 两个容器的进程可以通过 lo 网卡设备通信, 使用 `docker run --network container:容器名或 ID` 可以运行当前容器, 并使用指定容器名的容器共享网络

### None 网络模式

使用 None 模式, Docker容器拥有自己的 Network Namespace, 但是没有为 Docker 容器进行任何网络配置, 也就是说, 这个 Docker 容器没有网卡, IP, 路由等信息, 只有 lo 网络接口, 需要为 Docker 容器添加网卡, 配置IP等

None 模式下的容器不参与网络通信, 运行于此类容器中的进程仅能访问本地回环接口, 仅适用于进程无须网络通信的场景中, 例如: 备份, 进程诊断及各种离线任务等

### Overlay

Overlay 模式是 docker 自带的跨主机网络模型, 在 docker1.8 新加入

### macvlan

macvlan 网卡虚拟化技术, 可以在一张物理网卡上配置多个 MAC 地址, 对应多个 interfaces, 每个 interface 一个 ip, 优点是性能极好, 不需要创建 Linux bridge, 而是直接通过 interface 连接到物理网络

在 Docker 中, 可以为每个容器分配一个 MAC 地址, 直接通过 MAC 地址转发数据

需要考虑网络性能时, 或者希望应用直接通过物理网卡连接到网络时可以使用

### 自定义网络

启动 Docker 容器的时候，使用默认的网络是不支持指派固定 IP 的

```sh
docker run --rm --net bridge --ip 172.17.0.10 -d nginx
6eb1f228cf308d1c60db30093c126acbfd0cb21d76cb448c678bab0f1a7c0df6
docker: Error response from daemon: User specified IP address is supported on user defined networks only.
```

所以需要创建自定义网络

```sh
# 指定网段 172.18.0.0/16
docker network create --subnet=172.18.0.0/16 mynetwork

# 使用自定义网络和固定 IP
docker run --rm --net mynetwork --ip 172.17.0.10 -d nginx
```
