---
title: k8s-pod
date: 2021-01-30 17:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: k8s-pod
photo:
---

https://kubernetes.io/zh/docs/concepts/workloads/pods/

1.  [](https://kubernetes.io/zh/docs) Kubernetes 文档
2.  [](https://kubernetes.io/zh/docs/concepts) 概念
3.  [](https://kubernetes.io/zh/docs/concepts/workloads) 工作负载
4.  [](https://kubernetes.io/zh/docs/concepts/workloads/pods) Pods

# Pods

_Pod_ 是可以在 Kubernetes 中创建和管理的, 最小的可部署的计算单元.

_Pod_ (就像在鲸鱼荚或者豌豆荚中) 是一组 (一个或多个) [容器](/zh/docs/concepts/overview/what-is-kubernetes/#why-containers " 容器是可移植, 可执行的轻量级的镜像, 镜像中包含软件及其相关依赖."); 这些容器共享存储, 网络, 以及怎样运行这些容器的声明. Pod 中的内容总是并置 (colocated) 的并且一同调度, 在共享的上下文中运行. Pod 所建模的是特定于应用的 " 逻辑主机 ", 其中包含一个或多个应用容器, 这些容器是相对紧密的耦合在一起的. 在非云环境中, 在相同的物理机或虚拟机上运行的应用类似于 在同一逻辑主机上运行的云应用.

除了应用容器, Pod 还可以包含在 Pod 启动期间运行的 [](/zh/docs/concepts/workloads/pods/init-containers) Init 容器. 你也可以在集群中支持 [](/zh/docs/concepts/workloads/pods/ephemeral-containers) 临时性容器 的情况外, 为调试的目的注入临时性容器.

## 什么是 Pod?

> **说明:** 除了 Docker 之外, Kubernetes 支持 很多其他 [容器运行时](/zh/docs/setup/production-environment/container-runtimes " 容器运行时是负责运行容器的软件."), [](https://www.docker.com) Docker 是最有名的运行时, 使用 Docker 的术语来描述 Pod 会很有帮助.

Pod 的共享上下文包括一组 Linux 名字空间, 控制组 (cgroup) 和可能一些其他的隔离 方面, 即用来隔离 Docker 容器的技术. 在 Pod 的上下文中, 每个独立的应用可能会进一步实施隔离.

就 Docker 概念的术语而言, Pod 类似于共享名字空间和文件系统卷的一组 Docker 容器.

## 使用 Pod

通常你不需要直接创建 Pod, 甚至单实例 Pod. 相反, 你会使用诸如 [Deployment](/zh/docs/concepts/workloads/controllers/deployment/ "Deployment 是管理应用副本的 API 对象.") 或 [Job](/zh/docs/concepts/workloads/controllers/job/ "Job 是需要运行完成的确定性的或批量的任务.") 这类工作负载资源 来创建 Pod. 如果 Pod 需要跟踪状态, 可以考虑 [StatefulSet](/zh/docs/concepts/workloads/controllers/statefulset/ "StatefulSet 用来管理某 Pod 集合的部署和扩缩, 并为这些 Pod 提供持久存储和持久标识符.") 资源.

Kubernetes 集群中的 Pod 主要有两种用法:

*   **运行单个容器的 Pod**." 每个 Pod 一个容器 " 模型是最常见的 Kubernetes 用例; 在这种情况下, 可以将 Pod 看作单个容器的包装器, 并且 Kubernetes 直接管理 Pod, 而不是容器.

*   **运行多个协同工作的容器的 Pod**. Pod 可能封装由多个紧密耦合且需要共享资源的共处容器组成的应用程序. 这些位于同一位置的容器可能形成单个内聚的服务单元 —— 一个容器将文件从共享卷提供给公众, 而另一个单独的 " 挂斗 "(sidecar) 容器则刷新或更新这些文件. Pod 将这些容器和存储资源打包为一个可管理的实体.

    > **说明:** 将多个并置, 同管的容器组织到一个 Pod 中是一种相对高级的使用场景. 只有在一些场景中, 容器之间紧密关联时你才应该使用这种模式.


每个 Pod 都旨在运行给定应用程序的单个实例. 如果希望横向扩展应用程序 (例如, 运行多个实例 以提供更多的资源), 则应该使用多个 Pod, 每个实例使用一个 Pod. 在 Kubernetes 中, 这通常被称为 _副本 (Replication)_. 通常使用一种工作负载资源及其 [控制器](/zh/docs/concepts/architecture/controller/ " 控制器通过 apiserver 监控集群的公共状态, 并致力于将当前状态转变为期望的状态.") 来创建和管理一组 Pod 副本.

参见 [Pod 和控制器](#pods-and-controllers) 以了解 Kubernetes 如何使用工作负载资源及其控制器以实现应用的扩缩和自动修复.

### Pod 怎样管理多个容器

Pod 被设计成支持形成内聚服务单元的多个协作过程 (形式为容器). Pod 中的容器被自动安排到集群中的同一物理机或虚拟机上, 并可以一起进行调度. 容器之间可以共享资源和依赖, 彼此通信, 协调何时以及何种方式终止自身.

例如, 你可能有一个容器, 为共享卷中的文件提供 Web 服务器支持, 以及一个单独的 "sidecar(挂斗)" 容器负责从远端更新这些文件, 如下图所示:

! [example pod diagram](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg)

有些 Pod 具有 [Init 容器](/zh/docs/reference/glossary/? all= true#term-init-container " 应用容器运行前必须先运行完成的一个或多个初始化容器.") 和 [应用容器](/zh/docs/reference/glossary/? all= true#term-app-container " 用于运行部分工作负载的容器. 与初始化容器比较而言."). Init 容器会在启动应用容器之前运行并完成.

Pod 天生地为其成员容器提供了两种共享资源: [网络](#pod-networking) 和 [存储](#pod-storage).

## 使用 Pod

你很少在 Kubernetes 中直接创建一个个的 Pod, 甚至是单实例 (Singleton) 的 Pod. 这是因为 Pod 被设计成了相对临时性的, 用后即抛的一次性实体. 当 Pod 由你或者间接地由 [控制器](/zh/docs/concepts/architecture/controller/ " 控制器通过 apiserver 监控集群的公共状态, 并致力于将当前状态转变为期望的状态.") 创建时, 它被调度在集群中的 [节点](/zh/docs/concepts/architecture/nodes/ "Kubernetes 中的工作机器称作节点.") 上运行. Pod 会保持在该节点上运行, 直到 Pod 结束执行, Pod 对象被删除, Pod 因资源不足而被 _驱逐_ 或者节点失效为止.

> **说明:** 重启 Pod 中的容器不应与重启 Pod 混淆. Pod 不是进程, 而是容器运行的环境. 在被删除之前, Pod 会一直存在.

当你为 Pod 对象创建清单时, 要确保所指定的 Pod 名称是合法的 [DNS 子域名](/zh/docs/concepts/overview/working-with-objects/names#dns-subdomain-names).

### Pod 和控制器

你可以使用工作负载资源来创建和管理多个 Pod. 资源的控制器能够处理副本的管理, 上线, 并在 Pod 失效时提供自愈能力. 例如, 如果一个节点失败, 控制器注意到该节点上的 Pod 已经停止工作, 就可以创建替换性的 Pod. 调度器会将替身 Pod 调度到一个健康的节点执行.

下面是一些管理一个或者多个 Pod 的工作负载资源的示例:

*   [Deployment](/zh/docs/concepts/workloads/controllers/deployment/ "Deployment 是管理应用副本的 API 对象.")
*   [StatefulSet](/zh/docs/concepts/workloads/controllers/statefulset/ "StatefulSet 用来管理某 Pod 集合的部署和扩缩, 并为这些 Pod 提供持久存储和持久标识符.")
*   [DaemonSet](/zh/docs/concepts/workloads/controllers/daemonset/ " 确保 Pod 的副本在集群中的一组节点上运行.")

### Pod 模版

[负载](/zh/docs/concepts/workloads/ " 工作负载是在 Kubernetes 上运行的应用程序.") 资源的控制器通常使用 _Pod 模板 (Pod Template)_ 来替你创建 Pod 并管理它们.

Pod 模板是包含在工作负载对象中的规范, 用来创建 Pod. 这类负载资源包括 [](/zh/docs/concepts/workloads/controllers/deployment) Deployment, [](/zh/docs/concepts/workloads/controllers/job) Job 和 [](/zh/docs/concepts/workloads/controllers/daemonset) DaemonSets 等.

工作负载的控制器会使用负载对象中的 `PodTemplate` 来生成实际的 Pod. `PodTemplate` 是你用来运行应用时指定的负载资源的目标状态的一部分.

下面的示例是一个简单的 Job 的清单, 其中的 `template` 指示启动一个容器. 该 Pod 中的容器会打印一条消息之后暂停.

```
apiVersion: batch/v1 kind: Job metadata:  name: hello spec:  template:  # 这里是 Pod 模版  spec:  containers:  - name: hello  image: busybox  command: ['sh', '-c', 'echo "Hello, Kubernetes!" && sleep 3600']  restartPolicy: OnFailure  # 以上为 Pod 模版
```

修改 Pod 模版或者切换到新的 Pod 模版都不会对已经存在的 Pod 起作用. Pod 不会直接收到模版的更新. 相反, 新的 Pod 会被创建出来, 与更改后的 Pod 模版匹配.

例如, Deployment 控制器针对每个 Deployment 对象确保运行中的 Pod 与当前的 Pod 模版匹配. 如果模版被更新, 则 Deployment 必须删除现有的 Pod, 基于更新后的模版 创建新的 Pod. 每个工作负载资源都实现了自己的规则, 用来处理对 Pod 模版的更新.

在节点上, [kubelet](/docs/reference/generated/kubelet " 一个在集群中每个节点上运行的代理. 它保证容器都运行在 Pod 中.") 并不直接监测 或管理与 Pod 模版相关的细节或模版的更新, 这些细节都被抽象出来. 这种抽象和关注点分离简化了整个系统的语义, 并且使得用户可以在不改变现有代码的 前提下就能扩展集群的行为.

## Pod 更新与替换

正如前面章节所述, 当某工作负载的 Pod 模板被改变时, 控制器会基于更新的模板 创建新的 Pod 对象而不是对现有 Pod 执行更新或者修补操作.

Kubernetes 并不禁止你直接管理 Pod. 对运行中的 Pod 的某些字段执行就地更新操作 还是可能的. 不过, 类似 [`patch`](/docs/reference/generated/kubernetes-api/v1.20/#patch-pod-v1-core) 和 [`replace`](/docs/reference/generated/kubernetes-api/v1.20/#replace-pod-v1-core) 这类更新操作有一些限制:

*   Pod 的绝大多数元数据都是不可变的. 例如, 你不可以改变其 `namespace`,`name`, `uid` 或者 `creationTimestamp` 字段;`generation` 字段是比较特别的, 如果更新 该字段, 只能增加字段取值而不能减少.

*   如果 `metadata.deletionTimestamp` 已经被设置, 则不可以向 `metadata.finalizers` 列表中添加新的条目.

*   Pod 更新不可以改变除 `spec.containers[*].image`,`spec.initContainers[*].image`, `spec.activeDeadlineSeconds` 或 `spec.tolerations` 之外的字段. 对于 `spec.tolerations`, 你只被允许添加新的条目到其中.

*   在更新 `spec.activeDeadlineSeconds` 字段时, 以下两种更新操作是被允许的:

    1.  如果该字段尚未设置, 可以将其设置为一个正数;
    2.  如果该字段已经设置为一个正数, 可以将其设置为一个更小的, 非负的整数.

### 资源共享和通信

Pod 使它的成员容器间能够进行数据共享和通信.

### Pod 中的存储

一个 Pod 可以设置一组共享的存储 [卷](/zh/docs/concepts/storage/volumes/ " 包含可被 Pod 中容器访问的数据的目录."). Pod 中的所有容器都可以访问该共享卷, 从而允许这些容器共享数据. 卷还允许 Pod 中的持久数据保留下来, 即使其中的容器需要重新启动. 有关 Kubernetes 如何在 Pod 中实现共享存储并将其提供给 Pod 的更多信息, 请参考 [](/zh/docs/concepts/storage) 卷.

### Pod 联网

每个 Pod 都在每个地址族中获得一个唯一的 IP 地址. Pod 中的每个容器共享网络名字空间, 包括 IP 地址和网络端口. _Pod 内_ 的容器可以使用 `localhost` 互相通信. 当 Pod 中的容器与 _Pod 之外_ 的实体通信时, 它们必须协调如何使用共享的网络资源 (例如端口).

在同一个 Pod 内, 所有容器共享一个 IP 地址和端口空间, 并且可以通过 `localhost` 发现对方. 他们也能通过如 SystemV 信号量或 POSIX 共享内存这类标准的进程间通信方式互相通信. 不同 Pod 中的容器的 IP 地址互不相同, 没有 [](/zh/docs/concepts/policy/pod-security-policy) 特殊配置 就不能使用 IPC 进行通信. 如果某容器希望与运行于其他 Pod 中的容器通信, 可以通过 IP 联网的方式实现.

Pod 中的容器所看到的系统主机名与为 Pod 配置的 `name` 属性值相同. [](/zh/docs/concepts/cluster-administration/networking) 网络部分提供了更多有关此内容的信息.

## 容器的特权模式

Pod 中的任何容器都可以使用容器规约中的 [](/zh/docs/tasks/configure-pod-container/security-context) 安全性上下文中的 `privileged` 参数启用特权模式. 这对于想要使用使用操作系统管理权能 (Capabilities, 如操纵网络堆栈和访问设备) 的容器很有用. 容器内的进程几乎可以获得与容器外的进程相同的特权.

> **说明:** 你的 [容器运行时](/zh/docs/setup/production-environment/container-runtimes " 容器运行时是负责运行容器的软件.") 必须支持 特权容器的概念才能使用这一配置.

## 静态 Pod

_静态 Pod(Static Pod)_ 直接由特定节点上的 `kubelet` 守护进程管理, 不需要 [API 服务器](/zh/docs/reference/command-line-tools-reference/kube-apiserver/ " 提供 Kubernetes API 服务的控制面组件.") 看到它们. 尽管大多数 Pod 都是通过控制面 (例如, [Deployment](/zh/docs/concepts/workloads/controllers/deployment/ "Deployment 是管理应用副本的 API 对象.")) 来管理的, 对于静态 Pod 而言,`kubelet` 直接监控每个 Pod, 并在其失效时重启之.

静态 Pod 通常绑定到某个节点上的 [kubelet](/docs/reference/generated/kubelet " 一个在集群中每个节点上运行的代理. 它保证容器都运行在 Pod 中."). 其主要用途是运行自托管的控制面. 在自托管场景中, 使用 `kubelet` 来管理各个独立的 [控制面组件](/zh/docs/concepts/overview/components/#control-plane-components).

`kubelet` 自动尝试为每个静态 Pod 在 Kubernetes API 服务器上创建一个 [镜像 Pod](/zh/docs/reference/glossary/? all= true#term-mirror-pod "API 服务器中的一个对象, 用于跟踪 kubelet 上的静态 pod."). 这意味着在节点上运行的 Pod 在 API 服务器上是可见的, 但不可以通过 API 服务器来控制.

## 接下来

要了解为什么 Kubernetes 会在其他资源 (如 [StatefulSet](/zh/docs/concepts/workloads/controllers/statefulset/ "StatefulSet 用来管理某 Pod 集合的部署和扩缩, 并为这些 Pod 提供持久存储和持久标识符.") 或 [Deployment](/zh/docs/concepts/workloads/controllers/deployment/ "Deployment 是管理应用副本的 API 对象.")) 封装通用的 Pod API, 相关的背景信息可以在前人的研究中找到. 具体包括:

*   [Aurora](https://aurora.apache.org/documentation/latest/reference/configuration/#job-schema)
*   [Borg](https://research.google.com/pubs/pub43438.html)
*   [Marathon](https://mesosphere.github.io/marathon/docs/rest-api.html)
*   [](https://research.google/pubs/pub41684) Omega
*   [](https://engineering.fb.com/data-center-engineering/tupperware) Tupperware.
