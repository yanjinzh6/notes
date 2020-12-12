---
title: Linux 主机名
date: 2020-12-06 11:00:00
tags: 'Linux'
categories:
  - ['使用说明', '操作系统']
permalink: linux-hostname
---

## 简介

主机名是标识网络上的计算机的标签, 在同一网络上的两台不同的主机最好不要设置相同的主机名, 建议使用完全限定的域名 (FQDN) 来作为主机名

<!-- more -->

## FQDN

FQDN (fully qualified domain name, 完全限定域名) 是 internet 上特定计算机或主机的完整域名, FQDN 由两部分组成: 主机名和域名, 例如, 假设邮件服务器的 FQDN 可能是 mail.qq.com, 主机名是 mail, 主机位于域名 qq.com 中

DNS (Domain Name System), 负责将 FQDN 翻译成 IP, 是互联网绝大多数应用的寻址方式

## 查看

```sh
hostnamectl
   Static hostname: CommonWatchful-VM
         Icon name: computer-vm
           Chassis: vm
        Machine ID: xxx
           Boot ID: yyy
    Virtualization: kvm
  Operating System: Debian GNU/Linux 10 (buster)
            Kernel: Linux 4.19.0-6-amd64
      Architecture: x86-64
```

### 修改

```sh
# 设置新的主机名
hostnamectl set-hostname demo.com
# 修改文件
vi /etc/hosts
127.0.0.1   localhost

127.0.0.1   demo.com

# The following lines are desirable for IPv6 capable hosts (对于支持 IPv6 的主机, 以下行是理想的)

::1     localhost ip6-localhost ip6-loopback

ff02::1 ip6-allnodes

ff02::2 ip6-allrouters
```

使用 `hostnamectl` 验证是否成功修改
