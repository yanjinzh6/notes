---
title: kubelet-node-ip
date: 2021-03-21 17:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubelet-node-ip
photo:
---

https://github.com/geerlingguy/ansible-role-kubernetes/issues/15

kubelet 通过 KUBELET_EXTRA_ARGS 可以自定义参数, 配置文件为 `/etc/default/kubelet`

```conf
# This is a file that the user can use for overrides of the kubelet args as a last resort. Preferably, the user should use
# the .NodeRegistration.KubeletExtraArgs object in the configuration files instead. KUBELET_EXTRA_ARGS should be sourced from this file.
EnvironmentFile=-/etc/default/kubelet
```

通过配置 `KUBELET_EXTRA_ARGS=--node-ip=$PRIVATE_IP` 指定 node 注册到集群的 ip 地址

```sh
PRIVATE_IP=10.99.0.0
echo "KUBELET_EXTRA_ARGS=--node-ip=$PRIVATE_IP" > /etc/default/kubelet
systemctl daemon-reload
systemctl restart kubelet
```
