---
title: Helm3 配置使用
date: 2020-02-25 16:30:00
tags: '配置'
categories:
  - ['部署', 'k8s']
permalink: helm3-config
photo:
---

# 简介

Helm 是管理 Kubernetes 的应用管理工具

## 相关术语

- Helm 是一个命令行下的客户端工具. 主要用于 Kubernetes 应用程序 Chart 的创建, 打包, 发布以及创建和管理本地和远程的 Chart 仓库.
- Tiller 是 Helm 的服务端, 部署在 Kubernetes 集群中. Tiller 用于接收 Helm 的请求, 并根据 Chart 生成 Kubernetes 的部署文件 ( Helm 称为 Release ) , 然后提交给 Kubernetes 创建应用. Tiller 还提供了 Release 的升级, 删除, 回滚等一系列功能.
- Chart 是 Helm 的软件包, 采用 TAR 格式. 类似于 APT 的 DEB 包或者 YUM 的 RPM 包, 其包含了一组定义 Kubernetes 资源相关的 YAML 文件.
- Repoistory 是 Helm 的软件仓库, Repository 本质上是一个 Web 服务器, 该服务器保存了一系列的 Chart 软件包以供用户下载, 并且提供了一个该 Repository 的 Chart 包的清单文件以供查询. Helm 可以同时管理多个不同的 Repository.
- Release 是 使用 `helm install` 命令在 Kubernetes 集群中部署的 Chart

## Helm3 新功能

- 移除了 Tiller
- 不同的 namespace 可以使用相同的 Release Name
- 简化模板对象 .Capabilities
- 使用 JSONSchema 验证 charts 的 Values
- 将 requirements.yaml 合并到 Chart.yaml 中
- `helm install` 时需要指定 Release Name, 开启自动生成需要 --generate-name 参数
- 支持 push 到远端 registry  (如: harbor)
- 移除 helm serve
- 命令行变化 (将原先的命令保留为别名Aliases)
  - helm delete --> helm uninstall
  - helm inspect -> helm show
  - helm fetch -> helm pull
- go 导入路径改变 k8s.io/helm --> helm.sh/helm

