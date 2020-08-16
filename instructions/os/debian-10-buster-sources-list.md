---
title: Debian 10 Buster 国内常用镜像源
date: 2020-08-15 10:00:00
tags: 'Debian'
categories:
  - ['使用说明', '操作系统']
permalink: linux-awk-command
---

## 简介

日常使用中, 不管是容器还是物理机器都需要使用快一点的站点, 所以 Debian 10 发布后, 也需要将 `/etc/apt/sources.list` 更新为国内比较快的镜像源

## 方法

```sh
# 将源修改为 163
sed -i 's#http://deb.debian.org#https://mirrors.163.com#g' /etc/apt/sources.list
```

注意: **如果安装源是 HTTPS 协议的需要安装 apt-transport-https**

```sh
apt-get install apt-transport-https
apt-get update
```

<!-- more -->

## 常用安装源列表

### 163 镜像站

```sh
deb http://mirrors.163.com/debian/ buster main non-free contrib
deb http://mirrors.163.com/debian/ buster-updates main non-free contrib
deb http://mirrors.163.com/debian/ buster-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ buster/updates main non-free contrib

deb-src http://mirrors.163.com/debian/ buster main non-free contrib
deb-src http://mirrors.163.com/debian/ buster-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ buster-backports main non-free contrib
deb-src http://mirrors.163.com/debian-security/ buster/updates main non-free contrib
```

### 华为云镜像站

```sh
deb https://mirrors.huaweicloud.com/debian/ buster main contrib non-free
deb https://mirrors.huaweicloud.com/debian/ buster-updates main contrib non-free
deb https://mirrors.huaweicloud.com/debian/ buster-backports main contrib non-free
deb https://mirrors.huaweicloud.com/debian-security/ buster/updates main contrib non-free

deb-src https://mirrors.huaweicloud.com/debian/ buster main contrib non-free
deb-src https://mirrors.huaweicloud.com/debian/ buster-updates main contrib non-free
deb-src https://mirrors.huaweicloud.com/debian/ buster-backports main contrib non-free
```

### 腾讯云镜像站

```sh
deb http://mirrors.cloud.tencent.com/debian/ buster main non-free contrib
deb http://mirrors.cloud.tencent.com/debian-security buster/updates main
deb http://mirrors.cloud.tencent.com/debian/ buster-updates main non-free contrib
deb http://mirrors.cloud.tencent.com/debian/ buster-backports main non-free contrib

deb-src http://mirrors.cloud.tencent.com/debian-security buster/updates main
deb-src http://mirrors.cloud.tencent.com/debian/ buster main non-free contrib
deb-src http://mirrors.cloud.tencent.com/debian/ buster-updates main non-free contrib
deb-src http://mirrors.cloud.tencent.com/debian/ buster-backports main non-free contrib
```

### 中科大镜像站

```sh
deb https://mirrors.ustc.edu.cn/debian/ buster main contrib non-free
deb https://mirrors.ustc.edu.cn/debian/ buster-updates main contrib non-free
deb https://mirrors.ustc.edu.cn/debian/ buster-backports main contrib non-free
deb https://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free

deb-src https://mirrors.ustc.edu.cn/debian/ buster main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ buster-updates main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ buster-backports main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian-security/ buster/updates main contrib non-free
```

### 阿里云镜像站

```sh
deb http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb http://mirrors.aliyun.com/debian-security buster/updates main
deb http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib

deb-src http://mirrors.aliyun.com/debian-security buster/updates main
deb-src http://mirrors.aliyun.com/debian/ buster main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ buster-backports main non-free contrib
```

### 清华大学镜像站

```sh
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security/ buster/updates main contrib non-free

deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security/ buster/updates main contrib non-free
```

### 兰州大学镜像站

```sh
deb http://mirror.lzu.edu.cn/debian stable main contrib non-free
deb http://mirror.lzu.edu.cn/debian stable-updates main contrib non-free
deb http://mirror.lzu.edu.cn/debian/ buster-backports main contrib non-free
deb http://mirror.lzu.edu.cn/debian-security/ buster/updates main contrib non-free

deb-src http://mirror.lzu.edu.cn/debian stable main contrib non-free
deb-src http://mirror.lzu.edu.cn/debian stable-updates main contrib non-free
deb-src http://mirror.lzu.edu.cn/debian/ buster-backports main contrib non-free
deb-src http://mirror.lzu.edu.cn/debian-security/ buster/updates main contrib non-free
```

### 上海交大镜像站

```sh
deb https://mirror.sjtu.edu.cn/debian/ buster main contrib non-free
deb https://mirror.sjtu.edu.cn/debian/ buster-updates main contrib non-free
deb https://mirror.sjtu.edu.cn/debian/ buster-backports main contrib non-free
deb https://mirror.sjtu.edu.cn/debian-security/ buster/updates main contrib non-free

deb-src https://mirror.sjtu.edu.cn/debian/ buster-updates main contrib non-free
deb-src https://mirror.sjtu.edu.cn/debian/ buster-backports main contrib non-free
deb-src https://mirror.sjtu.edu.cn/debian/ buster main contrib non-free
deb-src https://mirror.sjtu.edu.cn/debian-security/ buster/updates main contrib non-free
```

## 参考

- [官方全球镜像站列表](https://www.debian.org/mirror/list)
