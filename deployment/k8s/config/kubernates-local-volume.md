---
title: Kubernates loval volume 组件
date: 2020-01-13 14:30:00
tags: '配置'
categories:
  - ['部署', 'k8s']
permalink: kubernates-loval-volume
photo:
---

# 简介

kubernetes 从 1.10 版本开始支持 local volume (本地卷) , workload (不仅是 statefulsets 类型) 可以充分利用本地快速 SSD, 从而获取比 remote volume (如 cephfs, RBD) 更好的性能.

在 local volume 出现之前, statefulsets 也可以利用本地 SSD, 方法是配置 hostPath, 并通过 nodeSelector 或者 nodeAffinity 绑定到具体 node 上. 但 hostPath 的问题是, 管理员需要手动管理集群各个 node 的目录, 不太方便.

下面两种类型应用适合使用 local volume.

- 数据缓存, 应用可以就近访问数据, 快速处理.
- 分布式存储系统, 如分布式数据库 Cassandra , 分布式文件系统 ceph/gluster

# 配置

## 创建一个 storage class

```yaml
kind:  StorageClass
apiVersion:  storage.k8s.io/v1
metadata:
  name:  local-volume
provisioner:  kubernetes.io/no-provisioner
volumeBindingMode:  WaitForFirstConsumer
```

sc 的 provisioner 是 `kubernetes.io/no-provisioner`

WaitForFirstConsumer 表示 PV 不要立即绑定 PVC , 而是直到有 Pod 需要用 PVC 的时候才绑定. 调度器会在调度时综合考虑选择合适的 local PV, 这样就不会导致跟 Pod 资源设置, selectors, affinity and anti-affinity 策略等产生冲突. 很明显：如果 PVC 先跟 local PV 绑定了, 由于 local PV 是跟n ode 绑定的, 这样 selectors, affinity 等等就基本没用了, 所以更好的做法是先根据调度策略选择 node, 然后再绑定 local PV

## 静态创建 PV

通过 kubectl 命令, 静态创建一个 1GiB 的 PV; 该 PV 使用 node ubuntu-node1 的 /data/local/vol 目录; 该 PV 的 sc 为 local-volume.

```yaml
apiVersion:  v1
kind:  PersistentVolume
metadata:
  name:  local-pv
spec:
  capacity:
    storage:  1Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy:  Retain
  storageClassName:  local-volume
  local:
    path:  /data/local/vol
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key:  kubernetes.io/hostname
          operator:  In
          values:
          - ubuntu-node1
```

参数 Retain (保留) : PV 跟PVC 释放后, 管理员需要手工清理, 重新设置该卷.

需要指定 PV 对应的 sc; 目录 `/data/local/vol` 也需要创建

## 使用 local volume PV

创建一个关联 sc: local-volume 的 PVC, 然后将该 PVC 挂到 nginx 容器里

```ymal
apiVersion:  v1
kind:  PersistentVolumeClaim
metadata:
  name:  local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage:  1Gi
  storageClassName:  local-volume
---
kind:  Pod
apiVersion:  v1
metadata:
  name:  mypod
spec:
  containers:
    - name:  myfrontend
      image:  nginx
      volumeMounts:
      - mountPath:  "/usr/share/nginx/html"
        name:  mypod-pvc
  volumes:
    - name:  mypod-pvc
      persistentVolumeClaim:
        claimName:  local-pvc
```

进入到容器里, 会看到挂载的目录, 大小其实就是上面创建的PV所在磁盘的 size.

在宿主机的 `/data/local/vol` 目录下创建一个 `index.html` 文件

```sh
echo "hello world" > /data/local/vol1/index.html
```

然后再去 curl 容器的 IP 地址, 就可以得到刚写入的字符串了.

删除 Pod/PVC, 之后 PV 状态改为 Released , 该 PV 不会再被绑定 PVC 了.

## 动态创建 PV

手工管理 local PV 显然是很费劲的, 社区提供了 external storage 可以动态的创建 PV (实际仍然不够自动化) .

local volume provisioner 的官方编排在 local-volume/provisioner/deployment/kubernetes/example/default_example_provisioner_generated.yaml 目录里, 不过官方文档一会 fast-disk, 一会 local-storage