具体新特性可以参考 [Helm 3 GitHub](https://github.com/helm/helm), 或者参考 [Helm 官方文档](https://v3.helm.sh/)

<!-- more -->

# 安装

## helm github

访问 [Helm GitHub release](https://github.com/helm/helm/releases) 下载最新的客户端

接下来解压下载的包, 然后将客户端放置到 `/usr/local/bin/` 目录下:

```sh
#下载Helm客户端
$ wget https://get.helm.sh/helm-v3.1.0-linux-amd64.tar.gz

#解压 Helm
$ tar -zxvf helm-v3.1.0-linux-amd64.tar.gz

#复制客户端执行文件到 bin 目录下, 方便在系统下能执行 helm 命令
$ cp linux-amd64/helm /usr/local/bin/
```

**注意: helm 客户端需要下载到安装了 kubectl 并且能执行能正常通过 kubectl 操作 kubernetes 的服务器上, 否则 helm 将不可用**

## microk8s

microk8s 自带 helm3 组件, 只需要使用 `microk8s.enable helm3` 命令, 等待下载完成即可使用

# 配置

Helm3 默认是不会添加 Chart 仓库, 需要添加常用的仓库

```sh
$ helm repo add  elastic    https://helm.elastic.co
$ helm repo add  gitlab     https://charts.gitlab.io
$ helm repo add  harbor     https://helm.goharbor.io
$ helm repo add  bitnami    https://charts.bitnami.com/bitnami
$ helm repo add  incubator  https://kubernetes-charts-incubator.storage.googleapis.com
$ helm repo add  stable     https://kubernetes-charts.storage.googleapis.com
```

增加完仓库后, 需要执行更新命令, 将仓库中的信息进行同步 `helm repo update`

**注意: 如果有的仓库不能正常解析, 请更换 DNS 地址, 在测试过程中, 发现有的能正常解析, 有的不能. 如果还不行, 就直接将域名和对应的地址写死在 Host 文件中. 或者配置代理**

## 使用国内仓库

```sh
helm repo add stable http://mirror.azure.cn/kubernetes/charts
helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
helm repo update
```

# 部署应用

## 查询应用

```sh
microk8s.helm3 search repo rabbitmq
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
stable/prometheus-rabbitmq-exporter     0.5.5           v0.29.0         Rabbitmq metrics exporter for prometheus
stable/rabbitmq                         6.17.4          3.8.2           Open source message broker software that implem...
stable/rabbitmq-ha                      1.40.1          3.8.0           Highly available RabbitMQ cluster, the open sou...
```

## 查看安装包的内容

`helm inspect values` 查看暴露的自定义参数

## 安装应用

```sh
microk8s.helm3 install my-rabbitmq stable/rabbitmq -n default

# -n, --namespace 参数指定安装的命名空间, Helm3 可以在不同的命名空间中部署相同名称的应用
# -f values.yaml 使用自定义参数配置
# --set 设置自定义参数列表
# --dry-run --debug 模拟安装过程并打印配置信息.
microk8s.helm3 template stable/rabbitmq -n default
---
# Source: rabbitmq/templates/secrets.yaml
apiVersion: v1
kind: Secret
metadata:
  name: RELEASE-NAME-rabbitmq
  labels:
    app: rabbitmq
    chart: rabbitmq-6.17.4
    release: "RELEASE-NAME"
    heritage: "Helm"
type: Opaque
data:

  rabbitmq-password: "Nll6SjJjRVZCNA=="


  rabbitmq-erlang-cookie: "a0k0YTlyYlZaTGRORG5vOGZXZTh0MmVodFlJY2xNd3A="
---
# Source: rabbitmq/templates/configuration.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-rabbitmq-config
  labels:
    app: rabbitmq
    chart: rabbitmq-6.17.4
    release: "RELEASE-NAME"
    heritage: "Helm"
data:
  enabled_plugins: |-
    [rabbitmq_management, rabbitmq_peer_discovery_k8s, rabbitmq_auth_backend_ldap].
  rabbitmq.conf: |-
    ##username and password
    default_user=user
    default_pass=CHANGEME
    ## Clustering
    cluster_formation.peer_discovery_backend  = rabbit_peer_discovery_k8s
    cluster_formation.k8s.host = kubernetes.default.svc.cluster.local
    cluster_formation.node_cleanup.interval = 10
    cluster_formation.node_cleanup.only_log_warning = true
    cluster_partition_handling = autoheal
    # queue master locator
    queue_master_locator=min-masters
    # enable guest user
    loopback_users.guest = false
    #disk_free_limit.absolute = 50MB
    #management.load_definitions = /app/load_definition.json
---
# Source: rabbitmq/templates/healthchecks.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-rabbitmq-healthchecks
  labels:
    app: rabbitmq
    chart: rabbitmq-6.17.4
    release: "RELEASE-NAME"
    heritage: "Helm"
data:
  rabbitmq-health-check: |-
    #!/bin/sh
    START_FLAG=/opt/bitnami/rabbitmq/var/lib/rabbitmq/.start
    if [ -f ${START_FLAG} ]; then
        rabbitmqctl node_health_check
        RESULT=$?
        if [ $RESULT -ne 0 ]; then
          rabbitmqctl status
          exit $?
        fi
        rm -f ${START_FLAG}
        exit ${RESULT}
    fi
    rabbitmq-api-check $1 $2
  rabbitmq-api-check: |-
    #!/bin/sh
    set -e
    URL=$1
    EXPECTED=$2
    ACTUAL=$(curl --silent --show-error --fail "${URL}")
    echo "${ACTUAL}"
    test "${EXPECTED}" = "${ACTUAL}"
---
# Source: rabbitmq/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: RELEASE-NAME-rabbitmq
  labels:
    app: rabbitmq
    chart: rabbitmq-6.17.4
    release: "RELEASE-NAME"
    heritage: "Helm"
---
# Source: rabbitmq/templates/role.yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: RELEASE-NAME-rabbitmq-endpoint-reader
  labels:
    app: rabbitmq
    chart: rabbitmq-6.17.4
    release: "RELEASE-NAME"
    heritage: "Helm"
rules:
- apiGroups: [""]
  resources: ["endpoints"]
  verbs: ["get"]
---
# Source: rabbitmq/templates/rolebinding.yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: RELEASE-NAME-rabbitmq-endpoint-reader
  labels:
    app: rabbitmq
    chart: rabbitmq-6.17.4
    release: "RELEASE-NAME"
    heritage: "Helm"
subjects:
- kind: ServiceAccount
  name: RELEASE-NAME-rabbitmq
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: RELEASE-NAME-rabbitmq-endpoint-reader
---
# Source: rabbitmq/templates/svc-headless.yaml
apiVersion: v1
kind: Service
metadata:
  name: RELEASE-NAME-rabbitmq-headless
  labels:
    app: rabbitmq
    chart: rabbitmq-6.17.4
    release: "RELEASE-NAME"
    heritage: "Helm"
spec:
  clusterIP: None
  ports:
  - name: epmd
    port: 4369
    targetPort: epmd
  - name: amqp
    port: 5672
    targetPort: amqp
  - name: dist
    port: 25672
    targetPort: dist
  - name: stats
    port: 15672
    targetPort: stats
  selector:
    app: rabbitmq
    release: "RELEASE-NAME"
---
# Source: rabbitmq/templates/svc.yaml
apiVersion: v1
kind: Service
metadata:
  name: RELEASE-NAME-rabbitmq
  labels:
    app: rabbitmq
    chart: rabbitmq-6.17.4
    release: "RELEASE-NAME"
    heritage: "Helm"
spec:
  type: ClusterIP
  ports:
  - name: epmd
    port: 4369
    targetPort: epmd
    nodePort: null
  - name: amqp
    port: 5672
    targetPort: amqp
    nodePort: null
  - name: dist
    port: 25672
    targetPort: dist
    nodePort: null
  - name: stats
    port: 15672
    targetPort: stats
    nodePort: null
  selector:
    app: rabbitmq
    release: "RELEASE-NAME"
---
# Source: rabbitmq/templates/statefulset.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: RELEASE-NAME-rabbitmq
  labels:
    app: rabbitmq
    chart: rabbitmq-6.17.4
    release: "RELEASE-NAME"
    heritage: "Helm"
spec:
  serviceName: RELEASE-NAME-rabbitmq-headless
  podManagementPolicy: OrderedReady
  replicas: 1
  updateStrategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app: rabbitmq
      release: "RELEASE-NAME"
  template:
    metadata:
      labels:
        app: rabbitmq
        release: "RELEASE-NAME"
        chart: rabbitmq-6.17.4
      annotations:
        checksum/secret: 674e9e7b3bee84098b9eb3fee68fa4c73e02927268215c350495627a9df62735
    spec:
      serviceAccountName: RELEASE-NAME-rabbitmq
      terminationGracePeriodSeconds: 10
      containers:
      - name: rabbitmq
        image: docker.io/bitnami/rabbitmq:3.8.2-debian-10-r25
        imagePullPolicy: "IfNotPresent"
        command:
         - bash
         - -ec
         - |
            mkdir -p /opt/bitnami/rabbitmq/.rabbitmq/
            mkdir -p /opt/bitnami/rabbitmq/etc/rabbitmq/
            touch /opt/bitnami/rabbitmq/var/lib/rabbitmq/.start
            #persist the erlang cookie in both places for server and cli tools
            echo $RABBITMQ_ERL_COOKIE > /opt/bitnami/rabbitmq/var/lib/rabbitmq/.erlang.cookie
            cp /opt/bitnami/rabbitmq/var/lib/rabbitmq/.erlang.cookie /opt/bitnami/rabbitmq/.rabbitmq/
            #change permission so only the user has access to the cookie file
            chmod 600 /opt/bitnami/rabbitmq/.rabbitmq/.erlang.cookie /opt/bitnami/rabbitmq/var/lib/rabbitmq/.erlang.cookie
            #copy the mounted configuration to both places
            cp  /opt/bitnami/rabbitmq/conf/* /opt/bitnami/rabbitmq/etc/rabbitmq
            # Apply resources limits
            ulimit -n "${RABBITMQ_ULIMIT_NOFILES}"
            #replace the default password that is generated
            sed -i "/CHANGEME/cdefault_pass=${RABBITMQ_PASSWORD//\\/\\\\}" /opt/bitnami/rabbitmq/etc/rabbitmq/rabbitmq.conf
            exec rabbitmq-server
        volumeMounts:
          - name: config-volume
            mountPath: /opt/bitnami/rabbitmq/conf
          - name: healthchecks
            mountPath: /usr/local/sbin/rabbitmq-api-check
            subPath: rabbitmq-api-check
          - name: healthchecks
            mountPath: /usr/local/sbin/rabbitmq-health-check
            subPath: rabbitmq-health-check
          - name: data
            mountPath: "/opt/bitnami/rabbitmq/var/lib/rabbitmq"
        ports:
        - name: epmd
          containerPort: 4369
        - name: amqp
          containerPort: 5672
        - name: dist
          containerPort: 25672
        - name: stats
          containerPort: 15672
        livenessProbe:
          exec:
            command:
              - sh
              - -c
              - rabbitmq-api-check "http://user:$RABBITMQ_PASSWORD@127.0.0.1:15672/api/healthchecks/node" '{"status":"ok"}'
          initialDelaySeconds: 120
          timeoutSeconds: 20
          periodSeconds: 30
          failureThreshold: 6
          successThreshold: 1
        readinessProbe:
          exec:
            command:
              - sh
              - -c
              - rabbitmq-health-check "http://user:$RABBITMQ_PASSWORD@127.0.0.1:15672/api/healthchecks/node" '{"status":"ok"}'
          initialDelaySeconds: 10
          timeoutSeconds: 20
          periodSeconds: 30
          failureThreshold: 3
          successThreshold: 1
        env:
          - name: BITNAMI_DEBUG
            value: "false"
          - name: MY_POD_IP
            valueFrom:
              fieldRef:
                fieldPath: status.podIP
          - name: MY_POD_NAME
            valueFrom:
              fieldRef:
                fieldPath: metadata.name
          - name: MY_POD_NAMESPACE
            valueFrom:
              fieldRef:
                fieldPath: metadata.namespace
          - name: K8S_SERVICE_NAME
            value: "RELEASE-NAME-rabbitmq-headless"
          - name: K8S_ADDRESS_TYPE
            value: hostname
          - name: RABBITMQ_NODENAME
            value: "rabbit@$(MY_POD_NAME).$(K8S_SERVICE_NAME).$(MY_POD_NAMESPACE).svc.cluster.local"
          - name: K8S_HOSTNAME_SUFFIX
            value: ".$(K8S_SERVICE_NAME).$(MY_POD_NAMESPACE).svc.cluster.local"
          - name: RABBITMQ_LOGS
            value: "-"
          - name: RABBITMQ_ULIMIT_NOFILES
            value: "65536"
          - name: RABBITMQ_SERVER_ADDITIONAL_ERL_ARGS
            value: +S 2:1
          - name: RABBITMQ_USE_LONGNAME
            value: "true"
          - name: RABBITMQ_ERL_COOKIE
            valueFrom:
              secretKeyRef:
                name: RELEASE-NAME-rabbitmq
                key: rabbitmq-erlang-cookie
          - name: RABBITMQ_PASSWORD
            valueFrom:
              secretKeyRef:
                name: RELEASE-NAME-rabbitmq
                key: rabbitmq-password
      securityContext:
        fsGroup: 1001
        runAsUser: 1001
      volumes:
        - name: config-volume
          configMap:
            name: RELEASE-NAME-rabbitmq-config
            items:
            - key: rabbitmq.conf
              path: rabbitmq.conf
            - key: enabled_plugins
              path: enabled_plugins
        - name: healthchecks
          configMap:
            name: RELEASE-NAME-rabbitmq-healthchecks
            items:
            - key: rabbitmq-health-check
              path: rabbitmq-health-check
              mode: 111
            - key: rabbitmq-api-check
              path: rabbitmq-api-check
              mode: 111
  volumeClaimTemplates:
    - metadata:
        name: data
        labels:
          app: rabbitmq
          release: "RELEASE-NAME"
          heritage: "Helm"
      spec:
        accessModes:
          - "ReadWriteOnce"
        resources:
            requests:
              storage: "8Gi"
```

有四种安装来源

- Reporitory
- Charts 打包后的 tgz 包
- 从 tgz 解压后的 Charts 目录
- 从 url 连接

## 查看状态

```sh
microk8s.helm3 status my-rabbitmq
NAME: my-rabbitmq
LAST DEPLOYED: Tue Feb 25 11:30:06 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **
...

microk8s.helm3 list --all --namespace default
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART           APP VERSION
my-rabbitmq     default         1               2020-02-25 11:30:06.341856864 +0800 CST deployed        rabbitmq-6.17.4 3.8.2
```

## 卸载应用

```sh
microk8s.helm3 uninstall my-rabbitmq -n default

# -n, --namespace 参数指定安装的命名空间, Helm3 可以在不同的命名空间中部署相同名称的应用
```

## 升级应用

通过更新配置文件的方式来更新部署

```sh
# values.yaml 参数配置文件
microk8s.helm3 upgrade -f values.yaml my-rabbitmq stable/rabbitmq -n default
# 查看新配置是否生效
microk8s.helm3 get values my-rabbitmq -n default
```

## 应用回滚

升级过程发生错误, 可以进行回滚, 操作过程为查看应用历史版本, 获取 REVISION 号后进行回滚操作

```sh
microk8s.helm3 history my-rabbitmq -n default
REVISION        UPDATED                         STATUS          CHART           APP VERSION     DESCRIPTION
1               Tue Feb 25 11:30:06 2020        deployed        rabbitmq-6.17.4 3.8.2           Install complete
# 回滚到 1 版本
microk8s.helm3 rollback my-rabbitmq 1 -n default
```

# 创建 Repository

- 查看当前的仓库 `helm repo list`
- 在本机创建仓库 `mkdir -p ~/my-repo & nohup helm serve --address 127.0.0.1:8879 --repo-path ~/my-repo &`
- 添加仓库 `helm repo add my-repo http://127.0.0.1:8879`
- 在仓库中添加包, 更新 index, 更新缓存

```sh
# 先去 github 上下载 charts
cp -r mysql  ~/my-repo
cd  ~/my-repo
helm package mysql --save=false
helm repo index --url=http://127.0.0.1:8879 .
helm update
```

# 创建 Charts

- 快速创建模板, `helm create my-charts` , 修改对应内容
- 打包, 然后拷贝至 repository 的目录, 然后执行更新index操作. helm package
- 安装, `helm install .` 或 `helm install my-charts.tgz`
- 验证 charts 格式, `helm lint`
- 查看 charts 文件内容. `helm inspect chart my-charts`
- 查看 value 文件内容. `helm inspect values my-charts`
- 查看 charts 目录下文件内容. `helm inspect my-charts`
- 查看 charts 模板渲染后 k8s 的 yaml, `helm template my-charts -f configfile --set a=b`

# 引用

- [安装 Helm3 管理 Kubernetes 应用](http://www.mydlq.club/article/51/)
- [helm3 基础使用](https://blog.csdn.net/qq_25611295/article/details/103624669)
- [k8s 包管理 Helm 命令大全](https://blog.51cto.com/qujunorz/2421027)