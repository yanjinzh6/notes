---
title: Git 多平台换行
date: 2020-07-05 15:00:00
tags: 'Git'
categories:
  - ['使用说明', '软件']
permalink: git-crlf-lf
photo:
---

## 简介

常见的换行如下

- `0x0D0A (CRLF)`: `\r\n`, 这是 DOS/Windows 下默认的换行方式
- `0x0A (LF)`: `\n`, Linux 和 MacOS 的换行方式
- `0x0D (CR)`: `\r`, 早期 MacOS 的换行方式

对于多人参与的仓库, 总会有各种各样的开发平台, 这样仓库中就经常会出现换行方式不一样的文件, 尽管仓库还是可以使用的, 但是每次文件一经过不同平台的修改后就会导致 Git 认为整个文件被修改, 这样影响 `diff` 工具的使用, 只能使用专用的代码分析工具才能比较出了, 如各种 IDE

所以团队的 Git 工具需要进行统一的配置

<!-- more -->

## 配置

### core.autocrlf 配置

该配置作用于提交和检出时自动转换换行符

```sh
# 提交时转换为 LF, 检出时转换为 CRLF
git config --global core.autocrlf true

# 提交时转换为 LF, 检出时不转换
git config --global core.autocrlf input

# 提交检出均不转换
git config --global core.autocrlf false
```

### core.safecrlf

该配置用于检查文件是否包含混合换行符

```sh
# 拒绝提交包含混合换行符的文件
git config --global core.safecrlf true

# 允许提交包含混合换行符的文件
git config --global core.safecrlf false

# 提交包含混合换行符的文件时给出警告
git config --global core.safecrlf warn
```

## 推荐使用

由于 Git 仓库一般都是部署在 UNIX 环境中, 所以统一使用 LR 换行符, 这样对于不同环境需要进行不同的设置

### Windows

```sh
# 提交时转换为 LF, 检出时转换为 CRLF
git config --global core.autocrlf true
# 拒绝提交包含混合换行符的文件
git config --global core.safecrlf true
```

Windows 环境下需要提交时自动转换为 LF, 检出时自动转换为 CRLF

### Linux/MacOS

```sh
# 提交时转换为 LF, 检出时不转换
git config --global core.autocrlf input
# 拒绝提交包含混合换行符的文件
git config --global core.safecrlf true
```

由于 Linux/MacOS 本身的换行符与 Git 仓库相同, 这时候只需要保证某一些 copy 来的文件正确提交即可
