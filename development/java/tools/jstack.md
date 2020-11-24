---
title: jstack java 堆栈跟踪工具
date: 2020-11-21 11:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '工具']
permalink: jstack
---

## 简介

jstack 是 JVM 自带的 Java 堆栈跟踪工具, 用于打印出给定的 java 进程 ID, core file, 远程调试服务的 Java 堆栈信息

- jstack 命令用于生成虚拟机当前时刻的线程快照, 线程快照是当前虚拟机内每一条线程正在执行的方法堆栈的集合, 生成线程快照的主要目的是定位线程出现长时间停顿的原因, 如线程间死锁、死循环、请求外部资源导致的长时间等待等问题
- 如果 java 程序崩溃生成 core 文件, jstack 工具可以用来获得 core 文件的 java stack 和 native stack 的信息, 从而可以轻松地知道 java 程序是如何崩溃和在程序何处发生问题
- jstack 工具可以附属到正在运行的 java 程序中, 看到当时运行的 java 程序的 java stack 和 native stack 的信息, 如果现在运行的 java 程序呈现 hung 的状态, jstack 是非常有用的

<!-- more -->

## 运行

```sh
jstack [ option ] pid
jstack [ option ] executable core
jstack [ option ] [server-id@]remote-hostname-or-IP
```

- pid: Java 进程 id
- executable: 产生 core dump 的 Java 可执行程序
- core: 打印出的 core 文件
- remote-hostname-or-ip: 远程 debug 服务器的名称或 IP
- server-id: 多个 debug 服务时指定服务 id
- option
  - -F: 当正常输出的请求不被响应时, 强制输出线程堆栈
  - -m: 如果调用到本地方法的话, 可以显示 C/C++ 的堆栈
  - -l: 除堆栈外, 显示关于锁的附件信息, 在发生死锁时可以查看锁的持有情况

## 输出解析

jstack 输出内容格式如下

```sh
"ajp-nio-8009-Acceptor-0" #180 daemon prio=5 os_prio=0 tid=0x00007f279865a000 nid=0xa825 runnable [0x00007f265e0c8000]
   java.lang.Thread.State: RUNNABLE
  at sun.nio.ch.ServerSocketChannelImpl.accept0(Native Method)
  at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:422)
  at sun.nio.ch.ServerSocketChannelImpl.accept(ServerSocketChannelImpl.java:250)
  - locked <0x00000006c0234640> (a java.lang.Object)
  at org.apache.tomcat.util.net.NioEndpoint$Acceptor.run(NioEndpoint.java:692)
  at java.lang.Thread.run(Thread.java:745)
```

第一行为线程简介

- "ajp-nio-8009-Acceptor-0": 线程名
- "#180": 线程编号
- "daemon": 后台守护线程
- "prio=5": 线程优先级
- "os_prio=0": 操作系统优先级
- "tid=0x00007f279865a000": 线程id
- "nid=0xa825": 操作系统映射的线程id (十六进制), 转成十进制后, 对应 Linux 下 ps -mp pid -o THREAD,tid,time 命令打印出来的线程 tid
- "0x00007f265e0c8000": 线程栈的起始地址
- "RUNNABLE": 线程状态

后续内容为线程的堆栈信息

## 线程状态

Dump 文件的线程状态一般包含 RUNNABLE, BLOCKED, WAITING

分析 Dump 文件主要关注以下消息

- deadlock: 出现死锁, 一般会提示发现多少个死锁
- blocked: 线程被阻塞
- Parked: 停止
- locked: 对象加锁
- waiting: 线程正无限制等待
- waiting to lock: 等待上锁
- Object.wait(): 对象无限制等待中
- waiting for monitor entry: 等待获取监视器
- waiting on condition: 等待资源, 比较常见的等待连接池, 等待网络读写
- parking to wait for: 正在等待哪个共享的对象

## 应用

### 分析线程 CPU 使用率高的原因

获取当前应用的 PID

```sh
jps -mlv
24690 sun.tools.jps.Jps -mlv -Denv.class.path=.:/root/home/java/jdk1.8.0_211/jre/lib/rt.jar:/root/home/java/jdk1.8.0_211/lib/dt.jar:/root/home/java/jdk1.8.0_211/lib/tools.jar -Dapplication.home=/root/home/java/jdk1.8.0_211 -Xms8m
6472 org.apache.zookeeper.server.quorum.QuorumPeerMain /root/home/zookeeper-3.4.8/bin/../conf/zoo.cfg -Dzookeeper.log.dir=. -Dzookeeper.root.logger=INFO,CONSOLE -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.local.only=false
17818 org.apache.catalina.startup.Bootstrap start -Djava.util.logging.config.file=/root/home/apache-tomcat-7.0.93/conf/logging.properties -Djava.util.logging.manager=org.apache.juli.ClassLoaderLogManager -Djdk.tls.ephemeralDHKeySize=2048 -agentlib:jdwp=transport=dt_socket,address=8000,server=y,suspend=n -Dignore.endorsed.dirs= -Dcatalina.base=/root/home/apache-tomcat-7.0.93 -Dcatalina.home=/root/home/apache-tomcat-7.0.93 -Djava.io.tmpdir=/root/home/apache-tomcat-7.0.93/temp
```

获取应用的线程使用情况

