---
title: ‘Jenkins’ doesn’t have label ‘slave-agent’ 异常问题
date: 2020-01-02 14:00:00
tags: 'Jenkins'
categories:
  - ['部署', 'k8s']
permalink: Jenkins-does-not-have-label-slave-agent
photo:
---

## 过程

最近没搭理 VPS 上的 minik8s 节点了, 后面升级的时候看了一下日志, 发现一大堆的错误. 其中大量出现创建 slave 的时候报错 `java.net.ProtocolException: Expected HTTP 101 response but was '403 Forbidden'`, 第一次看到 `403` 状态码的时候就觉得是不是新版本的权限配置更改了, 但是看了官网的 [service-account.yml](https://github.com/jenkinsci/kubernetes-plugin/blob/master/src/main/kubernetes/service-account.yml) 发现并没有问题, 再看后面插件的错误 `at io.fabric8.kubernetes.client.dsl.internal.WatchConnectionManager$1.onFailure(WatchConnectionManager.java:198)`, 经过查询, 这应该是插件的问题 [RP](https://github.com/jenkinsci/kubernetes-plugin/pull/582).

<!-- more -->

## 日志

```sh
Jan 02, 2020 1:42:16 PM hudson.slaves.NodeProvisioner$2 run
INFO: Kubernetes Pod Template provisioning successfully completed. We have now 2 computer(s)
Jan 02, 2020 1:42:16 PM org.csanchez.jenkins.plugins.kubernetes.KubernetesCloud provision
INFO: Excess workload after pending Kubernetes agents: 0
Jan 02, 2020 1:42:16 PM org.csanchez.jenkins.plugins.kubernetes.KubernetesCloud provision
INFO: Template for label slave-agent: Kubernetes Pod Template
Jan 02, 2020 1:42:16 PM org.csanchez.jenkins.plugins.kubernetes.KubernetesLauncher launch
INFO: Created Pod: jnlp-slave-0kxg8 in namespace jenkins-ops
Jan 02, 2020 1:42:17 PM okhttp3.internal.platform.Platform log
INFO: ALPN callback dropped: HTTP/2 is disabled. Is alpn-boot on the boot class path?
Jan 02, 2020 1:42:17 PM io.fabric8.kubernetes.client.dsl.internal.WatchConnectionManager$1 onFailure
WARNING: Exec Failure: HTTP 403, Status: 403 -
java.net.ProtocolException: Expected HTTP 101 response but was '403 Forbidden'
        at okhttp3.internal.ws.RealWebSocket.checkResponse(RealWebSocket.java:229)
        at okhttp3.internal.ws.RealWebSocket$2.onResponse(RealWebSocket.java:196)
        at okhttp3.RealCall$AsyncCall.execute(RealCall.java:206)
        at okhttp3.internal.NamedRunnable.run(NamedRunnable.java:32)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)

Jan 02, 2020 1:42:17 PM org.csanchez.jenkins.plugins.kubernetes.KubernetesLauncher launch
WARNING: Error in provisioning; agent=KubernetesSlave name: jnlp-slave-0kxg8, template=PodTemplate{inheritFrom='', name='jnlp-slave', namespace='', label='slave-agent', serviceAccount='jenkins', nodeSelector='', nodeUsageMode=EXCLUSIVE, workspaceVolume=EmptyDirWorkspaceVolume [memory=false], volumes=[HostPathVolume [mountPath=/var/run/docker.sock, hostPath=/var/run/docker.sock]], containers=[ContainerTemplate{name='jnlp', image='cnych/jenkins:jnlp', workingDir='/home/jenkins', command='', args='', ttyEnabled=true, resourceRequestCpu='', resourceRequestMemory='', resourceLimitCpu='', resourceLimitMemory='', livenessProbe=org.csanchez.jenkins.plugins.kubernetes.ContainerLivenessProbe@325d3daa}]}
io.fabric8.kubernetes.client.KubernetesClientException:
        at io.fabric8.kubernetes.client.dsl.internal.WatchConnectionManager$1.onFailure(WatchConnectionManager.java:198)
        at okhttp3.internal.ws.RealWebSocket.failWebSocket(RealWebSocket.java:571)
        at okhttp3.internal.ws.RealWebSocket$2.onResponse(RealWebSocket.java:198)
        at okhttp3.RealCall$AsyncCall.execute(RealCall.java:206)
        at okhttp3.internal.NamedRunnable.run(NamedRunnable.java:32)
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
        at java.lang.Thread.run(Thread.java:748)

Jan 02, 2020 1:42:17 PM org.csanchez.jenkins.plugins.kubernetes.KubernetesSlave _terminate
INFO: Terminating Kubernetes instance for agent jnlp-slave-0kxg8
Jan 02, 2020 1:42:17 PM org.csanchez.jenkins.plugins.kubernetes.KubernetesSlave deleteSlavePod
INFO: Terminated Kubernetes instance for agent jenkins-ops/jnlp-slave-0kxg8
Terminated Kubernetes instance for agent jenkins-ops/jnlp-slave-0kxg8
Jan 02, 2020 1:42:17 PM org.csanchez.jenkins.plugins.kubernetes.KubernetesSlave _terminate
INFO: Disconnected computer jnlp-slave-0kxg8
Disconnected computer jnlp-slave-0kxg8
Jan 02, 2020 1:42:26 PM org.csanchez.jenkins.plugins.kubernetes.KubernetesCloud provision
INFO: Excess workload after pending Kubernetes agents: 1
Jan 02, 2020 1:42:26 PM org.csanchez.jenkins.plugins.kubernetes.KubernetesCloud provision
INFO: Template for label slave-agent: Kubernetes Pod Template
Jan 02, 2020 1:42:26 PM hudson.slaves.NodeProvisioner$StandardStrategyImpl apply
INFO: Started provisioning Kubernetes Pod Template from kubernetes with 1 executors. Remaining excess workload: 0
```

## 版本

VPS 机器上的版本应该更新过了.

```sh
➜  jenkins kbc version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:20:10Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:12:17Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
```

## 插件

通过 Jenkins 自动将插件升级到 `1.16.7` 通过手动点立即获取可以看到当前最新版本为 [1.22.3](https://plugins.jenkins.io/kubernetes), 更新过程十分缓慢, 怀疑是[源地址](https://updates.jenkins.io/update-center.json)在 q 外的原因.

更新完成后发现插件部分配置改动了, 但是之前的配置还是生效的, 测试了可以正常连接.

## 新问题

```sh
Jan 02, 2020 4:35:58 PM INFO hudson.slaves.NodeProvisioner lambda$update$6
Kubernetes Pod Template provisioning successfully completed. We have now 2 computer(s)
Jan 02, 2020 4:35:58 PM INFO org.csanchez.jenkins.plugins.kubernetes.KubernetesLauncher launch
Created Pod: jenkins-ops/jnlp-slave-jd2cv
Jan 02, 2020 4:35:58 PM INFO okhttp3.internal.platform.Platform log
ALPN callback dropped: HTTP/2 is disabled. Is alpn-boot on the boot class path?
Jan 02, 2020 4:35:59 PM INFO org.csanchez.jenkins.plugins.kubernetes.KubernetesLauncher launch
Pod is running: jenkins-ops/jnlp-slave-jd2cv
Jan 02, 2020 4:35:59 PM INFO org.csanchez.jenkins.plugins.kubernetes.KubernetesLauncher launch
Waiting for agent to connect (0/100): jnlp-slave-jd2cv
Jan 02, 2020 4:36:01 PM SEVERE org.csanchez.jenkins.plugins.kubernetes.KubernetesLauncher logLastLines
Error in provisioning; agent=KubernetesSlave name: jnlp-slave-jd2cv, template=PodTemplate{inheritFrom='', name='jnlp-slave', namespace='', hostNetwork=false, label='slave-agent', serviceAccount='jenkins', nodeSelector='', nodeUsageMode=EXCLUSIVE, workspaceVolume=EmptyDirWorkspaceVolume [memory=false], volumes=[HostPathVolume [mountPath=/var/run/docker.sock, hostPath=/var/run/docker.sock]], containers=[ContainerTemplate{name='jnlp', image='cnych/jenkins:jnlp', workingDir='/home/jenkins/agent', command='', args='', ttyEnabled=true, resourceRequestCpu='', resourceRequestMemory='', resourceLimitCpu='', resourceLimitMemory='', livenessProbe=org.csanchez.jenkins.plugins.kubernetes.ContainerLivenessProbe@70d7620c}]}. Container jnlp. Logs: Warning: JnlpProtocol3 is disabled by default, use JNLP_PROTOCOL_OPTS to alter the behavior
"-workDir" is not a valid option
java -jar slave.jar [options...] <secret key> <slave name>
 -cert VAL                       : Specify additional X.509 encoded PEM
                                   certificates to trust when connecting to
                                   Jenkins root URLs. If starting with @ then
                                   the remainder is assumed to be the name of
                                   the certificate file to read.
 -credentials USER:PASSWORD      : HTTP BASIC AUTH header to pass in for making
                                   HTTP requests.
 -headless                       : Run in headless mode, without GUI
 -jar-cache DIR                  : Cache directory that stores jar files sent
                                   from the master
 -noKeepAlive                    : Disable TCP socket keep alive on connection
                                   to the master.
 -noreconnect                    : If the connection ends, don't retry and just
                                   exit.
 -proxyCredentials USER:PASSWORD : HTTP BASIC AUTH header to pass in for making
                                   HTTP authenticated proxy requests.
 -tunnel HOST:PORT               : Connect to the specified host and port,
                                   instead of connecting directly to Jenkins.
                                   Useful when connection to Hudson needs to be
                                   tunneled. Can be also HOST: or :PORT, in
                                   which case the missing portion will be
                                   auto-configured like the default behavior
 -url URL                        : Specify the Jenkins root URLs to connect to.

Jan 02, 2020 4:36:01 PM WARNING org.csanchez.jenkins.plugins.kubernetes.KubernetesLauncher launch
Error in provisioning; agent=KubernetesSlave name: jnlp-slave-jd2cv, template=PodTemplate{inheritFrom='', name='jnlp-slave', namespace='', hostNetwork=false, label='slave-agent', serviceAccount='jenkins', nodeSelector='', nodeUsageMode=EXCLUSIVE, workspaceVolume=EmptyDirWorkspaceVolume [memory=false], volumes=[HostPathVolume [mountPath=/var/run/docker.sock, hostPath=/var/run/docker.sock]], containers=[ContainerTemplate{name='jnlp', image='cnych/jenkins:jnlp', workingDir='/home/jenkins/agent', command='', args='', ttyEnabled=true, resourceRequestCpu='', resourceRequestMemory='', resourceLimitCpu='', resourceLimitMemory='', livenessProbe=org.csanchez.jenkins.plugins.kubernetes.ContainerLivenessProbe@70d7620c}]}
java.lang.IllegalStateException: Agent is not connected after 1 seconds, status: Succeeded
  at org.csanchez.jenkins.plugins.kubernetes.KubernetesLauncher.launch(KubernetesLauncher.java:191)
  at hudson.slaves.SlaveComputer.lambda$_connect$0(SlaveComputer.java:290)
  at jenkins.util.ContextResettingExecutorService$2.call(ContextResettingExecutorService.java:46)
  at jenkins.security.ImpersonatingExecutorService$2.call(ImpersonatingExecutorService.java:71)
  at java.util.concurrent.FutureTask.run(FutureTask.java:266)
  at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
  at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
  at java.lang.Thread.run(Thread.java:748)

Jan 02, 2020 4:36:01 PM INFO org.csanchez.jenkins.plugins.kubernetes.KubernetesSlave _terminate
Terminating Kubernetes instance for agent jnlp-slave-jd2cv
Jan 02, 2020 4:36:01 PM INFO org.csanchez.jenkins.plugins.kubernetes.KubernetesSlave deleteSlavePod
Terminated Kubernetes instance for agent jenkins-ops/jnlp-slave-jd2cv
Jan 02, 2020 4:36:01 PM INFO org.csanchez.jenkins.plugins.kubernetes.KubernetesSlave _terminate
Disconnected computer jnlp-slave-jd2cv
Jan 02, 2020 4:36:08 PM INFO org.csanchez.jenkins.plugins.kubernetes.KubernetesCloud provision
Excess workload after pending Kubernetes agents: 1
Jan 02, 2020 4:36:08 PM INFO org.csanchez.jenkins.plugins.kubernetes.KubernetesCloud provision
Template for label slave-agent: Kubernetes Pod Template
```

出现新的错误, 使用官方镜像 `jenkins/jnlp-slave` 就可以了, 应该是封装镜像里面命令错误了.
