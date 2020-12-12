---
title: screen 使用
date: 2020-12-06 13:00:00
tags: 'screen'
categories:
  - ['使用说明', '软件']
permalink: screen-introduction
---

## 简介

远程主机都是通过 ssh 连接的, 在关闭连接之后会结束当前正在执行的命令, 很多时候运行一些重要的命令或者需要长时间运行的命令时, 希望能在 ssh 连接断开后继续执行, 例如系统升级, 系统备份, ftp 传输等

Screen 是一款由 GNU 计划开发的用于命令行终端切换的自由软件, 可以通过该软件同时连接多个本地或者远程的命令行会话, 并且可以自由切换, 在结束 ssh 连接后, screen 仍然会保持其命令行会话一直执行

<!-- more -->

## 示例

```sh
# 新建一个名称为 name 的 session
screen -S name
# 列出当前所有的 session
screen -ls
# 恢复到 name 这个 session
screen -r name
# 远程 detach name 这个 session
screen -d name
# 结束掉当前的 session 并回到 name 这个 session
screen -d -r name
# 直接创建单窗口会话, 命令结束后会退出会话
screen vi txt
```

```sh
# 窗口列表, * 表示当前窗口, - 表示上次使用的窗口
0$ bash  1-$ bash  2*$ bash
# 可以使用 C-a A 命令来重命名当前窗口名称
```

```sh
# 暂时中断会话, 使用 C-a d
screen vi temp
# 这时候会话会暂停并回到后台, 使用 -r 恢复
screen -ls
screen -r sessionId
```

```sh
# session 会由某些原因变成 dead 状态, 这时候的会话已经无效了, 使用下面命令清除
screen -wipe
```

## 使用

```sh
screen [-AmRvx -ls -wipe][-d <作业名称>][-h <行数>][-r <作业名称>][-s ][-S <作业名称>]
```

### 参数

- -A :  将所有的视窗都调整为目前终端机的大小
- -d <作业名称> :  将指定的 screen 作业离线
- -h <行数> :  指定视窗的缓冲区行数
- -m :  即使目前已在作业中的 screen 作业, 仍强制建立新的 screen 作业
- -r <作业名称> :  恢复离线的 screen 作业
- -R :  先试图恢复离线的作业, 若找不到离线的作业, 即建立新的 screen 作业
- -s :  指定建立新视窗时, 所要执行的shell
- -S <作业名称> :  指定 screen 作业的名称
- -v :  显示版本信息
- -x :  恢复之前离线的 screen 作业
- -ls 或 --list :  显示目前所有的 screen 作业
- -wipe :  检查目前所有的 screen 作业, 并删除已经无法使用的 screen 作业

### 命令

在每个 screen session 中, 所有命令都是以 `ctrl + a (C-a)` 开始的

- `C-a ?` : 显示所有键绑定信息
- `C-a c` : 创建一个新的运行shell的窗口并切换到该窗口
- `C-a n` : Next, 切换到下一个 window
- `C-a p` : Previous, 切换到前一个 window
- `C-a 0..9` : 切换到第 0..9 个 window
- `Ctrl+a [Space]` : 由视窗0循序切换到视窗9
- `C-a C-a` : 在两个最近使用的 window 间切换
- `C-a x` : 锁住当前的 window, 需用用户密码解锁
- `C-a d` : detach, 暂时离开当前session, 将目前的 screen session (可能含有多个 windows) 丢到后台执行, 并会回到还没进 screen 时的状态, 此时在 screen session 里, 每个 window 内运行的 process (无论是前台/后台)都在继续执行, 即使 logout 也不影响
- `C-a z` : 把当前session放到后台执行, 用 shell 的 fg 命令则可回去
- `C-a w` : 显示所有窗口列表
- `C-a t` : Time, 显示当前时间, 和系统的 load
- `C-a k` : kill window, 强行关闭当前的 window
- `C-a [` : 进入 copy mode, 在 copy mode 下可以回滚, 搜索, 复制就像用使用 vi 一样
  - `C-b` : Backward, PageUp
  - `C-f` : Forward, PageDown
  - `H(大写)` : High, 将光标移至左上角
  - `L` : Low, 将光标移至左下角
  - `0` : 移到行首
  - `$` : 行末
  - `w` : forward one word, 以字为单位往前移
  - `b` : backward one word, 以字为单位往后移
  - `Space` : 第一次按为标记区起点, 第二次按为终点
  - `Esc` : 结束 copy mode
- `C-a ]` : Paste, 把刚刚在 copy mode 选定的内容贴上

## 更多功能

参考: [linux screen 命令详解](https://www.cnblogs.com/mchina/archive/2013/01/30/2880680.html)
