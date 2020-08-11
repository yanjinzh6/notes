---
title: Docker 输出到系统日志
date: 2020-08-08 15:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-syslog
photo:
---

## 简介

[Rsyslog](https://www.rsyslog.com/) 是 Syslog 标准的一种实现, 可以通过命令查看 Rsyslog 信息

```sh
rsyslogd -v
rsyslogd 8.32.0, compiled with:
        PLATFORM:                               x86_64-pc-linux-gnu
        PLATFORM (lsb_release -d):
        FEATURE_REGEXP:                         Yes
        GSSAPI Kerberos 5 support:              Yes
        FEATURE_DEBUG (debug build, slow code): No
        32bit Atomic operations supported:      Yes
        64bit Atomic operations supported:      Yes
        memory allocator:                       system default
        Runtime Instrumentation (slow code):    No
        uuid support:                           Yes
        systemd support:                        Yes
        Number of Bits in RainerScript integers: 64

See http://www.rsyslog.com for more information.
```

<!-- more -->

## 使用

### 开启 Rsyslog

```sh
vi /etc/rsyslog.conf

# 开启配置中相应的服务
#module(load="imtcp")
#input(type="imtcp" port="514")

# 也可以开启 UDP 服务
#module(load="imudp")
#input(type="imudp" port="514")

# 旧版本参数名如下
#$ModLoad imtcp
#$InputTCPServerRun 514

# 重启
systemctl restart rsyslog

# 查看是否监听 514 端口
netstat -anpt | grep 514
```

### docker 容器输出日志

```sh
docker run \
-d \
--rm \
-p 80:80 \
--log-driver syslog \
--log-opt syslog-address=tcp://localhost:514 \
--name nginx \
nginx
```

使用 `--log-driver` 指定 Syslog 为日志驱动, 使用 `--log-opt` 指定 Syslog 服务器的地址

可以通过 `tail -f /var/log/syslog` 来访问日志, 或者通过 `journalctl` 命令来输出相应日志

参考[自定义 syslog 模板](http://www.rsyslog.com/doc/v8-stable/configuration/properties.html)

### 配置容器输出内容

docker 自动将容器 ID 的前 12 个字符输出到日志中, 在运行容器的时候可以自定义相应的标签

Docker 已提供的模板标签

- {{.ID}}: 容器 ID 的前 12 个字符
- {{.FullID}}: 容器 ID 的完整名称
- {{.Name}}: 容器名称
- {{.ImageID}}: 容器镜像 ID 的前 12 个字符
- {{.ImageFullID}}: 容器镜像 ID 的完整名称
- {{.ImageName}}: 容器镜像名称
- {{.DaemonName}}: Docker 守护进程名称 (名为 docker)

例如

```sh
docker run \
-d \
--rm \
-p 80:80 \
--log-driver syslog \
--log-opt syslog-address=tcp://localhost:514 \
--log-opt tag="{{.ImageID}}/{{.Name}}/{{.ID}}" \
--name nginx \
nginx
```
