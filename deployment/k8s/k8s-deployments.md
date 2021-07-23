---
title: k8s-deployments
date: 2021-01-30 15:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: k8s-deployments
photo:
---

https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/

Deployments | KubernetesDeployments | Kubernetes

[](/zh/)

*   [文档](/zh/docs/)
*   [Kubernetes 博客](/zh/blog/)
*   [培训](/zh/training/)
*   [合作伙伴](/zh/partners/)
*   [社区](/zh/community/)
*   [案例分析](/zh/case-studies/)
*   [版本列表](#)

    [v1.20](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/) [v1.19](https://v1-19.docs.kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/) [v1.18](https://v1-18.docs.kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/) [v1.17](https://v1-17.docs.kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/) [v1.16](https://v1-16.docs.kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)

*   [中文 Chinese](#)

    [English](/docs/concepts/workloads/controllers/deployment/) [한국어 Korean](/ko/docs/concepts/workloads/controllers/deployment/) [日本語 Japanese](/ja/docs/concepts/workloads/controllers/deployment/) [Français](/fr/docs/concepts/workloads/controllers/deployment/) [Español](/es/docs/concepts/workloads/controllers/deployment/) [Bahasa Indonesia](/id/docs/concepts/workloads/controllers/deployment/)


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

    [测试页面 (中文版)](/zh/docs/test/)

1.  [Kubernetes 文档](https://kubernetes.io/zh/docs/)
2.  [概念](https://kubernetes.io/zh/docs/concepts/)
3.  [工作负载](https://kubernetes.io/zh/docs/concepts/workloads/)
4.  [工作负载资源](https://kubernetes.io/zh/docs/concepts/workloads/controllers/)
5.  [Deployments](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)

# Deployments

一个 _Deployment_ 为 [Pods](/docs/concepts/workloads/pods/pod-overview/ "Pod 表示您的集群上一组正在运行的容器.") 和 [ReplicaSets](/zh/docs/concepts/workloads/controllers/replicaset/ "ReplicaSet 是下一代副本控制器.") 提供声明式的更新能力.

你负责描述 Deployment 中的 _目标状态_, 而 Deployment [控制器 (Controller)](/zh/docs/concepts/architecture/controller/ " 控制器通过 apiserver 监控集群的公共状态, 并致力于将当前状态转变为期望的状态.") 以受控速率更改实际状态, 使其变为期望状态. 你可以定义 Deployment 以创建新的 ReplicaSet, 或删除现有 Deployment, 并通过新的 Deployment 收养其资源.

> **说明:** 不要管理 Deployment 所拥有的 ReplicaSet . 如果存在下面未覆盖的使用场景, 请考虑在 Kubernetes 仓库中提出 Issue.

## 用例

以下是 Deployments 的典型用例:

*   [创建 Deployment 以将 ReplicaSet 上线](#creating-a-deployment). ReplicaSet 在后台创建 Pods. 检查 ReplicaSet 的上线状态, 查看其是否成功.
*   通过更新 Deployment 的 PodTemplateSpec, [声明 Pod 的新状态](#updating-a-deployment) . 新的 ReplicaSet 会被创建, Deployment 以受控速率将 Pod 从旧 ReplicaSet 迁移到新 ReplicaSet. 每个新的 ReplicaSet 都会更新 Deployment 的修订版本.
*   如果 Deployment 的当前状态不稳定, [回滚到较早的 Deployment 版本](#rolling-back-a-deployment). 每次回滚都会更新 Deployment 的修订版本.
*   [扩大 Deployment 规模以承担更多负载](#scaling-a-deployment).
*   [暂停 Deployment](#pausing-and-resuming-a-deployment) 以应用对 PodTemplateSpec 所作的多项修改, 然后恢复其执行以启动新的上线版本.
*   [使用 Deployment 状态](#deployment-status) 来判定上线过程是否出现停滞.
*   [清理较旧的不再需要的 ReplicaSet](#clean-up-policy) .

## 创建 Deployment

下面是 Deployment 示例. 其中创建了一个 ReplicaSet, 负责启动三个 `nginx` Pods:

[`controllers/nginx-deployment.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/zh/examples/controllers/nginx-deployment.yaml) ! [](https://d33wubrfki0l68.cloudfront.net/0901162ab78eb4ff2e9e5dc8b17c3824befc91a6/44ccd/images/copycode.svg "Copy controllers/nginx-deployment.yaml to clipboard")

```
apiVersion: apps/v1 kind: Deployment metadata:  name: nginx-deployment  labels:  app: nginx spec:  replicas: 3  selector:  matchLabels:  app: nginx  template:  metadata:  labels:  app: nginx  spec:  containers:  - name: nginx  image: nginx:1.14.2  ports:  - containerPort: 80
```

在该例中:

*   创建名为 `nginx-deployment`(由 `.metadata.name` 字段标明) 的 Deployment.
*   该 Deployment 创建三个 (由 `replicas` 字段标明) Pod 副本.

*   `selector` 字段定义 Deployment 如何查找要管理的 Pods. 在这里, 你只需选择在 Pod 模板中定义的标签 (`app: nginx`). 不过, 更复杂的选择规则是也可能的, 只要 Pod 模板本身满足所给规则即可.

    > **说明:** `matchLabels` 字段是 `{key, value}` 偶对的映射. 在 `matchLabels` 映射中的单个 `{key, value}` 映射等效于 `matchExpressions` 中的一个元素, 即其 `key` 字段是 "key", operator 为 "In",`value` 数组仅包含 "value". 在 `matchLabels` 和 `matchExpressions` 中给出的所有条件都必须满足才能匹配.


*   `template` 字段包含以下子字段:
    *   Pod 被使用 `labels` 字段打上 `app: nginx` 标签.
    *   Pod 模板规约 (即 `.template.spec` 字段) 指示 Pods 运行一个 `nginx` 容器, 该容器运行版本为 1.14.2 的 `nginx` [Docker Hub](https://hub.docker.com/) 镜像.
    *   创建一个容器并使用 `name` 字段将其命名为 `nginx`.

开始之前, 请确保的 Kubernetes 集群已启动并运行. 按照以下步骤创建上述 Deployment :

1.  通过运行以下命令创建 Deployment :

    ```
    kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
    ```

    > **说明:** 你可以设置 `--record` 标志将所执行的命令写入资源注解 `kubernetes.io/change-cause` 中. 这对于以后的检查是有用的. 例如, 要查看针对每个 Deployment 修订版本所执行过的命令.


2.  运行 `kubectl get deployments` 检查 Deployment 是否已创建. 如果仍在创建 Deployment, 则输出类似于:

    ```
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3         0         0            0           1s
    ```

    在检查集群中的 Deployment 时, 所显示的字段有:

    *   `NAME` 列出了集群中 Deployment 的名称.
    *   `READY` 显示应用程序的可用的 _副本_ 数. 显示的模式是 " 就绪个数 / 期望个数 ".
    *   `UP-TO-DATE` 显示为了打到期望状态已经更新的副本数.
    *   `AVAILABLE` 显示应用可供用户使用的副本数.
    *   `AGE` 显示应用程序运行的时间.

    请注意期望副本数是根据 `.spec.replicas` 字段设置 3.


3.  要查看 Deployment 上线状态, 运行 `kubectl rollout status deployment/nginx-deployment`.

    输出类似于:

    ```
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    deployment "nginx-deployment" successfully rolled out
    ```

4.  几秒钟后再次运行 `kubectl get deployments`. 输出类似于:

    ```
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3         3         3            3           18s
    ```

    注意 Deployment 已创建全部三个副本, 并且所有副本都是最新的 (它们包含最新的 Pod 模板) 并且可用.


5.  要查看 Deployment 创建的 ReplicaSet(`rs`), 运行 `kubectl get rs`. 输出类似于:

    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-75675f5897   3         3         3       18s
    ```

    ReplicaSet 输出中包含以下字段:

    *   `NAME` 列出名字空间中 ReplicaSet 的名称;
    *   `DESIRED` 显示应用的期望副本个数, 即在创建 Deployment 时所定义的值. 此为期望状态;
    *   `CURRENT` 显示当前运行状态中的副本个数;
    *   `READY` 显示应用中有多少副本可以为用户提供服务;
    *   `AGE` 显示应用已经运行的时间长度.

    注意 ReplicaSet 的名称始终被格式化为 `[Deployment 名称]-[随机字符串]`. 其中的随机字符串是使用 pod-template-hash 作为种子随机生成的.


6.  要查看每个 Pod 自动生成的标签, 运行 `kubectl get pods --show-labels`. 返回以下输出:

    ```
    NAME                                READY     STATUS    RESTARTS   AGE       LABELS
    nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app= nginx, pod-template-hash=3123191453
    nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app= nginx, pod-template-hash=3123191453
    nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app= nginx, pod-template-hash=3123191453
    ```

    所创建的 ReplicaSet 确保总是存在三个 `nginx` Pod.


> **说明:** 你必须在 Deployment 中指定适当的选择算符和 Pod 模板标签 (在本例中为 `app: nginx`). 标签或者选择算符不要与其他控制器 (包括其他 Deployment 和 StatefulSet) 重叠. Kubernetes 不会阻止你这样做, 但是如果多个控制器具有重叠的选择算符, 它们可能会发生冲突 执行难以预料的操作.

### Pod-template-hash 标签

> **说明:** 不要更改此标签.

Deployment 控制器将 `pod-template-hash` 标签添加到 Deployment 所创建或收留的 每个 ReplicaSet .

此标签可确保 Deployment 的子 ReplicaSets 不重叠. 标签是通过对 ReplicaSet 的 `PodTemplate` 进行哈希处理. 所生成的哈希值被添加到 ReplicaSet 选择算符, Pod 模板标签, 并存在于在 ReplicaSet 可能拥有的任何现有 Pod 中.

## 更新 Deployment

> **说明:** 仅当 Deployment Pod 模板 (即 `.spec.template`) 发生改变时, 例如模板的标签或容器镜像被更新, 才会触发 Deployment 上线. 其他更新 (如对 Deployment 执行扩缩容的操作) 不会触发上线动作.

按照以下步骤更新 Deployment:

1.  先来更新 nginx Pod 以使用 `nginx:1.16.1` 镜像, 而不是 `nginx:1.14.2` 镜像.

    ```
    kubectl --record deployment.apps/nginx-deployment set image \    deployment.v1.apps/nginx-deployment nginx= nginx:1.16.1
    ```

    或者使用下面的命令:

    ```
    kubectl set image deployment/nginx-deployment nginx= nginx:1.16.1 --record
    ```

    输出类似于:

    ```
    deployment.apps/nginx-deployment image updated
    ```

    或者, 可以 `edit` Deployment 并将 `.spec.template.spec.containers[0].image` 从 `nginx:1.14.2` 更改至 `nginx:1.16.1`.

    ```
    kubectl edit deployment.v1.apps/nginx-deployment
    ```

    输出类似于:

    ```
    deployment.apps/nginx-deployment edited
    ```

2.  要查看上线状态, 运行:

    ```
    kubectl rollout status deployment/nginx-deployment
    ```

    输出类似于:

    ```
    Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
    ```

    或者

    ```
    deployment "nginx-deployment" successfully rolled out
    ```

获取关于已更新的 Deployment 的更多信息:

*   在上线成功后, 可以通过运行 `kubectl get deployments` 来查看 Deployment: 输出类似于:

    ```
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3         3         3            3           36s
    ```


*   运行 `kubectl get rs` 以查看 Deployment 通过创建新的 ReplicaSet 并将其扩容到 3 个副本并将旧 ReplicaSet 缩容到 0 个副本完成了 Pod 的更新操作:

    ```
    kubectl get rs
    ```

    输出类似于:

    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1564180365   3         3         3       6s
    nginx-deployment-2035384211   0         0         0       36s
    ```

*   现在运行 `get pods` 应仅显示新的 Pods:

    ```
    kubectl get pods
    ```

    输出类似于:

    ```
    NAME                                READY     STATUS    RESTARTS   AGE
    nginx-deployment-1564180365-khku8   1/1       Running   0          14s
    nginx-deployment-1564180365-nacti   1/1       Running   0          14s
    nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
    ```

    下次要更新这些 Pods 时, 只需再次更新 Deployment Pod 模板即可.

    Deployment 可确保在更新时仅关闭一定数量的 Pod. 默认情况下, 它确保至少所需 Pods 75% 处于运行状态 (最大不可用比例为 25%).

    Deployment 还确保仅所创建 Pod 数量只可能比期望 Pods 数高一点点. 默认情况下, 它可确保启动的 Pod 个数比期望个数最多多出 25%(最大峰值 25%).

    例如, 如果仔细查看上述 Deployment , 将看到它首先创建了一个新的 Pod, 然后删除了一些旧的 Pods, 并创建了新的 Pods. 它不会杀死老 Pods, 直到有足够的数量新的 Pods 已经出现. 在足够数量的旧 Pods 被杀死前并没有创建新 Pods. 它确保至少 2 个 Pod 可用, 同时 最多总共 4 个 Pod 可用.


*   获取 Deployment 的更多信息

    ```
    kubectl describe deployments
    ```

    输出类似于:

    ```
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
    Labels:                 app= nginx
    Annotations:            deployment.kubernetes.io/revision=2
    Selector:               app= nginx
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app= nginx
       Containers:
        nginx:
          Image:        nginx:1.16.1
          Port:         80/TCP
          Environment:  <none>
          Mounts:       <none>
        Volumes:        <none>
      Conditions:
        Type           Status  Reason
        ----           ------  ------
        Available      True    MinimumReplicasAvailable
        Progressing    True    NewReplicaSetAvailable
      OldReplicaSets:  <none>
      NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
      Events:
        Type    Reason             Age   From                   Message
        ----    ------             ----  ----                   -------
        Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
        Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
        Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
        Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
        Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
        Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
        Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
    ```

    可以看到, 当第一次创建 Deployment 时, 它创建了一个 ReplicaSet(nginx-deployment-2035384211) 并将其直接扩容至 3 个副本. 更新 Deployment 时, 它创建了一个新的 ReplicaSet (nginx-deployment-1564180365), 并将其扩容为 1, 然后将旧 ReplicaSet 缩容到 2, 以便至少有 2 个 Pod 可用且最多创建 4 个 Pod. 然后, 它使用相同的滚动更新策略继续对新的 ReplicaSet 扩容并对旧的 ReplicaSet 缩容. 最后, 你将有 3 个可用的副本在新的 ReplicaSet 中, 旧 ReplicaSet 将缩容到 0.


### 翻转 (多 Deployment 动态更新)

Deployment 控制器每次注意到新的 Deployment 时, 都会创建一个 ReplicaSet 以启动所需的 Pods. 如果更新了 Deployment, 则控制标签匹配 `.spec.selector` 但模板不匹配 `.spec.template` 的 Pods 的现有 ReplicaSet 被缩容. 最终, 新的 ReplicaSet 缩放为 `.spec.replicas` 个副本, 所有旧 ReplicaSets 缩放为 0 个副本.

当 Deployment 正在上线时被更新, Deployment 会针对更新创建一个新的 ReplicaSet 并开始对其扩容, 之前正在被扩容的 ReplicaSet 会被翻转, 添加到旧 ReplicaSets 列表 并开始缩容.

例如, 假定你在创建一个 Deployment 以生成 `nginx:1.14.2` 的 5 个副本, 但接下来 更新 Deployment 以创建 5 个 `nginx:1.16.1` 的副本, 而此时只有 3 个 `nginx:1.14.2` 副本已创建. 在这种情况下, Deployment 会立即开始杀死 3 个 `nginx:1.14.2` Pods, 并开始创建 `nginx:1.16.1` Pods. 它不会等待 `nginx:1.14.2` 的 5 个副本都创建完成 后才开始执行变更动作.

### 更改标签选择算符

通常不鼓励更新标签选择算符. 建议你提前规划选择算符. 在任何情况下, 如果需要更新标签选择算符, 请格外小心, 并确保自己了解 这背后可能发生的所有事情.

> **说明:** 在 API 版本 `apps/v1` 中, Deployment 标签选择算符在创建后是不可变的.

*   添加选择算符时要求使用新标签更新 Deployment 规约中的 Pod 模板标签, 否则将返回验证错误. 此更改是非重叠的, 也就是说新的选择算符不会选择使用旧选择算符所创建的 ReplicaSet 和 Pod, 这会导致创建新的 ReplicaSet 时所有旧 ReplicaSet 都会被孤立.
*   选择算符的更新如果更改了某个算符的键名, 这会导致与添加算符时相同的行为.
*   删除选择算符的操作会删除从 Deployment 选择算符中删除现有算符. 此操作不需要更改 Pod 模板标签. 现有 ReplicaSet 不会被孤立, 也不会因此创建新的 ReplicaSet, 但请注意已删除的标签仍然存在于现有的 Pod 和 ReplicaSet 中.

## 回滚 Deployment

有时, 你可能想要回滚 Deployment; 例如, 当 Deployment 不稳定时 (例如进入反复崩溃状态). 默认情况下, Deployment 的所有上线记录都保留在系统中, 以便可以随时回滚 (你可以通过修改修订历史记录限制来更改这一约束).

> **说明:** Deployment 被触发上线时, 系统就会创建 Deployment 的新的修订版本. 这意味着仅当 Deployment 的 Pod 模板 (`.spec.template`) 发生更改时, 才会创建新修订版本 -- 例如, 模板的标签或容器镜像发生变化. 其他更新, 如 Deployment 的扩缩容操作不会创建 Deployment 修订版本. 这是为了方便同时执行手动缩放或自动缩放. 换言之, 当你回滚到较早的修订版本时, 只有 Deployment 的 Pod 模板部分会被回滚.

*   假设你在更新 Deployment 时犯了一个拼写错误, 将镜像名称命名设置为 `nginx:1.161` 而不是 `nginx:1.16.1`:

    ```
    kubectl set image deployment.v1.apps/nginx-deployment nginx= nginx:1.161 --record= true
    ```

    输出类似于:

    ```
    deployment.apps/nginx-deployment image updated
    ```


*   此上线进程会出现停滞. 你可以通过检查上线状态来验证:

    ```
    kubectl rollout status deployment/nginx-deployment
    ```

    输出类似于:

    ```
    Waiting for rollout to finish: 1 out of 3 new replicas have been updated...
    ```

*   按 Ctrl-C 停止上述上线状态观测. 有关上线停滞的详细信息, [参考这里](#deployment-status).

*   你可以看到旧的副本有两个 (`nginx-deployment-1564180365` 和 `nginx-deployment-2035384211`), 新的副本有 1 个 (`nginx-deployment-3066724191`):

    ```
    kubectl get rs
    ```

    输出类似于:

    ```
    NAME                          DESIRED   CURRENT   READY   AGE
    nginx-deployment-1564180365   3         3         3       25s
    nginx-deployment-2035384211   0         0         0       36s
    nginx-deployment-3066724191   1         1         0       6s
    ```


*   查看所创建的 Pod, 你会注意到新 ReplicaSet 所创建的 1 个 Pod 卡顿在镜像拉取循环中.

    ```
    kubectl get pods
    ```

    输出类似于:

    ```
    NAME                                READY     STATUS             RESTARTS   AGE
    nginx-deployment-1564180365-70iae   1/1       Running            0          25s
    nginx-deployment-1564180365-jbqqo   1/1       Running            0          25s
    nginx-deployment-1564180365-hysrc   1/1       Running            0          25s
    nginx-deployment-3066724191-08mng   0/1       ImagePullBackOff   0          6s
    ```

    > **说明:** Deployment 控制器自动停止有问题的上线过程, 并停止对新的 ReplicaSet 扩容. 这行为取决于所指定的 rollingUpdate 参数 (具体为 `maxUnavailable`). 默认情况下, Kubernetes 将此值设置为 25%.


*   获取 Deployment 描述信息:

    ```
    kubectl describe deployment
    ```

    输出类似于:

    ```
    Name:           nginx-deployment
    Namespace:      default
    CreationTimestamp:  Tue, 15 Mar 2016 14:48:04 -0700
    Labels:         app= nginx
    Selector:       app= nginx
    Replicas:       3 desired | 1 updated | 4 total | 3 available | 1 unavailable
    StrategyType:       RollingUpdate
    MinReadySeconds:    0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app= nginx
      Containers:
       nginx:
        Image:        nginx:1.91
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    ReplicaSetUpdated
    OldReplicaSets:     nginx-deployment-1564180365 (3/3 replicas created)
    NewReplicaSet:      nginx-deployment-3066724191 (1/1 replicas created)
    Events:
      FirstSeen LastSeen    Count   From                    SubobjectPath   Type        Reason              Message
      --------- --------    -----   ----                    -------------   --------    ------              -------
      1m        1m          1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-2035384211 to 3
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 1
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 2
      22s       22s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 2
      21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 1
      21s       21s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-1564180365 to 3
      13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled down replica set nginx-deployment-2035384211 to 0
      13s       13s         1       {deployment-controller }                Normal      ScalingReplicaSet   Scaled up replica set nginx-deployment-3066724191 to 1
    ```

    要解决此问题, 需要回滚到以前稳定的 Deployment 版本.


### 检查 Deployment 上线历史

按照如下步骤检查回滚历史:

1.  首先, 检查 Deployment 修订历史:

    ```
    kubectl rollout history deployment.v1.apps/nginx-deployment
    ```

    输出类似于:

    ```
    deployments "nginx-deployment"
    REVISION    CHANGE-CAUSE
    1           kubectl apply --filename= https://k8s.io/examples/controllers/nginx-deployment.yaml --record= true
    2           kubectl set image deployment.v1.apps/nginx-deployment nginx= nginx:1.9.1 --record= true
    3           kubectl set image deployment.v1.apps/nginx-deployment nginx= nginx:1.91 --record= true
    ```

    `CHANGE-CAUSE` 的内容是从 Deployment 的 `kubernetes.io/change-cause` 注解复制过来的. 复制动作发生在修订版本创建时. 你可以通过以下方式设置 `CHANGE-CAUSE` 消息:

    *   使用 `kubectl annotate deployment.v1.apps/nginx-deployment kubernetes.io/change-cause="image updated to 1.9.1"` 为 Deployment 添加注解.
    *   追加 `--record` 命令行标志以保存正在更改资源的 `kubectl` 命令.
    *   手动编辑资源的清单.

2.  要查看修订历史的详细信息, 运行:

    ```
    kubectl rollout history deployment.v1.apps/nginx-deployment --revision=2
    ```

    输出类似于:

    ```
    deployments "nginx-deployment" revision 2
      Labels:       app= nginx
              pod-template-hash=1159050644
      Annotations:  kubernetes.io/change-cause= kubectl set image deployment.v1.apps/nginx-deployment nginx= nginx:1.16.1 --record= true
      Containers:
       nginx:
        Image:      nginx:1.16.1
        Port:       80/TCP
         QoS Tier:
            cpu:      BestEffort
            memory:   BestEffort
        Environment Variables:      <none>
      No volumes.
    ```


### 回滚到之前的修订版本

按照下面给出的步骤将 Deployment 从当前版本回滚到以前的版本 (即版本 2).

1.  假定现在你已决定撤消当前上线并回滚到以前的修订版本:

    ```
    kubectl rollout undo deployment.v1.apps/nginx-deployment
    ```

    输出类似于:

    ```
    deployment.apps/nginx-deployment
    ```

    或者, 你也可以通过使用 `--to-revision` 来回滚到特定修订版本:

    ```
    kubectl rollout undo deployment.v1.apps/nginx-deployment --to-revision=2
    ```

    输出类似于:

    ```
    deployment.apps/nginx-deployment
    ```

    与回滚相关的指令的更详细信息, 请参考 [`kubectl rollout`](/docs/reference/generated/kubectl/kubectl-commands#rollout).

    现在, Deployment 正在回滚到以前的稳定版本. 正如你所看到的, Deployment 控制器生成了 回滚到修订版本 2 的 `DeploymentRollback` 事件.


2.  检查回滚是否成功以及 Deployment 是否正在运行, 运行:

    ```
    kubectl get deployment nginx-deployment
    ```

    输出类似于:

    ```
    NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment   3         3         3            3           30m
    ```


3.  获取 Deployment 描述信息:

    ```
    kubectl describe deployment nginx-deployment
    ```

    输出类似于:

    ```
    Name:                   nginx-deployment
    Namespace:              default
    CreationTimestamp:      Sun, 02 Sep 2018 18:17:55 -0500
    Labels:                 app= nginx
    Annotations:            deployment.kubernetes.io/revision=4
                            kubernetes.io/change-cause= kubectl set image deployment.v1.apps/nginx-deployment nginx= nginx:1.16.1 --record= true
    Selector:               app= nginx
    Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
    StrategyType:           RollingUpdate
    MinReadySeconds:        0
    RollingUpdateStrategy:  25% max unavailable, 25% max surge
    Pod Template:
      Labels:  app= nginx
      Containers:
       nginx:
        Image:        nginx:1.16.1
        Port:         80/TCP
        Host Port:    0/TCP
        Environment:  <none>
        Mounts:       <none>
      Volumes:        <none>
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   nginx-deployment-c4747d96c (3/3 replicas created)
    Events:
      Type    Reason              Age   From                   Message
      ----    ------              ----  ----                   -------
      Normal  ScalingReplicaSet   12m   deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 3
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 1
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 2
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 2
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 1
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-c4747d96c to 3
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled down replica set nginx-deployment-75675f5897 to 0
      Normal  ScalingReplicaSet   11m   deployment-controller  Scaled up replica set nginx-deployment-595696685f to 1
      Normal  DeploymentRollback  15s   deployment-controller  Rolled back deployment "nginx-deployment" to revision 2
      Normal  ScalingReplicaSet   15s   deployment-controller  Scaled down replica set nginx-deployment-595696685f to 0
    ```

## 缩放 Deployment

你可以使用如下指令缩放 Deployment:

```
kubectl scale deployment.v1.apps/nginx-deployment --replicas=10
```

输出类似于:

```
deployment.apps/nginx-deployment scaled
```

假设集群启用了 [Pod 的水平自动缩放](/zh/docs/tasks/run-application/horizontal-pod-autoscale-walkthrough/), 你可以为 Deployment 设置自动缩放器, 并基于现有 Pods 的 CPU 利用率选择 要运行的 Pods 个数下限和上限.

```
kubectl autoscale deployment.v1.apps/nginx-deployment --min=10 --max=15 --cpu-percent=80
```

输出类似于:

```
deployment.apps/nginx-deployment scaled
```

### 比例缩放

RollingUpdate 的 Deployment 支持同时运行应用程序的多个版本. 当自动缩放器缩放处于上线进程 (仍在进行中或暂停) 中的 RollingUpdate Deployment 时, Deployment 控制器会平衡现有的活跃状态的 ReplicaSets(含 Pods 的 ReplicaSets) 中的额外副本, 以降低风险. 这称为 _比例缩放 (Proportional Scaling)_.

例如, 你正在运行一个 10 个副本的 Deployment, 其 [maxSurge](#max-surge)\=3, [maxUnavailable](#max-unavailable)\=2.

*   确保 Deployment 的这 10 个副本都在运行.

    ```
    kubectl get deploy
    ```

    输出类似于:

    ```
    NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx-deployment     10        10        10           10          50s
    ```

*   更新 Deployment 使用新镜像, 碰巧该镜像无法从集群内部解析.

    ```
    kubectl set image deployment.v1.apps/nginx-deployment nginx= nginx: sometag
    ```

    输出类似于:

    ```
    deployment.apps/nginx-deployment image updated
    ```

*   镜像更新使用 ReplicaSet `nginx-deployment-1989198191` 启动新的上线过程, 但由于上面提到的 `maxUnavailable` 要求, 该进程被阻塞了. 检查上线状态:

    ```
    kubectl get rs
    ```

    输出类似于:

    ```
    NAME                          DESIRED   CURRENT   READY     AGE
    nginx-deployment-1989198191   5         5         0         9s
    nginx-deployment-618515232    8         8         8         1m
    ```

*   然后, 出现了新的 Deployment 扩缩请求. 自动缩放器将 Deployment 副本增加到 15. Deployment 控制器需要决定在何处添加 5 个新副本. 如果未使用比例缩放, 所有 5 个副本 都将添加到新的 ReplicaSet 中. 使用比例缩放时, 可以将额外的副本分布到所有 ReplicaSet. 较大比例的副本会被添加到拥有最多副本的 ReplicaSet, 而较低比例的副本会进入到 副本较少的 ReplicaSet. 所有剩下的副本都会添加到副本最多的 ReplicaSet. 具有零副本的 ReplicaSets 不会被扩容.

在上面的示例中,3 个副本被添加到旧 ReplicaSet 中,2 个副本被添加到新 ReplicaSet. 假定新的副本都很健康, 上线过程最终应将所有副本迁移到新的 ReplicaSet 中. 要确认这一点, 请运行:

```
kubectl get deploy
```

输出类似于:

```
NAME                 DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment     15        18        7            8           7m
```

上线状态确认了副本是如何被添加到每个 ReplicaSet 的.

```
kubectl get rs
```

输出类似于:

```
NAME                          DESIRED   CURRENT   READY     AGE
nginx-deployment-1989198191   7         7         0         7m
nginx-deployment-618515232    11        11        11        7m
```

## 暂停, 恢复 Deployment

你可以在触发一个或多个更新之前暂停 Deployment, 然后再恢复其执行. 这样做使得你能够在暂停和恢复执行之间应用多个修补程序, 而不会触发不必要的上线操作.

*   例如, 对于一个刚刚创建的 Deployment: 获取 Deployment 信息:

    ```
    kubectl get deploy
    ```

    输出类似于:

    ```
    NAME      DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
    nginx     3         3         3            3           1m
    ```

    获取上线状态:

    ```
    kubectl get rs
    ```

    输出类似于:

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         1m
    ```


*   使用如下指令暂停运行:

    ```
    kubectl rollout pause deployment.v1.apps/nginx-deployment
    ```

    输出类似于:

    ```
    deployment.apps/nginx-deployment paused
    ```


*   接下来更新 Deployment 镜像:

    ```
    kubectl set image deployment.v1.apps/nginx-deployment nginx= nginx:1.16.1
    ```

    输出类似于:

    ```
    deployment.apps/nginx-deployment image updated
    ```

*   注意没有新的上线被触发:

    ```
    kubectl rollout history deployment.v1.apps/nginx-deployment
    ```

    输出类似于:

    ```
    deployments "nginx"
    REVISION  CHANGE-CAUSE
    1   <none>
    ```


*   获取上线状态确保 Deployment 更新已经成功:

    ```
    kubectl get rs
    ```

    输出类似于:

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   3         3         3         2m
    ```


*   你可以根据需要执行很多更新操作, 例如, 可以要使用的资源:

    ```
    kubectl set resources deployment.v1.apps/nginx-deployment -c= nginx --limits= cpu=200m, memory=512Mi
    ```

    输出类似于:

    ```
    deployment.apps/nginx-deployment resource requirements updated
    ```

    暂停 Deployment 之前的初始状态将继续发挥作用, 但新的更新在 Deployment 被 暂停期间不会产生任何效果.


*   最终, 恢复 Deployment 执行并观察新的 ReplicaSet 的创建过程, 其中包含了所应用的所有更新:

    ```
    kubectl rollout resume deployment.v1.apps/nginx-deployment
    ```

    输出:

    ```
    deployment.apps/nginx-deployment resumed
    ```


*   观察上线的状态, 直到完成.

    ```
    kubectl get rs -w
    ```

    输出类似于:

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   2         2         2         2m
    nginx-3926361531   2         2         0         6s
    nginx-3926361531   2         2         1         18s
    nginx-2142116321   1         2         2         2m
    nginx-2142116321   1         2         2         2m
    nginx-3926361531   3         2         1         18s
    nginx-3926361531   3         2         1         18s
    nginx-2142116321   1         1         1         2m
    nginx-3926361531   3         3         1         18s
    nginx-3926361531   3         3         2         19s
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         1         1         2m
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         20s
    ```

*   获取最近上线的状态:

    ```
    kubectl get rs
    ```

    输出类似于:

    ```
    NAME               DESIRED   CURRENT   READY     AGE
    nginx-2142116321   0         0         0         2m
    nginx-3926361531   3         3         3         28s
    ```

> **说明:** 你不可以回滚处于暂停状态的 Deployment, 除非先恢复其执行状态.

## Deployment 状态

Deployment 的生命周期中会有许多状态. 上线新的 ReplicaSet 期间可能处于 [Progressing(进行中)](#progressing-deployment), 可能是 [Complete(已完成)](#complete-deployment), 也可能是 [Failed(失败)](#failed-deployment) 以至于无法继续进行.

### 进行中的 Deployment

执行下面的任务期间, Kubernetes 标记 Deployment 为 _进行中 (Progressing)_:

*   Deployment 创建新的 ReplicaSet
*   Deployment 正在为其最新的 ReplicaSet 扩容
*   Deployment 正在为其旧有的 ReplicaSet(s) 缩容
*   新的 Pods 已经就绪或者可用 (就绪至少持续了 [MinReadySeconds](#min-ready-seconds) 秒).

你可以使用 `kubectl rollout status` 监视 Deployment 的进度.

### 完成的 Deployment

当 Deployment 具有以下特征时, Kubernetes 将其标记为 _完成 (Complete)_:

*   与 Deployment 关联的所有副本都已更新到指定的最新版本, 这意味着之前请求的所有更新都已完成.
*   与 Deployment 关联的所有副本都可用.
*   未运行 Deployment 的旧副本.

你可以使用 `kubectl rollout status` 检查 Deployment 是否已完成. 如果上线成功完成,`kubectl rollout status` 返回退出代码 0.

```
kubectl rollout status deployment/nginx-deployment
```

输出类似于:

```
Waiting for rollout to finish: 2 of 3 updated replicas are available...
deployment "nginx-deployment" successfully rolled out
$ echo $?
0
```

### 失败的 Deployment

你的 Deployment 可能会在尝试部署其最新的 ReplicaSet 受挫, 一直处于未完成状态. 造成此情况一些可能因素如下:

*   配额 (Quota) 不足
*   就绪探测 (Readiness Probe) 失败
*   镜像拉取错误
*   权限不足
*   限制范围 (Limit Ranges) 问题
*   应用程序运行时的配置错误

检测此状况的一种方法是在 Deployment 规约中指定截止时间参数: (\[`.spec.progressDeadlineSeconds`\](#progress-deadline-seconds)). `.spec.progressDeadlineSeconds` 给出的是一个秒数值, Deployment 控制器在 (通过 Deployment 状态) 标示 Deployment 进展停滞之前, 需要等待所给的时长.

以下 `kubectl` 命令设置规约中的 `progressDeadlineSeconds`, 从而告知控制器 在 10 分钟后报告 Deployment 没有进展:

```
kubectl patch deployment.v1.apps/nginx-deployment -p '{"spec": {"progressDeadlineSeconds":600}}'
```

输出类似于:

```
deployment.apps/nginx-deployment patched
```

超过截止时间后, Deployment 控制器将添加具有以下属性的 DeploymentCondition 到 Deployment 的 `.status.conditions` 中:

*   Type= Progressing
*   Status= False
*   Reason= ProgressDeadlineExceeded

参考 [Kubernetes API 约定](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#typical-status-properties) 获取更多状态状况相关的信息.

> **说明:**
>
> 除了报告 `Reason= ProgressDeadlineExceeded` 状态之外, Kubernetes 对已停止的 Deployment 不执行任何操作. 更高级别的编排器可以利用这一设计并相应地采取行动. 例如, 将 Deployment 回滚到其以前的版本.
>
> 如果你暂停了某个 Deployment, Kubernetes 不再根据指定的截止时间检查 Deployment 进展. 你可以在上线过程中间安全地暂停 Deployment 再恢复其执行, 这样做不会导致超出最后时限的问题.

Deployment 可能会出现瞬时性的错误, 可能因为设置的超时时间过短, 也可能因为其他可认为是临时性的问题. 例如, 假定所遇到的问题是配额不足. 如果描述 Deployment, 你将会注意到以下部分:

```
kubectl describe deployment nginx-deployment
```

输出类似于:

```
<...>
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     True    ReplicaSetUpdated
  ReplicaFailure  True    FailedCreate
<...>
```

如果运行 `kubectl get deployment nginx-deployment -o yaml`, Deployment 状态输出 将类似于这样:

```
status:
  availableReplicas: 2
  conditions:
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: Replica set "nginx-deployment-4262182780" is progressing.
    reason: ReplicaSetUpdated
    status: "True"
    type: Progressing
  - lastTransitionTime: 2016-10-04T12:25:42Z
    lastUpdateTime: 2016-10-04T12:25:42Z
    message: Deployment has minimum availability.
    reason: MinimumReplicasAvailable
    status: "True"
    type: Available
  - lastTransitionTime: 2016-10-04T12:25:39Z
    lastUpdateTime: 2016-10-04T12:25:39Z
    message: 'Error creating: pods "nginx-deployment-4262182780-" is forbidden: exceeded quota:
      object-counts, requested: pods=1, used: pods=3, limited: pods=2'
    reason: FailedCreate
    status: "True"
    type: ReplicaFailure
  observedGeneration: 3
  replicas: 2
  unavailableReplicas: 2
```

最终, 一旦超过 Deployment 进度限期, Kubernetes 将更新状态和进度状况的原因:

```
Conditions:
  Type            Status  Reason
  ----            ------  ------
  Available       True    MinimumReplicasAvailable
  Progressing     False   ProgressDeadlineExceeded
  ReplicaFailure  True    FailedCreate
```

可以通过缩容 Deployment 或者缩容其他运行状态的控制器, 或者直接在命名空间中增加配额 来解决配额不足的问题. 如果配额条件满足, Deployment 控制器完成了 Deployment 上线操作, Deployment 状态会更新为成功状况 (`Status= True` and `Reason= NewReplicaSetAvailable`).

```
Conditions:
  Type          Status  Reason
  ----          ------  ------
  Available     True    MinimumReplicasAvailable
  Progressing   True    NewReplicaSetAvailable
```

`Type= Available` 加上 `Status= True` 意味着 Deployment 具有最低可用性. 最低可用性由 Deployment 策略中的参数指定. `Type= Progressing` 加上 `Status= True` 表示 Deployment 处于上线过程中, 并且正在运行, 或者已成功完成进度, 最小所需新副本处于可用. 请参阅对应状况的 Reason 了解相关细节. 在我们的案例中 `Reason= NewReplicaSetAvailable` 表示 Deployment 已完成.

你可以使用 `kubectl rollout status` 检查 Deployment 是否未能取得进展. 如果 Deployment 已超过进度限期,`kubectl rollout status` 返回非零退出代码.

```
kubectl rollout status deployment/nginx-deployment
```

输出类似于:

```
Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
error: deployment "nginx" exceeded its progress deadline
```

`kubectl rollout` 命令的退出状态为 1(表明发生了错误):

```
$ echo $?
```

```
1
```

### 对失败 Deployment 的操作

可应用于已完成的 Deployment 的所有操作也适用于失败的 Deployment. 你可以对其执行扩缩容, 回滚到以前的修订版本等操作, 或者在需要对 Deployment 的 Pod 模板应用多项调整时, 将 Deployment 暂停.

## 清理策略

你可以在 Deployment 中设置 `.spec.revisionHistoryLimit` 字段以指定保留此 Deployment 的多少个旧有 ReplicaSet. 其余的 ReplicaSet 将在后台被垃圾回收. 默认情况下, 此值为 10.

> **说明:** 显式将此字段设置为 0 将导致 Deployment 的所有历史记录被清空, 因此 Deployment 将无法回滚.

## 金丝雀部署

如果要使用 Deployment 向用户子集或服务器子集上线版本, 则可以遵循 [资源管理](/zh/docs/concepts/cluster-administration/manage-deployment/#canary-deployments) 所描述的金丝雀模式, 创建多个 Deployment, 每个版本一个.

## 编写 Deployment 规约

同其他 Kubernetes 配置一样, Deployment 需要 `apiVersion`,`kind` 和 `metadata` 字段. 有关配置文件的其他信息, 请参考 [部署 Deployment](/zh/docs/tasks/run-application/run-stateless-application-deployment/) , 配置容器和 [使用 kubectl 管理资源](/zh/docs/concepts/overview/working-with-objects/object-management/) 等相关文档.

Deployment 对象的名称必须是合法的 [DNS 子域名](/zh/docs/concepts/overview/working-with-objects/names#dns-subdomain-names). Deployment 还需要 [`.spec` 部分](https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#spec-and-status).

### Pod 模板

`.spec` 中只有 `.spec.template` 和 `.spec.selector` 是必需的字段.

`.spec.template` 是一个 [Pod 模板](/zh/docs/concepts/workloads/pods/#pod-templates). 它和 [Pod](/docs/concepts/workloads/pods/pod-overview/ "Pod 表示您的集群上一组正在运行的容器.") 的语法规则完全相同. 只是这里它是嵌套的, 因此不需要 `apiVersion` 或 `kind`.

除了 Pod 的必填字段外, Deployment 中的 Pod 模板必须指定适当的标签和适当的重新启动策略. 对于标签, 请确保不要与其他控制器重叠. 请参考 [选择算符](#selector).

只有 [`.spec.template.spec.restartPolicy`](/zh/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy) 等于 `Always` 才是被允许的, 这也是在没有指定时的默认设置.

### 副本

`.spec.replicas` 是指定所需 Pod 的可选字段. 它的默认值是 1.

### 选择算符

`.spec.selector` 是指定本 Deployment 的 Pod [标签选择算符](/zh/docs/concepts/overview/working-with-objects/labels/) 的必需字段.

`.spec.selector` 必须匹配 `.spec.template.metadata.labels`, 否则请求会被 API 拒绝.

在 API `apps/v1` 版本中,`.spec.selector` 和 `.metadata.labels` 如果没有设置的话, 不会被默认设置为 `.spec.template.metadata.labels`, 所以需要明确进行设置. 同时在 `apps/v1` 版本中, Deployment 创建后 `.spec.selector` 是不可变的.

当 Pod 的标签和选择算符匹配, 但其模板和 `.spec.template` 不同时, 或者此类 Pod 的总数超过 `.spec.replicas` 的设置时, Deployment 会终结之. 如果 Pods 总数未达到期望值, Deployment 会基于 `.spec.template` 创建新的 Pod.

> **说明:** 你不应直接创建, 或者通过创建另一个 Deployment, 或者创建类似 ReplicaSet 或 ReplicationController 这类控制器来创建标签与此选择算符匹配的 Pod. 如果这样做, 第一个 Deployment 会认为它创建了这些 Pod. Kubernetes 不会阻止你这么做.

如果有多个控制器的选择算符发生重叠, 则控制器之间会因冲突而无法正常工作.

### 策略

`.spec.strategy` 策略指定用于用新 Pods 替换旧 Pods 的策略. `.spec.strategy.type` 可以是 "Recreate" 或 "RollingUpdate"."RollingUpdate" 是默认值.

#### 重新创建 Deployment

如果 `.spec.strategy.type== Recreate`, 在创建新 Pods 之前, 所有现有的 Pods 会被杀死.

#### 滚动更新 Deployment

Deployment 会在 `.spec.strategy.type== RollingUpdate` 时, 采取 滚动更新的方式更新 Pods. 你可以指定 `maxUnavailable` 和 `maxSurge` 来控制滚动更新 过程.

##### 最大不可用

`.spec.strategy.rollingUpdate.maxUnavailable` 是一个可选字段, 用来指定 更新过程中不可用的 Pod 的个数上限. 该值可以是绝对数字 (例如,5), 也可以是 所需 Pods 的百分比 (例如,10%). 百分比值会转换成绝对数并去除小数部分. 如果 `.spec.strategy.rollingUpdate.maxSurge` 为 0, 则此值不能为 0. 默认值为 25%.

例如, 当此值设置为 30% 时, 滚动更新开始时会立即将旧 ReplicaSet 缩容到期望 Pod 个数的 70%. 新 Pod 准备就绪后, 可以继续缩容旧有的 ReplicaSet, 然后对新的 ReplicaSet 扩容, 确保在更新期间 可用的 Pods 总数在任何时候都至少为所需的 Pod 个数的 70%.

##### 最大峰值

`.spec.strategy.rollingUpdate.maxSurge` 是一个可选字段, 用来指定可以创建的超出 期望 Pod 个数的 Pod 数量. 此值可以是绝对数 (例如,5) 或所需 Pods 的百分比 (例如,10%). 如果 `MaxUnavailable` 为 0, 则此值不能为 0. 百分比值会通过向上取整转换为绝对数. 此字段的默认值为 25%.

例如, 当此值为 30% 时, 启动滚动更新后, 会立即对新的 ReplicaSet 扩容, 同时保证新旧 Pod 的总数不超过所需 Pod 总数的 130%. 一旦旧 Pods 被杀死, 新的 ReplicaSet 可以进一步扩容, 同时确保更新期间的任何时候运行中的 Pods 总数最多为所需 Pods 总数的 130%.

### 进度期限秒数

`.spec.progressDeadlineSeconds` 是一个可选字段, 用于指定系统在报告 Deployment [进展失败](#failed-deployment) 之前等待 Deployment 取得进展的秒数. 这类报告会在资源状态中体现为 `Type= Progressing`,`Status= False`, `Reason= ProgressDeadlineExceeded`.Deployment 控制器将持续重试 Deployment. 将来, 一旦实现了自动回滚, Deployment 控制器将在探测到这样的条件时立即回滚 Deployment.

如果指定, 则此字段值需要大于 `.spec.minReadySeconds` 取值.

### 最短就绪时间

`.spec.minReadySeconds` 是一个可选字段, 用于指定新创建的 Pod 在没有任意容器崩溃情况下的最小就绪时间, 只有超出这个时间 Pod 才被视为可用. 默认值为 0(Pod 在准备就绪后立即将被视为可用). 要了解何时 Pod 被视为就绪, 可参考 [容器探针](/zh/docs/concepts/workloads/pods/pod-lifecycle/#container-probes).

### 修订历史限制

Deployment 的修订历史记录存储在它所控制的 ReplicaSets 中.

`.spec.revisionHistoryLimit` 是一个可选字段, 用来设定出于会滚目的所要保留的旧 ReplicaSet 数量. 这些旧 ReplicaSet 会消耗 etcd 中的资源, 并占用 `kubectl get rs` 的输出. 每个 Deployment 修订版本的配置都存储在其 ReplicaSets 中; 因此, 一旦删除了旧的 ReplicaSet, 将失去回滚到 Deployment 的对应修订版本的能力. 默认情况下, 系统保留 10 个旧 ReplicaSet, 但其理想值取决于新 Deployment 的频率和稳定性.

更具体地说, 将此字段设置为 0 意味着将清理所有具有 0 个副本的旧 ReplicaSet. 在这种情况下, 无法撤消新的 Deployment 上线, 因为它的修订历史被清除了.

### paused(暂停的)

`.spec.paused` 是用于暂停和恢复 Deployment 的可选布尔字段. 暂停的 Deployment 和未暂停的 Deployment 的唯一区别是, Deployment 处于暂停状态时, PodTemplateSpec 的任何修改都不会触发新的上线. Deployment 在创建时是默认不会处于暂停状态.
