---
title: Debian 9 Stretch 国内常用镜像源
date: 2020-08-15 16:00:00
tags: 'Debian'
categories:
  - ['使用说明', '操作系统']
permalink: debian-9-stretch-sources-list
---

## 简介

这是 Debian 9 的 `/etc/apt/sources.list` 更新为国内比较快的镜像源的记录文档

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
deb http://mirrors.163.com/debian/ stretch main non-free contrib
deb http://mirrors.163.com/debian/ stretch-updates main non-free contrib
deb http://mirrors.163.com/debian/ stretch-backports main non-free contrib
deb http://mirrors.163.com/debian-security/ stretch/updates main non-free contrib

deb-src http://mirrors.163.com/debian/ stretch main non-free contrib
deb-src http://mirrors.163.com/debian/ stretch-updates main non-free contrib
deb-src http://mirrors.163.com/debian/ stretch-backports main non-free contrib
deb-src http://mirrors.163.com/debian-security/ stretch/updates main non-free contrib
```

### 华为云镜像站

```sh
deb https://mirrors.huaweicloud.com/debian/ stretch main contrib non-free
deb https://mirrors.huaweicloud.com/debian/ stretch-updates main contrib non-free
deb https://mirrors.huaweicloud.com/debian/ stretch-backports main contrib non-free
deb https://mirrors.huaweicloud.com/debian-security/ stretch/updates main contrib non-free

deb-src https://mirrors.huaweicloud.com/debian/ stretch main contrib non-free
deb-src https://mirrors.huaweicloud.com/debian/ stretch-updates main contrib non-free
deb-src https://mirrors.huaweicloud.com/debian/ stretch-backports main contrib non-free
```

### 腾讯云镜像站

```sh
deb http://mirrors.cloud.tencent.com/debian/ stretch main non-free contrib
deb http://mirrors.cloud.tencent.com/debian-security stretch/updates main
deb http://mirrors.cloud.tencent.com/debian/ stretch-updates main non-free contrib
deb http://mirrors.cloud.tencent.com/debian/ stretch-backports main non-free contrib

deb-src http://mirrors.cloud.tencent.com/debian-security stretch/updates main
deb-src http://mirrors.cloud.tencent.com/debian/ stretch main non-free contrib
deb-src http://mirrors.cloud.tencent.com/debian/ stretch-updates main non-free contrib
deb-src http://mirrors.cloud.tencent.com/debian/ stretch-backports main non-free contrib
```

### 中科大镜像站

```sh
deb https://mirrors.ustc.edu.cn/debian/ stretch main contrib non-free
deb https://mirrors.ustc.edu.cn/debian/ stretch-updates main contrib non-free
deb https://mirrors.ustc.edu.cn/debian/ stretch-backports main contrib non-free
deb https://mirrors.ustc.edu.cn/debian-security/ stretch/updates main contrib non-free

deb-src https://mirrors.ustc.edu.cn/debian/ stretch main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ stretch-updates main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian/ stretch-backports main contrib non-free
deb-src https://mirrors.ustc.edu.cn/debian-security/ stretch/updates main contrib non-free
```

### 阿里云镜像站

```sh
deb http://mirrors.aliyun.com/debian/ stretch main non-free contrib
deb http://mirrors.aliyun.com/debian-security stretch/updates main
deb http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib
deb http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib

deb-src http://mirrors.aliyun.com/debian-security stretch/updates main
deb-src http://mirrors.aliyun.com/debian/ stretch main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ stretch-updates main non-free contrib
deb-src http://mirrors.aliyun.com/debian/ stretch-backports main non-free contrib
```

### 清华大学镜像站

```sh
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch-updates main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch-backports main contrib non-free
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security/ stretch/updates main contrib non-free

deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch-updates main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ stretch-backports main contrib non-free
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security/ stretch/updates main contrib non-free
```

### 兰州大学镜像站

```sh
deb http://mirror.lzu.edu.cn/debian stable main contrib non-free
deb http://mirror.lzu.edu.cn/debian stable-updates main contrib non-free
deb http://mirror.lzu.edu.cn/debian/ stretch-backports main contrib non-free
deb http://mirror.lzu.edu.cn/debian-security/ stretch/updates main contrib non-free

deb-src http://mirror.lzu.edu.cn/debian stable main contrib non-free
deb-src http://mirror.lzu.edu.cn/debian stable-updates main contrib non-free
deb-src http://mirror.lzu.edu.cn/debian/ stretch-backports main contrib non-free
deb-src http://mirror.lzu.edu.cn/debian-security/ stretch/updates main contrib non-free
```

### 上海交大镜像站

```sh
deb https://mirror.sjtu.edu.cn/debian/ stretch main contrib non-free
deb https://mirror.sjtu.edu.cn/debian/ stretch-updates main contrib non-free
deb https://mirror.sjtu.edu.cn/debian/ stretch-backports main contrib non-free
deb https://mirror.sjtu.edu.cn/debian-security/ stretch/updates main contrib non-free

deb-src https://mirror.sjtu.edu.cn/debian/ stretch-updates main contrib non-free
deb-src https://mirror.sjtu.edu.cn/debian/ stretch-backports main contrib non-free
deb-src https://mirror.sjtu.edu.cn/debian/ stretch main contrib non-free
deb-src https://mirror.sjtu.edu.cn/debian-security/ stretch/updates main contrib non-free
```
