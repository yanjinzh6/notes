---
title: Jenkins jnlp job 失败导致 pod 错误卡死
date: 2020-05-04 19:00:00
tags: 'Jenkins'
categories:
  - ['部署', 'CI']
permalink: jenkins-jnlp-error
photo:
---

## 简介

大环境说明一下, 得说 `2G` 的机器太难了

```sh
top - 16:33:35 up 101 days, 20:55,  0 users,  load average: 13.11, 12.22, 8.01
Tasks: 178 total,   7 running, 137 sleeping,   0 stopped,   2 zombie
%Cpu(s):  5.9 us, 92.4 sy,  0.0 ni,  0.0 id,  1.3 wa,  0.0 hi,  0.3 si,  0.0 st
KiB Mem :  2041368 total,    62824 free,  1588424 used,   390120 buff/cache
KiB Swap:  2097148 total,       56 free,  2097092 used.   293296 avail Mem
```

由于测试的机器配置很糟糕, 当出现 Jenkins job 失败情况后很容易就导致了 jnlp pod Error 状态, 如何就一直卡着

```sh
jenkins-ops          jnlp-slave-3dh8s                          0/1     Error     0          50m
```

<!-- more -->

这时候一般就手动进行删除操作 `kbc delete pod jnlp-slave-3dh8s -n jenkins-ops`

当然也很容易就出现 k8s 服务器连接不上了

```sh
Unable to connect to the server: net/http: TLS handshake timeout
```

查询后发现 Error 状态的容器并没有记录下什么关键信息的事件

```sh
Name:         jnlp-slave-xmbbf
Namespace:    jenkins-ops
Priority:     0
Node:         ubuntu/172.16.0.2
Start Time:   Sat, 09 May 2020 16:22:35 +0800
Labels:       jenkins=slave
              jenkins/label=slave-agent
Annotations:  <none>
Status:       Failed
IP:           10.1.1.106
IPs:
  IP:  10.1.1.106
Containers:
  jnlp:
    Container ID:   containerd://7c4c636b3b1d176f96d1a0e5d96c3561c242a64e88772df8599aaf20a3991f9f
    Image:          127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo:1
    Image ID:       127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo@sha256:e6f508efbd0bf226a6f6c12b3ff1e3c40d5583c00695edec84f520f59a6401c3
    Port:           <none>
    Host Port:      <none>
    State:          Terminated
      Reason:       Error
      Exit Code:    255
      Started:      Sat, 09 May 2020 16:22:59 +0800
      Finished:     Sat, 09 May 2020 16:23:56 +0800
    Ready:          False
    Restart Count:  0
    Environment:
      JENKINS_SECRET:         a9c282433692c376789affcb21888a7e719d27054987048ebc6ff3296cfa2af5
      JENKINS_AGENT_NAME:     jnlp-slave-xmbbf
      JENKINS_NAME:           jnlp-slave-xmbbf
      JENKINS_AGENT_WORKDIR:  /home/jenkins/agent
      JENKINS_URL:            http://jenkins.jenkins-ops.svc.cluster.local/
    Mounts:
      /home/jenkins/agent from workspace-volume (rw)
      /home/jenkins/agent/workspace from volume-1 (rw)
      /var/run/docker.sock from volume-0 (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from jenkins-token-5lpfl (ro)
Conditions:
  Type              Status
  Initialized       True
  Ready             False
  ContainersReady   False
  PodScheduled      True
Volumes:
  volume-0:
    Type:          HostPath (bare host directory volume)
    Path:          /var/run/docker.sock
    HostPathType:
  volume-1:
    Type:          HostPath (bare host directory volume)
    Path:          /data/jenkins_home/jnlp-hexo/
    HostPathType:
  workspace-volume:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:
    SizeLimit:  <unset>
  jenkins-token-5lpfl:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  jenkins-token-5lpfl
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  beta.kubernetes.io/os=linux
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
Events:
  Type    Reason     Age        From               Message
  ----    ------     ----       ----               -------
  Normal  Scheduled  <unknown>  default-scheduler  Successfully assigned jenkins-ops/jnlp-slave-xmbbf to ubuntu
  Normal  Pulling    13m        kubelet, ubuntu    Pulling image "127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo:1"
  Normal  Pulled     13m        kubelet, ubuntu    Successfully pulled image "127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo:1"
  Normal  Created    13m        kubelet, ubuntu    Created container jnlp
  Normal  Started    13m        kubelet, ubuntu    Started container jnlp`
