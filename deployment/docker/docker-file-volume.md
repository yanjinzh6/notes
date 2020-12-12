---
title: Dockerfile VOLUME 指令
date: 2020-12-06 09:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-file-volume
photo:
---

## 简介

VOLUME 指令可以指定一个或多个目录作为容器的数据卷, 之后就算容器被删除了, 被指定目录的数据依然存在

容器运行时应该尽量保持容器存储层不发生写操作, 对于需要动态修改数据的应用, 其动态数据文件应该保存在卷 (VOLUME) 中

VOLUME 可以简化 Dockerfile 的存储层复杂度, 如果有存储层的变化, 只需要更新对应目录的数据即可, 而不需要重新打包镜像

<!-- more -->

## 使用

使用格式

```dockerfile
VOLUME ["<路径 1>", "<路径 2>"...]
VOLUME <路径>
```

该指令可以防止运行时yoghurt忘记将动态文件保存的目录挂载为卷, 导致容器删除后丢失数据

使用该指令事先指定某些目录挂载为匿名卷, 这样在运行时如果没有主动指定挂载, 其修改的内容也会保存在 Docker 创建的挂载目录中

示例: Dockerfile 中使用 `VOLUME /data` 将 `/data` 目录作为容器数据卷目录

在运行时如果没有指定挂载, 那么 `/data` 目录会自动挂载为匿名卷, 向该目录中写入的信息都不会记录进容器, 从而保证了容器存储层的无状态化

在运行时可以指定挂载配置: `docker run -d --rm --name demo -v /your/path:/data xxx`

使用上面命令启动的容器, 就使用了自定义的 `/your/path` 目录挂载了 `/data`, 替换了 Dockerfile 中定义的匿名卷的配置

```sh
# 通过 -valumes-from 选项实现容器数据卷共享
docker run -it -volumes-from container1 image2 bash
```

container1 为需要被共享数据卷的容器 ID, image2 为需要共享数据的景象名
