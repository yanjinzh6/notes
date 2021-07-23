---
title: kubeadm-high-availability
date: 2021-01-30 21:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-high-availability
photo:
---

https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/high-availability/
https://qingmu.io/2019/05/17/Deploy-a-highly-available-cluster-with-kubeadm/

# 利用 kubeadm 创建高可用集群

本文讲述了使用 kubeadm 设置一个高可用的 Kubernetes 集群的两种不同方式:

*   使用堆控制平面节点. 这种方法所需基础设施较少.etcd 成员和控制平面节点位于同一位置.
*   使用外部集群. 这种方法所需基础设施较多. 控制平面的节点和 etcd 成员是分开的.

在下一步之前, 您应该仔细考虑哪种方法更好的满足您的应用程序和环境的需求. [](/zh/docs/setup/production-environment/tools/kubeadm/ha-topology) 这是对比文档 讲述了每种方法的优缺点.

如果您在安装 HA 集群时遇到问题, 请在 kubeadm [问题跟踪](https://github.com/kubernetes/kubeadm/issues/new) 里向我们提供反馈.

您也可以阅读 [](/zh/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade) 升级文件

> **注意:** 这篇文档没有讲述在云提供商上运行集群的问题. 在云环境中, 此处记录的方法不适用于类型为 LoadBalancer 的服务对象, 或者具有动态的 PersistentVolumes.

## 准备开始

对于这两种方法, 您都需要以下基础设施:

*   配置三台机器 [kubeadm 的最低要求](/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin) 给主节点
*   配置三台机器 [kubeadm 的最低要求](/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#before-you-begin) 给工作节点
*   在集群中, 所有计算机之间的完全网络连接 (公网或私网)
*   所有机器上的 sudo 权限
*   每台设备对系统中所有节点的 SSH 访问
*   在所有机器上安装 `kubeadm` 和 `kubelet`,`kubectl` 是可选的.

仅对于外部 etcd 集群来说, 您还需要:

*   给 etcd 成员使用的另外三台机器

## 这两种方法的第一步

### 为 kube-apiserver 创建负载均衡器

> **说明:** 使用负载均衡器需要许多配置. 您的集群搭建可能需要不同的配置. 下面的例子只是其中的一方面配置.

1.  创建一个名为 kube-apiserver 的负载均衡器解析 DNS.

    *   在云环境中, 应该将控制平面节点放置在 TCP 后面转发负载平衡. 该负载均衡器将流量分配给目标列表中所有运行状况良好的控制平面节点. 健康检查 apiserver 是在 kube-apiserver 监听端口 (默认值 `:6443`) 上的一个 TCP 检查.

    *   不建议在云环境中直接使用 IP 地址.

    *   负载均衡器必须能够在 apiserver 端口上与所有控制平面节点通信. 它还必须允许其监听端口的传入流量.

    *   确保负载均衡器的地址始终匹配 kubeadm 的 `ControlPlaneEndpoint` 地址.

    *   阅读 [软件负载平衡选项指南](https://github.com/kubernetes/kubeadm/blob/master/docs/ha-considerations.md#options-for-software-load-balancing) 以获取更多详细信息.


2.  添加第一个控制平面节点到负载均衡器并测试连接:

    ```
    nc -v LOAD_BALANCER_IP PORT
    ```

    *   由于 apiserver 尚未运行, 预期会出现一个连接拒绝错误. 然而超时意味着负载均衡器不能和控制平面节点通信. 如果发生超时, 请重新配置负载均衡器与控制平面节点进行通信.
3.  将其余控制平面节点添加到负载均衡器目标组.


## 使用堆控制平面和 etcd 节点

### 控制平面节点的第一步

1.  初始化控制平面:

    ```
    sudo kubeadm init --control-plane-endpoint "LOAD_BALANCER_DNS: LOAD_BALANCER_PORT" --upload-certs
    ```

    *   您可以使用 `--kubernetes-version` 标志来设置要使用的 Kubernetes 版本. 建议将 kubeadm, kebelet, kubectl 和 Kubernetes 的版本匹配.
    *   这个 `--control-plane-endpoint` 标志应该被设置成负载均衡器的地址或 DNS 和端口.
    *   这个 `--upload-certs` 标志用来将在所有控制平面实例之间的共享证书上传到集群. 如果正好相反, 你更喜欢手动地通过控制平面节点或者使用自动化 工具复制证书, 请删除此标志并参考如下部分 [证书分配手册](#manual-certs).

> **说明:** 标志 `kubeadm init`,`--config` 和 `--certificate-key` 不能混合使用, 因此如果您要使用 [kubeadm 配置](https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2), 您必须在相应的配置文件 (位于 `InitConfiguration` 和 `JoinConfiguration: controlPlane`) 添加 `certificateKey` 字段.

> **说明:** 一些 CNI 网络插件如 Calico 需要 CIDR 例如 `192.168.0.0/16` 和一些像 Weave 没有. 参考 [CNI 网络文档](/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network). 通过传递 `--pod-network-cidr` 标志添加 pod CIDR, 或者您可以使用 kubeadm 配置文件, 在 `ClusterConfiguration` 的 `networking` 对象下设置 `podSubnet` 字段.

*   命令完成后, 您应该会看到类似以下内容:

    ```
    ...
    现在, 您可以通过在根目录上运行以下命令来加入任意数量的控制平面节点:
    kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 --control-plane --certificate-key f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07

    请注意, 证书密钥可以访问集群内敏感数据, 请保密!
    为了安全起见, 将在两个小时内删除上传的证书; 如有必要, 您可以使用 kubeadm 初始化上传证书阶段, 之后重新加载证书.

    然后, 您可以通过在根目录上运行以下命令来加入任意数量的工作节点:
    kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
    ```

    *   将此输出复制到文本文件. 稍后您将需要它来将控制平面节点和辅助节点加入集群.

    *   当 `--upload-certs` 与 `kubeadm init` 一起使用时, 主控制平面的证书被加密并上传到 `kubeadm-certs` 密钥中.

    *   要重新上传证书并生成新的解密密钥, 请在已加入集群节点的控制平面上使用以下命令:

        ```
        sudo kubeadm init phase upload-certs --upload-certs
        ```

    *   您还可以在 `init` 期间指定自定义的 `--certificate-key`, 以后可以由 `join` 使用. 要生成这样的密钥, 可以使用以下命令:

        ```
        kubeadm alpha certs certificate-key
        ```


> **说明:** `kubeadm-certs` 密钥和解密密钥会在两个小时后失效.

> **注意:** 正如命令输出中所述, 证书密钥可访问群集敏感数据, 并将其保密!

1.  应用您选择的 CNI 插件: [请遵循以下指示](/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/#pod-network) 安装 CNI 提供程序. 如果适用, 请确保配置与 kubeadm 配置文件中指定的 Pod CIDR 相对应.

    在此示例中, 我们使用 Weave Net:

    ```
    kubectl apply -f "https://cloud.weave.works/k8s/net? k8s-version=$(kubectl version | base64 | tr -d '\n')"
    ```

2.  输入以下内容, 并查看 pods 的控制平面组件启动:

    ```
    kubectl get pod -n kube-system -w
    ```


### 其余控制平面节点的步骤

> **说明:** 从 kubeadm 1.15 版本开始, 您可以并行加入多个控制平面节点. 在此版本之前, 您必须在第一个节点初始化后才能依序的增加新的控制平面节点.

对于每个其他控制平面节点, 您应该:

1.  执行先前由第一个节点上的 `kubeadm init` 输出提供给您的 join 命令. 它看起来应该像这样:

    ```
    sudo kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866 --control-plane --certificate-key f8902e114ef118304e561c3ecd4d0b543adc226b7a07f675f56564185ffe0c07
    ```

    *   这个 `--control-plane` 命令通知 `kubeadm join` 创建一个新的控制平面.
    *   `--certificate-key ...` 将导致从集群中的 `kubeadm-certs` 秘钥下载控制平面证书并使用给定的密钥进行解密.

## 外部 etcd 节点

使用外部 etcd 节点设置集群类似于用于堆叠 etcd 的过程, 不同之处在于您应该首先设置 etcd, 并在 kubeadm 配置文件中传递 etcd 信息.

### 设置 ectd 集群

1.  按照 [](/zh/docs/setup/production-environment/tools/kubeadm/setup-ha-etcd-with-kubeadm) 这些指示 去设置 etcd 集群.

2.  设置 SSH 在 [这](#manual-certs) 描述.

3.  将以下文件从集群中的任何 etcd 节点复制到第一个控制平面节点:

    ```
    export CONTROL_PLANE="ubuntu@10.0.0.7"
    scp /etc/kubernetes/pki/etcd/ca.crt "${CONTROL_PLANE}":
    scp /etc/kubernetes/pki/apiserver-etcd-client.crt "${CONTROL_PLANE}":
    scp /etc/kubernetes/pki/apiserver-etcd-client.key "${CONTROL_PLANE}":
    ```

    *   用第一台控制平面机的 `user@host` 替换 `CONTROL_PLANE` 的值.

### 设置第一个控制平面节点

1.  用以下内容创建一个名为 `kubeadm-config.yaml` 的文件:

    ```
    apiVersion: kubeadm.k8s.io/v1beta2
    kind: ClusterConfiguration
    kubernetesVersion: stable
    controlPlaneEndpoint: "LOAD_BALANCER_DNS: LOAD_BALANCER_PORT"
    etcd:
        external:
            endpoints:
            - https://ETCD_0_IP:2379
            - https://ETCD_1_IP:2379
            - https://ETCD_2_IP:2379
            caFile: /etc/kubernetes/pki/etcd/ca.crt
            certFile: /etc/kubernetes/pki/apiserver-etcd-client.crt
            keyFile: /etc/kubernetes/pki/apiserver-etcd-client.key
    ```

> **说明:** 这里堆 etcd 和外部 etcd 之前的区别在于设置外部 etcd 需要一个 `etcd` 的 `external` 对象下带有 etcd 端点的配置文件. 如果是堆 etcd 技术, 是自动管理的.

*   在您的集群中, 将配置模板中的以下变量替换为适当值:

    *   `LOAD_BALANCER_DNS`
    *   `LOAD_BALANCER_PORT`
    *   `ETCD_0_IP`
    *   `ETCD_1_IP`
    *   `ETCD_2_IP`

以下的步骤与设置堆集群是相似的:

1.  在节点上运行 `sudo kubeadm init --config kubeadm-config.yaml --upload-certs` 命令.

2.  编写输出联接命令, 这些命令将返回到文本文件以供以后使用.

3.  应用您选择的 CNI 插件. 给定以下示例适用于 Weave Net:

    ```
    kubectl apply -f "https://cloud.weave.works/k8s/net? k8s-version=$(kubectl version | base64 | tr -d '\n')"
    ```


### 其他控制平面节点的步骤

步骤与设置堆 etcd 相同:

*   确保第一个控制平面节点已完全初始化.
*   使用保存到文本文件的连接命令将每个控制平面节点连接在一起. 建议一次加入一个控制平面节点.
*   不要忘记默认情况下,`--certificate-key` 中的解密秘钥会在两个小时后过期.

## 列举控制平面之后的常见任务

### 安装工作节点

您可以使用之前存储的命令将工作节点加入集群中 作为 `kubeadm init` 命令的输出:

```
sudo kubeadm join 192.168.0.200:6443 --token 9vr73a.a8uxyaju799qwdjv --discovery-token-ca-cert-hash sha256:7c2e69131a36ae2a042a339b33381c6d0d43887e2de83720eff5359e26aec866
```

## 手动证书分发

如果您选择不将 `kubeadm init` 与 `--upload-certs` 命令一起使用, 则意味着您将必须手动将证书从主控制平面节点复制到 将要加入的控制平面节点上.

有许多方法可以实现这种操作. 在下面的例子中我们使用 `ssh` 和 `scp`:

如果要在单独的一台计算机控制所有节点, 则需要 SSH.

1.  在您的主设备上启动 ssh-agent, 要求该设备能访问系统中的所有其他节点:

    ```
    eval $(ssh-agent)
    ```
2.  将 SSH 身份添加到会话中:

    ```
    ssh-add ~/.ssh/path_to_private_key
    ```
3.  检查节点间的 SSH 以确保连接是正常运行的

    *   SSH 到任何节点时, 请确保添加 `-A` 标志:

        ```
        ssh -A 10.0.0.7
        ```
    *   当在任何节点上使用 sudo 时, 请确保环境完善, 以便使用 SSH 转发任务:

        ```
        sudo -E -s
        ```

1.  在所有节点上配置 SSH 之后, 您应该在运行过 `kubeadm init` 命令的第一个控制平面节点上运行以下脚本. 该脚本会将证书从第一个控制平面节点复制到另一个控制平面节点:

    在以下示例中, 用其他控制平面节点的 IP 地址替换 `CONTROL_PLANE_IPS`.

    ```
    USER= ubuntu # 可自己设置
    CONTROL_PLANE_IPS="10.0.0.7 10.0.0.8"
    for host in ${CONTROL_PLANE_IPS}; do
        scp /etc/kubernetes/pki/ca.crt "${USER}"@$host:
        scp /etc/kubernetes/pki/ca.key "${USER}"@$host:
        scp /etc/kubernetes/pki/sa.key "${USER}"@$host:
        scp /etc/kubernetes/pki/sa.pub "${USER}"@$host:
        scp /etc/kubernetes/pki/front-proxy-ca.crt "${USER}"@$host:
        scp /etc/kubernetes/pki/front-proxy-ca.key "${USER}"@$host:
        scp /etc/kubernetes/pki/etcd/ca.crt "${USER}"@$host: etcd-ca.crt
        scp /etc/kubernetes/pki/etcd/ca.key "${USER}"@$host: etcd-ca.key
    done
    ```


> **注意:** 只需要复制上面列表中的证书.kubeadm 将负责生成其余证书以及加入控制平面实例所需的 SAN. 如果您错误地复制了所有证书, 由于缺少所需的 SAN, 创建其他节点可能会失败.

1.  然后, 在每个连接控制平面节点上, 您必须先运行以下脚本, 然后再运行 `kubeadm join`. 该脚本会将先前复制的证书从主目录移动到 `/etc/kubernetes/pki`:

    ```
    USER= ubuntu # 可自己设置
    mkdir -p /etc/kubernetes/pki/etcd
    mv /home/${USER}/ca.crt /etc/kubernetes/pki/
    mv /home/${USER}/ca.key /etc/kubernetes/pki/
    mv /home/${USER}/sa.pub /etc/kubernetes/pki/
    mv /home/${USER}/sa.key /etc/kubernetes/pki/
    mv /home/${USER}/front-proxy-ca.crt /etc/kubernetes/pki/
    mv /home/${USER}/front-proxy-ca.key /etc/kubernetes/pki/
    mv /home/${USER}/etcd-ca.crt /etc/kubernetes/pki/etcd/ca.crt
    mv /home/${USER}/etcd-ca.key /etc/kubernetes/pki/etcd/ca.key
    ```