```

## 分析

这次没有打印 logs 就把 pod 删除掉了, 操作过程非常不流畅, 感觉整个程序都已经失去响应了, 后面再发现情况需要试试能否打印容器日志

大概问题如下

- 由于多个 pod Error 状态导致没有资源再运行了
- 配置过低导致过渡使用交换区造成没有响应, 看上去 `kswapd0` 进程一直在消耗 CPU
- 能否自动删除 Error 状态的 pod
- [Kubernetes plugin](https://j.yzer.club/configureClouds/) 插件配置更新限制数量, 超时时间参数看看能否优化

## 日志

```sh
kbc logs jnlp-slave-gc4jm -n jenkins-ops
Warning: JnlpProtocol3 is disabled by default, use JNLP_PROTOCOL_OPTS to alter the behavior
May 09, 2020 10:00:34 AM hudson.remoting.jnlp.Main createEngine
INFO: Setting up agent: jnlp-slave-gc4jm
May 09, 2020 10:00:35 AM hudson.remoting.jnlp.Main$CuiListener <init>
INFO: Jenkins agent is running in headless mode.
May 09, 2020 10:00:36 AM hudson.remoting.Engine startEngine
INFO: Using Remoting version: 3.36
May 09, 2020 10:00:36 AM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /home/jenkins/agent/remoting as a remoting work directory
May 09, 2020 10:00:36 AM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
INFO: Both error and output logs will be printed to /home/jenkins/agent/remoting
May 09, 2020 10:00:37 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://jenkins.jenkins-ops.svc.cluster.local/]
May 09, 2020 10:00:38 AM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
INFO: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
May 09, 2020 10:00:38 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Agent discovery successful
  Agent address: jenkins.jenkins-ops.svc.cluster.local
  Agent port:    50000
  Identity:      6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 10:00:38 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Handshaking
May 09, 2020 10:00:38 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connecting to jenkins.jenkins-ops.svc.cluster.local:50000
May 09, 2020 10:00:38 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Trying protocol: JNLP4-connect
May 09, 2020 10:00:40 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Remote identity confirmed: 6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 10:00:43 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connected
May 09, 2020 10:01:02 AM hudson.remoting.UserRequest perform
WARNING: LinkageError while performing UserRequest:hudson.FilePath$Exists@7bea7926
java.lang.NoClassDefFoundError: hudson/FilePath$4
        at java.lang.Class.getDeclaredFields0(Native Method)
        at java.lang.Class.privateGetDeclaredFields(Class.java:2583)
        at java.lang.Class.getDeclaredField(Class.java:2068)
        at java.io.ObjectStreamClass.getDeclaredSUID(ObjectStreamClass.java:1857)
        at java.io.ObjectStreamClass.access$700(ObjectStreamClass.java:79)
        at java.io.ObjectStreamClass$3.run(ObjectStreamClass.java:506)
        at java.io.ObjectStreamClass$3.run(ObjectStreamClass.java:494)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.io.ObjectStreamClass.<init>(ObjectStreamClass.java:494)
        at java.io.ObjectStreamClass.lookup(ObjectStreamClass.java:391)
        at java.io.ObjectStreamClass.initNonProxy(ObjectStreamClass.java:681)
        at java.io.ObjectInputStream.readNonProxyDesc(ObjectInputStream.java:1885)
        at java.io.ObjectInputStream.readClassDesc(ObjectInputStream.java:1751)
        at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2042)
        at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1573)
        at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2287)
        at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2211)
        at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2069)
        at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1573)
        at java.io.ObjectInputStream.defaultReadFields(ObjectInputStream.java:2287)
        at java.io.ObjectInputStream.readSerialData(ObjectInputStream.java:2211)
        at java.io.ObjectInputStream.readOrdinaryObject(ObjectInputStream.java:2069)
        at java.io.ObjectInputStream.readObject0(ObjectInputStream.java:1573)
        at java.io.ObjectInputStream.readObject(ObjectInputStream.java:431)
        at hudson.remoting.UserRequest.deserialize(UserRequest.java:290)
        at hudson.remoting.UserRequest.perform(UserRequest.java:189)
        at hudson.remoting.UserRequest.perform(UserRequest.java:54)
        at hudson.remoting.Request$2.run(Request.java:369)
        at hudson.remoting.InterceptingExecutorService$1.call(InterceptingExecutorService.java:72)
        at java.util.concurrent.FutureTask.run(FutureTask.java:266)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at hudson.remoting.Engine$1.lambda$newThread$0(Engine.java:97)
        at java.lang.Thread.run(Thread.java:748)
