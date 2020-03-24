---
title: Squid 代理使用
date: 2020-03-24 10:00:00
tags: 'Hexo'
categories:
  - ['使用说明', '软件']
permalink: squid-introduction
---

# 简介

[Squid](http://www.squid-cache.org/) 是一个 Web 的缓存代理服务器, 支持 HTTP, HTTPS, FTP 等, 通过缓存和重用经常请求的网页, 可以减少带宽并缩短响应时间.

# 安装

最简单的使用[自定义镜像](https://hub.docker.com/r/sameersbn/squid)安装

```sh
# 拉取镜像
docker pull sameersbn/squid

# 运行自定义配置文件
docker run --name squid -d --restart=always \
  --publish 3128:3128 \
  --volume /data/squid/cache:/var/spool/squid \
  --volume /data/squid/etc/squid.conf:/etc/squid/squid.conf \
  --volume /data/squid/etc/resolv.conf:/etc/resolv.conf \
  sameersbn/squid:3.5.27-2

# 设置代理
export ftp_proxy=http://172.17.0.1:3128
export http_proxy=http://172.17.0.1:3128
export https_proxy=http://172.17.0.1:3128

# 简单查看访问日志
docker exec -it squid cat /var/log/squid/access.log
```

<!-- more -->

# 配置

打开 squid 自带的配置文件, 可以感受到项目开发组已经把配置文件当成使用说明书了. 将近 8k 行的配置文件里面有 99% 的说明. 这里有一个中文的[在线文档](https://wiki.ubuntu.org.cn/Squid)可以参考

容器安装启动后, 已经将配置文件复制出来并外挂上了, 现在可以进行配置

## 简单使用

```conf
# Example rule allowing access from your local networks.
# Adapt localnet in the ACL section to list your (internal) IP networks
# from where browsing should be allowed
#http_access allow localnet
http_access allow localhost

# And finally deny all other access to this proxy
# http_access deny all
# 使用的是容器配置, 所以需要配置能从宿主或者其他容器访问的权限, 最方便就是 allow all
# 解决 TCP_DENIED/403 4037 GET http://xxx - HIER_NONE/- text/html 的问题
http_access allow all
```

```sh
# 测试
curl -x 127.0.0.1:8081 http://httpbin.org/get
```

## 权限控制

如果 squid 容器需要开放外网连接的话, 那么不能单纯的 `http_access allow all`, 很容易导致了被其他机器调用, 需要修改一下默认接口并开启验证

```sh
# 生成 basic auth
htpasswd /data/squid/etc/passwd squid
New password:
Re-type new password:
Adding password for user squid
# 将 /data/squid/etc/passwd:/etc/squid/passwd 映射到容器中
```

修改配置文件

```conf
# Squid normally listens to port 3128
http_port 8123

# 添加认证流程
# 允许的客户端 ip
acl clients src 0.0.0.0/0.0.0.0
# 指定密码文件和用来验证密码的程序, 使用生成到 passwd 文件
auth_param basic program /usr/lib/squid3/basic_ncsa_auth /etc/squid/passwd
# 鉴权进程的数量
# auth_param basic children 5 startup=5 idle=1
# 用户输入用户名密码时看到的提示信息
# auth_param basic realm Squid proxy-caching web server
# 用户名和密码的缓存时间, 也就是说同一个用户名多久会调用 ncsa_auth 一次
# auth_param basic credentialsttl 2 hours
# 所有成功鉴权的用户都归于 authenticated 组
acl authenticated proxy_auth REQUIRED
# 允许 authenticated 和 clients 组的用户使用 Proxy
http_access allow authenticated clients

# And finally deny all other access to this proxy
http_access deny all
```

## 代理设置

代理分为透明代理、匿名代理、混淆代理、高匿代理, 这4种代理, 主要是在代理服务器端的配置不同, 导致其向目标地址发送请求时, REMOTE_ADDR,  HTTP_VIA, HTTP_X_FORWARDED_FOR三个变量不同.

- 透明代理(Transparent Proxy)

```
REMOTE_ADDR = Proxy IP
HTTP_VIA = Proxy IP
HTTP_X_FORWARDED_FOR = Your IP
```

透明代理虽然可以直接“隐藏”你的IP地址, 但是还是可以从HTTP_X_FORWARDED_FOR来查到你是谁.

- 匿名代理(Anonymous Proxy)

```
REMOTE_ADDR = proxy IP
HTTP_VIA = proxy IP
HTTP_X_FORWARDED_FOR = proxy IP
```

匿名代理比透明代理进步了一点：别人只能知道你用了代理, 无法知道你是谁.

- 混淆代理(Distorting Proxies)

```
REMOTE_ADDR = Proxy IP
HTTP_VIA = Proxy IP
HTTP_X_FORWARDED_FOR = Random IP address
```

如上, 与匿名代理相同, 如果使用了混淆代理, 别人还是能知道你在用代理, 但是会得到一个假的IP地址, 伪装的更逼真

- 高匿代理(Elite proxy或High Anonymity Proxy)

```
REMOTE_ADDR = Proxy IP
HTTP_VIA = not determined
HTTP_X_FORWARDED_FOR = not determined
```

可以看出来, 高匿代理让别人根本无法发现你是在用代理, 所以是最好的选择.

```conf
request_header_access X-Forwarded-For deny all
request_header_access From deny all
request_header_access Via deny all
```

也可以自定义允许的请求头

```conf
forwarded_for off
request_header_access Allow allow all
request_header_access Authorization allow all
request_header_access WWW-Authenticate allow all
request_header_access Proxy-Authorization allow all
request_header_access Proxy-Authenticate allow all
request_header_access Cache-Control allow all
request_header_access Content-Encoding allow all
request_header_access Content-Length allow all
request_header_access Content-Type allow all
request_header_access Date allow all
request_header_access Expires allow all
request_header_access Host allow all
request_header_access If-Modified-Since allow all
request_header_access Last-Modified allow all
request_header_access Location allow all
request_header_access Pragma allow all
request_header_access Accept allow all
request_header_access Accept-Charset allow all
request_header_access Accept-Encoding allow all
request_header_access Accept-Language allow all
request_header_access Content-Language allow all
request_header_access Mime-Version allow all
request_header_access Retry-After allow all
request_header_access Title allow all
request_header_access Connection allow all
request_header_access Proxy-Connection allow all
request_header_access User-Agent allow all
request_header_access Cookie allow all
request_header_access All deny all
```

# 引用

- [搭建需要身份认证的 Squid 代理](https://maoxian.de/2016/06/1415.html)
- [Ubuntu 下搭建高匿 HTTP 代理](https://www.cnblogs.com/dadonggg/p/10019026.html)