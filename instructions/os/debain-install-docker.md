---
title: Debian 安装 Docker Engine
date: 2020-11-29 11:00:00
tags: 'Debian'
categories:
  - ['使用说明', '操作系统']
permalink: debian-install-docker
---

## 简介

由于访问官网的流畅程度, 所以记录下基于官网的说明文档的简易步骤

## 系统需求

- Debian Buster 10 (stable)
- Debian Stretch 9 / Raspbian Stretch

Docker Engine 支持 x86_64 (or amd64), armhf, and arm64 版本

<!-- more -->

## 卸载旧版本

以前的版本命名有 docker, docker.io 或者 docker-engine, 可以运行下面命令卸载

```sh
sudo apt-get remove docker docker-engine docker.io containerd runc
```

现在最新的名称为 docker-ce

## 安装方法

### 使用 docker 的 repository 安装

最简单的安装和升级方法, 大多数时候应该选择这种方式

注意: **Raspbian 版本尚未支持**, 可以使用[便捷脚本](https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script)

#### 设置 repository

更新并安装支持 HTTPS 的软件包

```sh
# 更新
sudo apt-get update
# 安装支持 HTTPS 的软件包
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

添加 Docker 的官方 GPG 密钥

```sh
# 添加
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
# 验证
sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```

设置相应的 repository, 官方提供稳定的和每晚更新的和测试版本的 repository, 只需要将下面命令最后的 stable 修改为 nightly 或者 test 即可, [点击了解每晚更新和测试版本](https://docs.docker.com/engine/install/)

```sh
# x86_x64 / amd64
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
# armhf
sudo add-apt-repository \
   "deb [arch=armhf] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
# arm64
sudo add-apt-repository \
   "deb [arch=arm64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
```

注意: `lsb_release -cs` 命令用来获取 Debian 发行版名称

#### 安装 docker

```sh
# 更新
sudo apt-get update
# 安装
sudo apt-get install docker-ce docker-ce-cli containerd.io
```

注意: 如果使用多个 repository, 则在 `apt-get install or apt-get update` 命令中指定版本

如果需要安装特定版本, 可以先列出所有可用版本, 然后选择安装

```sh
# 列出可用版本
apt-cache madison docker-ce

  docker-ce | 5:18.09.1~3-0~debian-stretch | https://download.docker.com/linux/debian stretch/stable amd64 Packages
  docker-ce | 5:18.09.0~3-0~debian-stretch | https://download.docker.com/linux/debian stretch/stable amd64 Packages
  docker-ce | 18.06.1~ce~3-0~debian        | https://download.docker.com/linux/debian stretch/stable amd64 Packages
  docker-ce | 18.06.0~ce~3-0~debian        | https://download.docker.com/linux/debian stretch/stable amd64 Packages
  ...
# 选择安装
sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io
```

最后通过 `sudo docker info` 命令获取运行状态

#### 升级

先运行 `sudo apt-get update` 并按照上述安装过程选择新版本安装

### 从软件包安装

访问 [Docker Debian 软件下载目录](https://download.docker.com/linux/debian/dists/), 选择当前发行版本进行下载

按照按照 `.deb` 文件的过程安装即可

```sh
# /path/to 为下载的目录
sudo dpkg -i /path/to/package.deb
# 验证
sudo docker info
```

需要升级, 请下载更新的软件包, 然后重复上述安装过程即可

## 卸载

```sh
# 卸载程序
sudo apt-get purge docker-ce docker-ce-cli containerd.io
# 清理镜像, 容器, 卷
sudo rm -rf /var/lib/docker
sudo rm -rf /var/lib/containerd
```

如果有新增配置文件, 需要手动删除

## 连接

[Debian 安装 Docker](https://docs.docker.com/engine/install/debian/)
