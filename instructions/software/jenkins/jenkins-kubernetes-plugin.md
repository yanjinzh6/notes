---
title: Jenkins kubernetes 插件使用
date: 2020-1-13 14:00:00
tags: 'Jenkins'
categories:
  - ['使用说明', '软件']
permalink: jenkins-kubernetes-plugin
photo:
---

# 介绍

基于 Kubernete 的 CI/CD, 可以使用的工具有很多, 比如 Jenkins, Gitlab CI 和新兴的 drone

Jenkins 是一个流行的持续集成/发布的工具, 这里做个简单的流水账记录一下配置说明

传统的 Jenkins Slave 一主多从方式会存在一些痛点, 比如:

- 主 Master 发生单点故障时, 整个流程都不可用了
- 每个 Slave 的配置环境不一样, 来完成不同语言的编译打包等操作, 但是这些差异化的配置导致管理起来非常不方便, 维护起来也是比较费劲
- 资源分配不均衡, 有的 Slave 要运行的 job 出现排队等待, 而有的 Slave 处于空闲状态
- 资源有浪费, 每台 Slave 可能是物理机或者虚拟机, 当 Slave 处于空闲状态时, 也不会完全释放掉资源.

基于 Kubernetes 搭建 Jenkins 集群, Jenkins Master 和 Jenkins Slave 以 Pod 形式运行在 Kubernetes 集群的 Node 上, Master 运行在其中一个节点, 并且将其配置数据存储到一个 Volume 上去, Slave 运行在各个节点上, 并且它不是一直处于运行状态, 它会按照需求动态的创建并自动删除.

这种方式的工作流程大致为: 当 Jenkins Master 接受到 Build 请求时, 会根据配置的 Label 动态创建一个运行在 Pod 中的 Jenkins Slave 并注册到 Master 上, 当运行完 Job 后, 这个 Slave 会被注销并且这个 Pod 也会自动删除, 恢复到最初状态.

Kubernetes 搭建 Jenkins 集群的好处:

- 服务高可用, 当 Jenkins Master 出现故障时, Kubernetes 会自动创建一个新的 Jenkins Master 容器, 并且将 Volume 分配给新创建的容器, 保证数据不丢失, 从而达到集群服务高可用.
- 动态伸缩, 合理使用资源, 每次运行 Job 时, 会自动创建一个 Jenkins Slave, Job 完成后, Slave 自动注销并删除容器, 资源自动释放, 而且 Kubernetes 会根据每个资源的使用情况, 动态分配 Slave 到空闲的节点上创建, 降低出现因某节点资源利用率高, 还排队等待在该节点的情况.
- 扩展性好, 当 Kubernetes 集群的资源严重不足而导致 Job 排队等待时, 可以很容易的添加一个 Kubernetes Node 到集群中, 从而实现扩展.

<!-- more -->

# 安装

