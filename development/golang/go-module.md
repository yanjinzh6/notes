---
title: go-module
date: 2021-03-13 17:00:00
tags: 'golang'
categories:
  - ['开发', 'golang']
permalink: go-module
photo:
---

https://learnku.com/articles/27401

#

Go Modules 详解使用

[19](javascript:;) [19](javascript:;) [3](#replies " 评论 ") [](javascript: void(); " 内容目录 ") [

](https://learnku.com/topics/27401/patches/create " 发现错别字? 请点此纠错改进 ") [

](javascript: void(0))

 [! [](https://cdn.learnku.com//uploads/communities/sNljssWWQoW6J88O9G37.png!/both/44x44) Go](https://learnku.com/go " 发布于 Go 社区 ") /  10786 /  3 /  发布于 1 年前  / 更新于 1 年前

[](javascript: void(0);)

> 原文转载 [Go Modules 详解使用](https://www.njphper.com/posts/8b58ea6d.html)

## Module

自从 `Go` 官方从去年推出 1.11 之后, 增加新的依赖管理模块并且更加易于管理项目中所需要的模块. 模块是存储在文件树中的 Go 包的集合, 其根目录中包含 go.mod 文件. go.mod 文件定义了模块的模块路径, 它也是用于根目录的导入路径, 以及它的依赖性要求. 每个依赖性要求都被写为模块路径和特定语义版本.

从 Go 1.11 开始, Go 允许在 $GOPATH/src 外的任何目录下使用 go.mod 创建项目. 在 $GOPATH/src 中, 为了兼容性, Go 命令仍然在旧的 GOPATH 模式下运行. 从 Go 1.13 开始, 模块模式将成为默认模式.

本文将介绍使用模块开发 Go 代码时出现的一系列常见操作:

*   创建一个新模块.
*   添加依赖项.
*   升级依赖项.
*   删除未使用的依赖项.

下面使用的案例都是以 `GIN` 模块为例.
在这之前呢, 需要先设置一些环境变量:

```
export GO111MODULE= on
export GOPROXY= https://goproxy.io // 设置代理
```

### 创建一个新模块

你可以在 $GOPATH/src 之外的任何地方创建一个新的目录. 比如:

> mkdir backend && cd backend

然后初始化 `go mod init backend`, 成功之后你会发现目录下会生成一个 `go.mod` 文件.

> $ cat go.mod
>
> 内容如下
>
> module backend
>
> go 1.12

### 添加依赖项

创建一个文件 main.go 然后加入以下代码, 这里直接 import 了 gin 的依赖包.

> vim main.go

```
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(200, gin.H{
            "message": "pong",
        })
    })
    r.Run() // listen and serve on 0.0.0.0:8080
}
```

go build 之后, 会在 go.mod 引入所需要的依赖包. 之后再来看看 go.mod 文件的情况.

```
module backend

go 1.12

require (
    github.com/gin-contrib/sse v0.0.0-20190301062529-5545eab6dad3 // indirect
    github.com/gin-gonic/gin v1.3.0
    github.com/golang/protobuf v1.3.1 // indirect
    github.com/mattn/go-isatty v0.0.7 // indirect
    github.com/ugorji/go v1.1.4 // indirect
    gopkg.in/go-playground/validator.v8 v8.18.2 // indirect
    gopkg.in/yaml.v2 v2.2.2 // indirect
```

require 就是 gin 框架所需要的所有依赖包 并且在每个依赖包的后面已经表明了版本号

### 升级依赖项

首先我们需要查看以下我们使用到的依赖列表

```
> $ go list -m all
// 你会看到所有项目使用的依赖包
backend
github.com/gin-contrib/sse v0.0.0-20190301062529-5545eab6dad3
github.com/gin-gonic/gin v1.3.0
github.com/golang/protobuf v1.3.1
github.com/mattn/go-isatty v0.0.7
github.com/ugorji/go v1.1.4
golang.org/x/sys v0.0.0-20190222072716-a9d3bda3a223
gopkg.in/check.v1 v0.0.0-20161208181325-20d25e280405
gopkg.in/go-playground/validator.v8 v8.18.2
gopkg.in/yaml.v2 v2.2.2
```

因为这里使用的是最新的版本, 无法升级, 所以这里给出一个回退的例子. 将 GIN 框架的版本回退到上个版本. 这里需要使用一个命令查看依赖的版本历史.

```
>$ go list -m -versions github.com/gin-gonic/gin
// 将会列出 Gin 版本历史
github.com/gin-gonic/gin v1.1.1 v1.1.2 v1.1.3 v1.1.4 v1.3.0
```

将版本更新到上个版本, 这里只是个演示.

```
>$ go get github.com/gin-gonic/gin@v1.1.4 // 只需要在依赖后面加上 @version 就可以了
>$ go list -m all
// 看到了版本变化
github.com/gin-gonic/gin v1.1.4
```

或者可以使用 `go mod` 来进行版本的切换, 这样就需要两个步骤了

```
>$ go mod edit -require="github.com/gin-gonic/gin@v1.1.4" // 修改 go.mod 文件
>$ go tidy // 下载更新依赖
```

`go.tidy` 会自动清理掉不需要的依赖项, 同时可以将依赖项更新到当前版本

使用起来这是一个很简单过程, 只需要几个命令, 你便可以知道依赖的版本信息, 以及自由选择安装的版本, 一切都变得这么简单.

### 删除未使用的依赖项

如果你在项目过程需要移除一些不需要的依赖, 可以使用下面的命令来执行:

```
>$ go mod tidy
```

更多关于 go mod 的使用命令

```
>$ go mod
The commands are:

        download    download modules to local cache
        edit        edit go.mod from tools or scripts
        graph       print module requirement graph
        init        initialize new module in current directory
        tidy        add missing and remove unused modules
        vendor      make vendored copy of dependencies
        verify      verify dependencies have expected content
        why         explain why packages or modules are needed
```

### 结论

Go Module 是 Go 依赖管理的未来. 目前只有 1.11 和 1.12 版本支持该功能, 介绍了 Go 依赖管理的功能. 更多的功能会在以后补充. 也欢迎补充完善. 最后如果你是使用 `Goland`, 请移步这里 [Working with Go Modules
](https://blog.jetbrains.com/go/2019/01/22/working-with-go-modules/) 阅读关于使用 Modules 开发

[模块](https://learnku.com/blog/laravel-study/tags/modular_tn_50186)

> 本作品采用 [<CC 协议>](https://learnku.com/docs/guide/cc4.0/6589), 转载必须注明作者和本文链接

本帖由系统于 1 年前 自动加精

[

举报

](javascript: void(0))
