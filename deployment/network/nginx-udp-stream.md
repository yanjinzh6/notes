---
title: nginx-udp-stream
date: 2021-02-01 16:00:00
tags: 'Network'
categories:
  - ['运维', 'Network']
permalink: nginx-udp-stream
photo:
---

https://juejin.cn/post/6844903782363430926

[! [](data: image/png; base64, iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAAXNSR0IArs4c6QAAAERlWElmTU0AKgAAAAgAAYdpAAQAAAABAAAAGgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAPKADAAQAAAABAAAAPAAAAACL3+ lcAAAAV0lEQVRoBe3QMQEAAADCoPVP7WkJiEBhwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGPjAADh8AAFUUgPVAAAAAElFTkSuQmCC)](/user/3210229683594583)

2019 年 02 月 25 日 阅读 2239

关注

# 通过 nginx 进行 udp 报文负载均衡

### 1\. 安装 nginx

#### 1.1 通过 yum 安装

```
[root@yaoxiang ~]# yum install nginx
复制代码
```

#### 1.2 查看 nginx 的版本

```
[root@yaoxiang ~]# nginx -v
nginx version: nginx/1.12.2
复制代码
```

nginx 的版本必须高于 1.9.0, 因为从 1.9 开始 nginx 就支持对 TCP 的转发, 而到了 1.9.13 时, UDP 转发也支持了.

#### 1.3 查看默认编译参数

```
[root@yaoxiang ~]# nginx -V
nginx version: nginx/1.12.2
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-16) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/share/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib64/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --http-client-body-temp-path=/var/lib/nginx/tmp/client_body --http-proxy-temp-path=/var/lib/nginx/tmp/proxy --http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi --http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi --http-scgi-temp-path=/var/lib/nginx/tmp/scgi --pid-path=/run/nginx.pid --lock-path=/run/lock/subsys/nginx --user= nginx --group= nginx --with-file-aio --with-ipv6 --with-http_auth_request_module --with-http_ssl_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_xslt_module= dynamic --with-http_image_filter_module= dynamic --with-http_geoip_module= dynamic --with-http_sub_module --with-http_dav_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_random_index_module --with-http_secure_link_module --with-http_degradation_module --with-http_slice_module --with-http_stub_status_module --with-http_perl_module= dynamic --with-mail= dynamic --with-mail_ssl_module --with-pcre --with-pcre-jit --with-stream= dynamic --with-stream_ssl_module --with-google_perftools_module --with-debug --with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param= ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune= generic' --with-ld-opt='-Wl,-z, relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'
复制代码
```

nginx 实现 TCP/UDP 的转发依靠的是**Stream**模块, 查看默认编译参数中是含有 \*\*--with-stream**参数, 可以看到上面的参数里含有**\--with-stream= dynamic\*\*, 说明已动态加载 Stream 模块

### 2\. 修改 nginx 配置

#### 2.1 增加 stream 块

```
[root@yaoxiang ~]# vim /etc/nginx/nginx.conf
...
events {
    worker_connections 1024;
}
stream {
#定义被代理的服务器组 upstream 组名
upstream dns {
 # zone   dns 64k;
  server 172.27.9.204:4005;#代理的服务器地址
  server 172.27.9.204:4006 weight=5;#weight 权重, 默认都是 1
  server 172.27.9.204:4007 max_fails=3 fail_timeout=30s;#max_fails - NGINX 将服务器标记为不可用的连续失败尝试次数.fail_timeout 为 NGINX 认为服务器不可用的时间长度.
  server 172.27.9.204:4008 backup;#backup 备用服务器, 其他服务器不可用时, 才会收到转发
}

server {
  listen 4000 udp;#监听 udp4000 端口, 如不加 udp 则默认监听 tcp

  proxy_responses 1;#1 代表需要回应, 并将回应转发;0 代表不需要回应

  proxy_timeout 20s;#回应超时时间, 超时未回应暂停转发

  proxy_pass dns;#代理服务器, 服务器组

  proxy_buffer_size 512k;#响应缓冲区大小, 如果后端发送的响应头过大可以尝试增加此缓冲.

}

}
http {
...}
复制代码
```

#### 2.2 负载均衡策略

*   round-robin(轮询)——默认, Nginx 使用轮询算法负载均衡通信. 因为是默认方法, 所以没有 round-robin 指令; 只创建 upstream 配置块在顶级 stream 上下文并像之前步骤添加 server 指令.

*   last\_conn(最少连接)——Nginx 选择当前活跃连接数较少的服务器.

*   least\_tome——Nginx 选择最低平均延迟和最少活跃连接的服务器. 最低活跃连接基于 least\_time 指令的以下参数计算:

    *   connect——连接 upstream 服务器的时间

    *   first\_byte——接收第一个数据字节的时间

    *   last\_byte——从服务器接收完整响应的时间

*   hash——Nginx 基于用户定义键选择服务器, 例如, 源 IP 地址


#####例子: 采用最少连接和 hash 负载均衡策略

```
upstream dns {
 last_conn;#最少连接策略
 server 172.27.9.204:4005;#代理的服务器地址
 server 172.27.9.204:4006;
 server 172.27.9.204:4007;
 server 172.27.9.204:4008;
 }
复制代码
``````
upstream stream_backend {

    hash $remote_addr;

    server backend1.example.com:12345;

    server backend2.example.com:12345;

    server backend3.example.com:12346;

}
复制代码
```

### 3. 启动 nginx

```
[root@yaoxiang nginx]# sudo nginx
nginx: [emerg] bind() to 0.0.0.0:80 failed (98: Address already in use)
复制代码
```

出现以上提示说明端口被占用, 修改 nginx.conf 里 http 模块中的端口号

### 4\. 测试

#### 4.1 向 nginx 主机发送报文

#### 4.2 可以看到代理服务器组主机的不同端口都接收到了

**4006 端口**

**4005 端口**

#### 4.3 接收端代理服务器向 nginx 主机回复消息

#### 4.4 可以看到发送端收到了回复