Caused by: java.lang.ClassNotFoundException: hudson.FilePath$4
        at java.net.URLClassLoader.findClass(URLClassLoader.java:382)
        at hudson.remoting.RemoteClassLoader.findClass(RemoteClassLoader.java:171)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:424)
        at java.lang.ClassLoader.loadClass(ClassLoader.java:357)
        ... 34 more

May 09, 2020 10:01:02 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Terminated
May 09, 2020 10:01:02 AM hudson.remoting.Request$2 run
INFO: Failed to send back a reply to the request hudson.remoting.Request$2@74f61325: hudson.remoting.ChannelClosedException: Channel "hudson.remoting.Channel@714ad857:JNLP4-connect connection to jenkins.jenkins-ops.svc.cluster.local/10.152.183.5:50000": channel is already closed
May 09, 2020 10:01:15 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Performing onReconnect operation.
May 09, 2020 10:01:15 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: onReconnect operation completed.
May 09, 2020 10:01:15 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://jenkins.jenkins-ops.svc.cluster.local/]
May 09, 2020 10:01:15 AM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
INFO: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
May 09, 2020 10:01:15 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Agent discovery successful
  Agent address: jenkins.jenkins-ops.svc.cluster.local
  Agent port:    50000
  Identity:      6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 10:01:15 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Handshaking
May 09, 2020 10:01:15 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connecting to jenkins.jenkins-ops.svc.cluster.local:50000
May 09, 2020 10:01:15 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Trying protocol: JNLP4-connect
May 09, 2020 10:01:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Remote identity confirmed: 6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 10:01:16 AM org.jenkinsci.remoting.protocol.impl.ConnectionHeadersFilterLayer onRecv
INFO: [JNLP4-connect connection to jenkins.jenkins-ops.svc.cluster.local/10.152.183.5:50000] Local headers refused by remote: Unknown client name: jnlp-slave-gc4jm
May 09, 2020 10:01:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Protocol JNLP4-connect encountered an unexpected exception
java.util.concurrent.ExecutionException: org.jenkinsci.remoting.protocol.impl.ConnectionRefusalException: Unknown client name: jnlp-slave-gc4jm
        at org.jenkinsci.remoting.util.SettableFuture.get(SettableFuture.java:223)
        at hudson.remoting.Engine.innerRun(Engine.java:577)
        at hudson.remoting.Engine.run(Engine.java:488)
