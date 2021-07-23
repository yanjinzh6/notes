---
title: linux-mount-and-swap
date: 2021-03-06 14:00:00
tags: 'Linux'
categories:
  - ['使用说明', '操作系统']
permalink: linux-mount-and-swap
---

https://blog.51cto.com/13444271/2129132

[原创](https://blog.51cto.com/13444271? s=4)

# Linux 挂载新硬盘和创建 Swap 分区的方法

[! [](https://ucenter.51cto.com/images/noavatar_middle.gif)](https://blog.51cto.com/13444271)

[KJ\_老君丶](https://blog.51cto.com/13444271) 关注 0 人评论 [3288 人阅读](javascript:;) [2018-06-14 01:16:11](javascript:;)

Liunx 添加新硬盘其实和 Windows 的操作一样, 但一个是图形化操作, 另一个是命令行操作, 不过步骤是一样, 下面就动手演示和讲解

! [Linux 挂载新硬盘和创建 Swap 分区的方法](https://s4.51cto.com/images/blog/201806/14/34821015e479068c0fbd76208a89d6e5.jpg? x-oss-process= image/watermark, size_16, text_QDUxQ1RP5Y2a5a6i, color_FFFFFF, t_100, g_se, x_10, y_10, shadow_90, type_ZmFuZ3poZW5naGVpdGk=)

# Linux 挂载新硬盘

### 1, 查看硬盘信息

> 命令: fdisk -l

```
[root@center ~]# fdisk -l
Disk /dev/vda: 21.5 GB, 21474836480 bytes                                           #第一块硬盘的信息和分区信息
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0003a7b4

   Device Boot      Start         End      Blocks   Id  System
     /dev/vda1   *           1        2611    20970496   83  Linux

Disk /dev/vdb: 107.4 GB, 107374182400 bytes                                        #第二块硬盘的信息和分区信息
16 heads, 63 sectors/track, 208050 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x00000000
```

### 2, 创建新硬盘分区

> 命令: fdisk /dev/vdb

```
[root@center ~]# fdisk /dev/vdb
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0x1e694286.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won't be recoverable.

Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

WARNING: DOS-compatible mode is deprecated. It's strongly recommended to
         switch off the mode (command 'c') and change display units to
         sectors (command 'u').

Command (m for help): n
Command action
   e   extended                                 #e 为创建扩展分区
   p   primary partition (1-4)             #p 为创建逻辑分区
p
Partition number (1-4): 1           #在这里输入 1, 就进入划分逻辑分区阶段了;
First cylinder (1-208050, default 1): 1       #分区的 Start 值, 这里最好直接按回车, 否则可能会造成空间浪费;
Using default value 1
Last cylinder, + cylinders or + size{K, M, G} (1-208050, default 208050): 208050      #分区的 Over 值, 我就分一个区
Using default value 208050

Command (m for help): w                             #最后输入 w 回车保存退出.
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```

> fdisk 可以用 m 命令来看 fdisk 命令的内部命令;
> a: 命令指定启动分区;
> d: 命令删除一个存在的分区;
> l: 命令显示分区 ID 号的列表;
> m: 查看 fdisk 命令帮助;
> n: 命令创建一个新分区;
> p: 命令显示分区列表;
> t: 命令修改分区的类型 ID 号;
> w: 命令是将对分区表的修改存盘让它发生作用;

### 3, 确认新分区信息

> 命令: fdisk -l

```
[root@center ~]# fdisk -l
Disk /dev/vda: 21.5 GB, 21474836480 bytes
255 heads, 63 sectors/track, 2610 cylinders
Units = cylinders of 16065 * 512 = 8225280 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x0003a7b4

   Device Boot      Start         End      Blocks   Id  System
     /dev/vda1   *           1        2611    20970496   83  Linux

Disk /dev/vdb: 107.4 GB, 107374182400 bytes
16 heads, 63 sectors/track, 208050 cylinders
Units = cylinders of 1008 * 512 = 516096 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x1e694286

   Device Boot      Start         End      Blocks   Id  System
     /dev/vdb1               1      208050   104857168+  83  Linux           #刚创建好的分区信息
```

### 4, 格式化分区

> 命令: mkfs.ext4 /dev/vdb1

```
[root@center ~]# mkfs.ext4 /dev/vdb1
mke2fs 1.41.12 (17-May-2010)
Filesystem label=                   #文件系统标签
OS type: Linux                 #操作系统类型
Block size=4096 (log=2)          #块大小
Fragment size=4096 (log=2)      #分块大小
Stride=0 blocks, Stripe width=0 blocks
6553600 inodes, 26214292 blocks
1310714 blocks (5.00%) reserved for the super user
First data block=0          #第一个数据块
Maximum filesystem blocks=4294967296
800 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
    4096000, 7962624, 11239424, 20480000, 23887872

Writing inode tables: done                           #写入 inode 表
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done

This filesystem will be automatically checked every 26 mounts or
180 days, whichever comes first.  Use tune2fs -c or -i to override.
```

> Linux 的分区类型介绍:
> 随着 Linux 系统在现在业务中的应用, Linux 文件系统的弱点也渐渐显露出来了, 其中系统缺省使用的 ext2 文件系统是非日志文件系统. 这在关键行业的应用是一个致命的弱点. 而 Ext3 文件系统是直接从 Ext2 文件系统发展而来, 目前 ext3 文件系统已经非常稳定可靠. 它完全兼容 Ext2 文件系统.
> Ext3 的特点: 高可用性, 数据的完整性, 数据的完整性, 数据转换, 多种日志模式
> 同样的 Ext4 完全兼容 Ext3, 相对于 Ext3 来说, Ext4 支持更大的存储, Ext3 目前所支持的最大 16TB 文件系统和最大 2TB 文件, 而 Ext4 分别支持 1EB 的文件系统, 以及 16TB 的文件, 还有就是 Ext3 目前只支持 32,000 个子目录, 而 Ext4 支持无限数量的子目录, Ext4 引入了现代文件系统中流行的 extents 概念, 每个 extent 为一组连续的数据块, 上述文件则表示为 " 该文件数据保存在接下来的 25,600 个数据块中 ", 提高了不少效率.

### 5, 创建挂载目录

> 命令: mkdir /data

### 6, 挂载分区

> 命令: mount /dev/vdb1 /data

### 7, 查看硬盘大小以及挂载分区

> 命令: df -Th

```
[root@center ~]# df -Th
Filesystem     Type   Size  Used Avail Use% Mounted on
/dev/vda1      ext4    20G  1.1G   18G   6% /
tmpfs          tmpfs  3.9G     0  3.9G   0% /dev/shm
/dev/vdb1      ext4    99G   60M   94G   1% /data           #新挂载的分区
```

### 8, 配置开机自动挂载

> 命令: vim /etc/fstab
> /dev/vdb1 /data ext4 defaults 1 1

```
[root@center ~]# blkid
/dev/vda1: UUID="b7aae0d4-268c-4b60-914a-f3b48e22819c" TYPE="ext4"
/dev/vdb1: UUID="5de835dd-5322-46f0-8728-3d4ae7d83b54" TYPE="ext4"

[root@center ~]# cat /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Mar 27 04:51:55 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID= b7aae0d4-268c-4b60-914a-f3b48e22819c      /                       ext4      defaults      1 1
UUID=5de835dd-5322-46f0-8728-3d4ae7d83b54     /data                ext4   defaults    1 1
tmpfs                   /dev/shm                tmpfs   defaults        0 0
devpts                  /dev/pts                devpts  gid=5, mode=620  0 0
sysfs                   /sys                    sysfs   defaults        0 0
proc                    /proc                   proc    defaults        0 0
```

# Swap 分区

Swap 分区其实和 Windows 上的虚拟内存一样, Swap 分区在系统的物理内存不够用的时候, 把物理内存中的一部分空间释放出来, 以供当前运行的程序使用. 那些被释放的空间可能来自一些很长时间没有什么操作的程序, 这些被释放的空间被临时保存到 Swap 分区中, 等到那些程序要运行时, 再从 Swap 分区中恢复保存的数据到内存中.

下面介绍两种创建 swap 的方法:

*   新建磁盘分区作为 swap 分区
*   用文件作为 Swap 分区

### 新建磁盘分区作为 swap 分区

1.  用 fdisk 命令对磁盘进行分区, 添加 swap 分区, 新建分区, 在 fdisk 中用 "t" 命令将新添的分区 id 改为 82
    (Linux swap 类型)

2.  格式化 swap 分区, 这里的 sdb2 要看您加完后 p 命令显示的实际分区设备名

    > mkswap /dev/sdb1

3.  启动新的 swap 分区

    > swapon /dev/sdb1

4.  让系统启动时能自动启用这个交换分区, 可以编辑 /etc/fstab, 加入下面一行

    > /dev/sdb1 swap swap defaults 0 0


### 用文件作为 Swap 分区

1. 创建要作为 swap 分区的文件: 增加 1GB 大小的交换分区, 则命令写法如下, 其中的 count 等于想要的块的数量 (bs\*count= 文件大小).

> dd if=/dev/zero of=/root/swapfile bs=1M count=1024

2. 格式化为交换分区文件, 建立 swap 的文件系统:

> mkswap /root/swapfile

3. 启用交换分区文件:

> swapon /root/swapfile

4. 使系统开机时自启用, 在文件 /etc/fstab 中添加一行:

> /root/swapfile swap swap defaults 0 0

©著作权归作者所有: 来自 51CTO 博客作者 KJ\_老君丶的原创作品, 如需转载, 请注明出处, 否则将追究法律责任

[Linux](https://blog.51cto.com/search/result? q= Linux) [Swap](https://blog.51cto.com/search/result? q= Swap) [分区](https://blog.51cto.com/search/result? q=%E5%88%86%E5%8C%BA) [Linux 命令](https://blog.51cto.com/13444271/category3.html)

4

分享

微博 QQ 微信 ! [](/qr/qr-url? url= https%3A%2F%2Fblog.51cto.com%2F13444271%2F2129132)

收藏

[上一篇: SQL 注入其实很简单, 别一不留神...](https://blog.51cto.com/13444271/2126594 "SQL 注入其实很简单, 别一不留神就被利用了 ") [下一篇: 平民软件 OneProxy 的强大](https://blog.51cto.com/13444271/2130227 " 平民软件 OneProxy 的强大 ")
