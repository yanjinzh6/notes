---
title: Docker 资源限制
date: 2021-03-06 12:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-limit-resource
photo:
---

https://my.oschina.net/u/4270238/blog/4074295

# [Docker 的资源限制 (内存, CPU, IO) 详细篇](https://my.oschina.net/u/4270238/blog/4074295)

[osc\_6os21ekm](https://my.oschina.net/u/4270238)

2020/03/16 17:44

阅读数 2.1K

[昇腾众智计划火热上线!140 个算子 / 模型等你来挑战!>>>! [](https://www.oschina.net/img/hot3.png)](https://e.cn.miaozhen.com/r/k=2226845&p=7qZiO&dx=__IPDX__&rt=2&pro= s&ns=__IP__&ni=__IESID__&v=__LOC__&xa=__ADPLATFORM__&tr=__REQUESTID__&o= https://ascend.huawei.com/zh/#/ecosystem/all-wisdom? utm_campaign=%252004MHQHQ210KA01N&utm_medium= pm-display&utm_source= OSCHINA&source= pc_blog&utm_object= ai_NA&utm_content= ascend_wisdom_ad)

### **一个 docker host. 上会运行若干容器, 每个容器都需要 CPU, 内存和 I0 资源. 对于 KVM, VMware 等虚拟化技术, 用户可以控制分配多少 CPU, 内存资源给每个虚拟机. 对于容器, Docker 也提供了类似的机制避免某个容器因占用太多资源而影响其他容器乃至整个 host
的性能.**
 

# 内存限额

与操作系统类似, 容器可以使用的内存包括两部分: 物理内存和 Swap.

Docker 通过下面两组参数来控制容器内存的使用量

(1)-m 或 --memory : 设置内存的使用限额, 例如 100MB,2GB

(2)--memory-swap: 设置内存 + swawp 的使用限额

当我们执行如下的命令时

```
docker run -m 200M --memory-swap=300M ubuntu
```

其含义是允许该容器最多使用 200MB 的内存和 100MB 的 swap. 默认情况下, 上面两组参数为 -1, 即对容器内存和 swap 的使用没有限制.

下面我们将使用 progrium/stress 镜像来学习如何为容器分配内存. 该镜像可用于对容器执行压力测试. 执行如下命令:
 

```
docker run -it -m 200M --memory-swap=300M progrium/stress --vm 1 --vm-bytes 208M
```

*   \--vm1: 启动 1 个内存工作线程.
*   \--vm-bytes 280M: 每个线程分配 280MB 内存.

**运行如下图结果**

! [](https://oscimg.oschina.net/oscnet/7b0a783dff36e5676ffc7ab8713cd7b095a.png)

因为 280MB 在可分配的范围 (300MB) 内, 所以工作线程能够正常工作, 其过程是:
(1) 分配 280MB 内存.
(2) 释放 280MB 内存.
(3) 再分配 280MB 内存.
(4) 再释放 280MB 内存.
(5) 一 - 直循环.....
如果让工作线程分配的内存超过 300MB, 结果如图

```
docker run -it -m 200M --memory-swap=300M progrium/stress --vm 1 --vm-bytes 310M
```

! [](https://oscimg.oschina.net/oscnet/932968d7207523c13fc6238bf0c653afdc9.png)

分配的内存超过限额, stress 线程报错, 容器退出.
如果在启动容器时只指定 -m 而不指定 -memoryswap, 那么 -memory-swap 默认为 -m 的两倍, 比如:
 

```
docker run -it -m 200M ubuntu
```

容器最多使用 200M 绒里内存和 200swap

## CPU 限额

**默认设置下, 所有容器可以平等地使用 host CPU 资源并且没有限制**.


**Docker 可以通过 -c 或 -pu-shares 设置容器使用 CPU 的权重. 如果不指定, 默认值为 1024.**
与内存限额不同, 通过 -c 设置的 cpu share 并不是 CPU 资源的绝对数量, 而是一个相对的权重值. 某个容器最终能分配到的 CPU 资源取决于它的 cpu share 占所有容器 cpu share 总和的比例.
换句话说: 通过 cpu share 可以设置容器使用 CPU 的优先级.

比如在 host 中启动了两个容器:

```
docker run --name "cont_A" -c 1024 ubuntu docker run --name "cont_B" -c 512 ubuntu
```

**containerA 的 cpu share 1024, 是 containerB 的两倍. 当两个容器都需要 CPU 资源时, containerA 可以得到的 CPU 是 containerB 的两倍.
需要特别注意的是, 这种按权重分配 CPU 只会发生在 CPU 资源紧张的情况下. 如果 containerA 处于空闲状态, 这时, 为了充分利用 CPU 资源, containerB 也可以分配到全部可用的 CPU.**


下面我们继续用 progrium/stress 做实验.

###
(1) 启动 (container\_ A, cpu share 为 1024
 

```
docker run --name "cont_A" -it -c 1024  progrium/stress --cpu 1
```

! [](https://oscimg.oschina.net/oscnet/163ebc3f42cc3aa349562ef8a9243e9d33f.png)

\--cpu 用来设置工作线程的数量. 因为当前 host 只有 1 颗 CPU, 所以一个工作线程就能将 CPU 压满. 如果 host 有多颗 CPU, 则需要相应增加 --cpu 的数量.
 

### (2) 启动 (container\_B, cpu share 为 512

```
docker run --name "cont_B" -it -c 512  progrium/stress --cpu 1
```

! [](https://oscimg.oschina.net/oscnet/c391c3a98c144e8159b8af8a9e2a54b52e3.png)

### (3) 在 host 中执行 top, 查看容器对 CPU 的使用情况,
 

! [](https://oscimg.oschina.net/oscnet/a7044f99fc122e5cdd07b763d99a90248e6.png)

```
ps aux| head -1; ps aux| sort -k3nr | head -4
```

! [](https://oscimg.oschina.net/oscnet/527396693eba4f602f477c08ede6a299778.png)

### **containerA 消耗的 CPU 是 containerB 的两倍.**
 

### (4) 现在暂停 container. A

### ! [](https://oscimg.oschina.net/oscnet/2e5426ae5634f7e0599c68990a5e09b8a8f.png)

### (5) top 显示 containerB 在 containerA 空闲的情况下能够用满整颗 CPU

### ! [](https://oscimg.oschina.net/oscnet/ec9d68236aa24e2eaf0d2d9d216f208e807.png)

# Block IO 带宽限额

Block 10 是另一种可以限制容器使用的资源.Block I0 指的是磁盘的读写, docker 可通过设置权重, 限制 bps 和 iops 的方式控制容器读写磁盘的带宽, 下 面分别讨论.
**注: 目前 Block I0 限额只对 direct IO (不使用文件缓存) 有效.**
Block IO 权重
默认情况下, 所有容器能平等地读写磁盘, 可以通过设置**\-blkio-weight**参数来改变容器 block Io 的优先级.
**\-blkio-weight**与 --cpu-shares 类似, 设置的是相对权重值, 默认为 500. 在下面的例子中, containerA 读写磁盘的带宽是 containerB 的两倍.

```
docker run -it --name cont_A --blkip-weight 600 ubuntu
docker run -it --name cont_B --blkip-weight 300 ubuntu
```

### 限制 bps 和 iops

bps 是 byte per second , 每秒读写的数量

iops 是 io per second , 每秒 IO 的次数

可以同过下面的参数控制容器的 bps 和 iops;

*   \--device-read-bps: 限制读某个设备的 bps.
*   \--devce-write-bps: 限制写某个设备的 bps.
*   \--device- read-iops: 限制读某个设备的 iops.
*   \--device-write-iops: 限制写某个设备的 iops.
     

下面这个例子限制容器写 /dev/sda 的速率为 30 MB/s:
 

```
[root@kvm ~]# docker run -it --device-write-bps /dev/sda:30MB ubuntu
root@10845a98036e:/# time dd if=/dev/zero of= test.out bs=1M count=800 oflag= direct
800+0 records in
800+0 records out
838860800 bytes (839 MB, 800 MiB) copied, 26.6211 s, 31.5 MB/s

real	0m26.623s
user	0m0.000s
sys	0m0.106s
root@10845a98036e:/#
``````
docker run -it --device-write-bps /dev/sda:30MB ubuntu
```

**有限制**

! [](https://oscimg.oschina.net/oscnet/af31783aa4659f4c357f343ef8e7a634f7f.png)

**没有限制**

! [](https://oscimg.oschina.net/oscnet/353f961176d198de75c962494d1f77a601c.png)

通过 dd 测试在容器中写磁盘的速度. 因为容器的文件系统是在 host /dev/sda. 上的, 在容器中写文件相当于对 host /dev/sda 进行写操作. 另外, oflag= -direct 指定用 direct I0 方式写文件, 这样 --device-write-bps 才能生效.
 

看到, 没有限速的话, 速度很快,

其他参数, 大家也可以试试

展开阅读全文

[bps](https://www.oschina.net/p/bps)

本文转载自: https://www.cnblogs.com/heian99/p/12585725.html

举报

打赏

0 赞

0 收藏

微信 QQ 微博

分享

### 作者的其它热门文章

[Docker 安装 JIRA 和 Confluence(破解版)](https://my.oschina.net/u/4270238/blog/4074414 "Docker 安装 JIRA 和 Confluence(破解版)")

[安装 VMware16 兼容 Hyper-v+ WSL2+ Docker+ 解决 0x80370102 报错](https://my.oschina.net/u/4270238/blog/4074394 " 安装 VMware16 兼容 Hyper-v+ WSL2+ Docker+ 解决 0x80370102 报错 ")

[Docker-compose 搭建 Elasticsearch 集群及 kibana](https://my.oschina.net/u/4270238/blog/4074306 "Docker-compose 搭建 Elasticsearch 集群及 kibana")

[Centos8 安装最新稳定版 Docker-ce 出现 package docker-ce-3:19.03.8-3.el7.x86\_64 requires containerd.io >= 1....](https://my.oschina.net/u/4270238/blog/4074418 "Centos8 安装最新稳定版 Docker-ce 出现 package docker-ce-3:19.03.8-3.el7.x86_64 requires containerd.io >= 1....")
