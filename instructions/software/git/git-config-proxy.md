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







[! [](data: image/png; base64, iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAAXNSR0IArs4c6QAAAERlWElmTU0AKgAAAAgAAYdpAAQAAAABAAAAGgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAPKADAAQAAAABAAAAPAAAAACL3+ lcAAAAV0lEQVRoBe3QMQEAAADCoPVP7WkJiEBhwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGPjAADh8AAFUUgPVAAAAAElFTkSuQmCC)](/user/606586150591752)

2020 年 02 月 02 日 阅读 461

关注

# 配置 git 代理

配置 git 代理

git 代理一般会分为两种, 一种是走 http 协议的代理, 另一种是走 ssh 协议的代理.

对于走 http 协议的代理, 我们只需要执行如下命令 (端口号根据本地 ss 配置自行修改):

```
git config --global http.proxy http://127.0.0.1:1088
git config --global https.proxy http://127.0.0.1:1088
复制代码
```

如果要取消代理, 则执行如下命令

```
git config --global --unset http.proxy
git config --global --unset https.proxy
复制代码
```

加上如上配置后, 对于公司内网的 git 是无法进行访问的, 所以还得把内网的 git 地址加入到 no\_proxy 环境变量中

```
export no_proxy="localhost,127.0.0.1, 内网 git 域名 "
复制代码
```

除了 http 协议代理, git 可能还会走 ssh 协议, 对于 ssh 协议的配置其实也是类似

编辑 ssh config 文件

```
vim ~/.ssh/config
复制代码
```

加入如下内容 (端口号根据本地 ss 配置自行修改)

```
Host github.com
   HostName github.com
   User git
   # 走 HTTP 代理
   # ProxyCommand socat - PROXY:127.0.0.1:%h:%p, proxyport=1088
   # 走 socks5 代理 (如 ss)
   ProxyCommand nc -v -x 127.0.0.1:1086 %h %p
   # HTTPS 使用 -H, SOCKS 使用 -S
   # ProxyCommand connect -S 127.0.0.1:10808 %h %p
复制代码
```

注意此处使用的是 socks5 代理, 而 http/https 协议配置的是 http 代理, 至此就完成了 git 的代理配置

配置终端 http/https 代理

除了 git 需要配置代理之外, 其实还需要配置终端 http/https 代理, 因为源码下载过程中除了用 git 去下载源码外, 还会借用 python 去下载一些 zip 文件, 以及使用 cipd 下载一些文件, 使用这些工具的时候, 在终端中可能无法正常完成对应文件的下载.

因为这部分代理使用场景不多, 全局共两处

src/tools/dart/update.py src/tools/buildtools/update.py 为了尽可能的不污染全局环境, 我们进行临时的环境变量导出 (端口号根据本地 ss 配置自行修改).

```
export http_proxy= http://127.0.0.1:1088
export https_proxy= http://127.0.0.1:1088
复制代码
```

完成以上配置后, 源码同步过程会变得十分顺畅, 而不用进行漫长的等待.

https://juejin.cn/post/6844904055672668168