参考官方提供的 [yaml 文件](https://github.com/jenkinsci/kubernetes-plugin/tree/master/src/main/kubernetes), 这里官网使用的是比较规范的 StatefulSet (有状态集群服务) 方式进行部署, 并配置了 Ingress 和 RBAC 账户权限信息.

这里我们使用 `kind: Deployment` 替换 `kind: StatefulSet`
然后使用自定义的 pvc 来存储内容

```yaml
volumes:
  - name: jenkins-persistent-storage
    persistentVolumeClaim:
      claimName: local-pvc
```

通过 ingress 来暴露 Jenkins 服务并强制 https 访问

```yml
# jenkins
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
  namespace: jenkins-ops
  labels:
    app: jenkins
    version: v1
spec:
  # 新版本提示需要 selector 参数
  selector:
    matchLabels:
      app: jenkins
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  #  type: Recreate
  replicas: 1
  template:
    metadata:
      name: jenkins
      labels:
        app: jenkins
        version: v1
    spec:
      terminationGracePeriodSeconds: 10
      serviceAccountName: jenkins
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts
          # imagePullPolicy: IfNotPresent
          imagePullPolicy: Always
          ports:
            - containerPort: 8080
              name: http
              protocol: TCP
            - containerPort: 50000
              name: agent
              protocol: TCP
          resources:
            limits:
              cpu: '1'
              memory: '500Mi'
            requests:
              cpu: '0.1'
              memory: '100Mi'
          livenessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 60
            timeoutSeconds: 5
            failureThreshold: 12 # ~2 minutes
          readinessProbe:
            httpGet:
              path: /login
              port: 8080
            initialDelaySeconds: 60
            timeoutSeconds: 5
            failureThreshold: 12 # ~2 minutes
          volumeMounts:
            - name: jenkins-persistent-storage
              mountPath: /var/jenkins_home
          env:
            - name: LIMITS_MEMORY
              valueFrom:
                resourceFieldRef:
                  resource: limits.memory
                  divisor: 1Mi
            - name: JAVA_OPTS
              # value: -XX:+UnlockExperimentalVMOptions -XX:+UseCGroupMemoryLimitForHeap -XX:MaxRAMFraction=1 -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0 -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85
              value: -Xmx$(LIMITS_MEMORY)m -XshowSettings:vm -Dhudson.slaves.NodeProvisioner.initialDelay=0 -Dhudson.slaves.NodeProvisioner.MARGIN=50 -Dhudson.slaves.NodeProvisioner.MARGIN0=0.85 -Duser.timezone=Asia/Shanghai -Dhudson.model.DownloadService.noSignatureCheck=true
              # value: -Duser.timezone=Asia/Shanghai
      volumes:
        - name: jenkins-persistent-storage
          # persistentVolumeClaim:
          #   claimName: jenkins-pvc
          hostPath:
            # directory location on host
            path: /data/jenkins
            # this field is optional
            type: DirectoryOrCreate
      securityContext:
        runAsUser: 0
        fsGroup: 0
      # affinity:
      #   nodeAffinity:
      #     requiredDuringSchedulingIgnoredDuringExecution:
      #       nodeSelectorTerms:
      #       - matchExpressions:
      #         - key: kubernetes.io/hostname
      #           operator: NotIn
      #           values:
      #           - yanjin-ubuntu-master
      #           - yanjin-ubuntu-node
---
apiVersion: v1
kind: Service
metadata:
  name: jenkins
  namespace: jenkins-ops
  labels:
    app: jenkins
    service: jenkins
spec:
  selector:
    app: jenkins
  ports:
    - name: http
      port: 80
      # targetPort: web
      targetPort: 8080
    - name: agent
      port: 50000
      # targetPort: agent
      targetPort: 50000
  # type: NodePort
  # type: ClusterIP
```

这里使用一个名为 `jenkins/jenkins:lts` 的镜像, 这是 jenkins 官方的 Docker 镜像, 然后也有一些环境变量, 当然我们也可以根据自己的需求来定制一个镜像, 比如我们可以将一些插件打包在自定义的镜像当中, 可以参考文档: [https://github.com/jenkinsci/docker](https://github.com/jenkinsci/docker)

安装完成后通过命令验证

```sh
~$ kubectl get pods -n k8s-ops -o wide
NAME                      READY   STATUS    RESTARTS   AGE     IP            NODE                 NOMINATED NODE   READINESS GATES
jenkins-56dc668d6-nkhsg   1/1     Running   0          4m24s   10.244.1.25   yanjin-ubuntu-node   <none>           <none>
```

# 配置

## kubernetes plugin 配置

配置 Jenkins, 让他能够动态的生成 Slave 的 Pod

安装 jenkins 依赖的插件

- kubernetes
- managed scripts

1. 需要安装 kubernetes plugin, 点击 Manage Jenkins -> Manage Plugins -> Available -> Kubernetes 勾选安装即可
2. 安装完毕后, 点击 Manage Jenkins —> Configure System —> (拖到最下方)Add a new cloud —> 选择 Kubernetes, 然后填写 Kubernetes 和 Jenkins 配置信息

- Name 处默认为 kubernetes, 也可以修改为其他名称, 如果这里修改了, 下边在执行 Job 时指定 `podTemplate()` 参数 cloud 为其对应名称, 否则会找不到, cloud 默认值取: kubernetes
- Kubernetes URL 处默认 `https://kubernetes.default.svc.cluster.local` 完整 DNS 记录, 也可以简单填写 `https://kubernetes.default`, 这是 Kubernetes Service 对应的 DNS 记录, 通过该 DNS 记录可以解析成该 Service 的 Cluster IP, 命名方式要符合 `<svc_name>.<namespace_name>.svc.cluster.local`, 或者直接填写外部 Kubernetes 的地址 `https://<ClusterIP>:<Ports>`
- Jenkins URL 处填写 `http://jenkins.k8s-ops.svc.cluster.local:80`, 使用 Jenkins Service 对应的 DNS 记录, 不过要指定服务暴漏的端口. 同时也可以用 `http://<ClusterIP>:<Node_Port>` 方式
- 配置 Pod Template, 其实就是配置 Jenkins Slave 运行的 Pod 模板, 命名空间我们同样是用 k8s-ops, 这里的 Labels 名在配置非 pipeline 类型 Job 时, 用来指定任务运行的节点, 然后我们这里使用的是 `cnych/jenkins:jnlp` 这个镜像, 这个镜像是在官方的 jnlp 镜像基础上定制的, 加入了 kubectl 等一些实用的工具, Containers 下的 Name 字段的名字, 这里要注意的是, 如果 Name 配置为 jnlp, 那么 Kubernetes 会用下边指定的 Docker Image 代替默认的 `jenkinsci/jnlp-slave` 镜像, 否则, Kubernetes plugin 还是会用默认的 `jenkinsci/jnlp-slave` 镜像与 Jenkins Server 建立连接, 即使我们指定其他 Docker Image. 这里我随便配置为 jnlp-slave, 意思就是使用默认的 `jenkinsci/jnlp-slave` 镜像来运行

3. 配置完毕, 可以点击 "Test Connection" 按钮测试是否能够连接的到 Kubernetes, 如果显示 Connection test successful 则表示连接成功, 配置没有问题.

## Istio 配置

### 新建 k8s 命名空间

设置 label 自动注入 sidecar

### 重新部署

需要规范端口名称

### 配置外网连接

## 问题

由于官方的插件站点访问问题明显, 所以使用国内源 http://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/current/update-center.json

```sh
kubectl exec -it jenkins-56588c954c-9dvpn -c jenkins -n jenkins-ops /bin/bash
curl -i -k https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/current/updates/hudson.tasks.Maven.MavenInstaller.json
```

输出没问题, 但是在管理平台上就出现了 None of the tool installer metadata passed the signature check 提示使用插件更新站点时, 签名验证失败
查看 log

```
WARNING: signature check failed for https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/current/updates/hudson.tasks.Maven.MavenInstaller.json
ERROR: Signature verification failed in downloadable
```

根据[方法](https://support.cloudbees.com/hc/en-us/articles/115000494608-Why-is-there-Failed-Signature-Check-when-using-update-server-)
使用 `-Dhudson.model.DownloadService.noSignatureCheck=true` 参数解决

没有使用 istio 时好像不会有这种情况

# 测试

配置 Job 测试一下是否会根据配置的 Label 动态创建一个运行在 Docker Container 中的 Jenkins Slave 并注册到 Master 上, 而且运行完 Job 后, Slave 会被注销并且 Docker Container 也会自动删除

## pipeline 类型支持

创建一个 Pipeline 类型 Job , 然后在 Pipeline 脚本处填写一个简单的测试脚本

```
def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label, cloud: 'kubernetes') {
    node(label) {
        stage('Run shell') {
            sh 'sleep 30s'
            sh 'echo hello world.'
        }
    }
}
```

保存执行构建, 稍等一会, 就会看到 Master 和 jenkins-slave-jbs4z-xs2r8 已经创建完毕, 在等一会, 就会发现 jenkins-slave-jbs4z-xs2r8 已经注册到 Master 中, 并开始执行 Job, 点击该 Slave 节点, 我们可以看到通过标签 mypod-b538c04c-7c19-4b98-88f6-9e5bca6fc9ba 关联, 该 Label 就是我们定义的标签格式生成的, Job 执行完毕后, jenkins-slave 会自动注销, 我们通过 kubectl 命令行, 可以看到整个自动创建和删除过程

在 Slave 中构建任务

```
# label: 添加 Slave Pod 中配置的 label
node(label) {
    stage('Clone') {
      echo "1.Clone Stage"
    }
    stage('Test') {
      echo "2.Test Stage"
    }
    stage('Build') {
      echo "3.Build Stage"
    }
    stage('Deploy') {
      echo "4. Deploy Stage"
    }
}
```

重新触发立刻构建

```sh
$ kubectl get pods -n kube-ops
NAME                       READY     STATUS    RESTARTS   AGE
jenkins-7c85b6f4bd-rfqgv   1/1       Running   4          6d
jnlp-0hrrz                 1/1       Running   0          23s
```

## Container Group 类型支持

创建一个 Pipeline 类型 Job, 然后在 Pipeline 脚本处填写一个简单的测试脚本

```
def label = "mypod-${UUID.randomUUID().toString()}"
podTemplate(label: label, cloud: 'kubernetes', containers: [
    containerTemplate(name: 'maven', image: 'maven:3.3.9-jdk-8-alpine', ttyEnabled: true, command: 'cat'),
  ]) {

    node(label) {
        stage('Get a Maven Project') {
            git 'https://github.com/jenkinsci/kubernetes-plugin.git'
            container('maven') {
                stage('Build a Maven project') {
                    sh 'mvn -B clean install'
                }
            }
        }
    }
}
```

注意: 这里我们使用的 containers 定义了一个 containerTemplate 模板, 指定名称为 maven 和使用的 Image, 下边在执行 Stage 时, 使用 <code>container('maven'){...} </code>就可以指定在该容器模板里边执行相关操作了. 比如, 该示例会在 jenkins-slave 中执行 git clone 操作, 然后进入到 maven 容器内执行 <code>mvn -B clean install </code>编译操作. 这种操作的好处就是, 我们只需要根据代码类型分别制作好对应的编译环境镜像, 通过指定不同的 container 来分别完成对应代码类型的编译操作. 模板详细的各个参数配置可以参照 [Pod and container template configuration](https://github.com/jenkinsci/kubernetes-plugin/blob/master/README.md#pod-and-container-template-configuration).

## 非 Pipeline 类型支持

新建一个自由风格的 Job, 配置 "Restrict where this project can be run" 勾选, 在 "Label Expression" 后边输出我们上边创建模板是指定的 Labels 名称, 意思是指定该 Job 匹配 Labels 标签的 Slave 上运行

# 自定义 jenkins-slave 镜像

# 部署 Kubernetes 应用

1. 编写代码
2. 测试
3. 编写 Dockerfile
4. 构建打包 Docker 镜像
5. 推送 Docker 镜像到仓库
6. 编写 Kubernetes YAML 文件
7. 更改 YAML 文件中 Docker 镜像 TAG
8. 利用 kubectl 工具部署应用

## clone 代码

```
stage('Clone') {
    echo "1.Clone Stage"
    git url: "https://github.com/demo/jenkins-demo.git"
}
```

## 测试

```
stage('Test') {
    echo "2.Test"
}
```

## 构建镜像

```
stage('Build') {
    echo "3.Build Docker Image Stage"
    sh "docker build -t demo/jenkins-demo:${build_tag} ."
}
```

这里使用 Docker In Docker 的方式, 使用的 Slave 中已经配置好 docker 的 volume, 使用 sh 直接执行 docker build 命令即可, 采用和 git commit 的记录为镜像的 tag, 这里有一个好处就是镜像的 tag 可以和 git 提交记录对应起来, 也方便日后对应查看

```
def build_tag
stage('Clone') {
    echo "1.Clone Stage"
    git url: "https://github.com/cnych/jenkins-demo.git"
    script {
        build_tag = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
    }
}
stage('Build') {
    echo "3.Build Docker Image Stage"
    sh "docker build -t cnych/jenkins-demo:${build_tag} ."
}
```

## 推送镜像

通过 Credentials -> Stores scoped to Jenkins 下面的 Jenkins -> Global credentials (unrestricted) -> 左侧的 Add Credentials: 添加一个 Username with password 类型的认证信息

Add Credentials 输入 docker hub 的用户名和密码, ID 部分我们输入 dockerHub, 注意, 这个值非常重要, 在后面 Pipeline 的脚本中我们需要使用到这个 ID 值

```
stage('Push') {
    echo "4.Push Docker Image Stage"
    withCredentials([usernamePassword(credentialsId: 'dockerHub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {
        sh "docker login -u ${dockerHubUser} -p ${dockerHubPassword}"
        sh "docker push cnych/jenkins-demo:${build_tag}"
    }
}
```

私有仓库

```
stage('Push') {
    echo "4.Push Docker Image Stage"
    def DOCKER_REGISTRY_URI = "https://d.yzer.club"
    def CREDENTIALS_ID = "08267a76-19ab-40b4-a709-ce6e67c2243d"
    // withRegistry Wrote authentication to /var/jenkins_home/.dockercfg 但是 docker 使用的是 /root/.docker/config.json, 所以会出现 no basic auth credentials 错误
    withCredentials([usernamePassword(credentialsId: "${CREDENTIALS_ID}", usernameVariable: "USERNAME", passwordVariable: "PASSWORD")]) {
        sh "docker login --password=${PASSWORD} --username=${USERNAME} ${DOCKER_REGISTRY_URI}"
    }
    docker.withRegistry("${DOCKER_REGISTRY_URI}") {
        def customImage = null
        try {
            customImage = docker.build("youzhe/aiyongserver:1.0.${env.BUILD_ID}")

            // 推送远程仓库, 这里的 tag 可以使用 build_tag
            customImage.push("latest")
        } catch (e) {
            println e
            throw e
        } finally {
           //The finally block always executes.
           sh "docker logout  ${DOCKER_REGISTRY_URI}"
           // 删除
           if (customImage) {
               sh "docker rmi -f " + customImage.id
           }
        }
    }
}
```

## 更改 YAML, 更新 Kubernetes 系统中应用的镜像版本

使用一个 sed 命令

```
# sed 命令就是将 k8s.yaml 文件中的 标识给替换成变量 build_tag 的值
stage('YAML') {
    echo "5. Change YAML File Stage"
    sh "sed -i 's/<BUILD_TAG>/${build_tag}/' k8s.yaml"
}
```

## 部署

```
stage('Deploy') {
    echo "6. Deploy Stage"
    sh "kubectl apply -f k8s.yaml"
}
```

# 提供一个 shell 脚本, 实现一个 CI/CD 流程

```sh
#!/bin/bash
# Filename: k8s-deploy_v0.1.sh
# Description: jenkins CI/CD 持续发布脚本
# Author:  yi.hu
# Email: 345270016@qq.com
# Revision: 1.0
# Date: 2018-08-10
# Note: prd

# zookeeper基础服务,依照环境实际地址配置
init() {
    local lowerEnv="$(echo ${AppEnv} | tr '[:upper:]' 'lower')"
    case "${lowerEnv}" in
        dev)
            CFG_ADDR="10.34.11.186:4181"
            DR_CFG_ZOOKEEPER_ENV_URL="10.34.11.186:4181"
            ;;
        demo)
            CFG_ADDR="10.34.11.186:4181"
            DR_CFG_ZOOKEEPER_ENV_URL="10.34.11.186:4181"
            ;;
        *)
            echo "Not support AppEnv: ${AppEnv}"
            exit 1
            ;;
    esac
}


# 初始化变量
AppId=$(echo ${AppOrg}_${AppEnv}_${AppName} |sed 's/[^a-zA-Z0-9_-]//g' | tr "[:lower:]" "[:upper:]")
CFG_LABEL=${CfgLabelBaseNode}/${AppId}
CFG_ADDR=${CFG_ADDR}

# 登录harbor 仓库
docker_login () {
    docker login ${DOCKER_REGISTRY} -u${User} -p${PassWord}

}
# 编译代码, 制作镜像
build() {
echo $ACTION
    if [ "x${ACTION}" == "xDEPLOY" ] || [ "x${ACTION}" == "xPRE_DEPLOY" ]; then
        echo "Test harbor registry: ${DOCKER_REGISTRY}"
        #curl -X GET --connect-timeout 30 -I ${DOCKER_REGISTRY}/v2/ 2>/dev/null | grep 'HTTP/1.1 200 OK' > /dev/null
        curl --connect-timeout 30 -I ${DOCKER_REGISTRY}/api/projects 2>/dev/null | grep 'HTTP/1.1 200 OK' > /dev/null

        echo "Check image EXIST or NOT: ${ToImage}"
        #Image_Check=$(echo ${ToImage} | sed 's/\([^/]\+\)\([^:]\+\):/\1\/v2\2\/manifests\//')
        ImageCheck_Harbor=$(echo ${ToImage} | sed 's/\([^/]\+\)\([^:]\+\):/\1\/api\/repositories\2\/tags\//')
        Responed_Code=$(curl -u${User}:${PassWord} -so /dev/null -w '%{response_code}' ${ImageCheck_Harbor} || true)
        if [ "${NoCache}" == "true" ] || [ "x${Responed_Code}" != "x200" ] ; then
           if [  "x${BuildCmd}" ==  "x"  ]; then
               echo "Generating Dockerfile"
               echo "FROM ${FromImage}" > Dockerfile
               cat >> Dockerfile <<- EOF
               ${Dockerfile}
EOF
               echo "$(<Dockerfile)"
           else
               echo -e "Building\n${BuildCmd}"
               eval ${BuildCmd}

               echo -e "Packaging\n${TarCmd}"
               cd ${WORKSPACE}
               eval ${TarCmd}

               echo "Copy artifact: ${CodeTarget}"
               cd ${WORKSPACE}
               if [ $(expr match "${CodeTarget}" '.*/') -ne 0 ];then
                 CodeBasename=${CodeTarget##*/}
                 cp -av ${CodeTarget} ${CodeBasename}
               else
                 CodeBasename=${CodeTarget}
               fi
               echo "CodeBasename=#${CodeBasename}#"

               echo "Generate Dockerfile"
               cat > Dockerfile <<- EOF
               FROM ${FromImage}
               MAINTAINER devops <devops@dianrong.com>
               ADD ${CodeBasename} \${AppCode}
EOF
               echo "$(<Dockerfile)"
            fi

            if [ $(expr match "${FromImage}" 'scratch') -eq 0 ];then
               echo "Sync from image: ${FromImage}"
               docker  pull ${FromImage} # 同步上层镜像
            fi

            for i in $(sed -n '/FROM/{/scratch$/!{/[[:space:]]\+AS[[:space:]]\+/ba;s/FROM[[:space:]]\+\(.*\)\([[:space:]]*\)\?$/\1/p;:a;s/FROM[[:space:]]\+\(.*\)[[:space:]]\+AS.*/\1/p}}' Dockerfile | uniq);do
               docker pull ${i} # 同步Dockerfile中镜像
            done
            echo "Sync intermediate image in Dockerfile done"

            echo "Build image and push to registry: ${ToImage}"
            docker build --no-cache=${NoCache} -t ${ToImage} . && docker push ${ToImage} || exit 1 # 开始构建镜像, 成功后Push到仓库

            echo "Remove image: ${ToImage}"
            docker rmi ${ToImage} || echo # 删除镜像
        fi
    fi
}

# 发布、预发布、停止、重启
deploy() {
    if [ "x${ACTION}" == "xSTOP" ]; then
        # 停止当前实例
      kubectl delete -f ${AppName}-deploy.yaml
    elif [ "x${ACTION}" == "xRESTART" ]; then
      kubectl delete -f k8s-deploy-script.yaml
      sleep 10s
      kubectl apply -f ${AppName}-deploy.yaml
    elif [ "x${ACTION}" == "xDEPLOY" ]; then
      kubectl apply -f ${AppName}-deploy.yaml
    fi
}

# 函数执行
init
docker_login
build


cat > ${WORKSPACE}/${AppName}-deploy.yaml <<- EOF
#####################################################
#
#          ${ACTION} Deployment
#
#####################################################
apiVersion: apps/v1beta2 # for versions before 1.8.0 use apps/v1beta1
kind: Deployment
metadata:
  name: ${AppName}
  namespace: ${NameSpace}
  labels:
    app: ${AppName}
    version: ${GitBranch}
    AppEnv: ${AppEnv}
spec:
  replicas: ${Replicas}
  selector:
    matchLabels:
      app: ${AppName}
  template:
    metadata:
      labels:
        app: ${AppName}
    spec:
      containers:
      - name: ${AppName}
        image: ${ToImage}
        ports:
        - containerPort: ${ContainerPort}
        livenessProbe:
          httpGet:
            path: ${HealthCheckURL}
            port: ${ContainerPort}
          initialDelaySeconds: 90
          timeoutSeconds: 5
          periodSeconds: 5
        readinessProbe:
          httpGet:
            path: ${HealthCheckURL}
            port: ${ContainerPort}
          initialDelaySeconds: 5
          timeoutSeconds: 5
          periodSeconds: 5
        # configmap env
        env:
        - name: CFG_LABEL
          value: ${CFG_LABEL}
        - name: CFG_ADDR
          valueFrom:
            configMapKeyRef:
              name: ${ConfigMap}
              key: CFG_ADDR
        - name: DR_CFG_ZOOKEEPER_ENV_URL
          valueFrom:
            configMapKeyRef:
              name: ${ConfigMap}
              key: DR_CFG_ZOOKEEPER_ENV_URL
        - name: CFG_FILES
          valueFrom:
            configMapKeyRef:
              name: ${ConfigMap}
              key: CFG_FILES
        # configMap volume
        volumeMounts:
          - name: applogs
            mountPath: /volume_logs/
      volumes:
        - name: applogs
          hostPath:
            path: /opt/app_logs/${AppName}
      imagePullSecrets:
      - name: ${ImagePullSecrets}

---
apiVersion: v1
kind: Service
metadata:
  name: ${AppName}
  namespace: ${NameSpace}
  labels:
    app: ${AppName}
spec:
  ports:
  - port: ${ContainerPort}
    targetPort: ${ContainerPort}
  selector:
   app: ${AppName}
---
kind: ConfigMap
apiVersion: v1
metadata:
   name: ${ConfigMap}
   namespace: ${NameSpace}
data:
  CFG_ADDR: ${CFG_ADDR}
  DR_CFG_ZOOKEEPER_ENV_URL: ${DR_CFG_ZOOKEEPER_ENV_URL}
  CFG_FILES: ${Cfs_Files}


EOF

# 执行部署

deploy

# 打印配置

cat ${WORKSPACE}/${AppName}-deploy.yaml
```

# 参考

- [Set Up A CI/CD Pipeline With Kubernetes](https://www.linux.com/tutorials/set-cicd-pipeline-kubernetes-part-1-overview/)
- [jenkins-kubernetes-plugin](https://gitee.com/surenpi/kubernetes-plugin)
- [初试 Jenkins 使用 Kubernetes Plugin 完成持续构建与发布](https://cloud.tencent.com/developer/article/1433248)
