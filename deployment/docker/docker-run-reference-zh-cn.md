---
title: Docker 资源限制
date: 2021-03-07 18:00:00
tags: 'Docker'
categories:
  - ['部署', '容器化']
permalink: docker-limit-resource
photo:
---

[docker-run-reference-zh-cn](https://deepzz.com/post/docker-run-reference.html)

如何执行 docker run, docker run 命令参考文档 | Deepzz's Blog

[](/)

# [Deepzz](/)

唯爱与美食不可辜负也 :-D

*   [首页](/)
*   [专题](/series.html)
*   [归档](/archives.html)
*   [友链](/post/blogroll.html)
*   [关于](/post/about.html)

[Twitter](//twitter.com/deepzz0 "Twitter") [RSS](//deepzz.com/rss.html "RSS 订阅 ") [Search](/search.html " 站内搜索 ")

Jan 05, 2017

Jan 01, 0001

[0 Comments](#comments)

# 如何执行 docker run, docker run 命令参考文档

**预览目录**

*   [命令参数](#toc_0)
*   [网络设置](#toc_1)
*   [重启策略](#toc_2)
*   [退出状态](#toc_3)
*   [清除](#toc_4)
*   [安全配置](#toc_5)
*   [指定自定义 cgroups](#toc_6)
*   [资源的运行时约束](#toc_7)
*   [覆盖 Dockerfile 镜像默认值](#toc_8)

当安装好了 `Docker` 之后, 如何运行一个 docker 容器是个问题. 本篇文章讲诉如何在命令行终端执行 `docker run` 命令, 其主要内容来自官方文档 [Docker run reference](https://docs.docker.com/engine/reference/run/). 欢迎指出错误, 万分感谢. `docker run` 的基本格式是:

```
$ docker run [OPTIONS] IMAGE[: TAG|@DIGEST] [COMMAND] [ARG...]
```

简单说下,`IMAGE[: TAG|@DIGEST]` 是镜像名称, 在这之前的 `[OPTIONS]` 是 docker 配置, 如:`-d`,`-p 8080:8080` 等, 之后 `[COMMAND] [ARG...]` 是在容器启动时执行的命令 (容器内执行), 如:`echo hello world`.

> ` 注意 `, 在执行 `docker run` 命令时, 或许需要加上 `sudo`.
>
> ```
>
>  $ sudo groupadd docker                       #创建 docker 组, 免 sudo 执行 docker 命令
>   $ sudo usermod -aG docker ${username}        #将用户 username 添加到 docker 组, 重启 docker 服务, 注销, 登录
>
> ```

好的, 下面我们一一学习 `docker run` 的各个配置项.

### 命令参数

**后台运行**

默认情况下,`docker run` 命令是前台运行的, 如 `ctrl+ c` 会使容器停止. 使用 `-d` 参数, 容器后台运行.

```
$ docker run -d -p 80:80 my_image nginx -g 'daemon off;'
```

这里需要讲一下, 在前台模式下 (未加 -d). 容器附加 stdin, stdout 和 stderr 到当前控制台, 甚至可以模拟一个 TTY. 但这些都可以配置:

```
-a= []           : Attach to `STDIN`, `STDOUT` and/or `STDERR`
-t              : Allocate a pseudo-tty
--sig-proxy= true: Proxy all received signals to the process (non-TTY mode only)
-i              : Keep STDIN open even if not attached
```

如:

```
$ docker run -a stdin -a stdout -i -t ubuntu /bin/bash
```

这里分配了一个 TTY, 连接了 stdin 和 stdout . 我们可以容易的在当前控制台与容器交互.`-i -t` 后面会写为 `-it`, 请悉知.

**容器识别**

在很多操作中, 我们需要操作容器. 因此需要一个来标识容器, 如:`ID`,`Name`:

```
$ docker run -d -p 27017:27017 --name eidb mongo:3.2
...

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
3ed7b8f338ad        mongo:3.2           "/entrypoint.sh mo..."   7 weeks ago         Up 22 seconds       0.0.0.0:27017->27017/tcp   eidb
```

通过这两种标识符, 我们可以操作容器. 通过配置 `--name` 来指定容器名称, 否则启动容器时会分配一个随机的名称. 指定名称不仅容易记, 而且在容器互连的时候有用.

当然为了帮助自动化处理, 可以将容器 ID 写入文:

```
--cidfile="": Write the container ID to the file
```

**镜像**

运行镜像可以通过 `Image[: tag]` 和 `Image[@digest]`, 如:

```
$ docker run ubuntu:14.04
$ docker run alpine@sha256:9cacb71397b640eca97488cf08582ae4e4068513101088e9f96c9814bfda95e0 date
```

**PID 设置**

默认情况下, 容器启用了 PID 命名空间来提供进程隔离.PID 命名空间删除了系统进程的视图, 并允许重用包括 `pid 1` 的进程标识.

某些情况下, 你希望共享主机的进程命名空间, 允许容器进程查看系统所有进程. 如: 构建了调试工具 (strace/gdb) 镜像, 并想在容器中使用它们.

```
# 启动一个 redis
$ docker run --name my-redis -d redis

# 使用一个 strace 容器来调试 redis 容器
$ docker run -it --pid= container: my-redis my_strace_docker_image bash
$ strace -p 1
```

**UTS 设置**

```
--uts=""  : Set the UTS namespace mode for the container,
       'host': use the host's UTS namespace inside the container
```

UTS 命名空间用于设置主机名和对该命名空间中正在运行的进程可见的域. 默认情况下, 所有容器 (包括带有–network= host 都有自己的 UTS 命名空间.host 设置将导致容器使用与主机相同的 UTS 命名空间.

**IPC 设置**

```
--ipc=""  : Set the IPC mode for the container,
             'container: <name| id>': reuses another container's IPC namespace
             'host': use the host's IPC namespace inside the container
```

默认情况下, 所有容器都启用了 IPC 命名空间.

IPC(POSIX / SysV IPC) 命名空间提供了命名的共享内存段, 信号量和消息队列的分离.

共享内存段用于以内存速度加速进程间通信, 而不是通过管道或通过网络堆栈. 共享内存通常由数据库和定制 (通常是 C/OpenMPI, C++/using boost libraries) 的高性能应用程序用于科学计算和金融服务行业. 如果这些类型的应用程序分成多个容器, 您可能需要共享容器的 IPC 机制.

### 网络设置

```
--dns= []           : Set custom dns servers for the container
--network="bridge" : Connect a container to a network
                      'bridge': create a network stack on the default Docker bridge
                      'none': no networking
                      'container: <name| id>': reuse another container's network stack
                      'host': use the Docker host network stack
                      '<network-name> |<network-id>': connect to a user-defined network
--network-alias= [] : Add network-scoped alias for the container
--add-host=""      : Add a line to /etc/hosts (host: IP)
--mac-address=""   : Sets the container's Ethernet device's MAC address
--ip=""            : Sets the container's Ethernet device's IPv4 address
--ip6=""           : Sets the container's Ethernet device's IPv6 address
--link-local-ip= [] : Sets one or more container's Ethernet device's link local IPv4/IPv6 addresses
```

默认情况下, 所有容器都启用了 network, 可以进行任何连接. 我们也可以禁用 `docker run --network none`, 这种情况我们只能通过 `STDIN` 和 `STDOUT` 执行 I/O 操作了.

容器默认使用与主机相同的 DNS 服务器, 可以通过 `--dns` 覆盖.

默认使用分配给容器的 IP 地址生成 MAC 地址. 可通过 `--mac-address`(格式:`12:34:56:78:9a: bc`), Docker 不回检查手动设置的 MAC 地址的唯一性.

支持的网络:

| Network | Description |
| --- | --- |
| none | 禁用容器的网络 |
| bridge (default) | 通过 veth 接口将容器连接到网桥. |
| host | 在容器中使用主机的网络堆栈. |
| container: <name | id> |
| NETWORK | 将容器连接到用户创建的网络 (使用 Docker `docker network create` 命令) |

**管理 /etc/hosts**

如果你想在容器中的 `/etc/hosts` 添加一条记录, 手动添加可不是一条出路. 通过 `--add-host` 将会起到同样的作用:

```
$ docker run -it --add-host db-static:86.75.30.9 ubuntu cat /etc/hosts
172.17.0.22     09d03f76bf2c
fe00::0         ip6-localnet
ff00::0         ip6-mcastprefix
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
86.75.30.9      db-static
```

如果容器连接到用户定义的网络, 该网络中的容器的 `/etc/hosts` 文件都将更新.

### 重启策略

在启动容器的时候可以通过 `--restart` 指定在容器退出后的重启策略.

支持如下重启策略:

| Policy | Result |
| --- | --- |
| no | 默认值, 不重启 |
| on-failure\[: max-retries\] | 容器以非 0 状态退出时重启, 可选重启最大尝试次数 |
| always | 总是重新启动, 无限期尝试重启 |
| unless-stopped | 不管退出状态如何, 始终重启. 但如果容器早就被停止, 守护程序启动时将不回重启它. |

在每次重启时, 都将不断增加延迟 (总是之前的 2 倍, 从 100ms 开始). 如:100ms,200ms,400ms…等, 知道达到 `on-failure` 限制, 或 `docker stop`,`docker rm -f`.

通过 `docker inspect` 获取重启次数, 如容器 `my-container`:

```
$ docker inspect -f "{{ .RestartCount }}" my-container
# 2
```

例子:

```
# 退出后总是重启
$ docker run --restart= always redis
# 非 0 状态退出, 最大尝试 10 次
$ docker run --restart= on-failure:10 redis
```

### 退出状态

退出状态码反映了容器为什么运行失败或者退出. 当以非 0 状态退出时, 退出代码遵循 `chroot` 标准, 如:

```
# 125 Docker 守护进程错误
$ docker run --foo busybox; echo $?
# flag provided but not defined: --foo
  See 'docker run --help'.
  125

# 126 包含的命令不能被调用
$ docker run busybox /etc; echo $?
# docker: Error response from daemon: Container command '/etc' could not be invoked.
  126

# 127 找不到包含的命令
$ docker run busybox foo; echo $?
# docker: Error response from daemon: Container command 'foo' not found or does not exist.
  127

# 否则, 包含命令的 退出代码
$ docker run busybox /bin/sh -c 'exit 3'; echo $?
# 3
```

### 清除

默认情况, 容器的文件系统在退出后依然存在, 这利于调试. 如果你想让容器临时运行一个命令后就清楚, 则使用 `--rm` 配置:

```
--rm= false: Automatically remove the container when it exits (incompatible with -d)
```

> 使用 `--rm` 时, 容器卷也会删除. 如, 使用 `docker run --rm -v /foo -v awesome:/bar busybox top`,`/foo` 的卷将被删除, 但是 `/bar` 的卷不会.

### 安全配置

```
--security-opt="label= user: USER"     : Set the label user for the container
--security-opt="label= role: ROLE"     : Set the label role for the container
--security-opt="label= type: TYPE"     : Set the label type for the container
--security-opt="label= level: LEVEL"   : Set the label level for the container
--security-opt="label= disable"       : Turn off label confinement for the container
--security-opt="apparmor= PROFILE"    : Set the apparmor profile to be applied to the container
--security-opt="no-new-privileges"   : Disable container processes from gaining new privileges
--security-opt="seccomp= unconfined"  : Turn off seccomp confinement for the container
--security-opt="seccomp= profile.json": White listed syscalls seccomp Json file to be used as a seccomp filter
```

您可以通过指定 `--security-opt` 标志覆盖每个容器的默认标签方案. 在以下命令中指定级别允许您在容器之间共享相同的内容.

```
$ docker run --security-opt label= level: s0: c100, c200 -it fedora bash
```

> ` 注意 `: 目前不支持 MLS 标签的自动翻译.

要禁用此容器的安全标号, 而不使用 `--privileged` 标志运行, 请使用以下命令:

```
$ docker run --security-opt label= disable -it fedora bash
```

如果要对容器中的进程使用更严格的安全策略, 则可以为容器指定备用类型. 您可以运行一个容器, 只允许通过执行以下命令在 Apache 端口监听:

```
$ docker run --security-opt label= type: svirt_apache_t -it centos bash
```

> ` 注意 `: 您必须编写定义 svirt\_apache\_t 类型的策略.

如果要防止容器进程获得额外的权限, 可以执行以下命令:

```
$ docker run --security-opt no-new-privileges -it centos bash
```

这意味着提出诸如 `su` 或 `sudo` 权限将不再工作. 它也导致任何 seccomp 过滤器以后应用, 在权限被删除后, 这可能意味着您可以有一个更严格的过滤器集. 有关更多详细信息, 请参阅 [内核文档](https://www.kernel.org/doc/Documentation/prctl/no_new_privs.txt).

### 指定自定义 cgroups

使用 `--cgroup-parent` 标志, 可以传递特定的 cgroup 来运行容器. 这允许您自己创建和管理 cgroup. 您可以为这些 cgroup 定义自定义资源, 并将容器放在同一父组下.

### 资源的运行时约束

可以调整容器的性能参数:

| Option | Description |
| --- | --- |
| \-m, –memory="" | 内存限制 (格式: \[\]).Number 是正整数.Unit 可以是 b, k, m, or g. 最小是 4M. |
| –memory-swap="" | 总内存限制 (memory + swap, 格式: \[\]). 正整数, b, k, m, g. |
| –memory-reservation="" | 内存软限制 (格式: \[\]). 同上. |
| –kernel-memory="" | 内核内存限制 (格式: \[\]). 同上. 最小是 4M. |
| \-c, –cpu-shares=0 | CPU 份额 (相对权重) |
| –cpu-period=0 | 限制 CPU CFS(完全公平调度) 期间 |
| –cpuset-cpus="" | 允许执行的 CPU(0-3,0,1) |
| –cpuset-mems="" | 允许执行的内存节点 (MEM) (0-3,0,1). 仅在 NUMA 系统上有效. |
| –cpu-quota=0 | 限制 CPU CFS(完全公平计划程序) 配额 |
| –blkio-weight=0 | 块 IO 权重 (相对权重) 接受 10 和 1000 之间的权重值. |
| –blkio-weight-device="" | 块 IO 权重 (相对设备权重, 格式: DEVICE\_NAME: WEIGHT ) |
| –device-read-bps="" | 限制设备的读取速率 (格式: :\[\] ).Number 是一个正整数. 单位可以是 kb, mb 或 gb. |
| –device-write-bps="" | 限制对设备的写入速率 (格式: :\[\] ). 同上 |
| –device-read-iops="" | 限制设备的读取速率 (每秒 IO) (格式: : ). Number 是一个正整数. |
| –device-write-iops="" | 限制写入速率 (每秒 IO) 到设备 (格式: : ). Number 是一个正整数. |
| –oom-kill-disable= false | 是否禁用容器的 OOM 杀手. |
| –oom-score-adj=0 | 调整容器的 OOM 偏好 (-1000 到 1000) |
| –memory-swappiness="" | 调整容器的内存 swappiness 行为. 接受介于 0 和 100 之间的整数. |
| –shm-size="" | `/dev/shm` 大小. 格式为 `<number> <unit>`.number 必须大于 0. 单位是可选的, 可以是 b(字节), k(千字节), m(兆字节) 或 g(吉字节). 如果省略该单位, 系统将使用字节. 如果你完全忽略大小, 系统使用 64m. |

**用户内存约束**

| Option | Result |
| --- | --- |
| memory= inf, memory-swap= inf (default) | 容器没有内存限制. 容器可以根据需要使用尽可能多的内存. |
| memory= L<inf, memory-swap= inf | (指定内存并将内存交换设置为 -1 ) 容器不允许使用超过 L 字节的内存, 但可以使用所需的交换 (如果主机支持交换内存). |
| memory= L<inf, memory-swap=2\*L | (指定没有内存交换的内存) 容器不允许使用超过 L 个字节的内存, 交换加内存使用是其两倍. |
| memory= L<inf, memory-swap= S<inf, L<= S | (指定内存和内存交换) 容器不允许使用超过 L 字节的内存, 交换加内存使用受限于 S. |

例子:

```
$ docker run -it ubuntu:14.04 /bin/bash
```

无内存限制, 使用它们需要的尽可能多内存和交换内存.

```
$ docker run -it -m 300M --memory-swap -1 ubuntu:14.04 /bin/bash
```

我们设置内存限制和禁用交换内存限制, 这意味着容器中的进程可以使用 300M 内存和尽可能多的交换内存 (如果主机支持交换内存).

```
$ docker run -it -m 300M ubuntu:14.04 /bin/bash
```

我们只设置内存限制, 这意味着容器中的进程可以使用 300M 内存和 300M 交换内存, 默认情况下, 总虚拟内存大小 (-memory-swap) 将设置为内存的两倍, 在这种情况下, 内存 + 交换将是 2 \* 300M, 因此进程可以使用 300M 交换内存.

```
$ docker run -it -m 300M --memory-swap 1G ubuntu:14.04 /bin/bash
```

我们设置内存和交换内存, 因此容器中的进程可以使用 300M 内存和 700M 交换内存.

内存预留是一种内存软限制, 允许更大的内存共享. 在正常情况下, 容器可以根据需要使用尽可能多的内存, 并且只受到使用 `-m/--memory` 选项设置的硬限制. 当设置内存预留时, Docker 检测内存争用或低内存, 并强制容器将其消耗限制为预留限制.

始终将内存预留值设置为低于硬限制, 否则硬限制优先. 预约 0 与不设置预约相同. 默认情况下 (未设置预留), 内存预留与硬内存限制相同.

内存预留是软限制功能, 不保证不会超出限制. 相反, 该功能尝试确保, 当内存严重争用时, 内存是基于预留提示 / 设置分配的.

以下示例将内存 (`-m`) 限制为 500M, 并将内存预留设置为 200M.

```
$ docker run -it -m 500M --memory-reservation 200M ubuntu:14.04 /bin/bash
```

在此配置下, 当容器消耗的内存大于 200M 且小于 500M 时, 下一个系统内存回收尝试将容器内存缩小到 200M 以下.

以下示例将内存预留设置为 1G, 但没有硬内存限制.

```
$ docker run -it --memory-reservation 1G ubuntu:14.04 /bin/bash
```

容器可以使用所需的内存. 内存预留设置确保容器长时间不消耗太多内存, 因为每次内存回收都会缩减容器对预留的消耗.

默认情况下, 如果发生内存不足 (OOM) 错误, 内核会在容器中终止进程. 要更改此行为, 请使用 `--oom-kill-disable` 选项. 仅在您还设置了 `-m/--memory` 选项的容器上禁用 OOM 杀手. 如果未设置 `-m` 标志, 则这可能导致主机运行内存不足, 并且需要杀死主机的系统进程以释放内存.

以下示例将内存限制为 100M, 并禁用此容器的 OOM 杀手级:

```
$ docker run -it -m 100M --oom-kill-disable ubuntu:14.04 /bin/bash
```

以下示例说明了使用该标志的危险方法:

```
$ docker run -it --oom-kill-disable ubuntu:14.04 /bin/bash
```

容器有无限的内存, 可以导致主机运行内存, 并需要杀死系统进程释放内存.`--oom-score-adj` 参数可以更改为选择当系统内存不足时哪些容器将被杀死的优先级, 负分数使得它们不太可能被杀死为阳性.

**内核内存约束**

内核内存从根本上不同于用户内存, 因为内核内存不能被交换出来. 无法交换使得容器可能通过消耗过多的内核内存来阻止系统服务. 内核内存包括:

*   stack pages
*   slab pages
*   sockets memory pressure
*   tcp memory pressure

您可以设置内核内存限制来约束这些类型的内存. 例如, 每个进程都消耗一些堆栈页面. 通过限制内核内存, 可以防止在内核内存使用率过高时创建新进程.

内核内存从不完全独立于用户内存. 相反, 你在用户内存限制的上下文中限制内核内存. 假设 "U" 是用户内存限制,"K" 是内核限制. 有三种可能的方法来设置限制:

| Option | Result |
| --- | --- |
| U != 0, K = inf (default) | 这是在使用内核内存之前已经存在的标准内存限制机制. 内核内存被完全忽略. |
| U != 0, K < U | 内核内存是用户内存的一个子集. 此设置在过度使用每个 cgroup 的内存总量的部署中非常有用. 绝对不推荐过度使用内核内存限制, 因为框仍然可以用完不可回收的内存. 在这种情况下, 您可以配置 K, 以便所有组的总和永远不会大于总内存. 然后, 以系统的服务质量为代价自由设置 U. |
| U != 0, K > U | 因为内核内存费用也被馈送到用户计数器, 并且为两种存储器的容器触发回收. 此配置为管理员提供了统一的内存视图. 它也适用于只想跟踪内核内存使用的人. |

例子:

```
$ docker run -it -m 500M --kernel-memory 50M ubuntu:14.04 /bin/bash
```

我们设置内存和内核内存, 所以容器中的进程总共可以使用 500M 内存, 在这个 500M 内存中, 它可以是 50M 内核内核.

```
$ docker run -it --kernel-memory 50M ubuntu:14.04 /bin/bash
```

我们设置没有 -m 的内核内存, 因此容器中的进程可以使用尽可能多的内存, 但是它们只能使用 50M 内核内存.

**Swappiness 约束**

默认情况下, 容器的内核可以交换一定百分比的匿名页面. 要为容器设置此百分比, 请指定介于 0 和 100 之间的 `--memory-swappiness` 值. 值为 0 将关闭匿名页面交换. 值 100 将所有匿名页面设置为可交换. 默认情况下, 如果不使用 `--memory-swappiness`, 内存交换值将从父代继承.

例如, 您可以设置:

```
$ docker run -it --memory-swappiness=0 ubuntu:14.04 /bin/bash
```

当您希望保留容器的工作集并避免交换性能损失时, 设置 `--memory-swappiness` 选项非常有用.

**CPU 共享约束**

默认情况下, 所有容器获得相同比例的 CPU 周期. 这个比例可以通过改变容器的 CPU 份额权重相对于所有其他运行容器的权重来修改.

要从缺省值 1024 修改比例, 请使用 `-c` 或 `--cpu-sharessharess` 标志将权重设置为 2 或更高. 如果设置为 0, 系统将忽略该值, 并使用默认值 1024.

该比例将仅在 CPU 密集型进程正在运行时应用. 当一个容器中的任务空闲时, 其他容器可以使用剩余 CPU 时间. 实际的 CPU 时间将取决于系统上运行的容器数量.

例如, 考虑三个货柜, 一个人的 1024 CPU 份额和另外两人有一个 512 CPU 份额设置时在所有三个容器进程尝试使用 100%的 CPU, 第一个集装箱将获得的 50%总的 CPU 时间. 如果使用 1024 的 CPU 股添加第四容器, 第一容器只获取 CPU 的 33%. 剩余的容器接收 16.5%,16.5%和 CPU 的 33%.

在多核系统中, CPU 时间的份额被分发所有 CPU 内核上. 即使容器被限制的 CPU 时间小于 100%时, 它可以使用每个单独的 CPU 核心的 100%.

例如, 考虑具有三个以上内核的系统. 如果启动一个容器以运行一个过程, 和另一个容器与运行两个过程, 这可能导致的 CPU 份额以下划分:`{ C0 } -c=512` `{ C1 } -c=1024`

```
PID    container    CPU CPU share
100    {C0}     0   100% of CPU0
101    {C1}     1   100% of CPU1
102    {C1}     2   100% of CPU2
```

**CPU 时间段的约束**

默认 CPU CFS(完全公平调度器) 周期为 100ms. 我们可以用 `--cpu-period` 设置 CPU 的时间限制容器的 CPU 使用率. 通常 `--cpu-period` 应该一起工 `--cpu-quota`.

例子:

```
$ docker run -it --cpu-period=50000 --cpu-quota=25000 ubuntu:14.04 /bin/bash
```

如果有 1 个 CPU, 这意味着该容器可以得到的运行时每 50ms 50%的 CPU 价值.

欲了解更多信息, 请参阅 [带宽限制 CFS 文件](https://www.kernel.org/doc/Documentation/scheduler/sched-bwc.txt).

**Cpuset 约束**

我们可以设置的 CPU 中, 使集装箱执行.

例子:

```
$ docker run -it --cpuset-cpus="1,3" ubuntu:14.04 /bin/bash
```

这意味着在容器进程可以在 CPU 1 和 CPU 3 执行.

```
$ docker run -it --cpuset-cpus="0-2" ubuntu:14.04 /bin/bash
```

这意味着在容器进程可以在 CPU 0, CPU 1 和 CPU 2 来执行.

我们可以设置在允许容器执行 MEMS. 只有有效的 NUMA 系统.

例子:

```
$ docker run -it --cpuset-mems="1,3" ubuntu:14.04 /bin/bash
```

该实施例限制了工序在容器到只使用存储器从存储器节点 1 和 3.

```
$ docker run -it --cpuset-mems="0-2" ubuntu:14.04 /bin/bash
```

该实施例限制了工序在容器到只使用存储器从存储器节点 0,1 和 2.

**CPU 配额限制**

该 `--cpu-quota` 标志限制了容器的 CPU 使用率. 默认值 0 允许容器采取的 CPU 资源 (1 个 CPU) 的 100%. 食物安全中心 (完全公平调度器) 处理资源分配的执行进程, 是内核使用默认的 Linux 计划. 这个值设置为 50000 到容器限制到 CPU 资源的 50%. 对于多 CPU 的调整 `--cpu-quota` 是必要的. 欲了解更多信息, 请参阅 [带宽限制 CFS 文件](https://www.kernel.org/doc/Documentation/scheduler/sched-bwc.txt).

**块 IO 带宽 (Blkio) 约束**

默认情况下, 所有容器得到的块 IO 带宽 (blkio) 相同的比例. 这个比例是 500. 若要修改此比例, 容器的 blkio 相对权重更改为使用所有其他正在运行的容器的加权 `--blkio-weight` 标志.

> 注: 该 blkio 权重设置仅适用于直接 IO. 目前不支持缓冲 IO.

该 `--blkio-weight` 标志可以加权到 10 之间的值设置为 1000. 例如, 下面的命令创建两个容器具有不同 blkio 重量:

```
$ docker run -it --name c1 --blkio-weight 300 ubuntu:14.04 /bin/bash
$ docker run -it --name c2 --blkio-weight 600 ubuntu:14.04 /bin/bash
```

如果方框 IO 在两个容器同时, 通过, 例如:

```
$ time dd if=/mnt/zerofile of= test.out bs=1M count=1024 oflag= direct
```

你会发现的时间比例是相同的两个容器的重量 blkio 的比例.

该 `--blkio-weight-device="DEVICE_NAME: WEIGHT"` 标志设置一个特定设备的重量. 该 `DEVICE_NAME: WEIGHT` 是一个包含冒号分隔的设备名称和重量的字符串. 例如, 要设置 `/dev/sda` 的设备的重量 200:

```
$ docker run -it \ --blkio-weight-device "/dev/sda:200" \ ubuntu
```

如果同时指定了 `--blkio-weight` 和 `--blkio-weight-device`, 多克尔使用 `--blkio-weight` 的缺省权重, 并使用 `--blkio-weight-device` 可覆盖此默认与特定设备上的新的价值. 下面的示例使用的缺省权重 300 并覆盖这个默认的 `/dev/sda` 体重设置为 200:

```
$ docker run -it \ --blkio-weight 300 \ --blkio-weight-device "/dev/sda:200" \ ubuntu
```

该 `--device-read-bps` 旗从设备限制读取速率 (每秒字节). 例如, 下面的命令创建一个容器, 并限制读取速率 1mb 从每秒 `/dev/sda`:

```
$ docker run -it --device-read-bps /dev/sda:1mb ubuntu
```

该 `--device-write-bps` 标志限制的写速率 (每秒字节) 的装置. 例如, 下面的命令创建一个容器并限制写入速率, 以 1mb 每秒 `/dev/sda`:

```
$ docker run -it --device-write-bps /dev/sda:1mb ubuntu
```

两个标志采取的限制 `<device-path>: <limit> [unit]` 格式. 读取和写入速率必须是一个正整数. 您可以在指定的速度 kb(千字节), mb(兆字节), 或 gb(千兆字节).

该 `--device-read-iops` 标志限制读取设备速率 (每秒 IO). 例如, 下面的命令创建一个容器并限制读取速率, 以 1000 从 IO 每秒 `/dev/sda`:

```
$ docker run -ti --device-read-iops /dev/sda:1000 ubuntu
```

该 `--device-write-iops` 标志限制写入速度 (每秒 IO) 到设备. 例如, 下面的命令创建一个容器并限制写入速率为 1000IO 每秒 `/dev/sda`:

```
$ docker run -ti --device-write-iops /dev/sda:1000 ubuntu
```

两个标志采取的限制 `<device-path>: <limit>` 格式. 读取和写入速率必须是一个正整数.

**其他组**

```
--group-add: Add additional groups to run as
```

默认情况下, docker 容器进程运行时查找指定用户的补充组. 如果想要向该组列表中添加更多, 则可以使用此标志:

```
$ docker run --rm --group-add audio --group-add nogroup --group-add 777 busybox id
uid=0(root) gid=0(root) groups=10(wheel),29(audio),99(nogroup),777
```

**运行时的特权和 Linux 的功能**

```
--cap-add: Add Linux capabilities
--cap-drop: Drop Linux capabilities
--privileged= false: Give extended privileges to this container
--device= []: Allows you to run devices inside the container without the --privileged flag.
```

默认情况下, Docker 容器是 " 无特权的 ", 并且不能, 例如, 在 Docker 容器中运行 Docker 守护进程. 这是因为默认情况下不允许容器访问任何设备, 但允许 " 特权 " 容器访问所有设备 (请参阅 [cgroups 设备](https://www.kernel.org/doc/Documentation/cgroup-v1/devices.txt) 上的文档).

当操作员执行 `docker run -privileged` 时, Docker 将启用对主机上的所有设备的访问, 以及在 AppArmor 或 SELinux 中设置一些配置, 以允许容器几乎所有的访问主机作为在 主办. 有关使用 `--privileged` 运行的其他信息, 请参阅 [Docker 博客](http://blog.docker.com/2013/09/docker-can-now-run-within-docker/).

如果你想限制访问特定的设备或设备则可以使用 `--device` 标志. 它允许你指定一个或多个设备, 将容器内的访问.

```
$ docker run --device=/dev/snd:/dev/snd ...
```

默认情况下, 该容器就能 `read`,`write` 以及 `mknod` 这些设备. 这可以使用第三覆盖 `: rwm` 组选项到每个 `--device` 标志:

```
$ docker run --device=/dev/sda:/dev/xvdc --rm -it ubuntu fdisk  /dev/xvdc

Command (m for help): q
$ docker run --device=/dev/sda:/dev/xvdc: r --rm -it ubuntu fdisk  /dev/xvdc
You will not be able to write the partition table.

Command (m for help): q

$ docker run --device=/dev/sda:/dev/xvdc: w --rm -it ubuntu fdisk  /dev/xvdc
    crash....

$ docker run --device=/dev/sda:/dev/xvdc: m --rm -it ubuntu fdisk  /dev/xvdc
fdisk: unable to open /dev/xvdc: Operation not permitted
```

除了 `--privileged`, 操作者可以有超过使用功能细粒度控制 `--cap-add` 和 `--cap-drop`. 缺省情况下, 多克尔具有该保持能力的缺省列表. 下表列出了默认情况下允许的, 可以删除 Linux 的功能选项.

| Capability Key | Capability Description |
| --- | --- |
| SETPCAP | 修改过程的能力. |
| MKNOD | 创建使用 mknod 的特殊文件 (2). |
| AUDIT\_WRITE | 写记录到内核审核日志. |
| CHOWN | 让随意更改文件的 UID 和 GID(见 CHOWN(2)). |
| NET\_RAW | 使用 RAW 和 PACKET 套接字. |
| DAC\_OVERRIDE | 旁路文件读取, 写入和执行权限检查. |
| FOWNER | 对通常需要进程的文件系统 UID 以匹配文件的 UID 的操作绕过权限检查. |
| FSETID | 修改文件时, 不要清除 set-user-ID 和 set-group-ID 权限位. |
| KILL | 发送信号的旁路权限检查. |
| SETGID | 对进程 GOD 和补充 GID 列表进行任意操作. |
| SETUID | 对进程 UID 进行任意操作. |
| NET\_BIND\_SERVICE | 将套接字绑定到 Internet 域特权端口 (端口号小于 1024). |
| SYS\_CHROOT | 使用 chroot(2), 更改根目录. |
| SETFCAP | 设置文件功能. |

下表显示了默认情况下未授予的功能, 可以添加.

| Capability Key | Capability Description |
| --- | --- |
| SYS\_MODULE | 加载和卸载内核模块. |
| SYS\_RAWIO | 执行 I / O 端口操作 (iopl(2) 和 ioperm(2)). |
| SYS\_PACCT | 使用 acct(2), 切换过程记帐开或关. |
| SYS\_ADMIN | 执行一系列系统管理操作. |
| SYS\_NICE | 提高进程 nice 值 (nice(2), setpriority(2)) 并更改任意进程的 nice 值. |
| SYS\_RESOURCE | 覆盖资源限制. |
| SYS\_TIME | 设置系统时钟 (settimeofday(2), stime(2), adjtimex(2)); 设置实时 (硬件) 时钟. |
| SYS\_TTY\_CONFIG | 使用 vhangup(2); 在虚拟终端上使用各种特权 ioctl(2) 操作. |
| AUDIT\_CONTROL | 启用和禁用内核审计; 更改审计过滤规则; 检索审计状态和过滤规则. |
| MAC\_OVERRIDE | 允许 MAC 配置或状态更改. 实现了 Smack LSM. |
| MAC\_ADMIN | 覆盖强制访问控制 (MAC). 为 Smack Linux 安全模块 (LSM) 实现. |
| NET\_ADMIN | 执行各种网络相关操作. |
| SYSLOG | 执行特权 syslog(2) 操作. |
| DAC\_READ\_SEARCH | 旁路文件读取权限检查和目录读取和执行权限检查. |
| LINUX\_IMMUTABLE | 设置 FS\_APPEND\_FL 和 FS\_IMMUTABLE\_FL i 节点标志. |
| NET\_BROADCAST | 使套接字广播, 并监听多播. |
| IPC\_LOCK | 锁存器 (mlock(2), mlockall(2), mmap(2), shmctl(2)). |
| IPC\_OWNER | 绕过权限检查 System V IPC 对象上的操作. |
| SYS\_PTRACE | 使用 ptrace(2) 跟踪任意进程. |
| SYS\_BOOT | 使用 reboot(2) 和 kexec\_load(2), 重新启动并加载一个新的内核供以后执行. |
| LEASE | 在任意文件上建立租约 (见 fcntl(2)). |
| WAKE\_ALARM | 触发将唤醒系统的东西. |
| BLOCK\_SUSPEND | 使用可以阻止系统挂起的功能. |

更多参考信息可在 [功能 (7) - Linux 手册页](http://man7.org/linux/man-pages/man7/capabilities.7.html)

这两个标志都支持值 `ALL`, 所以如果操作员想要有所有的能力, 但 `MKNOD` 他们可以使用:

```
$ docker run --cap-add= ALL --cap-drop= MKNOD ...
```

用于与网络栈进行交互, 而不是使用 `--privileged` 他们应该使用 `--cap-add= NET_ADMIN` 修改的网络接口.

```
$ docker run -it --rm ubuntu:14.04 ip link add dummy0 type dummy RTNETLINK answers: Operation not permitted
$ docker run -it --rm --cap-add= NET_ADMIN ubuntu:14.04 ip link add dummy0 type dummy
```

要安装一个 FUSE 基于文件系统, 你需要把两者结合起来 `--cap-add`, 并 `--device`:

```
$ docker run --rm -it --cap-add SYS_ADMIN sshfs sshfs sven@10.10.10.20:/home/sven /mnt
fuse: failed to open /dev/fuse: Operation not permitted
$ docker run --rm -it --device /dev/fuse sshfs sshfs sven@10.10.10.20:/home/sven /mnt
fusermount: mount failed: Operation not permitted
$ docker run --rm -it --cap-add SYS_ADMIN --device /dev/fuse sshfs
# sshfs sven@10.10.10.20:/home/sven /mnt
The authenticity of host '10.10.10.20 (10.10.10.20)' can't be established.
ECDSA key fingerprint is 25:34:85:75:25: b0:17:46:05:19:04:93: b5: dd:5f: c6.
Are you sure you want to continue connecting (yes/no)? yes
sven@10.10.10.20's password:
root@30aa0cfaf1b5:/# ls -la /mnt/src/docker
total 1516
drwxrwxr-x 1 1000 1000   4096 Dec  4 06:08 .
drwxrwxr-x 1 1000 1000   4096 Dec  4 11:46 ..
-rw-rw-r-- 1 1000 1000     16 Oct  8 00:09 .dockerignore
-rwxrwxr-x 1 1000 1000    464 Oct  8 00:09 .drone.yml
drwxrwxr-x 1 1000 1000   4096 Dec  4 06:11 .git
-rw-rw-r-- 1 1000 1000    461 Dec  4 06:08 .gitignore
....
```

默认的 seccomp 配置文件将调整为所选的功能, 以允许使用功能允许的设施, 所以你不应该调整这一点, 因为从 Docker 1.12. 在 Docker 1.10 和 1.11 中, 这没有发生, 可能需要使用自定义 seccomp 配置文件或在添加功能时使用 `--security-opt seccomp = unconfined`.

**日志驱动器 (-log 驱动器)**

容器可以具有一个与 Docker 宿主程序不同的日志记录的驱动程序. 在 `docker run` 时指定 `--log-driver= VALUE` 配置容器的日志驱动程序. 将支持以下选项:

| 驱动程序 | 描述 |
| --- | --- |
| none | 禁用的容器中的任何记录.`docker logs` 将无法使用. |
| json-file | 默认. 写 JSON 消息记录到文件. |
| syslog | 写入日志消息的系统日志. |
| journald | 写入日志消息 journald. |
| gelf | Graylog 扩展日志格式 (GELF) 日志驱动. 写入日志消息发送到端点 GELF 或 likeGraylog Logstash. |
| fluentd | Fluentd 记录的驱动程序. 写入日志消息 fluentd(正向输入). |
| awslogs | 亚马逊 CloudWatch 的日志记录驱动泊坞窗. 写入日志消息到 Amazon CloudWatch 的日志 |
| splunk | Splunk 的日志驱动程序. 写入日志消息 splunk 使用事件的 Http 收藏家. |

该 `docker logs` 命令只对可 `json-file` 和 `journald` 记录的驱动程序. 有关使用记录司机工作的详细信息, 请参阅 [配置日志记录的驱动程序](https://docs.docker.com/engine/admin/logging/overview/).

### 覆盖 Dockerfile 镜像默认值

我们打包一个镜像一般通过 [Dockerfile](https://deepzz.com/post/dockerfile-reference.html) 构建, 之中已指定了容器启动时的默认参数

Dockerfile 中的 4 个命令不能覆盖: FROM, MAINTAINER, RUN, 和 ADD. 其他一切有相应 `docker run` 参数覆盖.

**CMD(默认的命令或选项)**

上面说过 `docker run` 格式:

```
$ docker run [OPTIONS] IMAGE[: TAG|@DIGEST] [COMMAND] [ARG...]
```

通过指定 `[COMMAND]` 我们可以直接覆盖 Dockerfile 中的命令.

如果图像还指定了 ENTRYPOINT 那么 CMD 还是 COMMAND 得到附加作为参数传递给 ENTRYPOINT. 如果 image 还指定了 `ENTRYPOINT`, 那么 `CMD` 或 `COMMAND` 将作为参数跟在 `ENTRYPOINT` 后面.

**ENTRYPOINT(默认命令在运行时执行)**

```
--entrypoint="": Overwrite the default entrypoint set by the image
```

`ENTRYPOINT` 类似于一个 `COMMAND`, 它可以指定容器启动时执行什么, 但它 (故意) 更难以覆盖.`ENTRYPOINT` 给出了一个容器的默认性质或行为, 我们可以指定 `COMMAND`, 这将作为参数传递给 `ENTRYPOINT`. 通过 `--entrypoint` 可以覆盖原来 Dockerfile 中的默认命令. 下面是如何将自动运行设置为其它东西 (如替换 `/usr/bin/redis-server` 而去运行 `shell`):

```
$ docker run -it --entrypoint /bin/bash example/redis
```

如何传跟多参数给 `ENTRYPOINT` 的两种方式:

```
$ docker run -it --entrypoint /bin/bash example/redis -c ls -l
$ docker run -it --entrypoint /usr/bin/redis-cli example/redis --help
```

> ` 注 `: 传递 `--entrypoint` 将清除出在 image 上的所有默认命令集 (如: Dockerfile 构建时的 CMD).

**EXPOSE(接收端口)**

下面的 run 命令选项与容器联网工作:

```
--expose= []: Expose a port or a range of ports inside the container.
             These are additional to those exposed by the `EXPOSE` instruction
-P         : Publish all exposed ports to the host interfaces
-p= []      : Publish a container᾿s port or a range of ports to the host
               format: ip: hostPort: containerPort | ip:: containerPort | hostPort: containerPort | containerPort
               Both hostPort and containerPort can be specified as a
               range of ports. When specifying ranges for both, the
               number of container ports in the range must match the
               number of host ports in the range, for example:
                   -p 1234-1236:1234-1236/tcp

               When specifying a range for hostPort only, the
               containerPort must not be a range.  In this case the
               container port is published somewhere within the
               specified hostPort range. (e.g., `-p 1234-1236:1234/tcp`)

               (use 'docker port' to see the actual mapping)

--link=""  : Add link to another container (<name or id>: alias or <name or id>)
```

使用该 `--expose` 选项添加到暴露的端口.

还可以使用 `-P` 或 `-p` 使容器端口与主机端口映射.

该 `-P` 选项发布的所有端口的主机接口.Docker 每个暴露的端口绑定到主机上的随机端口. 端口范围是一个内临时端口范围被定义 `/proc/sys/net/ipv4/ip_local_port_range`. 使用 `-p` 标志显式地映射一个端口或端口范围.

容器内的端口号 (其中, 服务侦听) 不需要匹配暴露在容器的外部 (其中客户端连接) 的端口号. 例如, HTTP 服务端口 80 监听的容器内 (指定 `EXPOSE 80` 在 Dockerfile). 在运行时, 端口可能被绑定到 42800 的主机上. 要查找主机端口与暴露的端口之间的映射, 请使用 `docker port`.

如果操作者使用 `--link` 在开始默认网桥网络中的一个新的客户端的容器时, 则客户端容器可以通过一个专用网络接口访问暴露端口. 如果 `--link` 开始在一个用户定义的网络的容器时如上述用于多克尔网络概览, 它将提供用于容器名为别名被链接到.

**ENV(环境变量)**

当创建一个新的集装箱, Docker 将自动设置以下环境变量:

| Variable | Value |
| --- | --- |
| HOME | Set based on the value of USER |
| HOSTNAME | The hostname associated with the container |
| PATH | Includes popular directories, such as :/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin |
| TERM | xterm if the container is allocated a pseudo-TTY |

此外, 操作者可以设置任何环境变量, 通过使用一个或更多个中的 `-e` 标志, 甚至覆盖那些上面提到的, 或已经用 Dockerfile 定义 de 的 ENV:

```
$ docker run -e "deep= purple" --rm ubuntu /bin/bash -c export
declare -x HOME="/"
declare -x HOSTNAME="85bc26a0e200"
declare -x OLDPWD
declare -x PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
declare -x PWD="/"
declare -x SHLVL="1"
declare -x deep="purple"
```

类似地, 可以用 `-h` 设置 `hostname`.

**HEALTHCHECK 健康检查**

```
--health-cmd            Command to run to check health
--health-interval       Time between running the check
--health-retries        Consecutive failures needed to report unhealthy
--health-timeout        Maximum time to allow one check to run
--no-healthcheck        Disable any container-specified HEALTHCHECK
```

例:

```
$ docker run --name= test -d \
    --health-cmd='stat /etc/passwd | | exit 1' \
    --health-interval=2s \
    busybox sleep 1d
$ sleep 2; docker inspect --format='{{.State.Health.Status}}' test
healthy
$ docker exec test rm /etc/passwd
$ sleep 2; docker inspect --format='{{json .State.Health}}' test
{
  "Status": "unhealthy",
  "FailingStreak": 3,
  "Log": [
    {
      "Start": "2016-05-25T17:22:04.635478668Z",
      "End": "2016-05-25T17:22:04.7272552Z",
      "ExitCode": 0,
      "Output": "  File: /etc/passwd\n  Size: 334       \tBlocks: 8          IO Block: 4096   regular file\nDevice: 32h/50d\tInode: 12          Links: 1\nAccess: (0664/-rw-rw-r--)  Uid: (    0/    root)   Gid: (    0/    root)\nAccess: 2015-12-05 22:05:32.000000000\nModify: 2015..."
    },
    {
      "Start": "2016-05-25T17:22:06.732900633Z",
      "End": "2016-05-25T17:22:06.822168935Z",
      "ExitCode": 0,
      "Output": "  File: /etc/passwd\n  Size: 334       \tBlocks: 8          IO Block: 4096   regular file\nDevice: 32h/50d\tInode: 12          Links: 1\nAccess: (0664/-rw-rw-r--)  Uid: (    0/    root)   Gid: (    0/    root)\nAccess: 2015-12-05 22:05:32.000000000\nModify: 2015..."
    },
    {
      "Start": "2016-05-25T17:22:08.823956535Z",
      "End": "2016-05-25T17:22:08.897359124Z",
      "ExitCode": 1,
      "Output": "stat: can't stat '/etc/passwd': No such file or directory\n"
    },
    {
      "Start": "2016-05-25T17:22:10.898802931Z",
      "End": "2016-05-25T17:22:10.969631866Z",
      "ExitCode": 1,
      "Output": "stat: can't stat '/etc/passwd': No such file or directory\n"
    },
    {
      "Start": "2016-05-25T17:22:12.971033523Z",
      "End": "2016-05-25T17:22:13.082015516Z",
      "ExitCode": 1,
      "Output": "stat: can't stat '/etc/passwd': No such file or directory\n"
    }
  ]
}
```

健康状况也显示在 docker ps 输出.

**TMPFS(安装的 tmpfs 文件系统)**

```
--tmpfs= []: Create a tmpfs mount with: container-dir[: <options>],
            where the options are identical to the Linux
            'mount -t tmpfs -o' command.
```

下面的例子挂载了一个空的 tmpfs 到容器用 `rw`,`noexec`,`nosuid`, 和 `size=65536k` 选项.

```
$ docker run -d --tmpfs /run: rw, noexec, nosuid, size=65536k my_image
```

**VOLUME(共享文件系统)**

```
-v, --volume= [host-src:]container-dest[: <options>]: Bind mount a volume.
The comma-delimited `options` are [rw| ro], [z| Z],
[[r]shared| [r]slave| [r]private], and [nocopy].
The 'host-src' is an absolute path or a name value.

If neither 'rw' or 'ro' is specified then the volume is mounted in
read-write mode.

The `nocopy` modes is used to disable automatic copying requested volume
path in the container to the volume storage location.
For named volumes, `copy` is the default mode. Copy modes are not supported
for bind-mounted volumes.

--volumes-from="": Mount all volumes from the given container(s)
```

在 `container-dest` 必须始终是绝对路径, 如 `/src/docs`.`host-src` 可以是绝对路径或一个 `name` 值. 如果你提供了一个绝对路径 `host-dir`, Docker 会绑定到指定路径. 如果提供 name, Docker 会创建一个名为量 name 的卷.

一个 name 值必须以字母数字字符, 然后是开始 `a-z0-9`,`_`(下划线), `.`(句号) 或 `-`(连字符). 绝对路径有一个开始 `/`(正斜杠).

**USER**

`root`(ID = 0) 是在一个容器内的默认用户. 我们可以创建更多的用户. 当我们指定一个 ID 时, 该用户不必存在.

开发人员可以设置默认用户使用 Dockerfile USER 指令运行第一个进程. 启动容器时, 操作员可以通过传递 `-u` 选项覆盖 `USER` 指令

```
-u="", --user="": Sets the username or UID used and optionally the groupname or GID for the specified command.

The followings examples are all valid:
--user= [ user | user: group | uid | uid: gid | user: gid | uid: group
```

> ` 注意 `: 如果你传递一个数字 UID, 它必须是在 0-2147483647 范围内.

**WORKDIR**

一个容器中运行的二进制文件的默认工作目录是根目录 (`/`), 通过 `-w` 可以指定与 Dockerfile `WORKDIR` 命令不同的工作目录.

```
-w="": Working directory inside the container
```

本文链接: [https://deepzz.com/post/docker-run-reference.html](//deepzz.com/post/docker-run-reference.html "Permalink to 如何执行 docker run, docker run 命令参考文档 "), [参与评论 »](//deepzz.com/post/docker-run-reference.html#comments)

\--EOF\--

发表于 2017-01-05 08:59:00.

本站使用 {[署名 4.0 国际](//creativecommons.org/licenses/by/4.0/)} 创作共享协议, 转载请注明作者及原网址.[更多说明 »](//deepzz.com/post/about.html#toc_1)

提醒: 本文最后更新于 1539 天前, 文中所描述的信息可能已发生改变, 请谨慎使用.

### 专题 {Docker 相关技术} 的其它文章 [»](/series.html#toc-1 " 更多 ")

*   [docker volume 容器卷的那些事 (二)](/post/the-docker-volumes-permissions.html) (Jan 07, 2018)
*   [docker volume 容器卷的那些事 (一)](/post/the-docker-volumes-basic.html) (Dec 03, 2017)
*   [如何写好 Dockerfile, Dockerfile 最佳实践](/post/dockerfile-best-practices.html) (Jun 10, 2017)
*   [如何写 Dockerfile, Dockerfile 参考文档](/post/dockerfile-reference.html) (Dec 15, 2016)
*   [如何写 docker-compose.yml, Docker compose file 参考文档](/post/docker-compose-file.html) (Dec 06, 2016)
*   [远程连接 docker daemon, Docker Remote API](/post/dockerd-and-docker-remote-api.html) (Nov 25, 2016)
*   [搭建安全的 docker private registry v2 指南 (Let's Encrypt)](/post/secure-docker-registry.html) (Oct 05, 2016)

[« Golang 包管理工具 Glide, 你值得拥有](/post/glide-package-management-introduce.html) [语义化版本号 2.0.0 文档, 怎么定义版本号 »](/post/semantic-versioning.html)

# Comments

© 2021 - Deepzz's Blog - [蜀 ICP 备 16021362 号 -1](https://beian.miit.gov.cn)
Powered by [Eiblog](//github.com/eiblog/eiblog) & [JerryQu](//imququ.com)
