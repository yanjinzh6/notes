---
title: Docker 服务代理
date: 2021-01-30 10:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-service-proxy
photo:
---

## 简介

有些私有仓库需要通过代理访问, 当使用 `docker pull` 后会出现网络错误, 使用 `export` 配置代理比较麻烦, 全局配置会导致其他网络连接走代理

Windows 和 Mac 下的 docker 客户端都提供了界面配置, Linux 下可以通过配置 docker 服务来实现代理

<!-- more -->

## 配置 HTTP/HTTPS 代理

创建 docker 服务的 systemd 访问目录, 在该目录下创建文件 `http-proxy.conf`

```sh
mkdir -p /etc/systemd/system/docker.service.d
touch /etc/systemd/system/docker.service.d/http-proxy.conf
```

在文件中添加环境变量代理配置, 将相应的代理 ip 配置上

```conf
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"
Environment="HTTPS_PROXY=https://proxy.example.com:443"
```

排除不需要代理的主机

```conf
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"
Environment="HTTPS_PROXY=https://proxy.example.com:443"
Environment="NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp"
```

`NO_PROXY` 参数可以通过配置字符串, 通配符等方式来排除不需要代理的主机, 多个配置使用 `,` 来分隔, 具体配置规则如下:

- 通过指定 ip 地址前缀, eg. `1.2.3.4`
- 通过域名或者特殊的 DNS 标签, eg. `*`
- 匹配相应的子域名, 通过 `.` 前缀配置仅匹配子域名
  - eg. 指定 `example.com` 将会匹配 `example.com` 和所有子域名
  - eg. 指定 `.example.com` 只会匹配子域名
- 单个 `*` 表示将不进行代理
- 使用 ip 或者域名可以配置端口号
  - eg. `1.2.3.4:80`
  - eg. `registry.example.com:443`

刷新配置并重启 docker 服务

```sh
systemctl daemon-reload
systemctl restart docker
```

验证配置是否已经加载

```sh
systemctl show --property=Environment docker
Environment=HTTP_PROXY=http://proxy.example.com:80 HTTPS_PROXY=https://proxy.example.com:443 NO_PROXY=localhost,127.0.0.1,docker-registry.example.com,.corp
```

## 参考

[Control Docker with systemd](https://docs.docker.com/config/daemon/systemd/)
