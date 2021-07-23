---
title: go-get-proxy
date: 2021-03-14 12:00:00
tags: 'golang'
categories:
  - ['开发', 'golang']
permalink: go-get-proxy
photo:
---

https://www.jianshu.com/p/bb16ea7afdda

# Go get 使用代理

在 vscode 中使用 golang 时, 经常会出现安装第三方工具的时候失败的问题, 一般来说都是下载了 `golang.org/x/...` 下面的包或者要下载的工具依赖于 `golang.org/x/...` 的包所导致的, 在国内是不会很顺利的下载和安装的.

```
Installing github.com/mdempsky/gocode FAILED
Installing github.com/uudashr/gopkgs/cmd/gopkgs FAILED
Installing github.com/ramya-rao-a/go-outline FAILED
Installing github.com/acroca/go-symbols FAILED
Installing golang.org/x/tools/cmd/guru FAILED
Installing golang.org/x/tools/cmd/gorename FAILED
Installing github.com/go-delve/delve/cmd/dlv FAILED
Installing github.com/rogpeppe/godef FAILED
Installing golang.org/x/tools/cmd/goimports FAILED
Installing golang.org/x/lint/golint FAILED
```

go get 默认使用 git 来作为版本管理工具, 如果对 git 设置了代理能否下载成功呢?

```
git config --global http.proxy 'http://127.0.0.1:8123'
git config --global https.proxy 'http://127.0.0.1:8123'
```

很遗憾, 即使你对 git 设置了代理, 也无法成功的下载安装 `golang.org` 下的包, 假如你要安装 `golang.org/x/tools/cmd/guru` 这个包, 当你使用 go get 去下载安装包时, 首先会访问 `https://golang.org/x/tools/cmd/guru? go-get=1` 这个 url 来获取版本库的类型, 是 git 还是 svn 或者其他的版本管理工具, 这个 http 请求是 git 之外的, 所以这有对这个 http 请求设置了代理, 然后对 git 设置代理, 才能成功的下载安装该库.

```
# 在终端中执行
export https_proxy= http://127.0.0.1:8123
export http_proxy= http://127.0.0.1:8123
git config --global http.proxy 'http://127.0.0.1:8123'
git config --global https.proxy 'http://127.0.0.1:8123'
```

前提是你拥有一个 http 或者 socks 代理, 如果没有的话, 还是老老实实的在 `github.com/golang/...` 上下载对应的包然后进行替换吧.

设置了代理之后, 再次执行 go get 命令:

```
 go get -v github.com/acroca/go-symbols

golang.org/x/tools/go/buildutil
# golang.org/x/tools/go/buildutil
Workbench/golang/src/golang.org/x/tools/go/buildutil/buildutil_go16.go:14:38: undefined: build.AllowVendor
```

这次又出错了, 原因应该是 `golang.org/x/tools/go/buildutil` 包太老了, 直接更新相关的依赖包即可:

```
go get -v -u github.com/acroca/go-symbols

github.com/acroca/go-symbols (download)
Fetching https://golang.org/x/tools/go/buildutil? go-get=1
Parsing meta tags from https://golang.org/x/tools/go/buildutil? go-get=1 (status code 200)
get "golang.org/x/tools/go/buildutil": found meta tag get.metaImport{Prefix:"golang.org/x/tools", VCS:"git", RepoRoot:"https://go.googlesource.com/tools"} at https://golang.org/x/tools/go/buildutil? go-get=1
get "golang.org/x/tools/go/buildutil": verifying non-authoritative meta tag
Fetching https://golang.org/x/tools? go-get=1
Parsing meta tags from https://golang.org/x/tools? go-get=1 (status code 200)
package golang.org/x/tools/go/buildutil: golang.org/x/tools is a custom import path for https://go.googlesource.com/tools, but /home/lu/Workbench/golang/src/golang.org/x/tools is checked out from https://github.com/golang/tools
```

这次就成功更新并安装了, 重启 vscode, 各项功能也正常了.

### 推荐阅读 [更多精彩内容](/)

*   [GO 语言静态代码测试 --- 应用于区块链构建性测试](/p/aa44d4a822d0)

    背景 随着区块链的这 2 年的快速发展, Go 语言和针对 GO 语言测试工具也越来越完善, 特别是 Go 语言的静态代码扫描工具完...

    [老余 2017](/u/c8860961c775) 阅读 8,978 评论 1 赞 17

    [](/p/aa44d4a822d0)
*   [Ubuntu18.04 系统下构建初 (True) 链基础环境](/p/212947a09e2b)

    本文将基于前文 Windows 操作系统用 VMware 虚拟机安装 Ubuntu 系统的基础上, 详细描述包括 GO 语言配置,...

    [ShawnXZhang](/u/4d65055f3f39) 阅读 268 评论 0 赞 0

    [](/p/212947a09e2b)
*   [学习笔记 6](/p/e0a4f6124fa0)

    1\. 分布式系统核心问题 参考书籍: <区块链原理, 设计与应用> 一致性问题例子: 两个不同的电影院买同一种电影票, 如...

    [molscar](/u/6c2dd37355f3) 阅读 440 评论 0 赞 0

    [](/p/e0a4f6124fa0)
