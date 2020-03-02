---
title: Git 设置代理
date: 2020-03-02 15:00:00
tags: 'Git'
categories:
  - ['使用说明', '软件']
permalink: git-config-proxy
photo:
---

```sh
# 设置 postBuffer
git config http.postBuffer 524288000
# 设置全局代理
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
# 取消全局代理
git config --global --unset http.proxy
git config --global --unset https.proxy
# 只对 github.com 设置代理
git config --global http.https://github.com.proxy socks5://127.0.0.1:1080
# 取消代理
git config --global --unset http.https://github.com.proxy)

npm config delete proxy
```

git 项目配置文件设置代理, 适用于独立项目

```
[http]
    proxy = socks5://127.0.0.1:1080
[https]
    proxy = socks5://127.0.0.1:1080
```