Caused by: org.jenkinsci.remoting.protocol.impl.ConnectionRefusalException: Unknown client name: jnlp-slave-gc4jm
        at org.jenkinsci.remoting.protocol.impl.ConnectionHeadersFilterLayer.newAbortCause(ConnectionHeadersFilterLayer.java:378)
        at org.jenkinsci.remoting.protocol.impl.ConnectionHeadersFilterLayer.onRecvClosed(ConnectionHeadersFilterLayer.java:433)
        at org.jenkinsci.remoting.protocol.ProtocolStack$Ptr.onRecvClosed(ProtocolStack.java:816)
        at org.jenkinsci.remoting.protocol.FilterLayer.onRecvClosed(FilterLayer.java:287)
        at org.jenkinsci.remoting.protocol.impl.SSLEngineFilterLayer.onRecvClosed(SSLEngineFilterLayer.java:172)
        at org.jenkinsci.remoting.protocol.ProtocolStack$Ptr.onRecvClosed(ProtocolStack.java:816)
        at org.jenkinsci.remoting.protocol.NetworkLayer.onRecvClosed(NetworkLayer.java:154)
        at org.jenkinsci.remoting.protocol.impl.BIONetworkLayer.access$1500(BIONetworkLayer.java:48)
        at org.jenkinsci.remoting.protocol.impl.BIONetworkLayer$Reader.run(BIONetworkLayer.java:247)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at hudson.remoting.Engine$1.lambda$newThread$0(Engine.java:97)
        at java.lang.Thread.run(Thread.java:748)
        Suppressed: java.nio.channels.ClosedChannelException
                ... 7 more

May 09, 2020 10:01:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connecting to jenkins.jenkins-ops.svc.cluster.local:50000
May 09, 2020 10:01:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Server reports protocol JNLP4-plaintext not supported, skipping
May 09, 2020 10:01:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Protocol JNLP3-connect is not enabled, skipping
May 09, 2020 10:01:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Server reports protocol JNLP2-connect not supported, skipping
May 09, 2020 10:01:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Server reports protocol JNLP-connect not supported, skipping
May 09, 2020 10:01:16 AM hudson.remoting.jnlp.Main$CuiListener error
SEVERE: The server rejected the connection: None of the protocols were accepted
java.lang.Exception: The server rejected the connection: None of the protocols were accepted
        at hudson.remoting.Engine.onConnectionRejected(Engine.java:662)
        at hudson.remoting.Engine.innerRun(Engine.java:602)
        at hudson.remoting.Engine.run(Engine.java:488)
```

```sh
kbc logs jnlp-slave-qk0w3 -n jenkins-ops
Warning: JnlpProtocol3 is disabled by default, use JNLP_PROTOCOL_OPTS to alter the behavior
May 09, 2020 9:29:51 AM hudson.remoting.jnlp.Main createEngine
INFO: Setting up agent: jnlp-slave-qk0w3
May 09, 2020 9:29:51 AM hudson.remoting.jnlp.Main$CuiListener <init>
INFO: Jenkins agent is running in headless mode.
May 09, 2020 9:29:52 AM hudson.remoting.Engine startEngine
INFO: Using Remoting version: 3.36
May 09, 2020 9:29:52 AM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /home/jenkins/agent/remoting as a remoting work directory
May 09, 2020 9:29:52 AM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
INFO: Both error and output logs will be printed to /home/jenkins/agent/remoting
May 09, 2020 9:29:52 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://jenkins.jenkins-ops.svc.cluster.local/]
May 09, 2020 9:29:56 AM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
INFO: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
May 09, 2020 9:29:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Agent discovery successful
  Agent address: jenkins.jenkins-ops.svc.cluster.local
  Agent port:    50000
  Identity:      6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 9:29:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Handshaking
