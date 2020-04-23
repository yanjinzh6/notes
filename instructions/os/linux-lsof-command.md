---
title: Linux lsof 命令
date: 2020-02-05 23:30:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: linux-lsof-command
---

## 简介

lsof (list open files) 是一个列出当前系统打开文件的工具. 在linux环境下, 任何事物都以文件的形式存在, 通过文件不仅仅可以访问常规数据, 还可以访问网络连接和硬件. 所以如传输控制协议 (TCP) 和用户数据报协议 (UDP) 套接字等, 系统在后台都为该应用程序分配了一个文件描述符, 无论这个文件的本质如何, 该文件描述符为应用程序与基础操作系统之间的交互提供了通用接口. 因为应用程序打开文件的描述符列表提供了大量关于这个应用程序本身的信息, 因此通过lsof工具能够查看这个列表对系统监测以及排错将是很有帮助的.

```sh
lsof [参数][文件]
```

## 常用

```sh
## 列出使用某个端口的应用
lsof -i :3306
## 列出多个进程多个打开的文件信息
lsof -c mysql -c apache
## 列出某个用户打开的文件信息
lsof -u username
```

<!-- more -->

## 详情

lsof打开的文件可以是:

1. 普通文件
2. 目录
3. 网络文件系统的文件
4. 字符或设备文件
5. (函数)共享库
6. 管道, 命名管道
7. 符号链接
8. 网络文件 (例如: NFS file, 网络socket, unix域名socket)
9. 还有其它类型的文件, 等等

```sh
lsof
  -a 列出打开文件存在的进程
  -c<进程名> 列出指定进程所打开的文件
  -g  列出GID号进程详情
  -d<文件号> 列出占用该文件号的进程
  +d<目录>  列出目录下被打开的文件
  +D<目录>  递归列出目录下被打开的文件
  -n<目录>  列出使用NFS的文件
  -i<条件>  列出符合条件的进程.  (4, 6, 协议, :端口,  @ip )
  -p<进程号> 列出指定进程号所打开的文件
  -u  列出UID号进程详情
  -h 显示帮助信息
  -v 显示版本信息
```

## 输出详解

- COMMAND: 进程的名称
- PID: 进程标识符
- PPID: 父进程标识符 (需要指定-R参数)
- USER: 进程所有者
- PGID: 进程所属组
- FD: 文件描述符, 应用程序通过文件描述符识别该文件. 如cwd, txt等
  - cwd: 表示current work dirctory, 即: 应用程序的当前工作目录, 这是该应用程序启动的目录, 除非它本身对这个目录进行更改
  - txt : 该类型的文件是程序代码, 如应用程序二进制文件本身或共享库, 如上列表中显示的 `/sbin/init` 程序
  - lnn: library references (AIX);
  - er: FD information error (see NAME column);
  - jld: jail directory (FreeBSD);
  - ltx: shared library text (code and data);
  - mxx : hex memory-mapped type number xx.
  - m86: DOS Merge mapped file;
  - mem: memory-mapped file;
  - mmap: memory-mapped device;
  - pd: parent directory;
  - rtd: root directory;
  - tr: kernel trace file (OpenBSD);
  - v86  VP/ix mapped file;
  - 0: 表示标准输出
  - 1: 表示标准输入
  - 2: 表示标准错误
- 一般在标准输出, 标准错误, 标准输入后还跟着文件状态模式: r, w, u等
  - u: 表示该文件被打开并处于读取/写入模式
  - r: 表示该文件被打开并处于只读模式
  - w: 表示该文件被打开并处于
  - 空格: 表示该文件的状态模式为unknow, 且没有锁定
  - -: 表示该文件的状态模式为unknow, 且被锁定
- 同时在文件状态模式后面, 还跟着相关的锁
  - N: for a Solaris NFS lock of unknown type;
  - r: for read lock on part of the file;
  - R: for a read lock on the entire file;
  - w: for a write lock on part of the file; (文件的部分写锁)
  - W: for a write lock on the entire file; (整个文件的写锁)
  - u: for a read and write lock of any length;
  - U: for a lock of unknown type;
  - x: for an SCO OpenServer Xenix lock on part      of the file;
  - X: for an SCO OpenServer Xenix lock on the      entire file;
  - space: if there is no lock.
- TYPE: 文件类型, 如DIR, REG等, 常见的文件类型
  - DIR: 表示目录
  - CHR: 表示字符类型
  - BLK: 块设备类型
  - UNIX:  UNIX 域套接字
  - FIFO: 先进先出 (FIFO) 队列
  - IPv4: 网际协议 (IP) 套接字
- DEVICE: 指定磁盘的名称
- SIZE: 文件的大小
- NODE: 索引节点 (文件在磁盘上的标识)
- NAME: 打开文件的确切名称
