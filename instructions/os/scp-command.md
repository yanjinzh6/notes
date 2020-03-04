---
title: Scp 命令
date: 2020-03-02 11:30:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: lscp-command
---

# 简介

scp 是 secure copy 的简写, 用于在 Linux 下进行远程拷贝文件的命令, 和它类似的命令有 cp, 不过 cp 只是在本机进行拷贝不能跨服务器, 而且 scp 传输是加密的. 可能会稍微影响一下速度. 当你服务器硬盘变为只读 read only system 时, 用 scp 可以帮你把文件移出来. 另外, scp 还非常不占资源, 不会提高多少系统负荷, 在这一点上, rsync 就远远不及它了. 虽然 rsync 比 scp 会快一点, 但当小文件众多的情况下, rsync 会导致硬盘 I/O 非常高, 而 scp 基本不影响系统正常使用.

# 常用

```sh
# 从本地服务器复制到远程服务器
scp local_file remote_username@remote_ip:remote_folder
# 使用 -r 支持复制目录
scp -r local_folder remote_username@remote_ip:remote_folder
# 反过来支持从远程复制到本地
scp -r remote_username@remote_ip:remote_folder local_folder
# 当目录存在时, 会出现目录下的文件不会被覆盖, 这时候需要直接复制目录所有的内容
scp -r remote_username@remote_ip:remote_folder/. local_folder/
```

<!-- more -->

# 命令格式

```sh
scp [参数] [原路径] [目标路径]

-1 强制 scp 命令使用协议 ssh1
-2 强制 scp 命令使用协议 ssh2
-4 强制 scp 命令只使用 IPv4 寻址
-6 强制 scp 命令只使用 IPv6 寻址
-B 使用批处理模式 (传输过程中不询问传输口令或短语)
-C 允许压缩.  (将 -C 标志传递给 ssh, 从而打开压缩功能)
-p 保留原文件的修改时间, 访问时间和访问权限.
-q 不显示传输进度条.
-r 递归复制整个目录.
-v 详细方式显示输出. scp 和 ssh(1) 会显示出整个过程的调试信息. 这些信息用于调试连接, 验证和配置问题.
-c cipher 以 cipher 将数据传输进行加密, 这个选项将直接传递给 ssh.
-F ssh_config 指定一个替代的 ssh 配置文件, 此参数直接传递给 ssh.
-i identity_file 从指定文件中读取传输时使用的密钥文件, 此参数直接传递给 ssh.
-l limit 限定用户所能使用的带宽, 以 Kbit/s 为单位.
-o ssh_option 如果习惯于使用 ssh_config(5) 中的参数传递方式,
-P port 注意是大写的 P, port 是指定数据传输用到的端口号
-S program 指定加密传输时所使用的程序. 此程序必须能够理解 ssh(1)的选项.
```

# 引用

- [每天一个 linux 命令 (60): scp命令](https://www.cnblogs.com/peida/archive/2013/03/15/2960802.HTML)