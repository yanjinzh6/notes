---
title: ssh 连接出现 REMOTE HOST IDENTIFICATION HAS CHANGED
date: 2020-08-01 10:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: ssh-remote-host-id-has-changed
---

## 简介

终端使用命令时出现

```sh
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:QDDo3lXU8WOoOyx5oVHOeizG6AJTbEYEhPPvtDGPLu4.
Please contact your system administrator.
Add correct host key in /Users/xxx/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /Users/xxx/.ssh/known_hosts:4
ECDSA host key for xxx.xxx.xxx.xxx has changed and you have requested strict checking.
Host key verification failed.
```

这个问题比较常见, 例如以前连接过的 ip 换公钥了, 具体的话更新公钥和换机器换系统都会出现

## 解决

使用命令清理相关 ip

```sh
## 使用现实中真实 ip
ssh-keygen -R xxx.xxx.xxx.xxx
```
