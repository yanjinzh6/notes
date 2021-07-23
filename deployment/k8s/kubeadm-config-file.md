---
title: kubeadm-config-file
date: 2021-01-30 19:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-config-file
photo:
---

https://blog.csdn.net/qq_38496902/article/details/106560163

记录 kubeadm --config 的可配置内容, 参考 [https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2)

## kubeadm --config 的配置文件内容

1.1  InitConfiguration

```
// 初始化参数配置
type InitConfiguration struct {
    metav1.TypeMeta `json:", inline"`

    // BootstrapTokens?
    BootstrapTokens []BootstrapToken `json:"bootstrapTokens, omitempty"`

    // 可配置节点 name, annotation, Taints, cloud-provider 等
    NodeRegistration NodeRegistrationOptions `json:"nodeRegistration, omitempty"`

    // 配置 apiserver 的 ip, 端口 (默认 6443), 另外一个参数 ControlPlaneEndpoint 用于在 HA 情况下配置全局的 apiserver 地址
    LocalAPIEndpoint APIEndpoint `json:"localAPIEndpoint, omitempty"`

    // 配置上传到集群作为 secret 的 ca 证书
    CertificateKey string `json:"certificateKey, omitempty"`
}
```

1.2  ClusterConfiguration

```
//kubeadm 集群的配置
type ClusterConfiguration struct {
    metav1.TypeMeta `json:", inline"`
    // etcd 集群的配置, 可以配置内部与外部的 etcd
    Etcd Etcd `json:"etcd, omitempty"`

    // service 网段, pod 网段, 域名
    Networking Networking `json:"networking, omitempty"`

    // k8s 版本
    KubernetesVersion string `json:"kubernetesVersion, omitempty"`

    // 在 HA 情况下, 可配置为 SLB 地址
    ControlPlaneEndpoint string `json:"controlPlaneEndpoint, omitempty"`

    //apiserver 的额外启动参数, 挂载的卷, SAN(多域名证书), apiserver 超时时间
    APIServer APIServer `json:"apiServer, omitempty"`


    //controller-manager 额外启动参数, 挂载的卷
    ControllerManager ControlPlaneComponent `json:"controllerManager, omitempty"`

    //schduler 额外启动参数, 挂载的卷
    Scheduler ControlPlaneComponent `json:"scheduler, omitempty"`

    // DNS 配置
    // 安装 core-dns 还是 kube-dns, 单独配置 dns 使用的镜像 (如果这里没有配置, 则使用 ClusterConfiguration 里配置的镜像)
    DNS DNS `json:"dns, omitempty"`

    // 证书存放位置
    CertificatesDir string `json:"certificatesDir, omitempty"`

    // 镜像仓库地址
    ImageRepository string `json:"imageRepository, omitempty"`

    // 是否使用 hyperkube 作为 k8s 组件 (弃用)
    UseHyperKubeImage bool `json:"useHyperKubeImage, omitempty"`

    // 启用哪些 FeatureGates
    FeatureGates map[string]bool `json:"featureGates, omitempty"`

    // 集群名称
    ClusterName string `json:"clusterName, omitempty"`
}
```

1.3  KubeletConfiguration

