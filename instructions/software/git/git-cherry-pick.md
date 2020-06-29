---
title: Git Cherry pick 命令
date: 2020-06-25 11:00:00
tags: 'Git'
categories:
  - ['使用说明', '软件']
permalink: git-cherry-pick
photo:
---

## 简介

多分支项目代码合并是常有的操作, 一般都是使用 `git merge` 将分支的代码合并, 如果仅需要合并一部分提交, 就可以使用 `git cherry-pick` 操作合并单个或多个提交

## 基本操作

```sh
# 将指定的提交应用于当前分支, 该操作会在当前分支产生一个新的分支, 具有不同的 Hash 值
git cherry-pick <commitHash>
```

如下仓库中有两个分支 `Master` 和 `Feature`

```
a - b - c - d   Master
         \
           e - f - g  Feature
```

这时候需要将 `Feature` 分支新开发的 `f` 提交合并到 `Master` 分支中

```sh
# 切换到 master 分支
$ git checkout master

# Cherry pick 操作
$ git cherry-pick f
```

完成操作后, 代码库分支状态如下

```
a - b - c - d - f  Master
         \
           e - f - g  Feature
```

结果是 `f` 提交在 `Master` 分支中重新提交了, 如果 `d` 和 `f` 之间有冲突需要解决

<!-- more -->

## 应用分支最新的提交

```sh
# 使用分支名作为参数可以将指定分支的最新的提交应用到当前分支中
git cherry-pick Feature
```

## 多个提交

```sh
# 将多个提交应用到当前分支, 会在当前分支生成多个对应的新提交
git cherry-pick <HashA> <HashB>

# 将从 A(不包含) 到 B 中间连续的分支应用到当前分支, 提交 A 必须早于提交 B
git cherry-pick A..B

# 包含 A
git cherry-pick A^..B
```

## 参数

```
-e, --edit: 打开外部编辑器, 编辑提交信息
-n, --no-commit: 只更新工作区和暂存区, 不产生新的提交
-x: 在提交信息的末尾追加一行 (cherry picked from commit ...), 方便以后查到这个提交是如何产生的
-s, --signoff: 在提交信息的末尾追加一行操作者的签名, 表示是谁进行了这个操作
-m parent-number, --mainline parent-number: 如果原始提交是一个合并节点, 来自于两个分支的合并, 那么 Cherry pick 默认将失败, 因为它不知道应该采用哪个分支的代码变动, 使用 -m 配置指定原始提交的父分支编号, 一般来说, 1号父分支是接受变动的分支 (the branch being merged into) , 2号父分支是作为变动来源的分支 (the branch being merged from)
```

## 代码冲突

使用过程中出现代码冲突, 需要用户确定接下来该如何操作

### 解决冲突

```sh
# 解决冲突
# 将修改的文件添加到暂存区
git add .
# 继续执行
git cherry-pick --continue
```

### 放弃

使用 `git cherry-pick --abort` 放弃操作, 回到操作前的状态

### 退出

使用 `git cherry-pick --quit` 退出操作, 保留当前的修改

## cherry-pick 其他仓库

```sh
# 添加一个远程仓库 target
git remote add target git://gitUrl
# 获取代码
git fetch target
# 获取远程仓库提交日志
git log target/master
# 将指定的提交应用到当前分支
git cherry-pick <commitHash>
```

## 参考

- [git cherry-pick 教程](https://www.ruanyifeng.com/blog/2020/04/git-cherry-pick.html)
