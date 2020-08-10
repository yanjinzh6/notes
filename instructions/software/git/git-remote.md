---
title: Git 远程仓库
date: 2020-08-08 11:00:00
tags: 'Git'
categories:
  - ['使用说明', '软件']
permalink: git-remote
photo:
---

## 简介

`git remote` 命令管理一组跟踪的存储库

远程仓库是指托管在网络上的项目仓库，可能会有好多个，其中有些你只能读，另外有些可以写。同他人协作开发某 个项目时，需要管理这些远程仓库，以便推送或拉取数据，分享各自的工作进展。管理远程仓库的工作，包括添加远程库，移除废弃的远程库，管理各式远程库分支，定义是否跟踪这些分支等等

## 应用

### 推送到新远程仓库

```sh
# 初始化本地仓库
git init
# 提交修改
git status
git add .
git commit
# 添加远程分支
git add remote origin https://github.com/xxx
# 推送本地代码到远程分支
git push -u origin master
```

### 添加新远程仓库并获取

```sh
# 当前远程仓库
git remote
# 添加新的远程仓库
git remote add target https://github.com/xxx
# 获取
git fetch target
# 可以将目标仓库提交 cherry-pick 到当前分支
```

<!-- more -->

## 语法

```sh
git remote [-v | --verbose]
git remote add [-t <branch>] [-m <master>] [-f] [--[no-]tags] [--mirror=<fetch|push>] <name> <url>
git remote rename <old> <new>
git remote remove <name>
git remote set-head <name> (-a | --auto | -d | --delete | <branch>)
git remote set-branches [--add] <name> <branch>…​
git remote get-url [--push] [--all] <name>
git remote set-url [--push] <name> <newurl> [<oldurl>]
git remote set-url --add [--push] <name> <newurl>
git remote set-url --delete [--push] <name> <url>
git remote [-v | --verbose] show [-n] <name>…​
git remote prune [-n | --dry-run] <name>…​
git remote [-v | --verbose] update [-p | --prune] [(<group> | <remote>)…​]
```
