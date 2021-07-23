---
title: go-get-auto-proxy
date: 2021-03-14 11:00:00
tags: 'golang'
categories:
  - ['开发', 'golang']
permalink: go-get-auto-proxy
photo:
---

[go-get-auto-proxy](https://segmentfault.com/a/1190000018610099)

原文链接: [https://blog.thinkeridea.com/201903/go/go\_get\_proxy.html](https://blog.thinkeridea.com/201903/go/go_get_proxy.html)

最近发现技术交流群里很多人在询问 go get 墙外包失败的问题, 大家给了很多解决方案:

*   从 Github 的代码库 clone
*   设置 GOPROXY 环境变量配置代理, 例如: GOPROXY= [https://goproxy.io](https://goproxy.io)
*   配置命令行代理, https\_proxy 环境变量
*   使用 go mod replace
*   使用 Gopm 类似的工具
*   ……

Go 的社区很活跃, 国内 gopher 对 Go 的热情不会因为墙的存在而减少, 从社区想到这么多翻墙方案就能看出来了.

上面的方法都是可行的, 但是总有一些不尽人意, 社区也一直在找更好的方法, 我一直使用自动代理的方式获取墙外的包, 可以支持所有 Go 原生拉取包操作, 比如 go get, go mod, dep, godep, glide 等各种方法, 只需要配置一次, 只要在任何原生命令前加前缀运行命令即可, 效率很高.

<!--more-->

## 实战操作

工具类就先不讲原理, 想直接获取方法的同学看这一部分即可, 想了解原理的同学可以看后面的 [原理部分](#%E5%8E%9F%E7%90%86%E7%AF%87).

#### 你需要准备如下工具:

*   一个 http 代理
*   Git
*   Github 账号设置好 ssh
*   其它 git 相关服务设置好 ssh (例如自建 gitlab)
*   一个可以运行 shell 的环境 (linux, Mac, windows 可以使用 git bash)

#### 具体步骤

*   首先通过 git 设置需要不代理的网站, 以 Github 为例, 执行 `git config --global url.git@github.com:.insteadof https://github.com/` 从 https 转到 ssh 协议, 这样会使我们设置的 https 代理不作用在 ssh 协议上, 如果有自建的服务只要更换地址就可以了.
*   新建一个脚本 (proxy), 修改里面的代理地址为自己的代理地址, 如下:

    ```
    #!/usr/bin/env bash
    # 如果你的系统没有 bash, 或者没有 /usr/bin/env , 请修改上一行指令为你的环境
    export http_proxy= http://127.0.0.1:1087  # 代理地址需要换成自己的
    export https_proxy= http://127.0.0.1:1087
    export ftp_proxy= http://127.0.0.1:1087

    exec ${@:1}
    ```
*   给 shell 脚本设置可执行权限, 然后放到 path 环境变量路径下.
*   测试 `proxy curl https://www.google.com` 和 `curl https://www.google.com` 第一个命令可以获取到结果, 第二个命令不可以.
*   测试 `proxy go get -v golang.org/x/text/…` 可以正常下载包, 其它任何拉取包命令都可以添加 `proxy` 前缀执行 , 比如 `proxy dep ensure -v`

截止当前你就配置了一个 `go get` 自动代理的环境, 以后需要翻墙操作的指令运行时加 `proxy` 就可以了, 该方法并不只适用于 `go get`, 任何需要命令行代理都可以使用.

## 原理篇

实际原理简单, 找到这种方法也是一种巧合, 在入坑 Go 之前我经常用 linux, 当时有一些需求需要命令行翻墙, 找到了三个环境变量 `http_proxy`,`https_proxy`,`ftp_proxy`, 但是全局设置导致很多请求变慢, 如果在一个窗口临时设置就导致需要记住那个窗口设置了代理, 切换窗口成本也比较高, 后来根据 shell 的特性, 任何一个脚本都有自己独立的环境变量, 所以用一个脚本设置代理环境变量,`exec ${@:1}` 可以执行脚本后面的指令, 也就是我们实际需要运行的指令, 这样在需要代理的命令前就加上这个脚本前缀就好了, 单行命令代理就这么简单的配置好了.

前期我使用 go 的时候遇到下载不了的包时, 就会在 `go get` 前加上 `proxy` 指令, 但是我发现拉取 Github 包的效率非常低, 本身国内现在访问 Github 已经很快了. 也是一个巧合, 当时我公司 Go 项目迁移到 Github 上, 所有项目全部是私有项目, 有同事提供了一个 git https 转 ssh 协议的操作,`git config --global url.git@….:.insteadof https://…./`, 这个操作让我看到一个隐性福利, 之前的代理只会代理 https 并不能代理 ssh 协议, 那么使用这个指令把不需要代理的网站全部转成 ssh 协议, 然后加上 `proxy` 运行 `go get` 就成了自动代理了, 尝试之后确实效率很高, 至此一直使用到今天.

**转载:**

**本文作者: 戚银 ([thinkeridea](https://blog.thinkeridea.com/))**

**本文链接: [https://blog.thinkeridea.com/201903/go/go\_get\_proxy.html](https://blog.thinkeridea.com/201903/go/go_get_proxy.html)**

**版权声明: 本博客所有文章除特别声明外, 均采用 [CC BY 4.0 CN 协议](http://creativecommons.org/licenses/by/4.0/deed.zh) 许可协议. 转载请注明出处!**