May 09, 2020 9:29:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connecting to jenkins.jenkins-ops.svc.cluster.local:50000
May 09, 2020 9:29:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Trying protocol: JNLP4-connect
May 09, 2020 9:29:57 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Remote identity confirmed: 6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 9:30:00 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connected
May 09, 2020 9:32:39 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Terminated
May 09, 2020 9:32:39 AM hudson.remoting.Request$2 run
INFO: Failed to send back a reply to the request hudson.remoting.Request$2@b670829: hudson.remoting.ChannelClosedException: Channel "hudson.remoting.Channel@75dd01e5:JNLP4-connect connection to jenkins.jenkins-ops.svc.cluster.local/10.152.183.5:50000": channel is already closed
May 09, 2020 9:32:50 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Performing onReconnect operation.
May 09, 2020 9:32:50 AM jenkins.slaves.restarter.JnlpSlaveRestarterInstaller$FindEffectiveRestarters$1 onReconnect
INFO: Restarting agent via jenkins.slaves.restarter.UnixSlaveRestarter@1123a38a
May 09, 2020 9:32:52 AM hudson.remoting.jnlp.Main createEngine
INFO: Setting up agent: jnlp-slave-qk0w3
May 09, 2020 9:32:53 AM hudson.remoting.jnlp.Main$CuiListener <init>
INFO: Jenkins agent is running in headless mode.
May 09, 2020 9:32:53 AM hudson.remoting.Engine startEngine
INFO: Using Remoting version: 3.36
May 09, 2020 9:32:53 AM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /home/jenkins/agent/remoting as a remoting work directory
May 09, 2020 9:32:53 AM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
INFO: Both error and output logs will be printed to /home/jenkins/agent/remoting
May 09, 2020 9:32:54 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://jenkins.jenkins-ops.svc.cluster.local/]
May 09, 2020 9:32:54 AM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
INFO: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
May 09, 2020 9:32:54 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Agent discovery successful
  Agent address: jenkins.jenkins-ops.svc.cluster.local
  Agent port:    50000
  Identity:      6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 9:32:54 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Handshaking
May 09, 2020 9:32:54 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connecting to jenkins.jenkins-ops.svc.cluster.local:50000
May 09, 2020 9:32:54 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Trying protocol: JNLP4-connect
May 09, 2020 9:32:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Remote identity confirmed: 6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 9:32:56 AM org.jenkinsci.remoting.protocol.impl.ConnectionHeadersFilterLayer onRecv
INFO: [JNLP4-connect connection to jenkins.jenkins-ops.svc.cluster.local/10.152.183.5:50000] Local headers refused by remote: Unknown client name: jnlp-slave-qk0w3
May 09, 2020 9:32:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Protocol JNLP4-connect encountered an unexpected exception
java.util.concurrent.ExecutionException: org.jenkinsci.remoting.protocol.impl.ConnectionRefusalException: Unknown client name: jnlp-slave-qk0w3
        at org.jenkinsci.remoting.util.SettableFuture.get(SettableFuture.java:223)
        at hudson.remoting.Engine.innerRun(Engine.java:577)
        at hudson.remoting.Engine.run(Engine.java:488)
Caused by: org.jenkinsci.remoting.protocol.impl.ConnectionRefusalException: Unknown client name: jnlp-slave-qk0w3
        at org.jenkinsci.remoting.protocol.impl.ConnectionHeadersFilterLayer.newAbortCause(ConnectionHeadersFilterLayer.java:378)
        at org.jenkinsci.remoting.protocol.impl.ConnectionHeadersFilterLayer.onRecvClosed(ConnectionHeadersFilterLayer.java:433)
        at org.jenkinsci.remoting.protocol.ProtocolStack$Ptr.onRecvClosed(ProtocolStack.java:816)
        at org.jenkinsci.remoting.protocol.FilterLayer.onRecvClosed(FilterLayer.java:287)
        at org.jenkinsci.remoting.protocol.impl.SSLEngineFilterLayer.onRecvClosed(SSLEngineFilterLayer.java:172)
        at org.jenkinsci.remoting.protocol.ProtocolStack$Ptr.onRecvClosed(ProtocolStack.java:816)
        at org.jenkinsci.remoting.protocol.NetworkLayer.onRecvClosed(NetworkLayer.java:154)
        at org.jenkinsci.remoting.protocol.impl.BIONetworkLayer.access$1500(BIONetworkLayer.java:48)
        at org.jenkinsci.remoting.protocol.impl.BIONetworkLayer$Reader.run(BIONetworkLayer.java:247)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at hudson.remoting.Engine$1.lambda$newThread$0(Engine.java:97)
        at java.lang.Thread.run(Thread.java:748)
        Suppressed: java.nio.channels.ClosedChannelException
                ... 7 more

