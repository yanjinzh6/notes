---
title: Docker 镜像保存和加载
date: 2020-01-17 10:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-save-load
photo:
---

## 简介

> 这篇文章主要介绍了一种根据便捷的镜像交付过程

一般项目的交付都需要一个甲乙双方私用的注册中心，然后乙方推送后自动在甲方那边集成部署，当然对于很多项目，大部分流程都是乙方打包，然后通过邮件或者其他形式，直接进行发送处理。

docker 也提供类似的功能，通过 `docker save`, `docker load` 来保存和导入镜像，即可以实现简单的镜像交付

<!-- more -->

## docker save

`docker save`: 将指定镜像保存成 tar 归档文件

```sh
docker save [OPTIONS] IMAGE [IMAGE...]

OPTIONS说明

  -o: 输出的文件, tar 格式
```

## docker load

`docker load`: 导入使用 `docker save` 命令导出的镜像

```sh
docker load [OPTIONS]

OPTIONS说明

  --input , -i: 指定导入的文件，代替 STDIN

  --quiet , -q: 精简输出信息
```