> ```
>
> type KubeletConfiguration struct {
>     metav1.TypeMeta `json:", inline"`
>
>     // 启用 kubelet, 默认 true
>     EnableServer *bool `json:"enableServer, omitempty"`
>
>     // 静态 pod 目录或文件, 默认 ""
>     StaticPodPath string `json:"staticPodPath, omitempty"`
>
>     // 同步运行中的容器与配置的最大周期, 默认 "1m"
>     // + optional
>     SyncFrequency metav1.Duration `json:"syncFrequency, omitempty"`
>
>     // 读取配置文件周期, 默认 "20s"
>     // 减小时间会造成频繁读取配置文件, 可能影响性能
>     FileCheckFrequency metav1.Duration `json:"fileCheckFrequency, omitempty"`
>
>     // 静态 pod http 检查周期, 默认 "20s"
>     // 减小时间会使轮询 StaticPodURL 更频繁, 默认 ""
>     HTTPCheckFrequency metav1.Duration `json:"httpCheckFrequency, omitempty"`
>
>     // 判断静态 pod 在运行的地址? 默认 ""
>     StaticPodURL string `json:"staticPodURL, omitempty"`
>
>     // 访问 podURL 时的 http header  , 默认 nil
>     StaticPodURLHeader map[string][]string `json:"staticPodURLHeader, omitempty"`
>
>     // kubelet 监听地址, 默认 "0.0.0.0"
>     Address string `json:"address, omitempty"`
>
>     //kubelet 监听端口, 默认 10250
>     Port int32 `json:"port, omitempty"`
>
>     //read-only 端口? 默认 0 (disabled)
>     ReadOnlyPort int32 `json:"readOnlyPort, omitempty"`
>
>     // 用于 https 认证的 X509 证书, 默认 ""
>     TLSCertFile string `json:"tlsCertFile, omitempty"`
>
>     // 私钥, 默认 ""
>     TLSPrivateKeyFile string `json:"tlsPrivateKeyFile, omitempty"`
>
>     //TLS 支持的加密算法? 值来自 (https://golang.org/pkg/crypto/tls/#pkg-constants), 默认 nil
>     TLSCipherSuites []string `json:"tlsCipherSuites, omitempty"`
>
>     // 支持的 TLS 最小版本, 默认 ""
>     TLSMinVersion string `json:"tlsMinVersion, omitempty"`
>
>     // 启用客户端证书循环? 默认 false
>     RotateCertificates bool `json:"rotateCertificates, omitempty"`
>
>     // kubelet 会用 CSR API 从 apiserver 得到证书, 需启用 RotateKubeletServerCertificate 特性, 默认 false
>     ServerTLSBootstrap bool `json:"serverTLSBootstrap, omitempty"`
>
>     // 请求 kubelet 时的认证配置, bear token, X509 等
>     // 默认:
>     //   anonymous:
>     //     enabled: false
>     //   webhook:
>     //     enabled: true
>     //     cacheTTL: "2m"
>     // + optional
>     Authentication KubeletAuthentication `json:"authentication"`
>
>     // 指定如何授权对 kubelet 的请求
>     // Defaults:
>     //   mode: Webhook
>     //   webhook:
>     //     cacheAuthorizedTTL: "5m"
>     //     cacheUnauthorizedTTL: "30s"
>     Authorization KubeletAuthorization `json:"authorization"`
>
>     // 拉取镜像的 QPS
>     // Default: 5
>     RegistryPullQPS *int32 `json:"registryPullQPS, omitempty"`
>
>     // bursty pulls 的最大数
>     // Default: 10
>     RegistryBurst int32 `json:"registryBurst, omitempty"`
>
>     // 每秒最大创建事件数
>     // Default: 5
>     EventRecordQPS *int32 `json:"eventRecordQPS, omitempty"`
>
>     // 每秒最大创建事件数 (burst)
>     // Default: 10
>     EventBurst int32 `json:"eventBurst, omitempty"`
>
>     // 是否支持访问 kubelet 端点的日志功能, 以及 exec, attach, logs, forward 命令
>     // Default: true
>     EnableDebuggingHandlers *bool `json:"enableDebuggingHandlers, omitempty"`
>
>     // 启用锁争用分析?
>     // Default: false
>     EnableContentionProfiling bool `json:"enableContentionProfiling, omitempty"`
>
>     // 健康检查端口
>     // Default: 10248
>     HealthzPort *int32 `json:"healthzPort, omitempty"`
>
>     // 健康检查地址
>     // Default: "127.0.0.1"
>     HealthzBindAddress string `json:"healthzBindAddress, omitempty"`
>
>     //kubelet 进程的 oom-score-adj
>     // Default: -999
>     OOMScoreAdj *int32 `json:"oomScoreAdj, omitempty"`
>
>     // 配置后所有容器都会使用该 DNS
>     // Default: ""
>     ClusterDomain string `json:"clusterDomain, omitempty"`
>
>     //DNS 服务器的列表, 配置后将代替宿主机的 DNS 解析
>     // Default: nil
>     ClusterDNS []string `json:"clusterDNS, omitempty"`
>
>     // 流连接的最长时间
>     // Default: "4h"
>     StreamingConnectionIdleTimeout metav1.Duration `json:"streamingConnectionIdleTimeout, omitempty"`
>
>     // 更新 node 状态的频率
>     // Default: "10s"
>     NodeStatusUpdateFrequency metav1.Duration `json:"nodeStatusUpdateFrequency, omitempty"`
>
>     // 上报 node 状态到 master 的频率
>     // nodeStatusUpdateFrequency for backward compatibility.
>     // Default: "1m"
>     NodeStatusReportFrequency metav1.Duration `json:"nodeStatusReportFrequency, omitempty"`
>
>     // kubelet 在租约上的持续时间 (参考 etcd lease 接口)
>     // Default: 40
>     NodeLeaseDurationSeconds int32 `json:"nodeLeaseDurationSeconds, omitempty"`
>
>     // 回收没有用的镜像的时间
>     // Default: "2m"
>     ImageMinimumGCAge metav1.Duration `json:"imageMinimumGCAge, omitempty"`
>
>     // 当磁盘使用率达到该值时, 会一直进行镜像 GC
>     // Default: 85
>     ImageGCHighThresholdPercent *int32 `json:"imageGCHighThresholdPercent, omitempty"`
>
>     // 当磁盘使用率低于该值时, 不会进行镜像 GC
>     // Default: 80
>     ImageGCLowThresholdPercent *int32 `json:"imageGCLowThresholdPercent, omitempty"`
>
>     // 计算, 缓存所有 pod 用到的 volume
>     // Default: "1m"
>     VolumeStatsAggPeriod metav1.Duration `json:"volumeStatsAggPeriod, omitempty"`
>
>     // 隔离 kubelet 的 cgroup 的绝对名称? (需要学习一下 cgroup)
>     // Default: ""
>     KubeletCgroups string `json:"kubeletCgroups, omitempty"`
>
>     // 略
>     // Default: ""
>     SystemCgroups string `json:"systemCgroups, omitempty"`
>
>     // 略
>     // Default: ""
>     CgroupRoot string `json:"cgroupRoot, omitempty"`
>
>     // 略
>     // Default: true
>     CgroupsPerQOS *bool `json:"cgroupsPerQOS, omitempty"`
>
>     //cgroup 驱动, cgroup/systemd
>     // Default: "cgroupfs"
>     CgroupDriver string `json:"cgroupDriver, omitempty"`
>
>     //cpu 管理策略 (可用哪些?)
>     // Default: "none"
>     CPUManagerPolicy string `json:"cpuManagerPolicy, omitempty"`
>
>     // CPU Manager 调节周期?
>     // Default: "10s"
>     CPUManagerReconcilePeriod metav1.Duration `json:"cpuManagerReconcilePeriod, omitempty"`
>
>     // 略
>     // Default: "none"
>     TopologyManagerPolicy string `json:"topologyManagerPolicy, omitempty"`
>
>     // 系统预留的资源 (百分比), 支持内存
>     // Default: nil
>     QOSReserved map[string]string `json:"qosReserved, omitempty"`
>
>     // 容器请求的超时时间, 除了 pull, logs, exec, attach
>     // Default: "2m"
>     RuntimeRequestTimeout metav1.Duration `json:"runtimeRequestTimeout, omitempty"`
>
>     // 当 service 想访问自己时, 可以配置此参数
>     // 可配置的值:
>     //promiscuous-bridge: 配置容器网桥模式为 promiscuous?
>     //hairpin-veth: 容器网卡上配置 hairpin flag?
>     //none: 啥也不干
>     // Default: "promiscuous-bridge"
>     HairpinMode string `json:"hairpinMode, omitempty"`
>
>     // 当前节点上可运行的最大 pod 数
>     // Default: 110
>     MaxPods int32 `json:"maxPods, omitempty"`
>
>     // Pod ID CIDR, standalone 模式 (?) 时可以配置该参数, cluster 模式时从 master 获取
>     // Default: ""
>     PodCIDR string `json:"podCIDR, omitempty"`
>
>     // pod 中的最大 pid 数
>     // Default: -1
>     PodPidsLimit *int64 `json:"podPidsLimit, omitempty"`
>
>     // 容器 dns 解析配置
>     // Default: "/etc/resolv.conf"
>     ResolverConfig string `json:"resolvConf, omitempty"`
>
>     // 让 kubelet 从 apiserver 获取一下 pod 情况, 然后退出
>     // Default: false
>     RunOnce bool `json:"runOnce, omitempty"`
>
>     // 当容器配置了 cpu limits 时, 启用 cpu cfs 限制
>     // Default: true
>     CPUCFSQuota *bool `json:"cpuCFSQuota, omitempty"`
>
>     // 配置 cpu 分配的周期, cpu.cfs_period_us
>     // Default: "100ms"
>     CPUCFSQuotaPeriod *metav1.Duration `json:"cpuCFSQuotaPeriod, omitempty"`
>
>     // 配置 node.status.images 的数量,
>     //-1: 没有上限
>     //0: 不返回 image
>     // Default: 50
>     NodeStatusMaxImages *int32 `json:"nodeStatusMaxImages, omitempty"`
>
>     // kubelet 进程可打开的最大文件数
>     // Default: 1000000
>     MaxOpenFiles int64 `json:"maxOpenFiles, omitempty"`
>
>     // 发送给 apiserver 的 contentType
>     // Default: "application/vnd.kubernetes.protobuf"
>     ContentType string `json:"contentType, omitempty"`
>
>     // 与 apiserver 交互的 QPS
>     // Default: 5
>     KubeAPIQPS *int32 `json:"kubeAPIQPS, omitempty"`
>
>     // 与 apiserver 交互的 QPS(burst)
>     // Default: 10
>     KubeAPIBurst int32 `json:"kubeAPIBurst, omitempty"`
>
>     // kubelet 一个一个拉取镜像
>     // Default: true
>     SerializeImagePulls *bool `json:"serializeImagePulls, omitempty"`
>
>     // 硬性驱逐的阈值, signal -> quantities
>     // Default:
>     //   memory.available:  "100Mi"
>     //   nodefs.available:  "10%"
>     //   nodefs.inodesFree: "5%"
>     //   imagefs.available: "15%"
>     EvictionHard map[string]string `json:"evictionHard, omitempty"`
>
>     // 软性驱逐的阈值 (grace)
>     // Default: nil
>     EvictionSoft map[string]string `json:"evictionSoft, omitempty"`
>
>     // 软性驱逐的周期, 比如 {"memory.available": "30s"}
>     // Default: nil
>     EvictionSoftGracePeriod map[string]string `json:"evictionSoftGracePeriod, omitempty"`
>
>     // 这是啥?
>     // Default: "5m"
>     EvictionPressureTransitionPeriod metav1.Duration `json:"evictionPressureTransitionPeriod, omitempty"`
>
>     // 这是啥?
>     // Default: 0
>     EvictionMaxPodGracePeriod int32 `json:"evictionMaxPodGracePeriod, omitempty"`
>
>     // Map of signal names to quantities that defines minimum reclaims, which describe the minimum
>     // amount of a given resource the kubelet will reclaim when performing a pod eviction while
>     // that resource is under pressure. For example: {"imagefs.available": "2Gi"}
>     // Dynamic Kubelet Config (beta): If dynamically updating this field, consider that
>     // it may change how well eviction can manage resource pressure.
>     // Default: nil
>     // + optional
>     EvictionMinimumReclaim map[string]string `json:"evictionMinimumReclaim, omitempty"`
>
>     // podsPerCore is the maximum number of pods per core. Cannot exceed MaxPods.
>     // 每核 cpu 能跑的最大 pod 数
>     // Default: 0
>     PodsPerCore int32 `json:"podsPerCore, omitempty"`
>
>     // 让 AD-controller 来执行卷的挂载 / 卸载
>     // Default: true
>     EnableControllerAttachDetach *bool `json:"enableControllerAttachDetach, omitempty"`
>
>     // true: 当内核配置不满足 kubelet 要求时, kubelet 会出现错误
>     //false: kubelet 会修改内核配置
>     // Default: false
>     ProtectKernelDefaults bool `json:"protectKernelDefaults, omitempty"`
>
>     // 会生成一些默认的 iptables 规则给组件用, 比如 kube-proxy
>     // Default: true
>     MakeIPTablesUtilChains *bool `json:"makeIPTablesUtilChains, omitempty"`
>
>     // 这是啥?
>     // Default: 14
>     IPTablesMasqueradeBit *int32 `json:"iptablesMasqueradeBit, omitempty"`
>
>     // 这是啥?
>     // Default: 15
>     IPTablesDropBit *int32 `json:"iptablesDropBit, omitempty"`
>
>     // 支持启用的特性, 查看 k8s.io/kubernetes/pkg/features/kube_features.go
>     // Default: nil
>     FeatureGates map[string]bool `json:"featureGates, omitempty"`
>
>     // 当 swap 启用时, kubelet 将不能启动
>     // Default: true
>     FailSwapOn *bool `json:"failSwapOn, omitempty"`
>
>     // 容器日志大小 (滚动更新前)
>     // Default: "10Mi"
>     ContainerLogMaxSize string `json:"containerLogMaxSize, omitempty"`
>
>     // 容器最大的日志文件数
>     // Default: 5
>     ContainerLogMaxFiles *int32 `json:"containerLogMaxFiles, omitempty"`
>
>     // 配置 confimap 和 secret 的 manager 以何种模式运行?
>     // Default: "Watch"
>     ConfigMapAndSecretChangeDetectionStrategy ResourceChangeDetectionStrategy `json:"configMapAndSecretChangeDetectionStrategy, omitempty"`
>
>     // 为系统预留资源, 只支持 cpu, mem 具体查看 http://kubernetes.io/docs/user-guide/compute-resources
>     // Default: nil
>     SystemReserved map[string]string `json:"systemReserved, omitempty"`
>
>     // 为 k8s 组件预留的资源, 支持 cpu, mem, loca storage 具体查看 http://kubernetes.io/docs/user-guide/compute-resources
>     // Default: nil
>     KubeReserved map[string]string `json:"kubeReserved, omitempty"`
>
>     // 预留的 cpu 列表, 将覆盖 system-reserved 和 kube-reserved
>     ReservedSystemCPUs string `json:"reservedSystemCPUs, omitempty"`
>
>     // 这是啥?
>     // Default: ""
>     ShowHiddenMetricsForVersion string `json:"showHiddenMetricsForVersion, omitempty"`
>
>     // 系统预留的 cgroup, 具体参考 https://git.k8s.io/community/contributors/design-proposals/node/node-allocatable.md
>     // Default: ""
>     SystemReservedCgroup string `json:"systemReservedCgroup, omitempty"`
>     // k8s 预留的 cgroup, 具体参考 https://git.k8s.io/community/contributors/design-proposals/node/node-allocatable.md
>     // Default: ""
>     KubeReservedCgroup string `json:"kubeReservedCgroup, omitempty"`
>
>     // 指定了 Node Allocatable enforcements, 具体参考 https://git.k8s.io/community/contributors/design-proposals/node/node-allocatable.md
>     // Default: ["pods"]
>     EnforceNodeAllocatable []string `json:"enforceNodeAllocatable, omitempty"`
>
>     // 允许不安全 sysctl 操作的白名单列表
>     // Default: []
>     AllowedUnsafeSysctls []string `json:"allowedUnsafeSysctls, omitempty"`
>
>     // 第三方卷插件目录
>     // Default: "/usr/libexec/kubernetes/kubelet-plugins/volume/exec/"
>     VolumePluginDir string `json:"volumePluginDir, omitempty"`
>
>     // 标识 cloudprovider 的实例, ccm 中会用到
>     // Default: ""
>     ProviderID string `json:"providerID, omitempty"`
> }
>
> ```

