---
title: Linux 重定向 IO
date: 2020-02-26 11:30:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: linux-shell-io
---

## 简介

Linux shell（比如 Bash）以字符序列或 流的形式接收输入和发送输出. 每个字符与它之前和之后的字符独立. 字符未组织为结构化记录或固定大小的字符块. 流使用了文件 IO 技术访问, 无论是否是来自或传到文件, 键盘, 显式窗口或某种其他 IO 设备的实际字符流. Linux shell 使用 3 种标准 I/O 流, 每种流与一种著名的文件描述符相关联:

- stdout 是 标准输出流, 它显示来自命令的输出. 它拥有文件描述符 1.
- stderr 是 标准错误流, 它显示来自命令的错误输出. 它拥有文件描述符 2.
- stdin 是 标准输入流, 它向命令提供输入. 它拥有文件描述符 0.

## 重定向标准 IO

| 命令 | 说明 |
| - | - |
| command > file | 将输出重定向到 file.  |
| command < file | 将输入重定向到 file.  |
| command >> file | 将输出以追加的方式重定向到 file.  |
| n > file | 将文件描述符为 n 的文件重定向到 file.  |
| n >> file | 将文件描述符为 n 的文件以追加的方式重定向到 file.  |
| n >& m | 将输出文件 m 和 n 合并.  |
| n <& m | 将输入文件 m 和 n 合并.  |
| << tag | 将开始标记 tag 和结束标记 tag 之间的内容作为输入.  |
| command 2 > file | stderr 重定向到 file.  |
| command 2 >> file | stderr 追加到 file.  |
| command > file 2>&1 | 将 stdout 和 stderr 合并后重定向到 file.  |
| command >> file 2>&1 | 将 stdout 和 stderr 合并后重定向到 file.  |
| command < file1 >file2 | 将 stdin 重定向到 file1, 将 stdout 重定向到 file2.  |

> 需要注意的是文件描述符 0 通常是标准输入（STDIN）, 1 是标准输出（STDOUT）, 2 是标准错误输出（STDERR）.

<!-- more -->

## Here Document

Here Document 是 Shell 中的一种特殊的重定向方式, 用来将输入重定向到一个交互式 Shell 脚本或程序.

```sh
command << delimiter
    document
delimiter
```

它的作用是将两个 delimiter 之间的内容(document) 作为输入传递给 command.

**注意:**

- 结尾的 delimiter 一定要顶格写, 前面不能有任何字符, 后面也不能有任何字符, 包括空格和 tab 缩进.
- 开始的 delimiter 前后的空格会被忽略掉.

```sh
cat << EOF
heredoc> 1
heredoc> 2
heredoc> 3
heredoc> EOF
1
2
3
```

## /dev/null 文件

如果希望执行某个命令, 但又不希望在屏幕上显示输出结果, 那么可以将输出重定向到 `/dev/null`: `command > /dev/null` `/dev/null` 是一个特殊的文件, 写入到它的内容都会被丢弃；如果尝试从该文件读取内容, 那么什么也读不到. 屏蔽 stdout 和 stderr `command > /dev/null 2>&1`

## 引用

- [Shell 输入/输出重定向](https://www.runoob.com/linux/linux-shell-io-redirections.html)
- [流, 管道和重定向](https://www.ibm.com/developerworks/cn/linux/l-lpic1-103-4/index.html)