May 09, 2020 9:32:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connecting to jenkins.jenkins-ops.svc.cluster.local:50000
May 09, 2020 9:32:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Server reports protocol JNLP4-plaintext not supported, skipping
May 09, 2020 9:32:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Protocol JNLP3-connect is not enabled, skipping
May 09, 2020 9:32:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Server reports protocol JNLP2-connect not supported, skipping
May 09, 2020 9:32:56 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Server reports protocol JNLP-connect not supported, skipping
May 09, 2020 9:32:56 AM hudson.remoting.jnlp.Main$CuiListener error
SEVERE: The server rejected the connection: None of the protocols were accepted
java.lang.Exception: The server rejected the connection: None of the protocols were accepted
        at hudson.remoting.Engine.onConnectionRejected(Engine.java:662)
        at hudson.remoting.Engine.innerRun(Engine.java:602)
        at hudson.remoting.Engine.run(Engine.java:488)
```

```sh
kbc logs jnlp-slave-8h4t9 -n jenkins-ops
Warning: JnlpProtocol3 is disabled by default, use JNLP_PROTOCOL_OPTS to alter the behavior
May 09, 2020 10:12:10 AM hudson.remoting.jnlp.Main createEngine
INFO: Setting up agent: jnlp-slave-8h4t9
May 09, 2020 10:12:10 AM hudson.remoting.jnlp.Main$CuiListener <init>
INFO: Jenkins agent is running in headless mode.
May 09, 2020 10:12:11 AM hudson.remoting.Engine startEngine
INFO: Using Remoting version: 3.36
May 09, 2020 10:12:11 AM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /home/jenkins/agent/remoting as a remoting work directory
May 09, 2020 10:12:11 AM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
INFO: Both error and output logs will be printed to /home/jenkins/agent/remoting
May 09, 2020 10:12:12 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://jenkins.jenkins-ops.svc.cluster.local/]
May 09, 2020 10:12:12 AM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
INFO: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
May 09, 2020 10:12:13 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Agent discovery successful
  Agent address: jenkins.jenkins-ops.svc.cluster.local
  Agent port:    50000
  Identity:      6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 10:12:13 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Handshaking
