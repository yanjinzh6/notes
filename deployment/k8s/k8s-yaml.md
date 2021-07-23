---
title: k8s-yaml
date: 2021-02-01 11:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: k8s-yaml
photo:
---

https://www.cnblogs.com/chanix/p/12744418.html

# [k8s kubernetes yml yaml 配置文件语法解析 以及 k8s kubernetes 软件目录配置文件说明](https://www.cnblogs.com/chanix/p/12744418.html)

k8s kubernetes yml yaml 配置文件语法解析 以及 k8s kubernetes 软件目录配置文件说明

k8s kubernetes yml yaml 配置文件语法解析
2019/03/01 陈信
参考:
[https://blog.csdn.net/phantom\_111/article/details/79427144](https://blog.csdn.net/phantom_111/article/details/79427144)
[https://blog.csdn.net/random\_w/article/details/80612881](https://blog.csdn.net/random_w/article/details/80612881)
[https://my.oschina.net/gibsonxue/blog/1840887](https://my.oschina.net/gibsonxue/blog/1840887)

YAML 基础
YAML 是专门用来写配置文件的语言, 非常简洁和强大, 使用比 json 更方便.
它实质上是一种通用的数据串行化格式. 后文会说明定义 YAML 文件创建 Pod 和创建 Deployment.

YAML 语法规则
大小写敏感
使用缩进表示层级关系
缩进时不允许使用 Tal 键, 只允许使用空格
缩进的空格数目不重要, 只要相同层级的元素左侧对齐即可
"#" 表示注释, 从这个字符一直到行尾, 都会被解析器忽略

Maps 和 Lists
在 Kubernetes 中, 只需要知道两种结构类型即可: Lists 和 Maps

使用 YAML 用于 K8s 的定义带来的好处包括:
便捷性: 不必添加大量的参数到命令行中执行命令
可维护性: YAML 文件可以通过源头控制, 跟踪每次操作
灵活性: YAML 可以创建比命令行更加复杂的结构

YAML Maps
Map 顾名思义指的是字典, 即一个 Key: Value 的键值对信息. 例如:

* * *

apiVersion: v1
kind: Pod
注:--- 为可选的分隔符 , 当需要在一个文件中定义多个结构的时候需要使用. 上述内容表示有两个键 apiVersion 和 kind, 分别对应的值为 v1 和 Pod.

## Maps 的 value 既能够对应字符串也能够对应一个 Maps. 例如:

apiVersion: v1
kind: Pod
metadata:
name: kube100-site
labels:
app: web
注: 上述的 YAML 文件中, metadata 这个 KEY 对应的值为一个 Maps, 而嵌套的 labels 这个 KEY 的值又是一个 Map. 实际使用中可视情况进行多层嵌套.

注意: 在 YAML 文件中绝对不要使用 tab 键

YAML Lists
List 即列表, 说白了就是数组, 例如:
args
\-beijing
\-shanghai
\-shenzhen
\-guangzhou
可以指定任何数量的项在列表中, 每个项的定义以破折号 (-) 开头, 并且与父元素之间存在缩进. 在 JSON 格式中, 表示如下:
{
"args": \["beijing", "shanghai", "shenzhen", "guangzhou"\]
}

## 当然 Lists 的子项也可以是 Maps, Maps 的子项也可以是 List, 例如:

apiVersion: v1
kind: Pod
metadata:
name: kube100-site
labels:
app: web
spec:
containers:
\- name: front-end
image: nginx
ports:
\- containerPort: 80
\- name: flaskapp-demo
image: jcdemo/flaskapp
ports:
\- containerPort: 5000

如上述文件所示, 定义一个 containers 的 List 对象, 每个子项都由 name, image, ports 组成, 每个 ports 都有一个 KEY 为 containerPort 的 Map 组成, 转成 JSON 格式文件:
{
"apiVersion": "v1",
"kind": "Pod",
"metadata": {
"name": "kube100-site",
"labels": {
"app": "web"
},
},
"spec": {
"containers": \[{
"name": "front-end",
"image": "nginx",
"ports": \[{
"containerPort": "80"
}\]
}, {
"name": "flaskapp-demo",
"image": "jcdemo/flaskapp",
"ports": \[{
"containerPort": "5000"
}\]
}\]
}
}

查看 api 版本与 yaml 参数语法
1.kubectl api-versions
kubectl api-versions 查看当前 k8s 支持哪些 api 版本

2.kubectl explain Deployment
kubectl explain Deployment.spec 查看用法 / 帮助手册 (支持多个子项目, 用 "." 表示)
kubectl explain Deployment.spec.template 略
默认情况下, kubectl explain 命令只会显示属性的一级数据, 我们可以使用 --recursive 参数来显示整个属性的数据:
kubectl explain deployment.spec --recursive

3.kubectl api-resources
如果你不太确定可以使用 kubectl explain 的资源名, 可以使用下面的命令来获取所有资源名称:
kubectl api-resources
该命令会线上资源名称的复数形式 (比如显示 deployments 而不是 deployment), 还会显示一个资源的简写 (比如 deploy), 不过不用担心, 我们可以用任意一个名称来结合 kubectl explain 命令使用的:
kubectl explain deployments.spec
或者
kubectl explain deployment.spec
或者
kubectl explain deploy.spec

## 使用 YAML 创建 Pod
创建 Pod
下面定义一个普通的 Pod 文件

apiVersion: v1 # apiVersion: 此处值是 v1, 这个版本号需要根据安装的 Kubernetes 版本和资源类型进行变化, 记住不是写死的.
kind: Pod # kind: 此处创建的是 Pod, 根据实际情况, 此处资源类型可以是 Deployment, Job, Ingress, Service 等.
metadata: # metadata: 包含 Pod 的一些 meta 信息, 比如名称, namespace, 标签等信息.
name: kube100-site
labels:
app: web
spec: # spec: 包括一些 container, storage, volume 以及其他 Kubernetes 需要的参数, 以及诸如是否在容器失败时重新启动容器的属性. 可在特定 Kubernetes API 找到完整的 Kubernetes Pod 的属性.
containers:
\- name: front-end
image: nginx
ports:
\- containerPort: 80
\- name: flaskapp-demo
image: jcdemo/flaskapp
ports:
\- containerPort: 5000

下面是一个典型的容器的定义:
…
spec:
containers:
\- name: front-end
image: nginx
ports:
\- containerPort: 80
…
上述例子只是一个简单的最小定义: 一个名字 (front-end), 基于 nginx 的镜像, 以及容器将会监听的指定端口号 (80).
除了上述的基本属性外, 还能够指定复杂的属性, 包括容器启动运行的命令, 使用的参数, 工作目录以及每次实例化是否拉取新的副本. 还可以指定更深入的信息, 例如容器的退出日志的位置. 容器可选的设置属性包括:
name, image, command, args, workingDir, ports, env, resource, volumeMounts, livenessProbe, readinessProbe, livecycle, terminationMessagePath, imagePullPolicy, securityContext, stdin, stdinOnce, tty

了解了 Pod 的定义后, 将上面创建 Pod 的 YAML 文件保存成 pod.yaml, 然后使用 Kubectl 创建 Pod:
$ kubectl create -f pod.yaml
pod "kube100-site" created
可以使用 Kubectl 命令查看 Pod 的状态
$ kubectl get pods
NAME READY STATUS RESTARTS AGE
kube100-site 2/2 Running 0 1m
注: Pod 创建过程中如果出现错误, 可以使用 kubectl describe 进行排查.

创建 Deployment
上述介绍了如何使用 YAML 文件创建 Pod 实例, 但是如果这个 Pod 出现了故障的话, 对应的服务也就挂掉了, 所以 Kubernetes 提供了一个 Deployment 的概念 , 目的是让 Kubernetes 去管理一组 Pod 的副本, 也就是副本集 , 这样就能够保证一定数量的副本一直可用, 不会因为某一个 Pod 挂掉导致整个服务挂掉.

## 下面一个完整的 Deployment 的 YAML 文件, 可以在 Kubernetes v1beta1 API 参考中找到完整的 Deployment 可指定的参数列表

apiVersion: extensions/v1beta1 # 注意这里 apiVersion 对应的值是 extensions/v1beta1, 同时也需要将 kind 的类型指定为 Deployment.
kind: Deployment
metadata: # metadata 指定一些 meta 信息, 包括名字或标签之类的.
name: kube100-site
spec: # spec 选项定义需要两个副本, 此处可以设置很多属性, 例如受此 Deployment 影响的 Pod 的选择器等
replicas: 2
template:
metadata:
labels:
app: web
spec: # spec 选项的 template 其实就是对 Pod 对象的定义
containers:
\- name: front-end
image: nginx
ports:
\- containerPort: 80
\- name: flaskapp-demo
image: jcdemo/flaskapp
ports:
\- containerPort: 5000

将上述的 YAML 文件保存为 deployment.yaml, 然后创建 Deployment:
$ kubectl create -f deployment.yaml
deployment "kube100-site" created

可以使用如下命令检查 Deployment 的列表:
$ kubectl get deployments
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
kube100-site 2 2 2 2 2m

yaml 格式的 pod 定义文件完整内容
apiVersion: v1 #必选, 版本号, 例如 v1
kind: Pod #必选, Pod
metadata: #必选, 元数据
name: string #必选, Pod 名称
namespace: string #必选, Pod 所属的命名空间
labels: #自定义标签
\- name: string #自定义标签名字
annotations: #自定义注释列表
\- name: string
spec: #必选, Pod 中容器的详细定义
containers: #必选, Pod 中容器列表

*   name: string #必选, 容器名称
    image: string #必选, 容器的镜像名称
    imagePullPolicy: \[Always | Never | IfNotPresent\] #获取镜像的策略 Alawys 表示下载镜像 IfnotPresent 表示优先使用本地镜像, 否则下载镜像, Nerver 表示仅使用本地镜像
    command: \[string\] #容器的启动命令列表, 如不指定, 使用打包时使用的启动命令
    args: \[string\] #容器的启动命令参数列表
    workingDir: string #容器的工作目录
    volumeMounts: #挂载到容器内部的存储卷配置
    *   name: string #引用 pod 定义的共享存储卷的名称, 需用 volumes\[\] 部分定义的的卷名
        mountPath: string #存储卷在容器内 mount 的绝对路径, 应少于 512 字符
        readOnly: boolean #是否为只读模式
        ports: #需要暴露的端口库号列表
    *   name: string #端口号名称
        containerPort: int #容器需要监听的端口号
        hostPort: int #容器所在主机需要监听的端口号, 默认与 Container 相同
        protocol: string #端口协议, 支持 TCP 和 UDP, 默认 TCP
        env: #容器运行前需设置的环境变量列表
    *   name: string #环境变量名称
        value: string #环境变量的值
        resources: #资源限制和请求的设置
        limits: #资源限制的设置
        cpu: string #Cpu 的限制, 单位为 core 数, 将用于 docker run --cpu-shares 参数
        memory: string #内存限制, 单位可以为 Mib/Gib, 将用于 docker run --memory 参数
        requests: #资源请求的设置
        cpu: string #Cpu 请求, 容器启动的初始可用数量
        memory: string #内存清楚, 容器启动的初始可用数量
        livenessProbe: #对 Pod 内各个容器健康检查的设置, 当探测无响应几次后将自动重启该容器, 检查方法有 exec, httpGet 和 tcpSocket, 对一个容器只需设置其中一种方法即可
        exec: #对 Pod 容器内检查方式设置为 exec 方式
        command: \[string\] #exec 方式需要制定的命令或脚本
        httpGet: #对 Pod 内个容器健康检查方法设置为 HttpGet, 需要制定 Path, port
        path: string
        port: number
        host: string
        scheme: string
        HttpHeaders:
        *   name: string
            value: string
            tcpSocket: #对 Pod 内个容器健康检查方式设置为 tcpSocket 方式
            port: number
            initialDelaySeconds: 0 #容器启动完成后首次探测的时间, 单位为秒
            timeoutSeconds: 0 #对容器健康检查探测等待响应的超时时间, 单位秒, 默认 1 秒
            periodSeconds: 0 #对容器监控检查的定期探测时间设置, 单位秒, 默认 10 秒一次
            successThreshold: 0
            failureThreshold: 0
            securityContext:
            privileged: false
            restartPolicy: \[Always | Never | OnFailure\]#Pod 的重启策略, Always 表示一旦不管以何种方式终止运行, kubelet 都将重启, OnFailure 表示只有 Pod 以非 0 退出码退出才重启, Nerver 表示不再重启该 Pod
            nodeSelector: obeject #设置 NodeSelector 表示将该 Pod 调度到包含这个 label 的 node 上, 以 key: value 的格式指定
            imagePullSecrets: #Pull 镜像时使用的 secret 名称, 以 key: secretkey 格式指定
    *   name: string
        hostNetwork: false #是否使用主机网络模式, 默认为 false, 如果设置为 true, 表示使用宿主机网络
        volumes: #在该 pod 上定义共享存储卷列表
    *   name: string #共享存储卷名称 (volumes 类型有很多种)
        emptyDir: {} #类型为 emtyDir 的存储卷, 与 Pod 同生命周期的一个临时目录. 为空值
        hostPath: string #类型为 hostPath 的存储卷, 表示挂载 Pod 所在宿主机的目录
        path: string #Pod 所在宿主机的目录, 将被用于同期中 mount 的目录
        secret: #类型为 secret 的存储卷, 挂载集群与定义的 secre 对象到容器内部
        scretname: string
        items:
        *   key: string
            path: string
            configMap: #类型为 configMap 的存储卷, 挂载预定义的 configMap 对象到容器内部
            name: string
            items:
        *   key: string
            path: string

Deployment 部署文件详解
apiVersion: extensions/v1beta1 #接口版本
kind: Deployment #接口类型
metadata:
name: cango-demo #Deployment 名称
namespace: cango-prd #命名空间
labels:
app: cango-demo #标签
spec:
replicas: 3
strategy:
rollingUpdate: ##由于 replicas 为 3, 则整个升级, pod 个数在 2-4 个之间
maxSurge: 1 #滚动升级时会先启动 1 个 pod
maxUnavailable: 1 #滚动升级时允许的最大 Unavailable 的 pod 个数
template:
metadata:
labels:
app: cango-demo #模板名称必填
sepc: #定义容器模板, 该模板可以包含多个容器
containers:
\- name: cango-demo #镜像名称
image: swr.cn-east-2.myhuaweicloud.com/cango-prd/cango-demo:0.0.1-SNAPSHOT #镜像地址
command: \[ "/bin/sh","-c","cat /etc/config/path/to/special-key" \] #启动命令
args: #启动参数
\- '-storage.local.retention=$(STORAGE\_RETENTION)'
\- '-storage.local.memory-chunks=$(STORAGE\_MEMORY\_CHUNKS)'
\- '-config.file=/etc/prometheus/prometheus.yml'
\- '-alertmanager.url= http://alertmanager:9093/alertmanager'
\- '-web.external-url=$(EXTERNAL\_URL)'
#如果 command 和 args 均没有写, 那么用 Docker 默认的配置.
#如果 command 写了, 但 args 没有写, 那么 Docker 默认的配置会被忽略而且仅仅执行.yaml 文件的 command(不带任何参数的).
#如果 command 没写, 但 args 写了, 那么 Docker 默认配置的 ENTRYPOINT 的命令行会被执行, 但是调用的参数是.yaml 中的 args.
#如果如果 command 和 args 都写了, 那么 Docker 默认的配置被忽略, 使用.yaml 的配置.
imagePullPolicy: IfNotPresent #如果不存在则拉取
livenessProbe: #表示 container 是否处于 live 状态. 如果 LivenessProbe 失败, LivenessProbe 将会通知 kubelet 对应的 container 不健康了. 随后 kubelet 将 kill 掉 container, 并根据 RestarPolicy 进行进一步的操作. 默认情况下 LivenessProbe 在第一次检测之前初始化值为 Success, 如果 container 没有提供 LivenessProbe, 则也认为是 Success;
httpGet:
path: /health #如果没有心跳检测接口就为 /
port: 8080
scheme: HTTP
initialDelaySeconds: 60 ##启动后延时多久开始运行检测
timeoutSeconds: 5
successThreshold: 1
failureThreshold: 5
readinessProbe:
readinessProbe:
httpGet:
path: /health #如果没有心跳检测接口就为 /
port: 8080
scheme: HTTP
initialDelaySeconds: 30 ##启动后延时多久开始运行检测
timeoutSeconds: 5
successThreshold: 1
failureThreshold: 5
resources: ##CPU 内存限制
requests:
cpu: 2
memory: 2048Mi
limits:
cpu: 2
memory: 2048Mi
env: ##通过环境变量的方式, 直接传递 pod= 自定义 Linux OS 环境变量
\- name: LOCAL\_KEY #本地 Key
value: value
\- name: CONFIG\_MAP\_KEY #局策略可使用 configMap 的配置 Key,
valueFrom:
configMapKeyRef:
name: special-config #configmap 中找到 name 为 special-config
key: special.type #找到 name 为 special-config 里 data 下的 key
ports:
\- name: http
containerPort: 8080 #对 service 暴露端口
volumeMounts: #挂载 volumes 中定义的磁盘
\- name: log-cache
mount: /tmp/log
\- name: sdb #普通用法, 该卷跟随容器销毁, 挂载一个目录
mountPath: /data/media
\- name: nfs-client-root #直接挂载硬盘方法, 如挂载下面的 nfs 目录到 /mnt/nfs
mountPath: /mnt/nfs
\- name: example-volume-config #高级用法第 1 种, 将 ConfigMap 的 log-script, backup-script 分别挂载到 /etc/config 目录下的一个相对路径 path/to/... 下, 如果存在同名文件, 直接覆盖.
mountPath: /etc/config
\- name: rbd-pvc #高级用法第 2 中, 挂载 PVC(PresistentVolumeClaim)

# 使用 volume 将 ConfigMap 作为文件或目录直接挂载, 其中每一个 key-value 键值对都会生成一个文件, key 为文件名, value 为内容,

volumes: # 定义磁盘给上面 volumeMounts 挂载

*   name: log-cache
    emptyDir: {}
*   name: sdb #挂载宿主机上面的目录
    hostPath:
    path: /any/path/it/will/be/replaced
*   name: example-volume-config # 供 ConfigMap 文件内容到指定路径使用
    configMap:
    name: example-volume-config #ConfigMap 中名称
    items:
    *   key: log-script #ConfigMap 中的 Key
        path: path/to/log-script #指定目录下的一个相对路径 path/to/log-script
    *   key: backup-script #ConfigMap 中的 Key
        path: path/to/backup-script #指定目录下的一个相对路径 path/to/backup-script
*   name: nfs-client-root #供挂载 NFS 存储类型
    nfs:
    server: 10.42.0.55 #NFS 服务器地址
    path: /opt/public #showmount -e 看一下路径
*   name: rbd-pvc #挂载 PVC 磁盘
    persistentVolumeClaim:
    claimName: rbd-pvc1 #挂载已经申请的 pvc 磁盘

k8s kubernetes 软件目录配置文件说明
2019/03/01 陈信

参考:
[https://my.oschina.net/u/2306127/blog/2980162](https://my.oschina.net/u/2306127/blog/2980162) Kubernetes 探秘—配置文件目录结构
[https://k8smeetup.github.io/docs/tasks/administer-cluster/kubelet-config-file/](https://k8smeetup.github.io/docs/tasks/administer-cluster/kubelet-config-file/)

Kubernetes 的配置目录包括
/etc/kubernetes/ 主要配置目录
/home/user/.kube/
/var/lib/kubelet/

主要配置目录 /etc/kubernetes/

master 上
\[root@ip-10-0-0-105 kubernetes\]# tree /etc/kubernetes/
├── admin.conf # 注意 conf 与 yaml 文件区别
├── controller-manager.conf
├── kubelet.conf
├── manifests
│ ├── etcd.yaml
│ ├── kube-apiserver.yaml
│ ├── kube-controller-manager.yaml
│ └── kube-scheduler.yaml
├── pki # 证书, 密钥
│ ├── apiserver.crt
│ ├── apiserver-etcd-client.crt
│ ├── apiserver-etcd-client.key
│ ├── apiserver.key
│ ├── apiserver-kubelet-client.crt
│ ├── apiserver-kubelet-client.key
│ ├── ca.crt
│ ├── ca.key
│ ├── etcd
│ │ ├── ca.crt
│ │ ├── ca.key
│ │ ├── healthcheck-client.crt
│ │ ├── healthcheck-client.key
│ │ ├── peer.crt
│ │ ├── peer.key
│ │ ├── server.crt
│ │ └── server.key
│ ├── front-proxy-ca.crt
│ ├── front-proxy-ca.key
│ ├── front-proxy-client.crt
│ ├── front-proxy-client.key
│ ├── sa.key
│ └── sa.pub
└── scheduler.conf

node 上
\[root@ip-10-0-0-106 ~\]# tree /etc/kubernetes/
/etc/kubernetes/
├── bootstrap-kubelet.conf
├── kubelet.conf
├── manifests
└── pki
└── ca.crt

kubernetes 用户配置目录 /home/centos/.kube/

master 上
\[centos@ip-10-0-0-105 ~\]$ tree /home/centos/.kube/
/home/centos/.kube/
├── cache
│ └── discovery
│ └── 10.0.0.105\_6443
│ ├── admissionregistration.k8s.io
│ │ └── v1beta1
│ │ └── serverresources.json
│ ├── apiextensions.k8s.io
│ │ └── v1beta1
│ │ └── serverresources.json
...
│ └── v1
│ └── serverresources.json
├── config
└── http-cache
├── 01bc06f73be10e8d87d9ee6dfd8b2217
├── 02d3387dd6e4aba4f63656f4c1984b78
...
└── fc4785aa1232654e2cb5c3415168eee3

其他 node 上: 无

kubelet 服务的配置目录 /var/lib/kubelet
每一个 ks 节点都需要运行 kubelet 服务.kubelet 服务的配置在 /var/lib/kubelet 目录下

mater 上
\[root@ip-10-0-0-105 kubelet\]# tree
.
├── config.yaml
├── cpu\_manager\_state
├── device-plugins
│ ├── DEPRECATION
│ ├── kubelet\_internal\_checkpoint
│ └── kubelet.sock
├── kubeadm-flags.env
├── pki
│ ├── kubelet-client-2019-03-01-11-25-53.pem
│ ├── kubelet-client-2019-03-01-11-26-21.pem
│ ├── kubelet-client-current.pem -> /var/lib/kubelet/pki/kubelet-client-2019-03-01-11-26-21.pem
│ ├── kubelet.crt
│ └── kubelet.key
├── plugin-containers
├── plugins
├── plugins\_registry
├── pod-resources
└── pods
├── 2c1838fa-3bd2-11e9-8350-068a71fa5690
│ ├── containers
│ │ ├── install-cni
│ │ │ ├── 0b4b4e10
│ │ │ ├── 19c66877
│ │ │ ├── 35c27ecf
│ │ │ ├── d467aa8a
│ │ │ └── d4b3d4ca
│ │ └── kube-flannel
│ │ ├── 14f96430
│ │ ├── 2532cbe0
│ │ ├── 4f31f687
│ │ ├── 52a0d626
│ │ └── 6a2fea18
│ ├── etc-hosts
│ ├── plugins
│ │ └── kubernetes.io~empty-dir
│ │ ├── wrapped\_flannel-cfg
│ │ │ └── ready
│ │ └── wrapped\_flannel-token-fn5rf
│ │ └── ready
│ └── volumes
│ ├── kubernetes.io~configmap
│ │ └── flannel-cfg
│ │ ├── cni-conf.json -> ..data/cni-conf.json
│ │ └── net-conf.json -> ..data/net-conf.json
│ └── kubernetes.io~secret
│ └── flannel-token-fn5rf
│ ├── ca.crt -> ..data/ca.crt
│ ├── namespace -> ..data/namespace
│ └── token -> ..data/token
├── 456ea23a46724821ed560a2db7e2d9f3
│ ├── containers
│ │ └── kube-apiserver
│ │ ├── 02f8d87a
│ │ ├── 2d988dc1
│ │ ├── 3302d612
│ │ ├── 46eb2273
│ │ └── 4ff2f287
│ ├── etc-hosts
│ ├── plugins
│ └── volumes
├── 4b52d75cab61380f07c0c5a69fb371d4
│ ├── containers
│ │ └── kube-scheduler
│ │ ├── 12e55ee8
│ │ ├── 20b64878
│ │ ├── 95b02faf
│ │ ├── 9c4b578d
│ │ └── d0764370
│ ├── etc-hosts
│ ├── plugins
│ └── volumes
├── 67337167e3ecf30096ecaf8c003e925a
│ ├── containers
│ │ └── etcd
│ │ ├── 0c97bb0d
│ │ ├── 3df93354
│ │ ├── 884eaf54
│ │ ├── b96335a0
│ │ └── cf847820
│ ├── etc-hosts
│ ├── plugins
│ └── volumes
├── c8a3ca19-3bd1-11e9-8350-068a71fa5690
│ ├── containers
│ │ └── coredns
│ │ ├── 09130d4f
│ │ ├── 4dac82a1
│ │ ├── 601eb4e4
│ │ ├── e3fc3048
│ │ └── fd799c7a
│ ├── etc-hosts
│ ├── plugins
│ │ └── kubernetes.io~empty-dir
│ │ ├── wrapped\_config-volume
│ │ │ └── ready
│ │ └── wrapped\_coredns-token-l59np
│ │ └── ready
│ └── volumes
│ ├── kubernetes.io~configmap
│ │ └── config-volume
│ │ └── Corefile -> ..data/Corefile
│ └── kubernetes.io~secret
│ └── coredns-token-l59np
│ ├── ca.crt -> ..data/ca.crt
│ ├── namespace -> ..data/namespace
│ └── token -> ..data/token
├── c8a5b8e6-3bd1-11e9-8350-068a71fa5690
│ ├── containers
│ │ └── coredns
│ │ ├── 3e77b789
│ │ ├── 607e8b98
│ │ ├── 68e8f6b8
│ │ ├── 7b0ecb21
│ │ └── b8ea8e88
│ ├── etc-hosts
│ ├── plugins
│ │ └── kubernetes.io~empty-dir
│ │ ├── wrapped\_config-volume
│ │ │ └── ready
│ │ └── wrapped\_coredns-token-l59np
│ │ └── ready
│ └── volumes
│ ├── kubernetes.io~configmap
│ │ └── config-volume
│ │ └── Corefile -> ..data/Corefile
│ └── kubernetes.io~secret
│ └── coredns-token-l59np
│ ├── ca.crt -> ..data/ca.crt
│ ├── namespace -> ..data/namespace
│ └── token -> ..data/token
├── c8a68d97-3bd1-11e9-8350-068a71fa5690
│ ├── containers
│ │ └── kube-proxy
│ │ ├── 1225589c
│ │ ├── 211e4539
│ │ ├── 27aca4dc
│ │ ├── 3b0a273a
│ │ └── b4305109
│ ├── etc-hosts
│ ├── plugins
│ │ └── kubernetes.io~empty-dir
│ │ ├── wrapped\_kube-proxy
│ │ │ └── ready
│ │ └── wrapped\_kube-proxy-token-55dm7
│ │ └── ready
│ └── volumes
│ ├── kubernetes.io~configmap
│ │ └── kube-proxy
│ │ ├── config.conf -> ..data/config.conf
│ │ └── kubeconfig.conf -> ..data/kubeconfig.conf
│ └── kubernetes.io~secret
│ └── kube-proxy-token-55dm7
│ ├── ca.crt -> ..data/ca.crt
│ ├── namespace -> ..data/namespace
│ └── token -> ..data/token
└── e0ebd09707c2bc7047d2353e7c72c457
├── containers
│ └── kube-controller-manager
│ ├── 23ee9aed
│ ├── 317117d8
│ ├── 5757d4f9
│ ├── a691a160
│ └── f871dc0c
├── etc-hosts
├── plugins
└── volumes

node 上
\[root@ip-10-0-0-106 kubelet\]# tree
.
├── config.yaml
├── cpu\_manager\_state
├── device-plugins
│ ├── DEPRECATION
│ ├── kubelet\_internal\_checkpoint
│ └── kubelet.sock
├── kubeadm-flags.env
├── pki
│ ├── kubelet-client-2019-03-01-11-32-45.pem
│ ├── kubelet-client-current.pem -> /var/lib/kubelet/pki/kubelet-client-2019-03-01-11-32-45.pem
│ ├── kubelet.crt
│ └── kubelet.key
├── plugin-containers
├── plugins
├── plugins\_registry
├── pod-resources
└── pods
├── ae1d9bae-3bd2-11e9-8350-068a71fa5690
│ ├── containers
│ │ ├── install-cni
│ │ │ └── 0db99f82
│ │ └── kube-flannel
│ │ └── 74e75121
│ ├── etc-hosts
│ ├── plugins
│ │ └── kubernetes.io~empty-dir
│ │ ├── wrapped\_flannel-cfg
│ │ │ └── ready
│ │ └── wrapped\_flannel-token-fn5rf
│ │ └── ready
│ └── volumes
│ ├── kubernetes.io~configmap
│ │ └── flannel-cfg
│ │ ├── cni-conf.json -> ..data/cni-conf.json
│ │ └── net-conf.json -> ..data/net-conf.json
│ └── kubernetes.io~secret
│ └── flannel-token-fn5rf
│ ├── ca.crt -> ..data/ca.crt
│ ├── namespace -> ..data/namespace
│ └── token -> ..data/token
└── ae1dd11c-3bd2-11e9-8350-068a71fa5690
├── containers
│ └── kube-proxy
│ └── ad15067b
├── etc-hosts
├── plugins
│ └── kubernetes.io~empty-dir
│ ├── wrapped\_kube-proxy
│ │ └── ready
│ └── wrapped\_kube-proxy-token-55dm7
│ └── ready
└── volumes
├── kubernetes.io~configmap
│ └── kube-proxy
│ ├── config.conf -> ..data/config.conf
│ └── kubeconfig.conf -> ..data/kubeconfig.conf
└── kubernetes.io~secret
└── kube-proxy-token-55dm7
├── ca.crt -> ..data/ca.crt
├── namespace -> ..data/namespace
└── token -> ..data/token

可以看到启动参数放到了 /var/lib/kubelet .

kubelet.service 服务目录 /etc/systemd/system/kubelet.service
kubelet 使用 systemd 管理, service 定义文件位于:
/etc/systemd/system/multi-user.target.wants/kubelet.service -> /etc/systemd/system/kubelet.service

最新的 dropin 文件位于: /etc/systemd/system/kubelet.service.d
ls /etc/systemd/system/kubelet.service.d/
10-kubeadm.conf

posted @ 2020-04-21 14:29  [ChanixChen](https://www.cnblogs.com/chanix/)  阅读 (1336)  评论 (0)  [编辑](https://i.cnblogs.com/EditPosts.aspx? postid=12744418)  [收藏](javascript: void(0))



[刷新评论](javascript: void(0);) [刷新页面](#) [返回顶部](#top)