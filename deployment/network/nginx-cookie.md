---
title: nginx-udp-stream
date: 2021-02-01 17:00:00
tags: 'Network'
categories:
  - ['运维', 'Network']
permalink: nginx-udp-stream
photo:
---

https://www.cnblogs.com/kevingrace/p/5707750.html

# [Nginx 防止 cookie 丢失的配置 <nginx proxy\_pass> <proxy\_cookie\_domain>](https://www.cnblogs.com/kevingrace/p/5707750.html)

**一, proxy\_cookie\_path** 参数的作用是用来改变 cookie 的路径
语法: proxy\_cookie\_path path replacement;
path 就是你要替换的路径 replacement 就是要替换的值

**为什么 cookie 会丢失?**
比如说一个没有经过代理的地址 : http://127.0.0.1/project       cookie\_path:/project
如果按照第二种方式代理那么地址就是 : http://127.0.0.1/proxy\_path       cookie\_path: /proxy\_path
如果 cookie\_path 与地址栏上的 path 不相符游览器就不会接受这个 cookie, 自然 session 就失效了.

**解决 nginx proxy\_pass 反向代理 cookie, session 丢失的问题**

下面是可能的三种情况
**1) host, 端口转换, cookie 不会丢失**

```
 location /project {
        proxy\_pass   http://127.0.0.1:8080/project;
    }
```

通过浏览器访问 http://127.0.0.1/project 时, 浏览器的 cookie 内有 jsessionid. 再次访问时, 浏览器会发送当前的 cookie.

**2) 如果路径也变化了, 则需要设置 cookie 的路径转换, nginx.conf 的配置如下**

```
location /proxy\_path {
        proxy\_pass   http://127.0.0.1:8080/project;
    }
```

通过浏览器访问 http://127.0.0.1/proxy\_path 时, 浏览器的 cookie 内没有 jsessionid. 再次访问时, 后台当然无法获取到 cookie 了.
加上路径转换: proxy\_cookie\_path  /project /proxy\_path; 则可以将 project 的 cookie 输出到 proxy\_path 上.
保证 cookie 不丢失的正确配置是:

```
 location /proxy\_path {
        proxy\_pass   http://127.0.0.1:8080/project;
        proxy\_cookie\_path  /project /proxy\_path;
    }
```

**3) 直接代理本地端口**

```
    location /proxy\_path {
        proxy\_pass   http://127.0.0.1:8080/;
        proxy\_cookie\_path  /project /proxy\_path; # project 为你的项目名 也可用变量代替
    }

```

**二, proxy\_cookie\_domain** 参数的作用是转换 response 的 set-cookie header 中的 domain 选项, 由后端设置的域名 domain 转换成你的域名 replacement, 来保证 cookie 的顺利传递并写入到当前页面中, 注意**proxy\_cookie\_domain 负责的只是处理 response set-cookie 头中的 domain 属性, 仅此而已!**

在了解了这个参数后, 发现不配置这个属性, 依然运转正常!

response 在写 set-cookie 的时候, domain 是一个可选项, 并不是必填项, 所以经常能看到如下这种情况:

! [](https://img2018.cnblogs.com/blog/907596/201907/907596-20190712181616354-2133081154.png)

这个时候由于 set-cookie 本身就没有 domain 内容, proxy\_cookie\_domain 也就不没有必要了, 这也是为什么在部分项目中不配置 proxy\_cookie\_domain 依然正常的原因. 但是对于一些设置了 domain 的项目, 比如:

! [](https://img2018.cnblogs.com/blog/907596/201907/907596-20190712181646310-1499523964.png)

这种情况下当你用 nginx 做反向代理的时候, 就必须要转换一下了.

\===================== **Nginx 反向代理理解误区之 proxy\_cookie\_domain \======================**
**之前错误地理解**:**proxy\_cookie\_domain 的作用是实现前后端 cookie 域名转换, 保证顺利传递!**比如说 Nginx 做反向代理的时候, 一般都习惯添加 proxy\_cookie\_domain 配置, 来做 cookie 的域名转换. 比如:

```
...
location /api {
   proxy\_pass https://b.test.com;
   proxy\_cookie\_domain b.test.com  a.test.com;
}
...

```

后面发现, 不配置这个属性, 依然运转正常! 这就是对 proxy\_cookie\_domain 错误理解导致地!! 乍一看好像也没错, 但是现在想想, 理解还是不够啊, 因为 proxy\_cookie\_domain 的作用是单向的, 并不是双向转换的. 我们先看下 cookie 的传递过程:

! [](https://img2018.cnblogs.com/blog/907596/201907/907596-20190712182500985-1665584566.png)

浏览器在发送请求的时候, 会在 request header 中带上 cookie 项 (有内容的话), 此时的 cookie 是一个字符串, 一个 key= value 并用分号分割的字符串,

! [](https://img2018.cnblogs.com/blog/907596/201907/907596-20190712182538591-522286072.png)

其中**并不包含任何域名信息**. 这是因为浏览器在设置 cookie 选项的时候, 所选取的内容都是缓存中接口域名下的. 然后 request 的只要请求发送出去之后, cookie 中有关 domain 信息其实是不存在的, 它只是一个普通的字符串, 随便 proxy\_pass 到任何位置, 都会正常携带下去. 因此**在前端到后端的 request 的过程中, proxy\_cookie\_domain 是没用的!!**

而 server 端在做响应的时候, 通过 set-cookie 的 domain 属性, 可以控制 cookie 的生效域名目标, 做到诸如二级域名 cookie 分离等等, 如果前端接收到的 set-cookie 的 domain 和当前域名不一致, 或者一级域名不一致 (二级域名可以共享一级域名下的 cookie), 这个 cookie 在后续的通信中就是无效的, 所以这里才需要去做 domain 的转换, 也就是说**response 中 set-cookie 的 domain 转换才是有意义的**, 这也正是 proxy\_cookie\_domain 的作用所在.
当 response 的 set-cookie 中 domain 不去设置时, cookie 顺利传入浏览器中, 浏览器会自动设置这个 cookie 的生效域名为当前域名.

和这个类似的还有**proxy\_cookie\_path 属性, 同样的该属性仅作用在修改 response set-cookie 的 path 属性**, 而一般情况下, 用的也比较少.

下面是 Nginx 里关于 proxy\_cookie\_domain 的一个配置:

```
页面地址是 a.com, 但是要用 b.com 的 cookie 需要

proxy\_set\_header Cookie $http\_cookie;
location / {
proxy\_cookie\_domain b.com a.com;            #注意别写错位置了 proxy\_cookie\_path / /;
proxy\_pass http://b.com;
 }
```

posted @ 2016-07-26 15:47  [散尽浮华](https://www.cnblogs.com/kevingrace/)  阅读 (12597)  评论 (0)  [编辑](https://i.cnblogs.com/EditPosts.aspx? postid=5707750)  [收藏](javascript: void(0))
