---
title: Git merge 和 rebase 应用
date: 2020-06-26 11:00:00
tags: 'Git'
categories:
  - ['使用说明', '软件']
permalink: git-merge-rebase
photo:
---

## 简介

单项目多分支一直都是团队开发的主流, 对于越来越多的分支出现, 如何对分支进行更好的合并就是一个问题了

merge 和 rebase 命令都有合并分支的功能, 具体的区别如下:

- 当合并的分支有冲突时, rebase 会省略了 merge commit
- rebase 合并分支后会合并之前的 commit 历史, 使得分支图更加简洁

rebase 虽然看着好, 但是合并后出现的代码问题不容易定位

所以需要保持多分支视图的简洁性, 或者合并自己的多个分支的时候, 就可以使用 rebase

- 使用 `git rebase master` 合并远程分支
- 解决冲突后使用 `git rebase --continue` 合并
- 将分支提交合并请求

<!-- more -->

## 简单示例

通过简单的两个分支更新一个文件造成冲突来看一下流程

```sh
# 新建仓库
g init
Initialized empty Git repository in /Users/xxx/git-demo/rebase/.git/
echo "# this is README.md" > README.md
ga .
gcmsg "docs: add README.md"
master (root-commit) c0cc178] docs: add README.md
 1 file changed, 1 insertion(+)
 create mode 100644 README.md
# 添加两个分支, 指向 master
commit c0cc1782c94e17644977c8f95f474adb6d712089 (HEAD -> b2, master, b)
Author: zhuyj <zhuyj@minstone.com.cn>
Date:   Fri Jul 3 14:51:57 2020 +0800

    docs: add README.md

 README.md | 1 +
 1 file changed, 1 insertion(+)
# 两个分支都修改 README.md 文件
# 修改 b 分支
gco b
Switched to branch 'b'
echo "# update" >> README.md
ga .
gcmsg "docs: b update README.md"
[b 401dfe9] docs: b update README.md
 1 file changed, 1 insertion(+)
# 修改 b2 分支
gco b2
Switched to branch 'b2'
echo "# update 2" >> README.md
ga .
gcmsg "docs: b2 update README.md"
[b2 23af3aa] docs: b2 update README.md
 1 file changed, 1 insertion(+)
# 查看当前状态
* 23af3aa (HEAD -> b2) docs: b2 update README.md
| * 401dfe9 (b) docs: b update README.md
|/
* c0cc178 (master) docs: add README.md
# 模拟分支 b 先提交合并请求
gco master
Switched to branch 'master'
gm b
Updating c0cc178..401dfe9
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
# 合并分支 b 后查看当前状态
* 23af3aa (b2) docs: b2 update README.md
| * 401dfe9 (HEAD -> master, b) docs: b update README.md
|/
* c0cc178 docs: add README.md
# 分支 b2 请求合并发现落后了需要同步 master
gco b2
Switched to branch 'b2'
grb master
First, rewinding head to replay your work on top of it...
Applying: docs: b2 update README.md
Using index info to reconstruct a base tree...
M README.md
Falling back to patching base and 3-way merge...
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
error: Failed to merge in the changes.
Patch failed at 0001 docs: b2 update README.md
Use 'git am --show-current-patch' to see the failed patch

Resolve all conflicts manually, mark them as resolved with
"git add/rm <conflicted_files>", then run "git rebase --continue".
You can instead skip this commit: run "git rebase --skip".
To abort and get back to the state before "git rebase", run "git rebase --abort".
# 提示冲突需要处理然后再 continue
code README.md
ga .
grbc
Applying: docs: b2 update README.md
# 查看当前的状态
* 1b772bb (HEAD -> b2) docs: b2 update README.md
* 401dfe9 (master, b) docs: b update README.md
* c0cc178 docs: add README.md
# 这时可以看到整个分支的图表很简单
# 模拟合并分支 b2
gco master
Switched to branch 'master'
gm b2
Updating 401dfe9..1b772bb
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
# 查看当前的状态
* 1b772bb (HEAD -> master, b2) docs: b2 update README.md
* 401dfe9 (b) docs: b update README.md
* c0cc178 docs: add README.md
# 可以看到 master 已经是最新的状态了
```

如上面所说的, 使用 rebase 可以使整个分支图变得精简, 但是会合并掉某一些请求的详细情况, 例如上面的 `23af3aa` 请求已经不见了

可以看一下如果分支 b2 使用 merge 合并 master 再提交合并会有什么样的情况

```sh
# 将 master 退回到 401dfe9
g reset 401dfe9 --hard
HEAD is now at 401dfe9 docs: b update README.md
# 查看当前的状态
* 1b772bb (b2) docs: b2 update README.md
* 401dfe9 (HEAD -> master, b) docs: b update README.md
* c0cc178 docs: add README.md
# 将分支 b2 退回到 c0cc178
gco b2
Switched to branch 'b2'
g reset c0cc178 --hard
HEAD is now at c0cc178 docs: add README.md
# 查看当前的状态
* 401dfe9 (master, b) docs: b update README.md
* c0cc178 (HEAD -> b2) docs: add README.md
# 修改分支 b2
echo "# update 2" >> README.md
ga .
gcmsg "docs: b2 update README.md"
[b2 9e1e200] docs: b2 update README.md
 1 file changed, 1 insertion(+)
# 查看当前的状态
* 9e1e200 (HEAD -> b2) docs: b2 update README.md
| * 401dfe9 (master, b) docs: b update README.md
|/
* c0cc178 docs: add README.md
# 分支 b2 合并 master
gm master
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
# 解决冲突
code README.md
ga .
gcmsg "docs: merge master"
[b2 5b31932] docs: merge master
# 查看当前的状态
*   5b31932 (HEAD -> b2) docs: merge master
|\
| * 401dfe9 (master, b) docs: b update README.md
* | 9e1e200 docs: b2 update README.md
|/
* c0cc178 docs: add README.md
# 模拟合并分支 b2
gco master
Switched to branch 'master'
gm b2
Updating 401dfe9..5b31932
Fast-forward
 README.md | 1 +
 1 file changed, 1 insertion(+)
# 查看当前的状态
*   5b31932 (HEAD -> master, b2) docs: merge master
|\
| * 401dfe9 (b) docs: b update README.md
* | 9e1e200 docs: b2 update README.md
|/
* c0cc178 docs: add README.md
```

## 小结

通过一个简单的操作, 可以看到 rebase 操作在操作合并分支后可以使整个分支图更加简洁, 虽然中间解决代码冲突的合并请求被隐藏了, 不方便追踪代码的变动, 但是当一个项目存在大量的分支, 一些小功能变动的分支应该使用 rebase 操作来保持分支图的简洁易懂性
