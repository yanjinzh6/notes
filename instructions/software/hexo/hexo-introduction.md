---
title: Hexo 博客使用
date: 2019-12-24 17:00:00
tags: 'Hexo'
categories:
  - ['使用说明', '软件']
permalink: hexo-introduction
---

## 起点

使用 markdown 来管理记录的一个优点是简洁, 可以使用版本管理等平台来进行保存, 自己也一直保存着源文件, 只要在 git 客户端的环境下就可以进行修改, 不好的地方就是需要自定义渲染才能将 markdown 文件转换成网站, 但是相比其他的博客平台, 都是只能用它们的格式来进行存储, 不方便迁移和私底下修改.

<!--more-->

## Hexo 简介

### npm 模块

[Hexo](https://github.com/hexojs/hexo) [官网](https://hexo.io/zh-cn/) 作为一个简洁的 markdown 渲染框架, 仅提供 npm 模块安装和 cli 命令行.

可以通过以下命令安装 `npm install -g hexo` 进行全局安装, 如果使用比较新的 nodejs 版本, 可以直接使用 `npx hexo` 来进行后续操作.

### hexo init 命令

`hexo init demo` 初始化博客目录, 这是一个标准的 nodejs 工程.

```
demo                 // 项目目录
├── _config.yml      // hexo 配置文件
├── package.json     // 项目依赖
├── scaffolds        // 模版
├── source
|   └── _posts       // 源文件
└── themes           // 主题
```

### hexo g && hexo s

在项目目录文件中使用 `hexo g` 就会在 public 文件夹内生成相应的 html 文件, 这个文件夹就是静态网站的根目录, `hexo s` 会启动一个本地的调试服务器, 通过 `localhost:4000` 就可以访问到渲染好的静态网站.

### 常用命令

```sh
hexo new "postName"      # 新建文章
hexo new page "pageName" # 新建页面
hexo generate            # 生成静态页面至 public 目录
hexo server              # 开启预览服务
hexo deploy              # 部署
hexo help                # 查看帮助
hexo version             # 查看 Hexo 的版本
```

### 快速添加概述

使用 `<!--more-->` 可以快速添加概述, 生成的页面将会在 html 头部添加概述标签.

### 使用主题

修改 `_config.yml` 中的 `theme` 参数, 这里注意到 `language` 参数, 这个设置是控制页面渲染国际化用的, 这里可以查看 `themes/${you_theme}/languages` 里的国际化文件, 配置相应的名称即可.

## 参考

[Hexo 建站教程](https://tding.top/archives/7f189df5.html)
