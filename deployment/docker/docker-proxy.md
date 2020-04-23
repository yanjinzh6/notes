---
title: Docker 命令使用代理
date: 2020-03-01 21:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-proxy
photo:
---

## 简介

当前环境下, 网络请求命令经常无法正常返回, 例如 `wget` 经常返回 4 状态, `curl` 也经常错误, 可以使用 `curl targetIP --max-time 3` 减少耗时, `docker build` 中经常要拉取的一下开发工具也下载很慢或者无法下载, 这时候可以使用代理进行下载

## 配置 ENV 变量

```dockerfile
ENV http_proxy "http://host.docker.internal:1080"
ENV HTTP_PROXY "http://host.docker.internal:1080"
ENV https_proxy "http://host.docker.internal:1080"
ENV HTTPS_PROXY "http://host.docker.internal:1080"
ENV ALL_PROXY "http://host.docker.internal:1080"
```

注意: **这里的 host.docker.internal 解析出来的是主机 IP, MacOS 中可以使用 docker.for.mac.localhost, Win 环境中使用 docker.for.win.localhost**

## 使用宿主网络

直接使用 `ENV` 命令固然能解决方法, 但是硬编码会导致各种环境需要相应修改, docker 中还有一种 host 网络模式, 让 container 使用宿主机的网络, 这里 `docker build --network host .` 也可以相应配置上

```sh
export http_proxy="http://host.docker.internal:1080"
export HTTP_PROXY="http://host.docker.internal:1080"
export https_proxy="http://host.docker.internal:1080"
export HTTPS_PROXY="http://host.docker.internal:1080"
export ALL_PROXY="http://host.docker.internal:1080"

docker build --network host .
```

## 小结

这里主要了解一下 docker 的网络模式, 当然如果基础镜像使用国内封装过的就方便多了, 最好还是改善网络环境