*   [vscode 安装 golang 开发环境](/p/db25a9378422)

    \[TOC\] 注意: 本文档默认已经安装好 go 语言环境且已经设置好相应环境变量 1. 安装 vscode 根据自身系统下...

    [木鸟飞鱼](/u/ebb726bcf17a) 阅读 9,280 评论 0 赞 3

    [](/p/db25a9378422)
*   [IntelliJ IDEA 搭建 Go 开发环境](/p/8c59598d597c)

    本文介绍 Windows7 x64 基于 IntelliJ IDEA 搭建 Go 语言开发环境. 主要是一些操作过程...

    [custa](/u/83741898c110) 阅读 7,987 评论 0 赞 1

https://www.cnblogs.com/sparkdev/p/10649159.html

# [Golang 入门 : 配置代理](https://www.cnblogs.com/sparkdev/p/10649159.html)

由于一些客观原因的存在, 我们开发 Golang 项目的过程总会碰到无法下载某些依赖包的问题. 这不是一个小问题, 因为你的工作会被打断, 即便你使用各种神通解决了问题, 很可能这时你的线程已经切换到其他的事情上了 (痛恨思路被打断!). 所以最好是一开始我们就重视这个问题, 并一劳永逸的解决它.

# 问题描述

当我们使用 go get, go install, go mod 等命令时, 类似于 golang.org/x/... 的包会是无法下载的. 比如通过下面的命令下载 sys 包:

```
$ go get -u golang.org/x/sys
```

下载肯定会失败:

! [](https://img2018.cnblogs.com/blog/952033/201904/952033-20190403152142826-1686706814.png)

# 设置代理

如果你有自己的代理服务器, 那就很容易了, 这也是一劳永逸的方法. 直接设置环境变量 http\_proxy 和 https\_proxy 就行了:

```
export http\_proxy= http://proxyAddress: port
export https\_proxy= http://proxyAddress: port
```

比如笔者在局域网中共享了代理 192.168.21.1:1080:

```
$ export http\_proxy= http://192.168.21.1:1080
$ export https\_proxy= http://192.168.21.1:1080
```

然后执行下面的命令后就能够下载了:

```
$ go get -u golang.org/x/sys
```

# 手动下载并安装包

如果一时找不到合适的代理, 还可以临时通过手动的方式下载包. 我们常见的 golang.org/x/... 包, 一般在 GitHub 上都有官方的镜像仓库对应. 比如 [zieckey/golang.org](https://github.com/zieckey/golang.org) 就是作为 golang.org/x 的镜像库存在的. 我们可以手动下载或 clone 对应的 GitHub 仓库到指定的目录下, 比如从 [zieckey/golang.org](https://github.com/zieckey/golang.org) 下载 x 目录下的所有包. 或者是每次只下载单个的库, 下面的示例演示如何下载 text 库:

```
$ mkdir $GOPATH/src/golang.org/x
$ cd $GOPATH/src/golang.org/x
$ git clone git@github.com: golang/text.git
```

当如果需要指定版本的时候, 该方法就无解了, 因为 GitHub 上的镜像仓库多数都没有 tag.

# 使用 go mod replace

从 Go 1.11 版本开始, 新增支持了 go modules 用于解决包依赖管理问题. 该工具提供了 replace, 就是为了解决包的别名问题, 也能替我们解决 golang.org/x 无法下载的的问题.

go module 被集成到原生的 go mod 命令中, 但是如果你的代码库在 $GOPATH 中, module 功能是默认不会开启的, 想要开启也非常简单, 通过一个环境变量即可开启 export GO111MODULE= on. 比如下面的示例, 在 go.mod 中指定下面的代码:

```
module example.com/demo

require (
    golang.org/x/text v0.3.0 )

replace (
    golang.org/x/text => github.com/golang/text v0.3.0 )
```

# 使用 GOPROXY 环境变量

从 Go 1.11 版本开始, 官方支持了 go module 包依赖管理工具. 其实还新增了 GOPROXY 环境变量. 如果设置了该变量, 下载源代码时将会通过这个环境变量设置的代理地址, 而不再是以前的直接从代码库下载. 比如我们可以指定自己的代理地址.

更可喜的是, goproxy.io 这个开源项目帮我们实现好了我们想要的. 该项目允许开发者一键构建自己的 GOPROXY 代理服务. 同时, 也提供了公用的代理服务 [https://goproxy.io](https://goproxy.io/), 我们只需设置该环境变量即可正常下载被墙的源码包了:

```
export GO111MODULE= on
export GOPROXY\= https://goproxy.io
```

也可以通过置空这个环境变量来关闭, export GOPROXY=.

对于 Windows 用户, 可以在 PowerShell 中设置:

```
$env: GOPROXY = "https://goproxy.io"
```

最后, 我们当然推荐使用 GOPROXY 这个环境变量的解决方式, 前提是 Go version >= 1.11.

下载 golang.org/x/sys 包:

```
$ go get -u golang.org/x/sys
```

! [](https://img2018.cnblogs.com/blog/952033/201904/952033-20190403152625747-2021837763.png)

下载到的包在 $GPATH/pkg/mod/golang.org/x/

! [](https://img2018.cnblogs.com/blog/952033/201904/952033-20190403152706484-547934815.png)

posted @ 2019-04-04 09:12  [sparkdev](https://www.cnblogs.com/sparkdev/)  阅读 (10936)  评论 (0)  [编辑](https://i.cnblogs.com/EditPosts.aspx? postid=10649159)  [收藏](javascript: void(0))