1.4  KubeProxyConfiguration

> ```
>
> type KubeProxyConfiguration struct {
>     metav1.TypeMeta `json:", inline"`
>
>     // 开启关闭一些 FeatureGates
>     FeatureGates map[string]bool `json:"featureGates, omitempty"`
>
>     //kube-proxy 绑定地址
>     BindAddress string `json:"bindAddress"`
>
>     // 健康检查绑定地址, 默认 0.0.0.0:10256
>     HealthzBindAddress string `json:"healthzBindAddress"`
>
>     //metrics server 地址, 默认 127.0.0.1:10249
>     MetricsBindAddress string `json:"metricsBindAddress"`
>
>     // 若为 true, 当端口绑定失败时, kube-proxy 将退出
>     BindAddressHardFail bool `json:"bindAddressHardFail"`
>
>     // 若为 true, 提供一个 web 接口 /debug/pprof, 给 metrics-server 使用
>     EnableProfiling bool `json:"enableProfiling"`
>
>     // 集群内 pod 的 CIDR
>     ClusterCIDR string `json:"clusterCIDR"`
>
>     // 覆盖真实的 hostname
>     HostnameOverride string `json:"hostnameOverride"`
>
>     // 与 apiserver 通信时的配置
>     //kube-proxy 的 kubeconfig, 请求 apiserer 时的请求头 (将覆盖 application/json), ContentType , QPS, Burst(当超过查询速率时允许将查询请求缓存下来?)
>     ClientConnection componentbaseconfigv1alpha1.ClientConnectionConfiguration `json:"clientConnection"`
>
>     //ipables 配置
>     // 掩码位数 [0,31], 是否对所有路由作 SNAT, iptables 刷新间隔 (SyncPeriod 和 MinSyncPeriod 有啥区别?),
>     IPTables KubeProxyIPTablesConfiguration `json:"iptables"`
>
>     //IPVS 配置
>     // 略...
>     IPVS KubeProxyIPVSConfiguration `json:"ipvs"`
>
>     //OOMScoreAdj [-1000,1000]
>     OOMScoreAdj *int32 `json:"oomScoreAdj"`
>
>     //kube-proxy 转发模式, userspace, iptables, ipvs
>     Mode ProxyMode `json:"mode"`
>
>     // 主机端口范围, 用于 service 服务转发
>     PortRange string `json:"portRange"`
>
>     //UDP 连接空闲的保持时间, 仅适用于 proxyMode= userspace
>     UDPIdleTimeout metav1.Duration `json:"udpIdleTimeout"`
>     //conntrack 配置
>     // 如跟踪每个 cpu 的最大 NAT 连接数, TCP 连接的空闲时间等
>     Conntrack KubeProxyConntrackConfiguration `json:"conntrack"`
>
>     //apiserver 的配置信息刷新频率
>     ConfigSyncPeriod metav1.Duration `json:"configSyncPeriod"`
>
>     // 这个设置了有啥用?
>     NodePortAddresses []string `json:"nodePortAddresses"`
>
>     // winkernel 配置 (windows)
>     Winkernel KubeProxyWinkernelConfiguration `json:"winkernel"`
>
>     // 这个参数是干嘛的?
>     ShowHiddenMetricsForVersion string `json:"showHiddenMetricsForVersion"`
>
>     // 检测本地 traffic, 默认 LocalModeClusterCIDR
>     DetectLocalMode LocalMode `json:"detectLocalMode"`
> }
>
> ```

1.5  JoinConfiguration

> ```
>
> type JoinConfiguration struct {
>     metav1.TypeMeta `json:", inline"`
>
>     // 可配置 kubelet 的参数, 比如 cloud-provider: "external"
>     NodeRegistration NodeRegistrationOptions `json:"nodeRegistration, omitempty"`
>
>     // 加入计算节点的配置, 可 Kubeadm token create --print-join-command 的输出信息
>     Discovery Discovery `json:"discovery"`
>
>     // 加入控制面的配置
>     ControlPlane *JoinControlPlane `json:"controlPlane, omitempty"`
> }
>
> ```
