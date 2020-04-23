---
title: SS 部署配置
date: 2020-02-05 17:30:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: ss-config-deploy
photo:
---

简单的客户端配置, 服务端配置同理

```yml
apiVersion: v1
kind: Pod
metadata:
  name: ss-client
  labels:
    app: ss-client
    version: v1
spec:
  hostNetwork: true
  containers:
    - name: shadowsocks
      image: mritd/shadowsocks
      imagePullPolicy: IfNotPresent
      ports:
        - containerPort: 1080
          name: ss-port
          protocol: TCP
      args:
        [
          '-m',
          'ss-local',
          '-s',
          '-s 127.0.0.1 -p 6500 -b 0.0.0.0 -l 1080 -m chacha20-ietf-poly1305 -k xxx',
          '-x',
          '-e',
          'kcpclient',
          '-k',
          '-r YOUR_IP:PORT -l :6500 --mode fast2 --key xxx --rcvwnd 1024',
        ]
```