```yaml
apiVersion:  v1
kind:  ConfigMap
metadata:
  name:  local-provisioner-config
  namespace:  default
data:
  storageClassMap:  |
    local-volume:
       hostDir:  /data/local
       mountDir:   /data/local
       blockCleanerCommand:
         - "/scripts/shred.sh"
         - "2"
       volumeMode:  Filesystem
       fsType:  ext4
---
apiVersion:  extensions/v1beta1
kind:  DaemonSet
metadata:
  name:  local-volume-provisioner
  namespace:  default
  labels:
    app:  local-volume-provisioner
spec:
  selector:
    matchLabels:
      app:  local-volume-provisioner
  template:
    metadata:
      labels:
        app:  local-volume-provisioner
    spec:
      serviceAccountName:  local-volume-admin
      containers:
        - image:  "silenceshell/local-volume-provisioner: v2.1.0"
          imagePullPolicy:  "Always"
          name:  provisioner
          securityContext:
            privileged:  true
          env:
          - name:  MY_NODE_NAME
            valueFrom:
              fieldRef:
                fieldPath:  spec.nodeName
          volumeMounts:
            - mountPath:  /etc/provisioner/config
              name:  provisioner-config
              readOnly:  true
            - mountPath:   /data/local
              name:  local
              mountPropagation:  "HostToContainer"
      volumes:
        - name:  provisioner-config
          configMap:
            name:  local-provisioner-config
        - name:  local
          hostPath:
            path:  /data/local
---
apiVersion:  v1
kind:  ServiceAccount
metadata:
  name:  local-volume-admin
  namespace:  default
---
apiVersion:  rbac.authorization.k8s.io/v1
kind:  ClusterRoleBinding
metadata:
  name:  local-volume-provisioner-pv-binding
  namespace:  default
subjects:
- kind:  ServiceAccount
  name:  local-volume-admin
  namespace:  default
roleRef:
  kind:  ClusterRole
  name:  system: persistent-volume-provisioner
  apiGroup:  rbac.authorization.k8s.io
---
apiVersion:  rbac.authorization.k8s.io/v1
kind:  ClusterRole
metadata:
  name:  local-volume-provisioner-node-clusterrole
  namespace:  default
rules:
- apiGroups:  [""]
  resources:  ["nodes"]
  verbs:  ["get"]
---
apiVersion:  rbac.authorization.k8s.io/v1
kind:  ClusterRoleBinding
metadata:
  name:  local-volume-provisioner-node-binding
  namespace:  default
subjects:
- kind:  ServiceAccount
  name:  local-volume-admin
  namespace:  default
roleRef:
  kind:  ClusterRole
  name:  local-volume-provisioner-node-clusterrole
  apiGroup:  rbac.authorization.k8s.io
```

kubectl 创建后, 由于是 daemonset 类型, 每个节点上都会启动一个 provisioner. 该 provisioner 会监视 "discovery directory", 即上面配置的 /data/local

```sh
$ kubectl get pods -o wide|grep local-volume
local-volume-provisioner-rrsjp            1/1     Running   0          5m    10.244.1.141   ubuntu-2   <none>
local-volume-provisioner-v87b7            1/1     Running   0          5m    10.244.2.69    ubuntu-3   <none>
local-volume-provisioner-x65k9            1/1     Running   0          5m    10.244.0.174   ubuntu-1   <none>
```

重新创建一个, 此时 pvc myclaim 是 Pending 状态, provisoner 并没有自动供给存储. 为什么呢?

原来 external-storage 的逻辑是这样的：其 Provisioner 本身其并不提供 local volume, 但它在各个节点上的 provisioner 会去动态的"发现"挂载点 (discovery directory) , 当某 node 的 provisioner 在 /data/local/ 目录下发现有挂载点时, 会创建 PV, 该 PV 的 local.path 就是挂载点, 并设置 nodeAffinity 为该 node.

那么如何获得挂载点呢?

直接去创建目录是行不通的, 因为 provsioner 希望 PV 是隔离的, 例如 capacity, io 等. 试着在 ubuntu-2 上的 /data/local/下创建一个 xxx 目录, 会得到这样的告警.

```sh
discovery.go: 201] Path "/data/local/xxx" is not an actual mountpoint
```

目录不是挂载点, 不能用.

该目录必须是真材实料的 mount 才行. 一个办法是加硬盘,  格式化,  mount, 比较麻烦, 实际可以通过本地文件格式化 (loopfs) 后挂载来"欺骗" provisioner, 让它以为是一个 mount 的盘, 从而自动创建 PV, 并与 PVC 绑定.

如下.

将下面的代码保存为文件 loopmount, 加执行权限并拷贝到 /bin 目录下, 就可以使用该命令来创建挂载点了.

```sh
#!/bin/bash

# Usage:  sudo loopmount file size mount-point

touch $1
truncate -s $2 $1
mke2fs -t ext4 -F $1 1> /dev/null 2> /dev/null
if [[ ! -d $3 ]]; then
        echo $3 " not exist, creating..."
        mkdir $3
fi
mount $1 $3
df -h |grep $3
```

使用脚本创建一个 6G 的文件, 并挂载到 /data/local 下. 之所以要 6G, 是因为前面 PVC 需要的是 5GB, 而格式化后剩余空间会小一点, 所以设置文件更大一些, 后面才好绑定 PVC.

```sh
# loopmount xxx 6G /data/local/xxx
/data/local/xxx  not exist, creating...
/dev/loop0     5.9G   24M  5.6G   1% /data/local/x1
```

查看 PV, 可见 Provisioner 自动创建了 PV, 而 kubernetes 会将该 PV 供给给前面的 PVC myclam, mypod 也 run 起来了.

```sh
# kubectl get pv
NAME              CAPACITY  ACCESS MODES   RECLAIM POLICY   STATUS  CLAIM            STORAGECLASS          REASON   AGE
local-pv-600377f7 5983Mi    RWO            Delete           Bound   default/myclaim  local-volume                   1s
```

可见, 目前版本的 local volume 还无法做到像 cephfs/RBD 一样的全自动化, 仍然需要管理员干涉

除了使用磁盘, 还可以考虑使用内存文件系统, 从而获取更高的 io 性能, 只是容量就没那么理想了. 一些特殊的应用可以考虑.

```sh
mount -t tmpfs -o size=1G,nr_inodes=10k,mode=700 tmpfs /data/local/tmpfs
```

# 参考

- [ref](https://ieevee.com/tech/2019/01/17/local-volume.html)