May 09, 2020 10:12:13 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connecting to jenkins.jenkins-ops.svc.cluster.local:50000
May 09, 2020 10:12:13 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Trying protocol: JNLP4-connect
May 09, 2020 10:12:14 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Remote identity confirmed: 6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 10:12:17 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connected
May 09, 2020 10:12:58 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Terminated
May 09, 2020 10:13:11 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Performing onReconnect operation.
May 09, 2020 10:13:11 AM jenkins.slaves.restarter.JnlpSlaveRestarterInstaller$FindEffectiveRestarters$1 onReconnect
INFO: Restarting agent via jenkins.slaves.restarter.UnixSlaveRestarter@5f8b4f36
May 09, 2020 10:13:14 AM hudson.remoting.jnlp.Main createEngine
INFO: Setting up agent: jnlp-slave-8h4t9
May 09, 2020 10:13:14 AM hudson.remoting.jnlp.Main$CuiListener <init>
INFO: Jenkins agent is running in headless mode.
May 09, 2020 10:13:15 AM hudson.remoting.Engine startEngine
INFO: Using Remoting version: 3.36
May 09, 2020 10:13:15 AM org.jenkinsci.remoting.engine.WorkDirManager initializeWorkDir
INFO: Using /home/jenkins/agent/remoting as a remoting work directory
May 09, 2020 10:13:15 AM org.jenkinsci.remoting.engine.WorkDirManager setupLogging
INFO: Both error and output logs will be printed to /home/jenkins/agent/remoting
May 09, 2020 10:13:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Locating server among [http://jenkins.jenkins-ops.svc.cluster.local/]
May 09, 2020 10:13:16 AM org.jenkinsci.remoting.engine.JnlpAgentEndpointResolver resolve
INFO: Remoting server accepts the following protocols: [JNLP4-connect, Ping]
May 09, 2020 10:13:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Agent discovery successful
  Agent address: jenkins.jenkins-ops.svc.cluster.local
  Agent port:    50000
  Identity:      6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 10:13:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Handshaking
May 09, 2020 10:13:16 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connecting to jenkins.jenkins-ops.svc.cluster.local:50000
May 09, 2020 10:13:17 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Trying protocol: JNLP4-connect
May 09, 2020 10:13:18 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Remote identity confirmed: 6f:d8:24:83:42:b8:53:bb:a4:42:8f:cf:26:39:22:71
May 09, 2020 10:13:18 AM org.jenkinsci.remoting.protocol.impl.ConnectionHeadersFilterLayer onRecv
INFO: [JNLP4-connect connection to jenkins.jenkins-ops.svc.cluster.local/10.152.183.5:50000] Local headers refused by remote: Unknown client name: jnlp-slave-8h4t9
May 09, 2020 10:13:18 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Protocol JNLP4-connect encountered an unexpected exception
java.util.concurrent.ExecutionException: org.jenkinsci.remoting.protocol.impl.ConnectionRefusalException: Unknown client name: jnlp-slave-8h4t9
        at org.jenkinsci.remoting.util.SettableFuture.get(SettableFuture.java:223)
        at hudson.remoting.Engine.innerRun(Engine.java:577)
        at hudson.remoting.Engine.run(Engine.java:488)
Caused by: org.jenkinsci.remoting.protocol.impl.ConnectionRefusalException: Unknown client name: jnlp-slave-8h4t9
        at org.jenkinsci.remoting.protocol.impl.ConnectionHeadersFilterLayer.newAbortCause(ConnectionHeadersFilterLayer.java:378)
        at org.jenkinsci.remoting.protocol.impl.ConnectionHeadersFilterLayer.onRecvClosed(ConnectionHeadersFilterLayer.java:433)
        at org.jenkinsci.remoting.protocol.ProtocolStack$Ptr.onRecvClosed(ProtocolStack.java:816)
        at org.jenkinsci.remoting.protocol.FilterLayer.onRecvClosed(FilterLayer.java:287)
        at org.jenkinsci.remoting.protocol.impl.SSLEngineFilterLayer.onRecvClosed(SSLEngineFilterLayer.java:172)
        at org.jenkinsci.remoting.protocol.ProtocolStack$Ptr.onRecvClosed(ProtocolStack.java:816)
        at org.jenkinsci.remoting.protocol.NetworkLayer.onRecvClosed(NetworkLayer.java:154)
        at org.jenkinsci.remoting.protocol.impl.BIONetworkLayer.access$1500(BIONetworkLayer.java:48)
        at org.jenkinsci.remoting.protocol.impl.BIONetworkLayer$Reader.run(BIONetworkLayer.java:247)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at hudson.remoting.Engine$1.lambda$newThread$0(Engine.java:97)
        at java.lang.Thread.run(Thread.java:748)
        Suppressed: java.nio.channels.ClosedChannelException
                ... 7 more

May 09, 2020 10:13:18 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Connecting to jenkins.jenkins-ops.svc.cluster.local:50000
May 09, 2020 10:13:18 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Server reports protocol JNLP4-plaintext not supported, skipping
May 09, 2020 10:13:18 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Protocol JNLP3-connect is not enabled, skipping
May 09, 2020 10:13:18 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Server reports protocol JNLP2-connect not supported, skipping
May 09, 2020 10:13:18 AM hudson.remoting.jnlp.Main$CuiListener status
INFO: Server reports protocol JNLP-connect not supported, skipping
May 09, 2020 10:13:18 AM hudson.remoting.jnlp.Main$CuiListener error
SEVERE: The server rejected the connection: None of the protocols were accepted
java.lang.Exception: The server rejected the connection: None of the protocols were accepted
        at hudson.remoting.Engine.onConnectionRejected(Engine.java:662)
        at hudson.remoting.Engine.innerRun(Engine.java:602)
        at hudson.remoting.Engine.run(Engine.java:488)
```
