---
title: Docker 日志驱动
date: 2020-08-08 14:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-logging-driver
photo:
---

## 简介

使用 `docker logs` 命令可以随时查看 Docker 容器内部应用程序运行时所产生的日志, 这样可以避免首先进入 Docker 容器, 再打开应用程序的日志文件的过程

使用 `docker info` 命令, 可以看到 Docker 容器的相关信息, 获取 `Logging Driver` 字段的内容如下

```sh
docker info | grep 'Logging Driver'
Logging Driver: json-file
```

可以了解到当前是使用 JSON 文件来保存容器内部应用程序所输出的日志

通过访问 `/var/lib/docker/containers/＜container_id＞` 目录, 可以看到目录下存放着 `＜container_id＞-json.log` 就是当前保存的日志内容

<!-- more -->

## 日志驱动类型

- none: 容器不输出任何日志
- json-file: 容器输出的日志以 JSON 格式写入文件中 (默认)
- syslog: 容器输出的日志写入宿主机的 Syslog 中
- journald: 容器输出的日志写入宿主机的 Journald 中
- gelf: 容器输出的日志以 GELF (Graylog Extended Log Format) 格式写入 Graylog 中
- fluentd: 容器输出的日志写入宿主机的 Fluentd 中
- awslogs: 容器输出的日志写入 Amazon CloudWatch Logs 中
- splunk: 容器输出的日志写入 splunk 中
- etwlogs: 容器输出的日志写入 ETW (Event Tracing for Windows) 中
- gcplogs: 容器输出的日志写入 GCP (Google Cloud Platform) 中
- nats: 容器输出的日志写入 NATS 服务器中

## 使用其他日志驱动类型

通过 `docker run` 命令中指定 `--log-driver` 参数来指定具体的日志驱动, 可以通过 `--log-opt` 参数来指定对应日志驱动的相关选项

例如

```sh
docker run \
-d \
--rm \
-p 80:80 \
--log-driver json-file \
--log-opt max-size=10m \
--log-opt max-file=3 \
--name nginx \
nginx
```

通过 `--log-opt` 参数为 json-file 日志驱动添加了两个选项, `max-size=10m` 表示 JSON 文件最大为 `10MB` (超过 `10MB` 就会自动生成新文件) , `max-file=3` 表示 JSON 文件最多为 `3` 个 (超过 `3` 个就会自动删除多余的旧文件)

## 参考

- [官方文档](https://docs.docker.com/engine/admin/logging/overview/)
