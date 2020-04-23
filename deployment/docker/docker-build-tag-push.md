---
title: Docker 构建和推送
date: 2020-01-17 09:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-build-tag-push
photo:
---

## 简介

> 这篇文章主要介绍了在持续继承中会用到构建镜像的基础命令

一般的构建流程是

- 开发者通过提交变更并将其推送 / 合并到 master 分支或打了个 git tag 或 release 后, 触发 ci 钩子来构建和测试
- 测试成功后进行打包 `docker build -t xxx/yyy:zz .`
- 登陆自建的镜像注册表
  - 添加一个镜像 tag 来标记构建的镜像 `docker tag xxx/yyy:zz 127.0.0.1:32000/xxx/yyy:zz`
  - 将镜像推送到镜像注册表 `docker push 127.0.0.1:32000/xxx/yyy:zz`
- 登陆 docker hub
  - 添加镜像 tag 不需要添加域名, 默认就是 docker hub
  - 直接推送

**推送时, 默认是 docker hub 需要自定义 tag 推送到自建的镜像注册表**

<!-- more -->

## docker build

`docker build` 命令用于使用 Dockerfile 创建镜像

```sh
docker build [OPTIONS] PATH | URL | -

OPTIONS说明

  --build-arg=[]: 设置镜像创建时的变量；
  --cpu-shares: 设置 cpu 使用权重；
  --cpu-period: 限制 CPU CFS周期；
  --cpu-quota: 限制 CPU CFS配额；
  --cpuset-cpus: 指定使用的CPU id；
  --cpuset-mems: 指定使用的内存 id；
  --disable-content-trust: 忽略校验, 默认开启；
  -f: 指定要使用的Dockerfile路径；
  --force-rm: 设置镜像过程中删除中间容器；
  --isolation: 使用容器隔离技术；
  --label=[]: 设置镜像使用的元数据；
  -m: 设置内存最大值；
  --memory-swap: 设置Swap的最大值为内存+swap, "-1"表示不限swap；
  --no-cache: 创建镜像的过程不使用缓存；
  --pull: 尝试去更新镜像的新版本；
  --quiet, -q: 安静模式, 成功后只输出镜像 ID；
  --rm: 设置镜像成功后删除中间容器；
  --shm-size: 设置/dev/shm的大小, 默认值是64M；
  --ulimit: Ulimit配置.
  --tag, -t: 镜像的名字及标签, 通常 name:tag 或者 name 格式；可以在一次构建中为一个镜像设置多个标签.
  --network: 默认 default. 在构建期间设置RUN指令的网络模式
```

## docker tag

`docker tag` : 标记本地镜像, 将其归入某一仓库

```sh
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]
```

## docker push

`docker push` : 将本地的镜像上传到镜像仓库, 要先登陆到镜像仓库

```sh
docker push [OPTIONS] NAME[:TAG]

OPTIONS说明

  --disable-content-trust : 忽略镜像的校验, 默认开启
```

## 小结

比较简单的构建过程, 可以很好的集成到 ci 中, 也可以从另一个思路走, 通过 `docker save`, `docker load` 来保存和导入镜像