```sh
top -Hp 17818
top - 02:32:54 up  4:00,  3 users,  load average: 0.22, 0.14, 0.09
Threads: 340 total,   0 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.5 us,  0.7 sy,  0.0 ni, 97.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
KiB Mem :  7864576 total,  4382304 free,  2307628 used,  1174644 buff/cache
KiB Swap:  8126460 total,  8126460 free,        0 used.  5233324 avail Mem

  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
17837 root      20   0 8395784 1.515g  23244 S  3.0 20.2   0:20.65 java
17836 root      20   0 8395784 1.515g  23244 S  1.0 20.2   0:42.20 java
17828 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:02.69 java
17833 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:00.30 java
17834 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:41.54 java
17835 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:42.01 java
17880 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:02.47 java
17915 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:00.23 java
18089 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:01.00 java
18105 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:00.78 java
22408 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:00.02 java
22470 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:00.01 java
23317 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:00.02 java
23318 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:00.05 java
23319 root      20   0 8395784 1.515g  23244 S  0.3 20.2   0:00.01 java
17818 root      20   0 8395784 1.515g  23244 S  0.0 20.2   0:00.00 java
17819 root      20   0 8395784 1.515g  23244 S  0.0 20.2   0:00.67 java
...
```

使用 jstack 获取当前的线程堆栈状态

```sh
jstack -l 17818 > 17818.log
```

因为线程 ID 是使用十六进制的, 所以需要将 17837 17836 转换为 0x45ad, 0x45ac

```sh
"C1 CompilerThread3" #10 daemon prio=9 os_prio=0 tid=0x00007fe9140db000 nid=0x45ad waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
  - None

"C2 CompilerThread2" #9 daemon prio=9 os_prio=0 tid=0x00007fe9140d8800 nid=0x45ac waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

   Locked ownable synchronizers:
  - None
```

这一系列线程占用比较高的 CPU 资源是因为 JVM 启动时所有代码都处于解释执行模式, 在 server 模式下字节码编译器 C2 Compiler 在某些代码被执行到一定阈值时, 会由 C2 Compiler 编译成机器码, 提升后续效率, 所以在一开始进行压测时会频繁占用 CPU, 等到编译完成后就会降低了

如何进行解决这边就不详细解释了, 主要通过调整 JVM -XX:CICompilerCount=threads 参数和 Warm Up (预热) 等方法进行处理

## 线程卡住

```sh
"ThreadPoolTaskExecutor-1" #149 prio=5 os_prio=0 tid=0x00007f581c240800 nid=0x6d5c waiting on condition [0x00007f5888746000]

   java.lang.Thread.State: WAITING (parking)

  at sun.misc.Unsafe.park(Native Method)

  - parking to wait for  <0x00000005d4e168d8> (a java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject)

  at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)

  at java.util.concurrent.locks.AbstractQueuedSynchronizer$ConditionObject.await(AbstractQueuedSynchronizer.java:2039)

  at org.apache.http.pool.PoolEntryFuture.await(PoolEntryFuture.java:139)

  at org.apache.http.pool.AbstractConnPool.getPoolEntryBlocking(AbstractConnPool.java:307)

  at org.apache.http.pool.AbstractConnPool.access$000(AbstractConnPool.java:65)

  at org.apache.http.pool.AbstractConnPool$2.getPoolEntry(AbstractConnPool.java:193)

  at org.apache.http.pool.AbstractConnPool$2.getPoolEntry(AbstractConnPool.java:186)

  at org.apache.http.pool.PoolEntryFuture.get(PoolEntryFuture.java:108)

  at org.apache.http.impl.conn.PoolingHttpClientConnectionManager.leaseConnection(PoolingHttpClientConnectionManager.java:276)

  at org.apache.http.impl.conn.PoolingHttpClientConnectionManager$1.get(PoolingHttpClientConnectionManager.java:263)

  at org.apache.http.impl.execchain.MainClientExec.execute(MainClientExec.java:190)

  at org.apache.http.impl.execchain.ProtocolExec.execute(ProtocolExec.java:184)

  at org.apache.http.impl.execchain.RetryExec.execute(RetryExec.java:88)

  at org.apache.http.impl.execchain.RedirectExec.execute(RedirectExec.java:110)

  at org.apache.http.impl.client.InternalHttpClient.doExecute(InternalHttpClient.java:184)

  at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:82)

  at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:107)

  ...

     Locked ownable synchronizers:

  - <0x00000005cf72d818> (a java.util.concurrent.ThreadPoolExecutor$Worker)

"Thread-15" #142 daemon prio=5 os_prio=0 tid=0x00007f593680f000 nid=0x6d28 waiting on condition [0x00007f5889c4f000]

   java.lang.Thread.State: WAITING (parking)

  at sun.misc.Unsafe.park(Native Method)

  - parking to wait for  <0x00000005d4e20c30> (a java.util.concurrent.CountDownLatch$Sync)

  at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)

  at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:836)

  at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:997)

  at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1304)

  at java.util.concurrent.CountDownLatch.await(CountDownLatch.java:231)
```

在多线程环境中经常会出现各种因为资源抢夺而导致线程等待或者阻塞的情况

上面的日志出现了线程 ThreadPoolTaskExecutor-1 一直在等待的情况, 分析原因后发现等待的位置是在 httpClient 获取连接池的时候

```sh
at org.apache.http.pool.PoolEntryFuture.await(PoolEntryFuture.java:139)
```

经过检查, 发现没有对 httpClient 进行优化导致了连接池不够, 而 httpClient 4.5 版本的时候会无限制的等待连接池

添加连接池超时配置

```java
RequestConfig defaultRequestConfig = RequestConfig.custom()
    // 等待连接池超时
    .setConnectionRequestTimeout(5000)
    .build();
```

优化连接池, 在出现问题后及时释放连接
