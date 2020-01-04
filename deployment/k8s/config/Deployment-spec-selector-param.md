---
title: k8s 更新 Deployment 组件配置问题
date: 2020-01-02 17:00:00
tags: '配置'
categories:
  - ['部署', 'k8s']
permalink: Deployment-spec-selector-param
photo:
---

# 错误

好几个月前部署的 Jenkins 后便再也没有改动配置文件了,今天提示 Jenkins 版本落后才发现配置文件配的是不拉取镜像的,所有修改成 `imagePullPolicy: Always` 准备更新镜像便发现问题.

```sh
➜  jenkins kbc apply -f jenkins.yml
error: error validating "jenkins.yml": error validating data: ValidationError(Deployment.spec): missing required field "selector" in io.k8s.api.apps.v1.DeploymentSpec; if you choose to ignore these errors, turn validation off with --validate=false
```

<!-- more -->

# 版本

```sh
➜  jenkins kbc version
Client Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:20:10Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
Server Version: version.Info{Major:"1", Minor:"17", GitVersion:"v1.17.0", GitCommit:"70132b0f130acc0bed193d9ba59dd186f0e634cf", GitTreeState:"clean", BuildDate:"2019-12-07T21:12:17Z", GoVersion:"go1.13.4", Compiler:"gc", Platform:"linux/amd64"}
```

# 原因

查看[官方文档](https://kubernetes.io/zh/docs/concepts/workloads/controllers/deployment/)发现里面有个注意需要配置.

> 注意： You must specify an appropriate selector and pod template labels in a Deployment (in this case, app = nginx). That is, don’t overlap with other controllers (including other Deployments, ReplicaSets, StatefulSets, etc.). Kubernetes doesn’t stop you from overlapping, and if multiple controllers have overlapping selectors, those controllers may fight with each other and won’t behave correctly.

大概意思就是说: Deployment 必须指定适当的选择器和 pod 模版标签, 请勿与其他控制器 (including other Deployments, ReplicaSets, StatefulSets, etc.) 配置重复, 有可能导致互相竞争, 无法正常运行.

这里的选择器应该是指定模版对应的标签, 看一下官方的实例.

```yml
controllers/nginx-deployment.yaml

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.4
        ports:
        - containerPort: 80
```

这里的 selector 匹配的是 template 中配置的 labels, 另外很多配置中都在不同的层级中存在大量相同的 labels, 有时候确实很难看清楚, 还是应该定义好名称规范.

# 总结

* Deployment 必须配置 selector
* 定义 template 时必须定义 labels
* Deployment.spec.selector 必须与 template.labels 对应上
* template 里面定义的内容会应用到下面所有的副本集里面, 在 template.spec.containers 里面不能定义labels标签.