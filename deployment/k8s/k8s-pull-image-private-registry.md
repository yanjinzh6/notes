---
title: k8s-pull-image-private-registry
date: 2021-01-31 10:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: k8s-pull-image-private-registry
photo:
---

https://kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/

从私有仓库拉取镜像 | Kubernetes 从私有仓库拉取镜像 | Kubernetes

[](/zh/)

*   [文档](/zh/docs/)
*   [Kubernetes 博客](/zh/blog/)
*   [培训](/zh/training/)
*   [合作伙伴](/zh/partners/)
*   [社区](/zh/community/)
*   [案例分析](/zh/case-studies/)
*   [版本列表](#)

    [v1.20](https://kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/) [v1.19](https://v1-19.docs.kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/) [v1.18](https://v1-18.docs.kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/) [v1.17](https://v1-17.docs.kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/) [v1.16](https://v1-16.docs.kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/)

*   [中文 Chinese](#)

    [English](/docs/tasks/configure-pod-container/pull-image-private-registry/) [한국어 Korean](/ko/docs/tasks/configure-pod-container/pull-image-private-registry/) [Français](/fr/docs/tasks/configure-pod-container/pull-image-private-registry/) [Bahasa Indonesia](/id/docs/tasks/configure-pod-container/pull-image-private-registry/)


*   *   [主页](/zh/docs/home/)

    *   [Kubernetes 文档支持的版本](/zh/docs/home/supported-doc-versions/)

    *   [入门](/zh/docs/setup/)

    *   *   [Kubernetes 发行说明和版本偏差](/zh/docs/setup/release/)

        *   [v1.18 发布说明](/zh/docs/setup/release/notes/) [Kubernetes 版本及版本偏差支持策略](/zh/docs/setup/release/version-skew-policy/)

        *   [学习环境](/zh/docs/setup/learning-environment/)

        *   [生产环境](/zh/docs/setup/production-environment/)

        *   [容器运行时](/zh/docs/setup/production-environment/container-runtimes/) [Turnkey 云解决方案](/zh/docs/setup/production-environment/turnkey-solutions/)

            *   [使用部署工具安装 Kubernetes](/zh/docs/setup/production-environment/tools/)

            *   *   [使用 kubeadm 引导集群](/zh/docs/setup/production-environment/tools/kubeadm/)

                *   [安装 kubeadm](/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/) [对 kubeadm 进行故障排查](/zh/docs/setup/production-environment/tools/kubeadm/troubleshooting-kubeadm/) [使用 kubeadm 创建集群](/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/) [使用 kubeadm 定制控制平面配置](/zh/docs/setup/production-environment/tools/kubeadm/control-plane-flags/) [高可用拓扑选项](/zh/docs/setup/production-environment/tools/kubeadm/ha-topology/) [利用 kubeadm 创建高可用集群](/zh/docs/setup/production-environment/tools/kubeadm/high-availability/) [使用 kubeadm 创建一个高可用 etcd 集群](/zh/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm/) [使用 kubeadm 配置集群中的每个 kubelet](/zh/docs/setup/production-environment/tools/kubeadm/kubelet-integration/) [配置您的 kubernetes 集群以自托管控制平台](/zh/docs/setup/production-environment/tools/kubeadm/self-hosting/)

                [使用 Kops 安装 Kubernetes](/zh/docs/setup/production-environment/tools/kops/) [使用 Kubespray 安装 Kubernetes](/zh/docs/setup/production-environment/tools/kubespray/)

            *   [Windows Kubernetes](/zh/docs/setup/production-environment/windows/)

            *   [Kubernetes 对 Windows 的支持](/zh/docs/setup/production-environment/windows/intro-windows-in-kubernetes/) [Kubernetes 中调度 Windows 容器的指南](/zh/docs/setup/production-environment/windows/user-guide-windows-containers/)


        *   [最佳实践](/zh/docs/setup/best-practices/)

        *   [运行于多区环境](/zh/docs/setup/best-practices/multiple-zones/) [创建大型集群](/zh/docs/setup/best-practices/cluster-large/) [校验节点设置](/zh/docs/setup/best-practices/node-conformance/) [PKI 证书和要求](/zh/docs/setup/best-practices/certificates/)


    *   [概念](/zh/docs/concepts/)

    *   *   [概述](/zh/docs/concepts/overview/)

        *   [Kubernetes 是什么?](/zh/docs/concepts/overview/what-is-kubernetes/) [Kubernetes 组件](/zh/docs/concepts/overview/components/) [Kubernetes API](/zh/docs/concepts/overview/kubernetes-api/)
            *   [使用 Kubernetes 对象](/zh/docs/concepts/overview/working-with-objects/)

            *   [理解 Kubernetes 对象](/zh/docs/concepts/overview/working-with-objects/kubernetes-objects/) [Kubernetes 对象管理](/zh/docs/concepts/overview/working-with-objects/object-management/) [对象名称和 IDs](/zh/docs/concepts/overview/working-with-objects/names/) [名字空间](/zh/docs/concepts/overview/working-with-objects/namespaces/) [标签和选择算符](/zh/docs/concepts/overview/working-with-objects/labels/) [注解](/zh/docs/concepts/overview/working-with-objects/annotations/) [字段选择器](/zh/docs/concepts/overview/working-with-objects/field-selectors/) [推荐使用的标签](/zh/docs/concepts/overview/working-with-objects/common-labels/)


        *   [Kubernetes 架构](/zh/docs/concepts/architecture/)

        *   [节点](/zh/docs/concepts/architecture/nodes/) [控制面到节点通信](/zh/docs/concepts/architecture/control-plane-node-communication/) [控制器](/zh/docs/concepts/architecture/controller/) [云控制器管理器的基础概念](/zh/docs/concepts/architecture/cloud-controller/)

        *   [容器](/zh/docs/concepts/containers/)

        *   [镜像](/zh/docs/concepts/containers/images/) [容器环境](/zh/docs/concepts/containers/container-environment/) [容器运行时类 (Runtime Class)](/zh/docs/concepts/containers/runtime-class/) [容器生命周期回调](/zh/docs/concepts/containers/container-lifecycle-hooks/)

        *   [工作负载](/zh/docs/concepts/workloads/)

        *   *   [Pods](/zh/docs/concepts/workloads/pods/)

            *   [Pod 的生命周期](/zh/docs/concepts/workloads/pods/pod-lifecycle/) [Init 容器](/zh/docs/concepts/workloads/pods/init-containers/) [Pod 拓扑分布约束](/zh/docs/concepts/workloads/pods/pod-topology-spread-constraints/) [干扰 (Disruptions)](/zh/docs/concepts/workloads/pods/disruptions/) [临时容器](/zh/docs/concepts/workloads/pods/ephemeral-containers/)

            *   [工作负载资源](/zh/docs/concepts/workloads/controllers/)

            *   [Deployments](/zh/docs/concepts/workloads/controllers/deployment/) [ReplicaSet](/zh/docs/concepts/workloads/controllers/replicaset/) [StatefulSets](/zh/docs/concepts/workloads/controllers/statefulset/) [DaemonSet](/zh/docs/concepts/workloads/controllers/daemonset/) [Jobs](/zh/docs/concepts/workloads/controllers/job/) [垃圾收集](/zh/docs/concepts/workloads/controllers/garbage-collection/) [已完成资源的 TTL 控制器](/zh/docs/concepts/workloads/controllers/ttlafterfinished/) [CronJob](/zh/docs/concepts/workloads/controllers/cron-jobs/) [ReplicationController](/zh/docs/concepts/workloads/controllers/replicationcontroller/)


        *   [服务, 负载均衡和联网](/zh/docs/concepts/services-networking/)

        *   [服务](/zh/docs/concepts/services-networking/service/) [服务拓扑 (Service Topology)](/zh/docs/concepts/services-networking/service-topology/) [Pod 与 Service 的 DNS](/zh/docs/concepts/services-networking/dns-pod-service/) [使用 Service 连接到应用](/zh/docs/concepts/services-networking/connect-applications-service/) [端点切片 (Endpoint Slices)](/zh/docs/concepts/services-networking/endpoint-slices/) [Ingress](/zh/docs/concepts/services-networking/ingress/) [Ingress 控制器](/zh/docs/concepts/services-networking/ingress-controllers/) [网络策略](/zh/docs/concepts/services-networking/network-policies/) [使用 HostAliases 向 Pod /etc/hosts 文件添加条目](/zh/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/) [IPv4/IPv6 双协议栈](/zh/docs/concepts/services-networking/dual-stack/)

        *   [存储](/zh/docs/concepts/storage/)

        *   [卷](/zh/docs/concepts/storage/volumes/) [卷快照](/zh/docs/concepts/storage/volume-snapshots/) [持久卷](/zh/docs/concepts/storage/persistent-volumes/) [CSI 卷克隆](/zh/docs/concepts/storage/volume-pvc-datasource/) [卷快照类](/zh/docs/concepts/storage/volume-snapshot-classes/) [存储类](/zh/docs/concepts/storage/storage-classes/) [动态卷供应](/zh/docs/concepts/storage/dynamic-provisioning/) [存储容量](/zh/docs/concepts/storage/storage-capacity/) [临时卷](/zh/docs/concepts/storage/ephemeral-volumes/) [特定于节点的卷数限制](/zh/docs/concepts/storage/storage-limits/)

        *   [配置](/zh/docs/concepts/configuration/)

        *   [配置最佳实践](/zh/docs/concepts/configuration/overview/) [ConfigMap](/zh/docs/concepts/configuration/configmap/) [Secret](/zh/docs/concepts/configuration/secret/) [为容器管理资源](/zh/docs/concepts/configuration/manage-resources-containers/) [使用 kubeconfig 文件组织集群访问](/zh/docs/concepts/configuration/organize-cluster-access-kubeconfig/) [Pod 优先级与抢占](/zh/docs/concepts/configuration/pod-priority-preemption/)

        *   [安全](/zh/docs/concepts/security/)

        *   [Pod 安全性标准](/zh/docs/concepts/security/pod-security-standards/) [云原生安全概述](/zh/docs/concepts/security/overview/) [Kubernetes API 访问控制](/zh/docs/concepts/security/controlling-access/)

        *   [策略](/zh/docs/concepts/policy/)

        *   [限制范围](/zh/docs/concepts/policy/limit-range/) [资源配额](/zh/docs/concepts/policy/resource-quotas/) [Pod 安全策略](/zh/docs/concepts/policy/pod-security-policy/) [进程 ID 约束与预留](/zh/docs/concepts/policy/pid-limiting/)

        *   [调度和驱逐 (Scheduling and Eviction)](/zh/docs/concepts/scheduling-eviction/)

        *   [Pod 开销](/zh/docs/concepts/scheduling-eviction/pod-overhead/) [污点和容忍度](/zh/docs/concepts/scheduling-eviction/taint-and-toleration/) [Kubernetes 调度器](/zh/docs/concepts/scheduling-eviction/kube-scheduler/) [将 Pod 分配给节点](/zh/docs/concepts/scheduling-eviction/assign-pod-node/) [扩展资源的资源装箱](/zh/docs/concepts/scheduling-eviction/resource-bin-packing/) [驱逐策略](/zh/docs/concepts/scheduling-eviction/eviction-policy/) [调度框架](/zh/docs/concepts/scheduling-eviction/scheduling-framework/) [调度器性能调优](/zh/docs/concepts/scheduling-eviction/scheduler-perf-tuning/)

        *   [集群管理](/zh/docs/concepts/cluster-administration/)

        *   [证书](/zh/docs/concepts/cluster-administration/certificates/) [管理资源](/zh/docs/concepts/cluster-administration/manage-deployment/) [集群网络系统](/zh/docs/concepts/cluster-administration/networking/) [Kubernetes 系统组件指标](/zh/docs/concepts/cluster-administration/system-metrics/) [日志架构](/zh/docs/concepts/cluster-administration/logging/) [系统日志](/zh/docs/concepts/cluster-administration/system-logs/) [容器镜像的垃圾收集](/zh/docs/concepts/cluster-administration/kubelet-garbage-collection/) [Kubernetes 中的代理](/zh/docs/concepts/cluster-administration/proxies/) [API 优先级和公平性](/zh/docs/concepts/cluster-administration/flow-control/) [安装扩展 (Addons)](/zh/docs/concepts/cluster-administration/addons/)

        *   [扩展 Kubernetes](/zh/docs/concepts/extend-kubernetes/)

        *   [扩展 Kubernetes 集群](/zh/docs/concepts/extend-kubernetes/extend-cluster/)

            *   [扩展 Kubernetes API](/zh/docs/concepts/extend-kubernetes/api-extension/)

            *   [定制资源](/zh/docs/concepts/extend-kubernetes/api-extension/custom-resources/) [通过聚合层扩展 Kubernetes API](/zh/docs/concepts/extend-kubernetes/api-extension/apiserver-aggregation/)

            [Operator 模式](/zh/docs/concepts/extend-kubernetes/operator/)

            *   [计算, 存储和网络扩展](/zh/docs/concepts/extend-kubernetes/compute-storage-net/)

            *   [网络插件](/zh/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/) [设备插件](/zh/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/)

            [服务目录](/zh/docs/concepts/extend-kubernetes/service-catalog/)


    *   [任务](/zh/docs/tasks/)

    *   *   [安装工具](/zh/docs/tasks/tools/)

        *   [安装并配置 kubectl](/zh/docs/tasks/tools/install-kubectl/)

        *   [管理集群](/zh/docs/tasks/administer-cluster/)

        *   *   [Migrating from dockershim](/docs/tasks/administer-cluster/migrating-from-dockershim/)

            *   [Check whether Dockershim deprecation affects you (EN)](/docs/tasks/administer-cluster/migrating-from-dockershim/check-if-dockershim-deprecation-affects-you/) [Migrating telemetry and security agents from dockershim (EN)](/docs/tasks/administer-cluster/migrating-from-dockershim/migrating-telemetry-and-security-agents/)

            *   [用 kubeadm 进行管理](/zh/docs/tasks/administer-cluster/kubeadm/)

            *   [使用 kubeadm 进行证书管理](/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-certs/) [升级 kubeadm 集群](/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/) [添加 Windows 节点](/zh/docs/tasks/administer-cluster/kubeadm/adding-windows-nodes/) [升级 Windows 节点](/zh/docs/tasks/administer-cluster/kubeadm/upgrading-windows-nodes/)

            *   [管理内存, CPU 和 API 资源](/zh/docs/tasks/administer-cluster/manage-resources/)

            *   [为命名空间配置默认的内存请求和限制](/zh/docs/tasks/administer-cluster/manage-resources/memory-default-namespace/) [为命名空间配置默认的 CPU 请求和限制](/zh/docs/tasks/administer-cluster/manage-resources/cpu-default-namespace/) [配置命名空间的最小和最大内存约束](/zh/docs/tasks/administer-cluster/manage-resources/memory-constraint-namespace/) [为命名空间配置 CPU 最小和最大约束](/zh/docs/tasks/administer-cluster/manage-resources/cpu-constraint-namespace/) [为命名空间配置内存和 CPU 配额](/zh/docs/tasks/administer-cluster/manage-resources/quota-memory-cpu-namespace/) [配置命名空间下 Pod 配额](/zh/docs/tasks/administer-cluster/manage-resources/quota-pod-namespace/)

            *   [安装网络规则驱动](/zh/docs/tasks/administer-cluster/network-policy-provider/)

            *   [使用 Calico 提供 NetworkPolicy](/zh/docs/tasks/administer-cluster/network-policy-provider/calico-network-policy/) [使用 Cilium 提供 NetworkPolicy](/zh/docs/tasks/administer-cluster/network-policy-provider/cilium-network-policy/) [使用 kube-router 提供 NetworkPolicy](/zh/docs/tasks/administer-cluster/network-policy-provider/kube-router-network-policy/) [使用 Romana 提供 NetworkPolicy](/zh/docs/tasks/administer-cluster/network-policy-provider/romana-network-policy/) [使用 Weave Net 提供 NetworkPolicy](/zh/docs/tasks/administer-cluster/network-policy-provider/weave-network-policy/)

            [IP Masquerade Agent 用户指南](/zh/docs/tasks/administer-cluster/ip-masq-agent/) [Kubernetes 云管理控制器](/zh/docs/tasks/administer-cluster/running-cloud-controller/) [为 Kubernetes 运行 etcd 集群](/zh/docs/tasks/administer-cluster/configure-upgrade-etcd/) [为系统守护进程预留计算资源](/zh/docs/tasks/administer-cluster/reserve-compute-resources/) [为节点发布扩展资源](/zh/docs/tasks/administer-cluster/extended-resource-node/) [使用 CoreDNS 进行服务发现](/zh/docs/tasks/administer-cluster/coredns/) [使用 KMS 驱动进行数据加密](/zh/docs/tasks/administer-cluster/kms-provider/) [使用 Kubernetes API 访问集群](/zh/docs/tasks/administer-cluster/access-cluster-api/) [保护集群安全](/zh/docs/tasks/administer-cluster/securing-a-cluster/) [关键插件 Pod 的调度保证](/zh/docs/tasks/administer-cluster/guaranteed-scheduling-critical-addon-pods/) [升级集群](/zh/docs/tasks/administer-cluster/cluster-upgrade/) [名字空间演练](/zh/docs/tasks/administer-cluster/namespaces-walkthrough/) [启用 EndpointSlices](/zh/docs/tasks/administer-cluster/enabling-endpointslices/) [启用 / 禁用 Kubernetes API](/zh/docs/tasks/administer-cluster/enable-disable-api/) [在 Kubernetes 集群中使用 NodeLocal DNSCache](/zh/docs/tasks/administer-cluster/nodelocaldns/) [在 Kubernetes 集群中使用 sysctl](/zh/docs/tasks/administer-cluster/sysctl-cluster/) [在运行中的集群上重新配置节点的 kubelet](/zh/docs/tasks/administer-cluster/reconfigure-kubelet/) [声明网络策略](/zh/docs/tasks/administer-cluster/declare-network-policy/) [安全地清空一个节点](/zh/docs/tasks/administer-cluster/safely-drain-node/) [开发云控制器管理器](/zh/docs/tasks/administer-cluster/developing-cloud-controller-manager/) [开启服务拓扑](/zh/docs/tasks/administer-cluster/enabling-service-topology/) [控制节点上的 CPU 管理策略](/zh/docs/tasks/administer-cluster/cpu-management-policies/) [控制节点上的拓扑管理策略](/zh/docs/tasks/administer-cluster/topology-manager/) [搭建高可用的 Kubernetes Masters](/zh/docs/tasks/administer-cluster/highly-available-master/) [改变默认 StorageClass](/zh/docs/tasks/administer-cluster/change-default-storage-class/) [更改 PersistentVolume 的回收策略](/zh/docs/tasks/administer-cluster/change-pv-reclaim-policy/) [自动扩缩集群 DNS 服务](/zh/docs/tasks/administer-cluster/dns-horizontal-autoscaling/) [自定义 DNS 服务](/zh/docs/tasks/administer-cluster/dns-custom-nameservers/) [访问集群上运行的服务](/zh/docs/tasks/administer-cluster/access-cluster-services/) [调试 DNS 问题](/zh/docs/tasks/administer-cluster/dns-debugging-resolution/) [通过名字空间共享集群](/zh/docs/tasks/administer-cluster/namespaces/) [通过配置文件设置 Kubelet 参数](/zh/docs/tasks/administer-cluster/kubelet-config-file/) [配置 API 对象配额](/zh/docs/tasks/administer-cluster/quota-api-object/) [配置资源不足时的处理方式](/zh/docs/tasks/administer-cluster/out-of-resource/) [限制存储消耗](/zh/docs/tasks/administer-cluster/limit-storage-consumption/) [静态加密 Secret 数据](/zh/docs/tasks/administer-cluster/encrypt-data/)

        *   [配置 Pods 和容器](/zh/docs/tasks/configure-pod-container/)

        *   [为容器和 Pod 分配内存资源](/zh/docs/tasks/configure-pod-container/assign-memory-resource/) [为 Windows Pod 和容器配置 GMSA](/zh/docs/tasks/configure-pod-container/configure-gmsa/) [为 Windows 的 Pod 和容器配置 RunAsUserName](/zh/docs/tasks/configure-pod-container/configure-runasusername/) [为容器和 Pods 分配 CPU 资源](/zh/docs/tasks/configure-pod-container/assign-cpu-resource/) [配置 Pod 的服务质量](/zh/docs/tasks/configure-pod-container/quality-service-pod/) [为容器分派扩展资源](/zh/docs/tasks/configure-pod-container/extended-resource/) [配置 Pod 以使用卷进行存储](/zh/docs/tasks/configure-pod-container/configure-volume-storage/) [配置 Pod 以使用 PersistentVolume 作为存储](/zh/docs/tasks/configure-pod-container/configure-persistent-volume-storage/) [配置 Pod 使用投射卷作存储](/zh/docs/tasks/configure-pod-container/configure-projected-volume-storage/) [为 Pod 或容器配置安全性上下文](/zh/docs/tasks/configure-pod-container/security-context/) [为 Pod 配置服务账户](/zh/docs/tasks/configure-pod-container/configure-service-account/) [从私有仓库拉取镜像](/zh/docs/tasks/configure-pod-container/pull-image-private-registry/) [配置存活, 就绪和启动探测器](/zh/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) [将 Pod 分配给节点](/zh/docs/tasks/configure-pod-container/assign-pods-nodes/) [用节点亲和性把 Pods 分配到节点](/zh/docs/tasks/configure-pod-container/assign-pods-nodes-using-node-affinity/) [配置 Pod 初始化](/zh/docs/tasks/configure-pod-container/configure-pod-initialization/) [为容器的生命周期事件设置处理函数](/zh/docs/tasks/configure-pod-container/attach-handler-lifecycle-event/) [配置 Pod 使用 ConfigMap](/zh/docs/tasks/configure-pod-container/configure-pod-configmap/) [在 Pod 中的容器之间共享进程命名空间](/zh/docs/tasks/configure-pod-container/share-process-namespace/) [创建静态 Pod](/zh/docs/tasks/configure-pod-container/static-pod/) [将 Docker Compose 文件转换为 Kubernetes 资源](/zh/docs/tasks/configure-pod-container/translate-compose-kubernetes/)

        *   [管理 Kubernetes 对象](/zh/docs/tasks/manage-kubernetes-objects/)

        *   [使用配置文件对 Kubernetes 对象进行声明式管理](/zh/docs/tasks/manage-kubernetes-objects/declarative-config/) [使用 Kustomize 对 Kubernetes 对象进行声明式管理](/zh/docs/tasks/manage-kubernetes-objects/kustomization/) [使用指令式命令管理 Kubernetes 对象](/zh/docs/tasks/manage-kubernetes-objects/imperative-command/) [使用配置文件对 Kubernetes 对象进行命令式管理](/zh/docs/tasks/manage-kubernetes-objects/imperative-config/) [使用 kubectl patch 更新 API 对象](/zh/docs/tasks/manage-kubernetes-objects/update-api-object-kubectl-patch/)

        *   [管理 Secrets](/zh/docs/tasks/configmap-secret/)

        *   [使用 kubectl 管理 Secret](/zh/docs/tasks/configmap-secret/managing-secret-using-kubectl/) [使用配置文件管理 Secret](/zh/docs/tasks/configmap-secret/managing-secret-using-config-file/) [使用 Kustomize 管理 Secret](/zh/docs/tasks/configmap-secret/managing-secret-using-kustomize/)

        *   [给应用注入数据](/zh/docs/tasks/inject-data-application/)

        *   [为容器设置启动时要执行的命令和参数](/zh/docs/tasks/inject-data-application/define-command-argument-container/) [为容器设置环境变量](/zh/docs/tasks/inject-data-application/define-environment-variable-container/) [定义相互依赖的环境变量](/zh/docs/tasks/inject-data-application/define-interdependent-environment-variables/) [通过环境变量将 Pod 信息呈现给容器](/zh/docs/tasks/inject-data-application/environment-variable-expose-pod-information/) [通过文件将 Pod 信息呈现给容器](/zh/docs/tasks/inject-data-application/downward-api-volume-expose-pod-information/) [使用 Secret 安全地分发凭证](/zh/docs/tasks/inject-data-application/distribute-credentials-secure/)

        *   [运行应用](/zh/docs/tasks/run-application/)

        *   [运行一个单实例有状态应用](/zh/docs/tasks/run-application/run-single-instance-stateful-application/) [运行一个有状态的应用程序](/zh/docs/tasks/run-application/run-replicated-stateful-application/) [删除 StatefulSet](/zh/docs/tasks/run-application/delete-stateful-set/) [强制删除 StatefulSet 类型的 Pods](/zh/docs/tasks/run-application/force-delete-stateful-set-pod/) [Pod 水平自动扩缩](/zh/docs/tasks/run-application/horizontal-pod-autoscale/) [Horizontal Pod Autoscaler 演练](/zh/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/) [为应用程序设置干扰预算 (Disruption Budget)](/zh/docs/tasks/run-application/configure-pdb/) [使用 Deployment 运行一个无状态应用](/zh/docs/tasks/run-application/run-stateless-application-deployment/) [扩缩 StatefulSet](/zh/docs/tasks/run-application/scale-stateful-set/)

        *   [运行 Jobs](/zh/docs/tasks/job/)

        *   [使用 CronJob 运行自动化任务](/zh/docs/tasks/job/automated-tasks-with-cron-jobs/) [使用展开的方式进行并行处理](/zh/docs/tasks/job/parallel-processing-expansion/) [使用工作队列进行粗粒度并行处理](/zh/docs/tasks/job/coarse-parallel-processing-work-queue/) [使用工作队列进行精细的并行处理](/zh/docs/tasks/job/fine-parallel-processing-work-queue/)

        *   [访问集群中的应用程序](/zh/docs/tasks/access-application-cluster/)

        *   [Web 界面 (Dashboard)](/zh/docs/tasks/access-application-cluster/web-ui-dashboard/) [访问集群](/zh/docs/tasks/access-application-cluster/access-cluster/) [使用端口转发来访问集群中的应用](/zh/docs/tasks/access-application-cluster/port-forward-access-application-cluster/) [使用服务来访问集群中的应用](/zh/docs/tasks/access-application-cluster/service-access-application-cluster/) [使用 Service 把前端连接到后端](/zh/docs/tasks/access-application-cluster/connecting-frontend-backend/) [创建外部负载均衡器](/zh/docs/tasks/access-application-cluster/create-external-load-balancer/) [列出集群中所有运行容器的镜像](/zh/docs/tasks/access-application-cluster/list-all-running-container-images/) [在 Minikube 环境中使用 NGINX Ingress 控制器配置 Ingress](/zh/docs/tasks/access-application-cluster/ingress-minikube/) [为集群配置 DNS](/zh/docs/tasks/access-application-cluster/configure-dns-cluster/) [同 Pod 内的容器使用共享卷通信](/zh/docs/tasks/access-application-cluster/communicate-containers-same-pod-shared-volume/) [配置对多集群的访问](/zh/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

        *   [监控, 日志和排错](/zh/docs/tasks/debug-application-cluster/)

        *   [StackDriver 中的事件](/zh/docs/tasks/debug-application-cluster/events-stackdriver/) [使用 crictl 对 Kubernetes 节点进行调试](/zh/docs/tasks/debug-application-cluster/crictl/) [使用 ElasticSearch 和 Kibana 进行日志管理](/zh/docs/tasks/debug-application-cluster/logging-elasticsearch-kibana/) [使用 Stackdriver 生成日志](/zh/docs/tasks/debug-application-cluster/logging-stackdriver/) [在本地开发和调试服务](/zh/docs/tasks/debug-application-cluster/local-debugging/) [审计](/zh/docs/tasks/debug-application-cluster/audit/) [应用故障排查](/zh/docs/tasks/debug-application-cluster/debug-application/) [应用自测与调试](/zh/docs/tasks/debug-application-cluster/debug-application-introspection/) [故障诊断](/zh/docs/tasks/debug-application-cluster/troubleshooting/) [确定 Pod 失败的原因](/zh/docs/tasks/debug-application-cluster/determine-reason-pod-failure/) [节点健康监测](/zh/docs/tasks/debug-application-cluster/monitor-node-health/) [获取正在运行容器的 Shell](/zh/docs/tasks/debug-application-cluster/get-shell-running-container/) [调试 Init 容器](/zh/docs/tasks/debug-application-cluster/debug-init-containers/) [调试 Pods 和 ReplicationControllers](/zh/docs/tasks/debug-application-cluster/debug-pod-replication-controller/) [调试 Service](/zh/docs/tasks/debug-application-cluster/debug-service/) [调试 StatefulSet](/zh/docs/tasks/debug-application-cluster/debug-stateful-set/) [调试运行中的 Pod](/zh/docs/tasks/debug-application-cluster/debug-running-pod/) [资源指标管道](/zh/docs/tasks/debug-application-cluster/resource-metrics-pipeline/) [资源监控工具](/zh/docs/tasks/debug-application-cluster/resource-usage-monitoring/) [集群故障排查](/zh/docs/tasks/debug-application-cluster/debug-cluster/)

        *   [扩展 Kubernetes](/zh/docs/tasks/extend-kubernetes/)

        *   *   [使用自定义资源](/zh/docs/tasks/extend-kubernetes/custom-resources/)

            *   [使用 CustomResourceDefinition 扩展 Kubernetes API](/zh/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definitions/) [CustomResourceDefinition 的版本](/zh/docs/tasks/extend-kubernetes/custom-resources/custom-resource-definition-versioning/)

            [配置聚合层](/zh/docs/tasks/extend-kubernetes/configure-aggregation-layer/) [安装一个扩展的 API server](/zh/docs/tasks/extend-kubernetes/setup-extension-api-server/) [配置多个调度器](/zh/docs/tasks/extend-kubernetes/configure-multiple-schedulers/) [使用 HTTP 代理访问 Kubernetes API](/zh/docs/tasks/extend-kubernetes/http-proxy-access-api/) [设置 Konnectivity 服务](/zh/docs/tasks/extend-kubernetes/setup-konnectivity/)

        *   [TLS](/zh/docs/tasks/tls/)

        *   [为 kubelet 配置证书轮换](/zh/docs/tasks/tls/certificate-rotation/) [手动轮换 CA 证书](/zh/docs/tasks/tls/manual-rotation-of-ca-certificates/) [管理集群中的 TLS 认证](/zh/docs/tasks/tls/managing-tls-in-a-cluster/)

        *   [管理集群守护进程](/zh/docs/tasks/manage-daemon/)

        *   [对 DaemonSet 执行滚动更新](/zh/docs/tasks/manage-daemon/update-daemon-set/) [对 DaemonSet 执行回滚](/zh/docs/tasks/manage-daemon/rollback-daemon-set/)

        *   [安装服务目录](/zh/docs/tasks/service-catalog/)

        *   [使用 Helm 安装 Service Catalog](/zh/docs/tasks/service-catalog/install-service-catalog-using-helm/) [使用 SC 安装服务目录](/zh/docs/tasks/service-catalog/install-service-catalog-using-sc/)

        *   [网络](/zh/docs/tasks/network/)

        *   [验证 IPv4/IPv6 双协议栈](/zh/docs/tasks/network/validate-dual-stack/)

        [Configure a kubelet image credential provider (EN)](/docs/tasks/kubelet-credential-provider/kubelet-credential-provider/) [用插件扩展 kubectl](/zh/docs/tasks/extend-kubectl/kubectl-plugins/) [管理巨页 (HugePages)](/zh/docs/tasks/manage-hugepages/scheduling-hugepages/) [调度 GPUs](/zh/docs/tasks/manage-gpus/scheduling-gpus/)

    *   [教程](/zh/docs/tutorials/)

    *   [你好, Minikube](/zh/docs/tutorials/hello-minikube/)

        *   [学习 Kubernetes 基础知识](/zh/docs/tutorials/kubernetes-basics/)

        *   *   [创建集群](/zh/docs/tutorials/kubernetes-basics/create-cluster/)

            *   [使用 Minikube 创建集群](/zh/docs/tutorials/kubernetes-basics/create-cluster/cluster-intro/) [交互式教程 - 创建集群](/zh/docs/tutorials/kubernetes-basics/create-cluster/cluster-interactive/)

            *   [部署应用](/zh/docs/tutorials/kubernetes-basics/deploy-app/)

            *   [使用 kubectl 创建 Deployment](/zh/docs/tutorials/kubernetes-basics/deploy-app/deploy-intro/) [交互式教程 - 部署应用](/zh/docs/tutorials/kubernetes-basics/deploy-app/deploy-interactive/)

            *   [了解你的应用](/zh/docs/tutorials/kubernetes-basics/explore/)

            *   [查看 pod 和工作节点](/zh/docs/tutorials/kubernetes-basics/explore/explore-intro/) [交互式教程 - 了解你的应用](/zh/docs/tutorials/kubernetes-basics/explore/explore-interactive/)

            *   [公开地暴露你的应用](/zh/docs/tutorials/kubernetes-basics/expose/)

            *   [使用 Service 暴露您的应用](/zh/docs/tutorials/kubernetes-basics/expose/expose-intro/) [交互式教程 - 暴露你的应用](/zh/docs/tutorials/kubernetes-basics/expose/expose-interactive/)

            *   [缩放你的应用](/zh/docs/tutorials/kubernetes-basics/scale/)

            *   [运行应用程序的多个实例](/zh/docs/tutorials/kubernetes-basics/scale/scale-intro/) [交互教程 - 缩放你的应用](/zh/docs/tutorials/kubernetes-basics/scale/scale-interactive/)

            *   [更新你的应用](/zh/docs/tutorials/kubernetes-basics/update/)

            *   [执行滚动更新](/zh/docs/tutorials/kubernetes-basics/update/update-intro/) [交互式教程 - 更新你的应用](/zh/docs/tutorials/kubernetes-basics/update/update-interactive/)


        *   [配置](/zh/docs/tutorials/configuration/)

        *   *   [示例: 配置 java 微服务](/zh/docs/tutorials/configuration/configure-java-microservice/)

            *   [使用 MicroProfile, ConfigMaps, Secrets 实现外部化应用配置](/zh/docs/tutorials/configuration/configure-java-microservice/configure-java-microservice/) [互动教程 - 配置 java 微服务](/zh/docs/tutorials/configuration/configure-java-microservice/configure-java-microservice-interactive/)

            [使用 ConfigMap 来配置 Redis](/zh/docs/tutorials/configuration/configure-redis-using-configmap/)

        *   [无状态应用程序](/zh/docs/tutorials/stateless-application/)

        *   [公开外部 IP 地址以访问集群中应用程序](/zh/docs/tutorials/stateless-application/expose-external-ip-address/) [示例: 使用 Redis 部署 PHP 留言板应用程序](/zh/docs/tutorials/stateless-application/guestbook/) [示例: 添加日志和指标到 PHP / Redis Guestbook 案例](/zh/docs/tutorials/stateless-application/guestbook-logs-metrics-with-elk/)

        *   [有状态的应用](/zh/docs/tutorials/stateful-application/)

        *   [示例: 使用 Persistent Volumes 部署 WordPress 和 MySQL](/zh/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/) [StatefulSet 基础](/zh/docs/tutorials/stateful-application/basic-stateful-set/) [示例: 使用 Stateful Sets 部署 Cassandra](/zh/docs/tutorials/stateful-application/cassandra/) [运行 ZooKeeper, 一个 CP 分布式系统](/zh/docs/tutorials/stateful-application/zookeeper/)

        *   [集群](/zh/docs/tutorials/clusters/)

        *   [使用 Seccomp 限制容器的系统调用](/zh/docs/tutorials/clusters/seccomp/) [AppArmor](/zh/docs/tutorials/clusters/apparmor/)

        *   [Services](/zh/docs/tutorials/services/)

        *   [使用 Source IP](/zh/docs/tutorials/services/source-ip/)


    *   [参考](/zh/docs/reference/)

    *   [标准化词汇表](/zh/docs/reference/glossary/)

        *   [Kubernetes 问题和安全](/zh/docs/reference/issues-security/)

        *   [Kubernetes 问题追踪](/zh/docs/reference/issues-security/issues/) [Kubernetes 安全和信息披露](/zh/docs/reference/issues-security/security/)

        *   [使用 Kubernetes API](/zh/docs/reference/using-api/)

        *   [Kubernetes API 概念](/zh/docs/reference/using-api/api-concepts/) [服务器端应用 (Server-Side Apply)](/zh/docs/reference/using-api/server-side-apply/) [客户端库](/zh/docs/reference/using-api/client-libraries/) [Kubernetes 弃用策略](/zh/docs/reference/using-api/deprecation-policy/) [Kubernetes API 健康端点](/zh/docs/reference/using-api/health-checks/)

        [Well-Known Labels, Annotations and Taints (EN)](/docs/reference/labels-annotations-taints/)

        *   [访问 API](/zh/docs/reference/access-authn-authz/)

        *   [用户认证](/zh/docs/reference/access-authn-authz/authentication/) [使用启动引导令牌 (Bootstrap Tokens) 认证](/zh/docs/reference/access-authn-authz/bootstrap-tokens/) [证书签名请求](/zh/docs/reference/access-authn-authz/certificate-signing-requests/) [使用准入控制器](/zh/docs/reference/access-authn-authz/admission-controllers/) [动态准入控制](/zh/docs/reference/access-authn-authz/extensible-admission-controllers/) [管理 Service Accounts](/zh/docs/reference/access-authn-authz/service-accounts-admin/) [鉴权概述](/zh/docs/reference/access-authn-authz/authorization/) [使用 RBAC 鉴权](/zh/docs/reference/access-authn-authz/rbac/) [使用 Node 鉴权](/zh/docs/reference/access-authn-authz/node/) [Webhook 模式](/zh/docs/reference/access-authn-authz/webhook/) [使用 ABAC 鉴权](/zh/docs/reference/access-authn-authz/abac/)

        *   [API 参考](/zh/docs/reference/kubernetes-api/)

        *   *   [Workloads Resources](/docs/reference/kubernetes-api/workloads-resources/)

            *   [Pod (EN)](/docs/reference/kubernetes-api/workloads-resources/pod-v1/) [Container (EN)](/docs/reference/kubernetes-api/workloads-resources/container/) [EphemeralContainer (EN)](/docs/reference/kubernetes-api/workloads-resources/ephemeral-container/) [PodTemplate (EN)](/docs/reference/kubernetes-api/workloads-resources/pod-template-v1/) [ReplicationController (EN)](/docs/reference/kubernetes-api/workloads-resources/replication-controller-v1/) [ReplicaSet (EN)](/docs/reference/kubernetes-api/workloads-resources/replica-set-v1/) [Deployment (EN)](/docs/reference/kubernetes-api/workloads-resources/deployment-v1/) [StatefulSet (EN)](/docs/reference/kubernetes-api/workloads-resources/stateful-set-v1/) [ControllerRevision (EN)](/docs/reference/kubernetes-api/workloads-resources/controller-revision-v1/) [DaemonSet (EN)](/docs/reference/kubernetes-api/workloads-resources/daemon-set-v1/) [Job (EN)](/docs/reference/kubernetes-api/workloads-resources/job-v1/) [CronJob v1beta1 (EN)](/docs/reference/kubernetes-api/workloads-resources/cron-job-v1beta1/) [CronJob v2alpha1 (EN)](/docs/reference/kubernetes-api/workloads-resources/cron-job-v2alpha1/) [HorizontalPodAutoscaler (EN)](/docs/reference/kubernetes-api/workloads-resources/horizontal-pod-autoscaler-v1/) [HorizontalPodAutoscaler v2beta2 (EN)](/docs/reference/kubernetes-api/workloads-resources/horizontal-pod-autoscaler-v2beta2/) [PriorityClass (EN)](/docs/reference/kubernetes-api/workloads-resources/priority-class-v1/)

            *   [Services Resources](/docs/reference/kubernetes-api/services-resources/)

            *   [Service (EN)](/docs/reference/kubernetes-api/services-resources/service-v1/) [Endpoints (EN)](/docs/reference/kubernetes-api/services-resources/endpoints-v1/) [EndpointSlice v1beta1 (EN)](/docs/reference/kubernetes-api/services-resources/endpoint-slice-v1beta1/) [Ingress (EN)](/docs/reference/kubernetes-api/services-resources/ingress-v1/) [IngressClass (EN)](/docs/reference/kubernetes-api/services-resources/ingress-class-v1/)

            *   [Config and Storage Resources](/docs/reference/kubernetes-api/config-and-storage-resources/)

            *   [ConfigMap (EN)](/docs/reference/kubernetes-api/config-and-storage-resources/config-map-v1/) [Secret (EN)](/docs/reference/kubernetes-api/config-and-storage-resources/secret-v1/) [Volume (EN)](/docs/reference/kubernetes-api/config-and-storage-resources/volume/) [PersistentVolumeClaim (EN)](/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-claim-v1/) [PersistentVolume (EN)](/docs/reference/kubernetes-api/config-and-storage-resources/persistent-volume-v1/) [StorageClass (EN)](/docs/reference/kubernetes-api/config-and-storage-resources/storage-class-v1/) [VolumeAttachment (EN)](/docs/reference/kubernetes-api/config-and-storage-resources/volume-attachment-v1/) [CSIDriver (EN)](/docs/reference/kubernetes-api/config-and-storage-resources/csi-driver-v1/) [CSINode (EN)](/docs/reference/kubernetes-api/config-and-storage-resources/csi-node-v1/)

            *   [Authentication Resources](/docs/reference/kubernetes-api/authentication-resources/)

            *   [ServiceAccount (EN)](/docs/reference/kubernetes-api/authentication-resources/service-account-v1/) [TokenRequest (EN)](/docs/reference/kubernetes-api/authentication-resources/token-request-v1/) [TokenReview (EN)](/docs/reference/kubernetes-api/authentication-resources/token-review-v1/) [CertificateSigningRequest (EN)](/docs/reference/kubernetes-api/authentication-resources/certificate-signing-request-v1/)

            *   [Authorization Resources](/docs/reference/kubernetes-api/authorization-resources/)

            *   [LocalSubjectAccessReview (EN)](/docs/reference/kubernetes-api/authorization-resources/local-subject-access-review-v1/) [SelfSubjectAccessReview (EN)](/docs/reference/kubernetes-api/authorization-resources/self-subject-access-review-v1/) [SelfSubjectRulesReview (EN)](/docs/reference/kubernetes-api/authorization-resources/self-subject-rules-review-v1/) [SubjectAccessReview (EN)](/docs/reference/kubernetes-api/authorization-resources/subject-access-review-v1/) [ClusterRole (EN)](/docs/reference/kubernetes-api/authorization-resources/cluster-role-v1/) [ClusterRoleBinding (EN)](/docs/reference/kubernetes-api/authorization-resources/cluster-role-binding-v1/) [Role (EN)](/docs/reference/kubernetes-api/authorization-resources/role-v1/) [RoleBinding (EN)](/docs/reference/kubernetes-api/authorization-resources/role-binding-v1/)

            *   [Policies Resources](/docs/reference/kubernetes-api/policies-resources/)

            *   [LimitRange (EN)](/docs/reference/kubernetes-api/policies-resources/limit-range-v1/) [ResourceQuota (EN)](/docs/reference/kubernetes-api/policies-resources/resource-quota-v1/) [NetworkPolicy (EN)](/docs/reference/kubernetes-api/policies-resources/network-policy-v1/) [PodDisruptionBudget v1beta1 (EN)](/docs/reference/kubernetes-api/policies-resources/pod-disruption-budget-v1beta1/) [PodSecurityPolicy v1beta1 (EN)](/docs/reference/kubernetes-api/policies-resources/pod-security-policy-v1beta1/)

            *   [Extend Resources](/docs/reference/kubernetes-api/extend-resources/)

            *   [CustomResourceDefinition (EN)](/docs/reference/kubernetes-api/extend-resources/custom-resource-definition-v1/) [MutatingWebhookConfiguration (EN)](/docs/reference/kubernetes-api/extend-resources/mutating-webhook-configuration-v1/) [ValidatingWebhookConfiguration (EN)](/docs/reference/kubernetes-api/extend-resources/validating-webhook-configuration-v1/)

            *   [Cluster Resources](/docs/reference/kubernetes-api/cluster-resources/)

            *   [Node (EN)](/docs/reference/kubernetes-api/cluster-resources/node-v1/) [Namespace (EN)](/docs/reference/kubernetes-api/cluster-resources/namespace-v1/) [Event (EN)](/docs/reference/kubernetes-api/cluster-resources/event-v1/) [APIService (EN)](/docs/reference/kubernetes-api/cluster-resources/api-service-v1/) [Lease (EN)](/docs/reference/kubernetes-api/cluster-resources/lease-v1/) [RuntimeClass (EN)](/docs/reference/kubernetes-api/cluster-resources/runtime-class-v1/) [FlowSchema v1beta1 (EN)](/docs/reference/kubernetes-api/cluster-resources/flow-schema-v1beta1/) [PriorityLevelConfiguration v1beta1 (EN)](/docs/reference/kubernetes-api/cluster-resources/priority-level-configuration-v1beta1/) [Binding (EN)](/docs/reference/kubernetes-api/cluster-resources/binding-v1/) [ComponentStatus (EN)](/docs/reference/kubernetes-api/cluster-resources/component-status-v1/)

            *   [Common Definitions](/docs/reference/kubernetes-api/common-definitions/)

            *   [DeleteOptions (EN)](/docs/reference/kubernetes-api/common-definitions/delete-options/) [DownwardAPIVolumeFile (EN)](/docs/reference/kubernetes-api/common-definitions/downward-api-volume-file/) [ExecAction (EN)](/docs/reference/kubernetes-api/common-definitions/exec-action/) [HTTPGetAction (EN)](/docs/reference/kubernetes-api/common-definitions/http-get-action/) [JSONSchemaProps (EN)](/docs/reference/kubernetes-api/common-definitions/json-schema-props/) [KeyToPath (EN)](/docs/reference/kubernetes-api/common-definitions/key-to-path/) [LabelSelector (EN)](/docs/reference/kubernetes-api/common-definitions/label-selector/) [ListMeta (EN)](/docs/reference/kubernetes-api/common-definitions/list-meta/) [LocalObjectReference (EN)](/docs/reference/kubernetes-api/common-definitions/local-object-reference/) [NodeAffinity (EN)](/docs/reference/kubernetes-api/common-definitions/node-affinity/) [NodeSelectorRequirement (EN)](/docs/reference/kubernetes-api/common-definitions/node-selector-requirement/) [ObjectFieldSelector (EN)](/docs/reference/kubernetes-api/common-definitions/object-field-selector/) [ObjectMeta (EN)](/docs/reference/kubernetes-api/common-definitions/object-meta/) [ObjectReference (EN)](/docs/reference/kubernetes-api/common-definitions/object-reference/) [Patch (EN)](/docs/reference/kubernetes-api/common-definitions/patch/) [PodAffinity (EN)](/docs/reference/kubernetes-api/common-definitions/pod-affinity/) [PodAntiAffinity (EN)](/docs/reference/kubernetes-api/common-definitions/pod-anti-affinity/) [Quantity (EN)](/docs/reference/kubernetes-api/common-definitions/quantity/) [ResourceFieldSelector (EN)](/docs/reference/kubernetes-api/common-definitions/resource-field-selector/) [Status (EN)](/docs/reference/kubernetes-api/common-definitions/status/) [TCPSocketAction (EN)](/docs/reference/kubernetes-api/common-definitions/tcp-socket-action/) [TypedLocalObjectReference (EN)](/docs/reference/kubernetes-api/common-definitions/typed-local-object-reference/)

            [Common Parameters (EN)](/docs/reference/kubernetes-api/common-parameters/common-parameters/) [v1.20](/zh/docs/reference/kubernetes-api/api-index/) [知名标签 (Label), 注解 (Annotation) 和 污点 (Taint)](/zh/docs/reference/kubernetes-api/labels-annotations-taints/)

        *   [安装工具](/zh/docs/reference/setup-tools/)

        *   *   [Kubeadm](/zh/docs/reference/setup-tools/kubeadm/)

            *   *   [Kubeadm Generated](/docs/reference/setup-tools/kubeadm/generated/)

                *   [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_alpha/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_alpha_kubeconfig/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_alpha_kubeconfig_user/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_alpha_kubelet/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_alpha_kubelet_config/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_alpha_kubelet_config_enable-dynamic/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_alpha_selfhosting/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_alpha_selfhosting_pivot/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_certificate-key/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_check-expiration/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_generate-csr/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_admin.conf/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_all/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_apiserver-etcd-client/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_apiserver-kubelet-client/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_apiserver/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_controller-manager.conf/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_etcd-healthcheck-client/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_etcd-peer/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_etcd-server/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_front-proxy-client/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_certs_renew_scheduler.conf/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_completion/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_config/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_config_images/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_config_images_list/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_config_images_pull/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_config_migrate/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_config_print/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_config_print_init-defaults/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_config_print_join-defaults/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_config_view/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_addon/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_addon_all/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_addon_coredns/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_addon_kube-proxy/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_bootstrap-token/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_certs/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_certs_all/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_certs_apiserver-etcd-client/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_certs_apiserver-kubelet-client/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_certs_apiserver/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_certs_ca/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_certs_etcd-ca/) [(EN)](/docs/reference/setup-tools/kubeadm/generated/kubeadm_init_phase_certs_etcd-healthcheck-client/)

                [kubeadm init](/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/) [kubeadm join](/zh/docs/reference/setup-tools/kubeadm/kubeadm-join/) [kubeadm upgrade](/zh/docs/reference/setup-tools/kubeadm/kubeadm-upgrade/) [kubeadm config](/zh/docs/reference/setup-tools/kubeadm/kubeadm-config/) [kubeadm reset](/zh/docs/reference/setup-tools/kubeadm/kubeadm-reset/) [kubeadm token](/zh/docs/reference/setup-tools/kubeadm/kubeadm-token/) [kubeadm version](/zh/docs/reference/setup-tools/kubeadm/kubeadm-version/) [kubeadm alpha](/zh/docs/reference/setup-tools/kubeadm/kubeadm-alpha/) [kubeadm certs](/zh/docs/reference/setup-tools/kubeadm/kubeadm-certs/) [kubeadm init phase](/zh/docs/reference/setup-tools/kubeadm/kubeadm-init-phase/) [kubeadm join phase](/zh/docs/reference/setup-tools/kubeadm/kubeadm-join-phase/) [kubeadm reset phase](/zh/docs/reference/setup-tools/kubeadm/kubeadm-reset-phase/) [kubeadm upgrade phase](/zh/docs/reference/setup-tools/kubeadm/kubeadm-upgrade-phase/) [实现细节](/zh/docs/reference/setup-tools/kubeadm/implementation-details/)


        *   [kubectl 命令行界面](/zh/docs/reference/kubectl/)

        *   [kubectl 概述](/zh/docs/reference/kubectl/overview/) [JSONPath 支持](/zh/docs/reference/kubectl/jsonpath/) [kubectl](/zh/docs/reference/kubectl/kubectl/) [kubectl 命令](/zh/docs/reference/kubectl/kubectl-cmds/) [kubectl 备忘单](/zh/docs/reference/kubectl/cheatsheet/) [kubectl 的用法约定](/zh/docs/reference/kubectl/conventions/) [适用于 Docker 用户的 kubectl](/zh/docs/reference/kubectl/docker-cli-to-kubectl/)

        *   [命令行工具参考](/zh/docs/reference/command-line-tools-reference/)

        *   [特性门控](/zh/docs/reference/command-line-tools-reference/feature-gates/) [kubelet](/zh/docs/reference/command-line-tools-reference/kubelet/) [kube-apiserver](/zh/docs/reference/command-line-tools-reference/kube-apiserver/) [kube-controller-manager](/zh/docs/reference/command-line-tools-reference/kube-controller-manager/) [kube-proxy](/zh/docs/reference/command-line-tools-reference/kube-proxy/) [kube-scheduler](/zh/docs/reference/command-line-tools-reference/kube-scheduler/) [Kubelet 认证 / 鉴权](/zh/docs/reference/command-line-tools-reference/kubelet-authentication-authorization/) [TLS bootstrapping](/zh/docs/reference/command-line-tools-reference/kubelet-tls-bootstrapping/)

        *   [调度](/zh/docs/reference/scheduling/)

        *   [调度策略](/zh/docs/reference/scheduling/policies/) [调度器配置](/zh/docs/reference/scheduling/config/)

        [工具](/zh/docs/reference/tools/)

    *   [贡献](/zh/docs/contribute/)

    *   [提出内容改进建议](/zh/docs/contribute/suggest-improvements/)

        *   [贡献新内容](/zh/docs/contribute/new-content/)

        *   [概述](/zh/docs/contribute/new-content/overview/) [发起拉取请求 (PR)](/zh/docs/contribute/new-content/open-a-pr/) [为发行版本撰写文档](/zh/docs/contribute/new-content/new-features/) [博客和案例分析](/zh/docs/contribute/new-content/blogs-case-studies/)

        *   [评阅变更](/zh/docs/contribute/review/)

        *   [评阅 PRs](/zh/docs/contribute/review/reviewing-prs/) [评阅人和批准人](/zh/docs/contribute/review/for-approvers/)

        [本地化 Kubernetes 文档](/zh/docs/contribute/localization/)

        *   [参与 SIG Docs](/zh/docs/contribute/participate/)

        *   [角色与责任](/zh/docs/contribute/participate/roles-and-responsibilities/) [PR 管理者](/zh/docs/contribute/participate/pr-wranglers/)

        *   [参考文档概述](/zh/docs/contribute/generate-ref-docs/)

        *   [为上游 Kubernetes 代码库做出贡献](/zh/docs/contribute/generate-ref-docs/contribute-upstream/) [快速入门](/zh/docs/contribute/generate-ref-docs/quickstart/) [为 Kubernetes API 生成参考文档](/zh/docs/contribute/generate-ref-docs/kubernetes-api/) [为 kubectl 命令集生成参考文档](/zh/docs/contribute/generate-ref-docs/kubectl/) [为 Kubernetes 组件和工具生成参考文档](/zh/docs/contribute/generate-ref-docs/kubernetes-components/) [](/zh/docs/contribute/generate-ref-docs/prerequisites-ref-docs/)

        *   [文档样式概述](/zh/docs/contribute/style/)

        *   [内容指南](/zh/docs/contribute/style/content-guide/) [样式指南](/zh/docs/contribute/style/style-guide/) [撰写新主题](/zh/docs/contribute/style/write-new-topic/) [页面内容类型](/zh/docs/contribute/style/page-content-types/) [内容组织](/zh/docs/contribute/style/content-organization/) [定制 Hugo 短代码](/zh/docs/contribute/style/hugo-shortcodes/)

        [高级贡献](/zh/docs/contribute/advanced/) [中文本地化样式指南](/zh/docs/contribute/localization_zh/)

    [](/zh/docs/doc-contributor-tools/linkchecker/readme/) [测试页面 (中文版)](/zh/docs/test/)

1.  [Kubernetes 文档](https://kubernetes.io/zh/docs/)
2.  [任务](https://kubernetes.io/zh/docs/tasks/)
3.  [配置 Pods 和容器](https://kubernetes.io/zh/docs/tasks/configure-pod-container/)
4.  [从私有仓库拉取镜像](https://kubernetes.io/zh/docs/tasks/configure-pod-container/pull-image-private-registry/)

# 从私有仓库拉取镜像

本文介绍如何使用 Secret 从私有的 Docker 镜像仓库或代码仓库拉取镜像来创建 Pod.

## 准备开始

*   你必须拥有一个 Kubernetes 的集群, 同时你的 Kubernetes 集群必须带有 kubectl 命令行工具. 如果你还没有集群, 你可以通过 [Minikube](/zh/docs/tasks/tools/#minikube) 构建一 个你自己的集群, 或者你可以使用下面任意一个 Kubernetes 工具构建:

    *   [Katacoda](https://www.katacoda.com/courses/kubernetes/playground)
    *   [玩转 Kubernetes](http://labs.play-with-k8s.com/)

    要获知版本信息, 请输入 `kubectl version`.

你需要 [Docker ID](https://docs.docker.com/docker-id/) 和密码来进行本练习.

## 登录 Docker 镜像仓库

在个人电脑上, 要想拉取私有镜像必须在镜像仓库上进行身份验证.

```
docker login
```

当出现提示时, 输入 Docker 用户名和密码.

登录过程会创建或更新保存有授权令牌的 `config.json` 文件.

查看 `config.json` 文件:

```
cat ~/.docker/config.json
```

输出结果包含类似于以下内容的部分:

```
{
    "auths": {
        "https://index.docker.io/v1/": {
            "auth": "c3R...zE2"
        }
    }
}
```

> **说明:** 如果使用 Docker 凭证仓库, 则不会看到 `auth` 条目, 看到的将是以仓库名称作为值的 `credsStore` 条目.

## 在集群中创建保存授权令牌的 Secret

Kubernetes 集群使用 `docker-registry` 类型的 Secret 来通过容器仓库的身份验证, 进而提取私有映像.

创建 Secret, 命名为 `regcred`:

```
kubectl create secret docker-registry regcred \   --docker-server= <你的镜像仓库服务器> \   --docker-username= <你的用户名> \   --docker-password= <你的密码> \   --docker-email= <你的邮箱地址>
```

在这里:

*   `<your-registry-server>` 是你的私有 Docker 仓库全限定域名 (FQDN). (参考 [https://index.docker.io/v1/](https://index.docker.io/v1/) 中关于 DockerHub 的部分)
*   `<your-name>` 是你的 Docker 用户名.
*   `<your-pword>` 是你的 Docker 密码.
*   `<your-email>` 是你的 Docker 邮箱.

这样你就成功地将集群中的 Docker 凭据设置为名为 `regcred` 的 Secret.

## 检查 Secret `regcred`

要了解你创建的 `regcred` Secret 的内容, 可以用 YAML 格式进行查看:

```
kubectl get secret regcred --output= yaml
```

输出和下面类似:

```
apiVersion: v1 data:  .dockerconfigjson: eyJodHRwczovL2luZGV4L ... J0QUl6RTIifX0= kind: Secret metadata:  ...  name: regcred  ... type: kubernetes.io/dockerconfigjson
```

`.dockerconfigjson` 字段的值是 Docker 凭据的 base64 表示.

要了解 `dockerconfigjson` 字段中的内容, 请将 Secret 数据转换为可读格式:

```
kubectl get secret regcred --output="jsonpath= {.data.\.dockerconfigjson}" | base64 --decode
```

输出和下面类似:

```
{"auths": {"yourprivateregistry.com": {"username":"janedoe","password":"xxxxxxxxxxx","email":"jdoe@example.com","auth":"c3R...zE2"}}}
```

要了解 `auth` 字段中的内容, 请将 base64 编码过的数据转换为可读格式:

```
echo "c3R...zE2" | base64 --decode
```

输出结果中, 用户名和密码用 `:` 链接, 类似下面这样:

```
janedoe: xxxxxxxxxxx
```

注意, Secret 数据包含与本地 `~/.docker/config.json` 文件类似的授权令牌.

这样你就已经成功地将 Docker 凭据设置为集群中的名为 `regcred` 的 Secret.

## 创建一个使用你的 Secret 的 Pod

下面是一个 Pod 配置文件, 它需要访问 `regcred` 中的 Docker 凭据:

[`pods/private-reg-pod.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/zh/examples/pods/private-reg-pod.yaml) ! [](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy pods/private-reg-pod.yaml to clipboard")

```
apiVersion: v1 kind: Pod metadata:  name: private-reg spec:  containers:  - name: private-reg-container  image: <your-private-image>  imagePullSecrets:  - name: regcred
```

下载上述文件:

```
wget -O my-private-reg-pod.yaml https://k8s.io/examples/pods/private-reg-pod.yaml
```

在 `my-private-reg-pod.yaml` 文件中, 使用私有仓库的镜像路径替换 `<your-private-image>`, 例如:

```
janedoe/jdoe-private: v1
```

要从私有仓库拉取镜像, Kubernetes 需要凭证. 配置文件中的 `imagePullSecrets` 字段表明 Kubernetes 应该通过名为 `regcred` 的 Secret 获取凭证.

创建使用了你的 Secret 的 Pod, 并检查它是否正常运行:

```
kubectl apply -f my-private-reg-pod.yaml
kubectl get pod private-reg
```

## 接下来

*   进一步了解 [Secret](/zh/docs/concepts/configuration/secret/)
*   进一步了解 [使用私有仓库](/zh/docs/concepts/containers/images/#using-a-private-registry)
*   参考 [kubectl create secret docker-registry](/docs/reference/generated/kubectl/kubectl-commands/#-em-secret-docker-registry-em-)
*   参考 [Secret](/docs/reference/generated/kubernetes-api/v1.20/#secret-v1-core)
*   参考 [PodSpec](/docs/reference/generated/kubernetes-api/v1.20/#podspec-v1-core) 中的 `imagePullSecrets` 字段
