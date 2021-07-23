---
title: k8s-pod-yaml
date: 2021-02-01 10:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: k8s-pod-yaml
photo:
---

https://www.cnblogs.com/flying1819/articles/9039529.html

        k8s yaml 格式的 Pod 配置文件 - flyoss - 博客园

 [! [](https://img2020.cnblogs.com/blog/35695/202102/35695-20210205110158186-1063387091.jpg)](https://developer.aliyun.com/learning/trainingcamp/java/1? utm_content= g_1000236192)

*   [! [博客园 Logo](/images/logo.svg? v= R9M0WmLAIPVydmdzE2keuvnjl-bPR7_35oHqtiBzGsM)](https://www.cnblogs.com/ " 开发者的网上家园 ")
*   [首页](/)
*   [新闻](https://news.cnblogs.com/)
*   [博问](https://q.cnblogs.com/)
*   [专区](https://brands.cnblogs.com/)
*   [闪存](https://ing.cnblogs.com/)
*   [班级](https://edu.cnblogs.com/)

*    ! [搜索](/images/aggsite/search.svg)

*    [! [我的博客](/images/aggsite/myblog.svg)](https://passport.cnblogs.com/GetBlogApplyStatus.aspx " 我的博客 ") [! [短消息](/images/aggsite/message.svg? v= oS4PkibyMjZ9rGD5XAcLt99uW_s76Javy2up4dbnZNY)](https://msg.cnblogs.com/ " 短消息 ")

     [! [用户头像](/images/aggsite/avatar-default.svg)](https://home.cnblogs.com/)

    [我的博客](https://passport.cnblogs.com/GetBlogApplyStatus.aspx) [我的园子](https://home.cnblogs.com/) [账号设置](https://account.cnblogs.com/settings/account) [退出登录](javascript: void(0))

    [注册](https://account.cnblogs.com/signup/) [登录](javascript: void(0);)

# [flyoss](https://www.cnblogs.com/flying1819/)

## [k8s yaml 格式的 Pod 配置文件](https://www.cnblogs.com/flying1819/articles/9039529.html)

### kubernetes 核心技术概念

```
kubernetes 核心技术概念
复制控制器 (Replication Controller, RC)

RC 是 K8s 集群中保证 Pod 高可用的 API 对象. 通过监控运行中的 Pod 来保证集群中运行指定数目的 Pod 副本. 指定的数目可以是多个也可以是 1 个; 少于指定数目, RC 就会启动运行新的 Pod 副本;
多于指定数目, RC 就会杀死多余的 Pod 副本. 即使在指定数目为 1 的情况下, 通过 RC 运行 Pod 也比直接运行 Pod 更明智, 因为 RC 也可以发挥它高可用的能力, 保证永远有 1 个 Pod 在运行.
RC 是 K8s 较早期的技术概念, 只适用于长期伺服型的业务类型, 比如控制小机器人提供高可用的 Web 服务.



副本集 (Replica Set, RS)

RS 是新一代 RC, 提供同样的高可用能力, 区别主要在于 RS 后来居上, 能支持更多种类的匹配模式. 副本集对象一般不单独使用, 而是作为 Deployment 的理想状态参数使用.



部署 (Deployment)

部署表示用户对 K8s 集群的一次更新操作. 部署是一个比 RS 应用模式更广的 API 对象, 可以是创建一个新的服务, 更新一个新的服务, 也可以是滚动升级一个服务.
滚动升级一个服务, 实际是创建一个新的 RS, 然后逐渐将新 RS 中副本数增加到理想状态, 将旧 RS 中的副本数减小到 0 的复合操作;
这样一个复合操作用一个 RS 是不太好描述的, 所以用一个更通用的 Deployment 来描述.



服务 (Service)

   RC, RS 和 Deployment 只是保证了支撑服务的微服务 Pod 的数量, 但是没有解决如何访问这些服务的问题.
一个 Pod 只是一个运行服务的实例, 随时可能在一个节点上停止, 在另一个节点以一个新的 IP 启动一个新的 Pod, 因此不能以确定的 IP 和端口号提供服务.
要稳定地提供服务需要服务发现和负载均衡能力. 服务发现完成的工作, 是针对客户端访问的服务, 找到对应的的后端服务实例. 在 K8s 集群中, 客户端需要访问的服务就是 Service 对象.
每个 Service 会对应一个集群内部有效的虚拟 IP, 集群内部通过虚拟 IP 访问一个服务.
在 K8s 集群中微服务的负载均衡是由 Kube\-proxy 实现的.Kube-proxy 是 K8s 集群内部的负载均衡器.
它是一个分布式代理服务器, 在 K8s 的每个节点上都有一个; 这一设计体现了它的伸缩性优势, 需要访问服务的节点越多, 提供负载均衡能力的 Kube-proxy 就越多, 高可用节点也随之增多.
与之相比, 我们平时在服务器端做个反向代理做负载均衡, 还要进一步解决反向代理的负载均衡和高可用问题.



任务 (Job)

Job 是 K8s 用来控制批处理型任务的 API 对象. 批处理业务与长期伺服业务的主要区别是批处理业务的运行有头有尾, 而长期伺服业务在用户不停止的情况下永远运行.
Job 管理的 Pod 根据用户的设置把任务成功完成就自动退出了.
成功完成的标志根据不同的 spec.completions 策略而不同: 单 Pod 型任务有一个 Pod 成功就标志完成; 定数成功型任务保证有 N 个任务全部成功; 工作队列型任务根据应用确认的全局成功而标志成功.



后台支撑服务集 (DaemonSet)

长期伺服型和批处理型服务的核心在业务应用, 可能有些节点运行多个同类业务的 Pod,
有些节点上又没有这类 Pod 运行; 而后台支撑型服务的核心关注点在 K8s 集群中的节点 (物理机或虚拟机), 要保证每个节点上都有一个此类 Pod 运行.
节点可能是所有集群节点也可能是通过 nodeSelector 选定的一些特定节点. 典型的后台支撑型服务包括, 存储, 日志和监控等在每个节点上支持 K8s 集群运行的服务.



有状态服务集 (PetSet)

RC 和 RS 主要是控制提供无状态服务的, 其所控制的 Pod 的名字是随机设置的, 一个 Pod 出故障了就被丢弃掉, 在另一个地方重启一个新的 Pod, 名字变了, 名字和启动在哪儿都不重要, 重要的只是 Pod 总数;
而 PetSet 是用来控制有状态服务, PetSet 中的每个 Pod 的名字都是事先确定的, 不能更改.PetSet 中 Pod 的名字的作用, 是关联与该 Pod 对应的状态.

对于 RC 和 RS 中的 Pod, 一般不挂载存储或者挂载共享存储, 保存的是所有 Pod 共享的状态, Pod 像牲畜一样没有分别 (这似乎也确实意味着失去了人性特征);
对于 PetSet 中的 Pod, 每个 Pod 挂载自己独立的存储, 如果一个 Pod 出现故障, 从其他节点启动一个同样名字的 Pod, 要挂载上原来 Pod 的存储继续以它的状态提供服务.

适合于 PetSet 的业务包括数据库服务 MySQL 和 PostgreSQL, 集群化管理服务 Zookeeper, etcd 等有状态服务.
PetSet 的另一种典型应用场景是作为一种比普通容器更稳定可靠的模拟虚拟机的机制.
传统的虚拟机正是一种有状态的宠物, 运维人员需要不断地维护它, 容器刚开始流行时, 我们用容器来模拟虚拟机使用, 所有状态都保存在容器里, 而这已被证明是非常不安全, 不可靠的.
使用 PetSet, Pod 仍然可以通过漂移到不同节点提供高可用, 而存储也可以通过外挂的存储来提供高可靠性, PetSet 做的只是将确定的 Pod 与确定的存储关联起来保证状态的连续性.
PetSet 还只在 Alpha 阶段, 后面的设计如何演变, 我们还要继续观察.



集群联邦 (Federation)

K8s 在 1.3 版本里发布了 beta 版的 Federation 功能. 在云计算环境中, 服务的作用距离范围从近到远一般可以有:
同主机 (Host, Node), 跨主机同可用区 (Available Zone), 跨可用区同地区 (Region), 跨地区同服务商 (Cloud Service Provider), 跨云平台.
K8s 的设计定位是单一集群在同一个地域内, 因为同一个地区的网络性能才能满足 K8s 的调度和计算存储连接要求. 而联合集群服务就是为提供跨 Region 跨服务商 K8s 集群服务而设计的.

每个 K8s Federation 有自己的分布式存储, API Server 和 Controller Manager. 用户可以通过 Federation 的 API Server 注册该 Federation 的成员 K8s Cluster.
当用户通过 Federation 的 API Server 创建, 更改 API 对象时, Federation API Server 会在自己所有注册的子 K8s Cluster 都创建一份对应的 API 对象.
在提供业务请求服务时, K8s Federation 会先在自己的各个子 Cluster 之间做负载均衡, 而对于发送到某个具体 K8s Cluster 的业务请求,
会依照这个 K8s Cluster 独立提供服务时一样的调度模式去做 K8s Cluster 内部的负载均衡. 而 Cluster 之间的负载均衡是通过域名服务的负载均衡来实现的.

所有的设计都尽量不影响 K8s Cluster 现有的工作机制, 这样对于每个子 K8s 集群来说, 并不需要更外层的有一个 K8s Federation,
也就是意味着所有现有的 K8s 代码和机制不需要因为 Federation 功能有任何变化.



存储卷 (Volume)

K8s 集群中的存储卷跟 Docker 的存储卷有些类似, 只不过 Docker 的存储卷作用范围为一个容器, 而 K8s 的存储卷的生命周期和作用范围是一个 Pod.
每个 Pod 中声明的存储卷由 Pod 中的所有容器共享.K8s 支持非常多的存储卷类型, 特别的, 支持多种公有云平台的存储, 包括 AWS, Google 和 Azure 云;
支持多种分布式存储包括 GlusterFS 和 Ceph; 也支持较容易使用的主机本地目录 hostPath 和 NFS.
K8s 还支持使用 Persistent Volume Claim 即 PVC 这种逻辑存储, 使用这种存储, 使得存储的使用者可以忽略后台的实际存储技术 (例如 AWS, Google 或 GlusterFS 和 Ceph),
而将有关存储实际技术的配置交给存储管理员通过 Persistent Volume 来配置.

持久存储卷 (Persistent Volume, PV) 和持久存储卷声明 (Persistent Volume Claim, PVC)

PV 和 PVC 使得 K8s 集群具备了存储的逻辑抽象能力, 使得在配置 Pod 的逻辑里可以忽略对实际后台存储技术的配置, 而把这项配置的工作交给 PV 的配置者, 即集群的管理者.
存储的 PV 和 PVC 的这种关系, 跟计算的 Node 和 Pod 的关系是非常类似的; PV 和 Node 是资源的提供者, 根据集群的基础设施变化而变化, 由 K8s 集群管理员配置; 而 PVC 和 Pod 是资源的使用者, 根据业务服务的需求变化而变化, 有 K8s 集群的使用者即服务的管理员来配置.



节点 (Node)

K8s 集群中的计算能力由 Node 提供, 最初 Node 称为服务节点 Minion, 后来改名为 Node.
K8s 集群中的 Node 也就等同于 Mesos 集群中的 Slave 节点, 是所有 Pod 运行所在的工作主机, 可以是物理机也可以是虚拟机.
不论是物理机还是虚拟机, 工作主机的统一特征是上面要运行 kubelet 管理节点上运行的容器.



密钥对象 (Secret)

Secret 是用来保存和传递密码, 密钥, 认证凭证这些敏感信息的对象. 使用 Secret 的好处是可以避免把敏感信息明文写在配置文件里.
在 K8s 集群中配置和使用服务不可避免的要用到各种敏感信息实现登录, 认证等功能, 例如访问 AWS 存储的用户名密码.
为了避免将类似的敏感信息明文写在所有需要使用的配置文件中, 可以将这些信息存入一个 Secret 对象, 而在配置文件中通过 Secret 对象引用这些敏感信息.
这种方式的好处包括: 意图明确, 避免重复, 减少暴漏机会.



用户帐户 (User Account) 和服务帐户 (Service Account)

顾名思义, 用户帐户为人提供账户标识, 而服务账户为计算机进程和 K8s 集群中运行的 Pod 提供账户标识.
用户帐户和服务帐户的一个区别是作用范围; 用户帐户对应的是人的身份, 人的身份与服务的 namespace 无关, 所以用户账户是跨 namespace 的;
而服务帐户对应的是一个运行中程序的身份, 与特定 namespace 是相关的.



名字空间 (Namespace)

名字空间为 K8s 集群提供虚拟的隔离作用, K8s 集群初始有两个名字空间, 分别是默认名字空间 default 和系统名字空间 kube\-system, 除此以外, 管理员可以可以创建新的名字空间满足需要.



RBAC 访问授权

K8s 在 1.3 版本中发布了 alpha 版的基于角色的访问控制 (Role\-based Access Control, RBAC) 的授权模式.
相对于基于属性的访问控制 (Attribute-based Access Control, ABAC), RBAC 主要是引入了角色 (Role) 和角色绑定 (RoleBinding) 的抽象概念.
在 ABAC 中, K8s 集群中的访问策略只能跟用户直接关联; 而在 RBAC 中, 访问策略可以跟某个角色关联, 具体的用户在跟一个或多个角色相关联.
显然, RBAC 像其他新功能一样, 每次引入新功能, 都会引入新的 API 对象, 从而引入新的概念抽象, 而这一新的概念抽象一定会使集群服务管理和使用更容易扩展和重用.
```

###
kubernetes yaml 文件解析

! [复制代码](https://common.cnblogs.com/images/copycode.gif)

```
\# yaml 格式的 pod 定义文件完整内容:
apiVersion: v1          #必选, 版本号, 例如 v1
kind: Pod             #必选, Pod
metadata:             #必选, 元数据
  name: string          #必选, Pod 名称
  namespace: string       #必选, Pod 所属的命名空间
  labels:             #自定义标签
    - name: string       #自定义标签名字
  annotations:          #自定义注释列表
    - name: string
spec:                #必选, Pod 中容器的详细定义
  containers:           #必选, Pod 中容器列表
  - name: string        #必选, 容器名称
    image: string       #必选, 容器的镜像名称
    imagePullPolicy: \[Always | Never | IfNotPresent\]  #获取镜像的策略 Alawys 表示下载镜像 IfnotPresent 表示优先使用本地镜像, 否则下载镜像, Nerver 表示仅使用本地镜像
    command: \[string\]       #容器的启动命令列表, 如不指定, 使用打包时使用的启动命令
    args: \[string\]         #容器的启动命令参数列表
    workingDir: string      #容器的工作目录
    volumeMounts:         #挂载到容器内部的存储卷配置
    - name: string         #引用 pod 定义的共享存储卷的名称, 需用 volumes\[\] 部分定义的的卷名
      mountPath: string     #存储卷在容器内 mount 的绝对路径, 应少于 512 字符
      readOnly: boolean     #是否为只读模式
    ports:              #需要暴露的端口库号列表
    - name: string         #端口号名称
      containerPort: int    #容器需要监听的端口号
      hostPort: int        #容器所在主机需要监听的端口号, 默认与 Container 相同
      protocol: string      #端口协议, 支持 TCP 和 UDP, 默认 TCP
    env:              #容器运行前需设置的环境变量列表
    - name: string        #环境变量名称
      value: string       #环境变量的值
    resources:          #资源限制和请求的设置
      limits:           #资源限制的设置
        cpu: string       #Cpu 的限制, 单位为 core 数, 将用于 docker run --cpu-shares 参数
        memory: string      #内存限制, 单位可以为 Mib/Gib, 将用于 docker run --memory 参数
      requests:         #资源请求的设置
        cpu: string       #Cpu 请求, 容器启动的初始可用数量
        memory: string      #内存清楚, 容器启动的初始可用数量
    livenessProbe:        #对 Pod 内个容器健康检查的设置, 当探测无响应几次后将自动重启该容器, 检查方法有 exec, httpGet 和 tcpSocket, 对一个容器只需设置其中一种方法即可
      exec:             #对 Pod 容器内检查方式设置为 exec 方式
        command: \[string\]   #exec 方式需要制定的命令或脚本
      httpGet:            #对 Pod 内个容器健康检查方法设置为 HttpGet, 需要制定 Path, port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        - name: string
          value: string
      tcpSocket:            #对 Pod 内个容器健康检查方式设置为 tcpSocket 方式
         port: number
       initialDelaySeconds: 0   #容器启动完成后首次探测的时间, 单位为秒
       timeoutSeconds: 0      #对容器健康检查探测等待响应的超时时间, 单位秒, 默认 1 秒
       periodSeconds: 0       #对容器监控检查的定期探测时间设置, 单位秒, 默认 10 秒一次
       successThreshold: 0
       failureThreshold: 0
       securityContext:
         privileged: false
    restartPolicy: \[Always | Never | OnFailure\] #Pod 的重启策略, Always 表示一旦不管以何种方式终止运行, kubelet 都将重启, OnFailure 表示只有 Pod 以非 0 退出码退出才重启, Nerver 表示不再重启该 Pod
    nodeSelector: obeject     #设置 NodeSelector 表示将该 Pod 调度到包含这个 label 的 node 上, 以 key: value 的格式指定
    imagePullSecrets:         #Pull 镜像时使用的 secret 名称, 以 key: secretkey 格式指定
    - name: string
    hostNetwork: false        #是否使用主机网络模式, 默认为 false, 如果设置为 true, 表示使用宿主机网络
    volumes:              #在该 pod 上定义共享存储卷列表
    - name: string          #共享存储卷名称 (volumes 类型有很多种)
      emptyDir: {}          #类型为 emtyDir 的存储卷, 与 Pod 同生命周期的一个临时目录. 为空值
      hostPath: string        #类型为 hostPath 的存储卷, 表示挂载 Pod 所在宿主机的目录
        path: string        #Pod 所在宿主机的目录, 将被用于同期中 mount 的目录
      secret:             #类型为 secret 的存储卷, 挂载集群与定义的 secre 对象到容器内部
        scretname: string
        items:
        - key: string
          path: string
      configMap:          #类型为 configMap 的存储卷, 挂载预定义的 configMap 对象到容器内部
        name: string
        items:
        - key: string
          path: string
```

! [复制代码](https://common.cnblogs.com/images/copycode.gif)

### [k8s dns](http://www.cnblogs.com/FRESHMANS/p/8446067.html)


Cluster DNS 主要包含如下几项:

1) SkyDNS 
提供 DNS 解析服务. 
2) Etcd 
用于 DNS 的存储. 
3) Kube2sky 
监听 Kubernetes, 当有新的 Service 创建时, 将其注册到 etcd 上. 
4) healthz 
提供对 skydns 服务的健康检查功能.


**在 master 服务器上**

! [复制代码](https://common.cnblogs.com/images/copycode.gif)

```
Cluster DNS 在 Kubernetes 发布包的 cluster/addons/dns 目录下

yum -y install wget
wget https://codeload.github.com/kubernetes/kubernetes/tar.gz/v1.2.7
tar zxvf v1.2.7
cd kubernetes-1.2.7/cluster/addons/dns
```

! [复制代码](https://common.cnblogs.com/images/copycode.gif)

skydns 服务使用的 clusterIP 需要我们指定一个固定的 IP 地址, 每个 Node 的 kubelet 进程都将使用这个 IP 地址, 不能通过 Kuberneters 自动给 skydns 分配.

```
通过环境变量, 配置参数
export DNS\_SERVER\_IP="10.254.10.2"
export DNS\_DOMAIN="cluster.local"
export DNS\_REPLICAS=1
```

设置 Cluster DNS Service 的 IP 为 10.254.10.2(不能和已分配的 IP 重复), Cluster DNS 的本地域为 cluster.local.

修改每台 Node 上的 kubelet 启动参数

```
vim /etc/kubernetes/kubelet
#在 KUBELET\_ARGS 里增加:
--cluster\_dns=10.254.10.2
--cluster\_domain= cluster.local

``````
#中间用空格隔开
```

! [](https://images2017.cnblogs.com/blog/1045282/201802/1045282-20180213091734640-1442872489.png)

重启 kubelet 服务

```
systemctl restart kubelet
```

master 下

生成 dns-rc.yaml 和 dns-svc.yaml
kubernetes-1.2.7/cluster/addons/dns 目录下.

! [](https://images2017.cnblogs.com/blog/1045282/201802/1045282-20180213091859437-179327361.png)

skydns-rc.yaml.in 和 skydns-svc.yaml.in 是两个模板文件, 通过设置的环境变量修改其中的相应属性值, 可以生成 Replication Controller 和 Service 的定义文件.

生成 Replication Controller 的定义文件 dns-rc.yaml 创建 RC

```
sed -e "s/{{ pillar\\\['dns\_replicas'\\\] }}/${DNS\_REPLICAS}/g; s/{{ pillar\\\['dns\_domain'\\\] }}/${DNS\_DOMAIN}/g" \\skydns-rc.yaml.in > dns-rc.yaml
```

### **查看 dns-rc.yaml**

! [复制代码](https://common.cnblogs.com/images/copycode.gif)

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: kube-dns-v11
  namespace: kube-system
  labels:
    k8s-app: kube-dns
    version: v11
    kubernetes.io/cluster-service: "true"
spec:
  replicas: 1
  selector:
    k8s-app: kube-dns
    version: v11
  template:
    metadata:
      labels:
        k8s-app: kube-dns
        version: v11
        kubernetes.io/cluster-service: "true"
    spec:
      containers:
      - name: etcd
        image: index.tenxcloud.com/google\_containers/etcd-amd64:2.2.1
        resources:
          # TODO: Set memory limits when we've profiled the container for large
          # clusters, then set request = limit to keep this container in
          # guaranteed class. Currently, this container falls into the
          # "burstable" category so the kubelet doesn't backoff from restarting it.
          limits:
            cpu: 100m
            memory: 500Mi
          requests:
            cpu: 100m
            memory: 50Mi
        command:
        - /usr/local/bin/etcd
        - -data-dir
        - /var/etcd/data
        - -listen-client-urls
        - http://127.0.0.1:2379, http://127.0.0.1:4001
        - -advertise-client-urls
        - http://127.0.0.1:2379, http://127.0.0.1:4001
        - -initial-cluster-token
        - skydns-etcd
        volumeMounts:
        - name: etcd-storage
          mountPath: /var/etcd/data
      - name: kube2sky
        image: index.tenxcloud.com/google\_containers/kube2sky:1.14
        resources:
          # TODO: Set memory limits when we've profiled the container for large
          # clusters, then set request = limit to keep this container in
          # guaranteed class. Currently, this container falls into the
          # "burstable" category so the kubelet doesn't backoff from restarting it.
          limits:
            cpu: 100m
            # Kube2sky watches all pods.
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 50Mi
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
            scheme: HTTP
          initialDelaySeconds: 60
          timeoutSeconds: 5
          successThreshold: 1
          failureThreshold: 5
        readinessProbe:
          httpGet:
            path: /readiness
            port: 8081
            scheme: HTTP
          # we poll on pod startup for the Kubernetes master service and
          # only setup the /readiness HTTP server once that's available.
          initialDelaySeconds: 30
          timeoutSeconds: 5
        args:
        # command = "/kube2sky"
        #特别注意
        #- -domain= cluster.local 是错误的写法
        #- -kube-master-url= http://192.168.121.143:8080 是错误的设置
        #会导致 CrashLoopBackOff 的错误
        #如果已经进行 CA 认证, 则可以不指定 kube-master-url
        - --domain= cluster.local
        - --kube-master-url= http://192.168.132.148:8080    ##master 地址
      - name: skydns
        image: index.tenxcloud.com/google\_containers/skydns:2015-10-13-8c72f8c
        resources:
          # TODO: Set memory limits when we've profiled the container for large
          # clusters, then set request = limit to keep this container in
          # guaranteed class. Currently, this container falls into the
          # "burstable" category so the kubelet doesn't backoff from restarting it.
          limits:
            cpu: 100m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 50Mi
        args:
        # command = "/skydns"
        - -machines= http://127.0.0.1:4001
        - -addr=0.0.0.0:53
        - -ns-rotate= false
        - -domain= cluster.local.
        ports:
        - containerPort: 53
          name: dns
          protocol: UDP
        - containerPort: 53
          name: dns-tcp
          protocol: TCP
      - name: healthz
        image: index.tenxcloud.com/google\_containers/exechealthz:1.0
        resources:
          # keep request = limit to keep this container in guaranteed class
          limits:
            cpu: 10m
            memory: 20Mi
          requests:
            cpu: 10m
            memory: 20Mi
        args:
        - -cmd= nslookup kubernetes.default.svc.cluster.local 127.0.0.1 >/dev/null
        - -port=8080
        ports:
        - containerPort: 8080
          protocol: TCP
      volumes:
      - name: etcd-storage
        emptyDir: {}
      dnsPolicy: Default  # Don't use cluster DNS.
```

! [复制代码](https://common.cnblogs.com/images/copycode.gif)

需要注意 kube2sky 需要**ServiceAccount**来调用 Kubernetes API. 
而 ServiceAccount 的使用需要对 Kubernetes 集群进行安全认证, 否则可能会导致 RC 无法自动创建 Pod 等错误.

 ! [](https://images2017.cnblogs.com/blog/1045282/201802/1045282-20180213092240874-939741633.png)

通过定义文件 dns-rc.yaml 创建 Cluster DNS Replication Controller

```
kubectl create -f dns-rc.yaml
```

验证 Cluster DNS Pod 是否创建运行成功:

```
kubectl get pod --namespace= kube-system -o wide
```

! [](https://images2017.cnblogs.com/blog/1045282/201802/1045282-20180213092354859-524306351.png)

生成 Service 的定义文件 dns-svc.yaml 创建 Service

```
sed -e "s/{{ pillar\\\['dns\_server'\\\] }}/${DNS\_SERVER\_IP}/g" \\skydns-svc.yaml.in > dns-svc.yaml
```

### **dns-svc.yaml**

! [复制代码](https://common.cnblogs.com/images/copycode.gif)

```
apiVersion: v1
kind: Service
metadata:
  name: kube-dns
  namespace: kube-system
  labels:
    k8s-app: kube-dns
    kubernetes.io/cluster-service: "true"
    kubernetes.io/name: "KubeDNS"
spec:
  selector:
    k8s-app: kube-dns
  clusterIP: 10.254.10.2
  ports:
  - name: dns
    port: 53
    protocol: UDP
  - name: dns-tcp
    port: 53
    protocol: TCP
```

! [复制代码](https://common.cnblogs.com/images/copycode.gif)

**根据 dns-svc.yaml 创建 Cluster DNS Service**

```
kubectl create -f dns-svc.yaml
```

**验证:**

```
kubectl get svc --namespace= kube-system -o wide
```

创建 Pod 验证 Cluster DNS

使用一个带有 nslookup 的工具来验证 DNS 是否能够正常工作: 
**busybox.yaml**

! [复制代码](https://common.cnblogs.com/images/copycode.gif)

```
apiVersion: v1
kind: Pod
metadata:
  name: busybox
  namespace: default
spec:
  containers:
  - name: busybox
    image: index.tenxcloud.com/google\_containers/busybox
    command:
      - sleep
      - "3600"
```

! [复制代码](https://common.cnblogs.com/images/copycode.gif)

```
#kubectl create -f busybox.yaml
#kubectl exec busybox -- nslookup kubernetes.default.svc.cluster.local
```

! [](https://images2017.cnblogs.com/blog/1045282/201802/1045282-20180213092849062-1958353476.png)

ok, 搭建完成

### done

posted on 2018-05-15 10:17  [flyoss](https://www.cnblogs.com/flying1819/)  阅读 (2472)  评论 (0)  [编辑](https://i.cnblogs.com/EditArticles.aspx? postid=9039529)  [收藏](javascript: void(0))



[刷新评论](javascript: void(0);) [刷新页面](#) [返回顶部](#top)

### 导航

*   [博客园](https://www.cnblogs.com/)
*   [首页](https://www.cnblogs.com/flying1819/)
*   [新随笔](https://i.cnblogs.com/EditPosts.aspx? opt=1)

*   [订阅](javascript: void(0)) [! [订阅](/skins/anothereon001/images/xml.gif)](https://www.cnblogs.com/flying1819/rss/)
*   [管理](https://i.cnblogs.com/)

Powered by:
[博客园](https://www.cnblogs.com/)
Copyright © 2021 flyoss
Powered by .NET 5.0 on Kubernetes
