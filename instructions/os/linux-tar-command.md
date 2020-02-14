---
title: Linux tar 命令
date: 2020-02-14 17:00:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: linux-tar-command
---

# 简介

tar 命令主要提供打包命令

- 打包: 将一堆文件或目录变成一个总的文件
- 压缩: 通过算法将文件变成小文件

由于 Linux 中很多压缩程序只能针对一个文件进行压缩, 所以压缩一堆文件时, 需要先将文件打包, 然后使用压缩程序进行压缩

# 常用命令

```sh
# 将文件全部打包成 tar 包
tar -cvf FILENAME.tar ./
# 打包后, 以 gzip 压缩
tar -zcvf FILENAME.tar.gz ./
# 打包后, 以 bzip2 压缩
tar -jcvf FILENAME.tar.bz2 ./
# 查看 tar 包内文件
tar -tvf FILENAME.tar ./
# 解压
tar -zxvf FILENAME.tar.gz [可以指定仅解压的文件列表] [-C 指定解压目录]
# 筛选文件夹文件
tar -N [压缩比 yyyy/MM/DD 日期新的文件] --exclude [排除的文件或目录, 支持正则] -zcvf FILENAME.tar.gz ./
```

<!-- more -->

# 参数

```sh
# 必要参数有如下:

-A 新增压缩文件到已存在的压缩

-B 设置区块大小

-c 建立新的压缩文件

-d 记录文件的差别

-r 添加文件到已经压缩的文件

-u 添加改变了和现有的文件到已经存在的压缩文件

-x 从压缩的文件中提取文件

-t 显示压缩文件的内容

-z 支持gzip解压文件

-j 支持bzip2解压文件

-Z 支持compress解压文件

-v 显示操作过程

-l 文件系统边界设置

-k 保留原有文件不覆盖

-m 保留文件不被覆盖

-W 确认压缩文件的正确性

# 可选参数如下:

-b 设置区块数目

-C 切换到指定目录

-f 指定压缩文件

--help 显示帮助信息

--version 显示版本信息
```

# 使用命令

- tar
  - 解包: tar xvf FileName.tar
  - 打包: tar cvf FileName.tar DirName
- .gz
  - 解压1: gunzip FileName.gz
  - 解压2: gzip -d FileName.gz
  - 压缩: gzip FileName
- .tar.gz 和 .tgz
  - 解压: tar zxvf FileName.tar.gz
  - 压缩: tar zcvf FileName.tar.gz DirName
- .bz2
  - 解压1: bzip2 -d FileName.bz2
  - 解压2: bunzip2 FileName.bz2
  - 压缩:  bzip2 -z FileName
- .tar.bz2
  - 解压: tar jxvf FileName.tar.bz2
  - 压缩: tar jcvf FileName.tar.bz2 DirName
- .bz
  - 解压1: bzip2 -d FileName.bz
  - 解压2: bunzip2 FileName.bz
  - 压缩: 未知
- .tar.bz
  - 解压: tar jxvf FileName.tar.bz
  - 压缩: 未知
- .Z
  - 解压: uncompress FileName.Z
  - 压缩: compress FileName
- .tar.Z
  - 解压: tar Zxvf FileName.tar.Z
  - 压缩: tar Zcvf FileName.tar.Z DirName
- .zip
  - 解压: unzip FileName.zip
  - 压缩: zip FileName.zip DirName
- .rar
  - 解压: rar x FileName.rar
  - 压缩: rar a FileName.rar DirName

# 引用

[每天一个linux命令 (28) : tar命令](https://www.cnblogs.com/peida/archive/2012/11/30/2795656.html)