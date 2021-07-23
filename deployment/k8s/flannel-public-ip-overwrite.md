---
title: flannel-public-ip-overwrite
date: 2021-03-21 18:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: flannel-public-ip-overwrite
photo:
---

通过配置 public-ip-overwrite 注解来指定 flannel 通信 ip

```sh
kubectl annotate nodes tx-gz-a.atjog.com flannel.alpha.coreos.com/public-ip-overwrite=xxx.xxx.xxx.xxx
```

重启后好像会丢失该注解
