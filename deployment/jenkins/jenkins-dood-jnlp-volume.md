---
title: Jenkins dood 模式 jnlp 使用
date: 2020-05-04 18:00:00
tags: 'Jenkins'
categories:
  - ['部署', 'CI']
permalink: jenkins-dood-jnlp-volume
photo:
---

## 简介

Jenkins 采用 k8s 的本地集群, 其任务使用自定义的 jnlp 镜像来跑, 这样有个好处就是不同的任务可以使用不同的 jnlp, 任务处理完成后会直接结束掉容器减少资源消耗

之前没有配置 `volumeMounts` 导致每次发布新版本都要全量的从 git 上面拉取整个项目和相关工具, 造成不必要的浪费

<!-- more -->

## 添加 volumeMounts

Agent jnlp-slave-3dh8s is provisioned from template Kubernetes Pod Template

```yaml
apiVersion: "v1"
kind: "Pod"
metadata:
  annotations: {}
  labels:
    jenkins: "slave"
    jenkins/label: "slave-agent"
  name: "jnlp-slave-3dh8s"
spec:
  containers:
  - env:
    - name: "JENKINS_SECRET"
      value: "********"
    - name: "JENKINS_AGENT_NAME"
      value: "jnlp-slave-3dh8s"
    - name: "JENKINS_NAME"
      value: "jnlp-slave-3dh8s"
    - name: "JENKINS_AGENT_WORKDIR"
      value: "/home/jenkins/agent"
    - name: "JENKINS_URL"
      value: "http://jenkins.jenkins-ops.svc.cluster.local/"
    image: "127.0.0.1:32000/yanjinzh6/jnlp-slave-hexo:1"
    imagePullPolicy: "Always"
    name: "jnlp"
    resources:
      limits: {}
      requests: {}
    securityContext:
      privileged: false
    tty: true
    volumeMounts:
    - mountPath: "/home/jenkins/agent/workspace"
      name: "volume-1"
      readOnly: false
    - mountPath: "/var/run/docker.sock"
      name: "volume-0"
      readOnly: false
    - mountPath: "/home/jenkins/agent"
      name: "workspace-volume"
      readOnly: false
    workingDir: "/home/jenkins/agent"
  hostNetwork: false
  nodeSelector:
    beta.kubernetes.io/os: "linux"
  restartPolicy: "Never"
  securityContext: {}
  serviceAccount: "jenkins"
  volumes:
  - hostPath:
      path: "/var/run/docker.sock"
    name: "volume-0"
  - hostPath:
      path: "/data/jenkins_home/jnlp-hexo/"
    name: "volume-1"
  - emptyDir:
      medium: ""
    name: "workspace-volume"
```

这里 `JENKINS_AGENT_WORKDIR` 变量设置了 `jenkins_home` 的目录, 只需要添加一个本地的文件路径即可

当然, 一开始没意识到采用 dood 模式都是用到宿主的容器的, 一直在 Jenkins 映射的目录里面查找刚刚配置的 hostPath, 一直蒙圈没找到, 后面知道了其实在宿主机器里面的路径.

## 项目配置

由于使用到了 Jenkins 的密钥, 还有项目配置的 groove 脚本也看着挺顺眼的, 所以就直接使用 Jenkins 管道项目

```groove
node('slave-agent') {
    def home = '/home/jenkins'
    def workspace = 'hexo-notes'
    def next = 'next'
    def cos = 'hexo-cos-tool'
    def source = 'source/_posts'

    environment {
        HTTP_PROXY='socks5://172.17.0.1:1080'
        HTTPS_PROXY='socks5://172.17.0.1:1080'
        ALL_PROXY='socks5://172.17.0.1:1080'
    }

    stage('clone notes') {
        sh "cp -r ${home}/. ./ || echo 'copy home'"
        sh "mkdir -p ${workspace}/${source}/ || echo 'create folder ${source}'"
        dir("${workspace}/${source}/") {
            def exise = fileExists 'notes'
            if (exise) {
                dir('notes/') {
                    sh 'git pull'
                }
            } else {
                sh "export ALL_PROXY='socks5://172.17.0.1:1080' && git clone https://github.com/yanjinzh6/notes.git"
            }
        }
    }
    stage('init workspace') {
        dir("${workspace}/") {
            sh "yarn --registry=https://registry.npm.taobao.org && yarn build"
            sh "cp ./fixed/inject.swig ./node_modules/hexo-math/asset/inject.swig"
        }
    }
    stage('init cos tool') {
        def exise = fileExists "${cos}"
        if (exise) {
            dir("${cos}/") {
                sh 'git pull'
            }
        } else {
            sh "export ALL_PROXY='socks5://172.17.0.1:1080' && git clone https://github.com/yanjinzh6/hexo-cos-tool.git"
        }
        dir("${cos}/") {
            sh "yarn --registry=https://registry.npm.taobao.org"
        }
    }
    stage('deploy') {
        withCredentials([string(credentialsId: 'xxx', variable: 'cosAppId'), string(credentialsId: 'yyy', variable: 'cosSecret')]) {
            sh "node ${cos}/index.js ${cosAppId} ${cosSecret} yozh-kkk ap-cc ${workspace}/public"
        }
    }
}
```

## 小结

由于项目中经常改变 Hexo 的插件, 好几次因为缓存导致一直以为配置有问题而排查许久, 现在使用 `volumeMounts` 更得考虑到如何清理 hexo 缓存的问题, 如果需要的话, 应该得在项目中主动调用 `hexo clean` 来清理缓存, 现在还没有好的想法如何能在 `package.json`, `_config.yml` 等关键配置改动时自动清理掉以前的缓存, 估计只能根据 `git pull` 拉取回来的记录来处理了

[Kubernetes plugin](https://j.yzer.club/configureClouds/) 插件提供了相应的配置页面, 可以了解一下里面工作空间的配置是否就是需要的配置
