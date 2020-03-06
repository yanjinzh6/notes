---
title: 使用 Secret 安全地分发凭证
date: 2020-03-06 16:30:00
tags: '配置'
categories:
  - ['部署', 'k8s']
permalink: use-secret-credentials
photo:
---

# 简介

在之前 docker-compose 的配置中，通常将一些自定义的参数配置在 `.env` 文件中，并通过 `env-file` 参数来引入配置文件，这样就可以在配置中通过 `${}` 使用参数, 现在使用 Kubernetes 的 `Secret` 组件可以安全将地敏感数据注入到 Pods 中

# 准备数据

将 secret 数据转换为 base-64 形式

在 Linux 中，可以通过如下方式查看 base64 数据

```sh
echo -n 'user' | base64
dXNlcg==
echo -n '123456' | base64
MTIzNDU2
```

# 创建 Secret

这里是一个配置文件，可以用来创建存有用户名和密码的 Secret

```yaml
# pods/inject/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: test-secret
data:
  username: dXNlcg==
  password: MTIzNDU2
```

```sh
kubectl create -f pods/inject/secret.yaml
```

> **注意：**如果想要跳过 Base64 编码的步骤，可以使用 kubectl create secret 命令来创建 Secret：

```sh
kubectl create secret generic test-secret --from-literal=username='user' --from-literal=password='123456'
```

```sh
# 查看 Secret 相关信息
kubectl get secret test-secret
NAME          TYPE     DATA   AGE
test-secret   Opaque   2      16s

# 查看 Secret 相关的更多详细信息
kbc describe secret test-secret
Name:         test-secret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  6 bytes
username:  4 bytes
```

<!-- more -->

# 创建可以通过卷访问 secret 数据的 Pod

配置文件

```yaml
# pods/inject/secret-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-test-pod
spec:
  containers:
    - name: test-container
      image: nginx
      volumeMounts:
          # name must match the volume name below
          - name: secret-volume
            mountPath: /etc/secret-volume
  # The secret data is exposed to Containers in the Pod through a Volume.
  volumes:
    - name: secret-volume
      secret:
        secretName: test-secret
```

操作过程

```sh
# 创建 Pod
kubectl create -f secret-pod.yaml

# 确认 Pod 正在运行
kubectl get pod secret-test-pod
NAME              READY     STATUS    RESTARTS   AGE
secret-test-pod   1/1       Running   0          42m

# 在 Pod 中运行的容器中获取一个 shell
kubectl exec -it secret-test-pod -- /bin/bash

# secret 数据通过挂载在 /etc/secret-volume 目录下的卷暴露在容器中。 在 shell 中，进入 secret 数据被暴露的目录：
cd /etc/secret-volume

#在 shell 中，列出 /etc/secret-volume 目录的文件：
root@secret-test-pod:/etc/secret-volume# ls

# 输出显示了两个文件，每个对应一条 secret 数据：
password username

# 在 shell 中，显示 username 和 password 文件的内容：
root@secret-test-pod:/etc/secret-volume# cat username; echo; cat password; echo

#输出为用户名和密码：
user
123456
```

# 创建通过环境变量访问 secret 数据的 Pod

配置文件

```yaml
# pods/inject/secret-envars-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-envars-test-pod
spec:
  containers:
  - name: envars-test-container
    image: nginx
    env:
    - name: SECRET_USERNAME
      valueFrom:
        secretKeyRef:
          name: test-secret
          key: username
    - name: SECRET_PASSWORD
      valueFrom:
        secretKeyRef:
          name: test-secret
          key: password
```

这里的 `test-secret` 作为一个 key-value 参数输入，通过 `valueFrom` 参数来获取对应 name 和 key 的数据

操作过程

```sh
# 创建 Pod
kubectl create -f https://k8s.io/examples/pods/inject/secret-envars-pod.yaml

# 确认 Pod 正在运行
kubectl get pod secret-envars-test-pod

# 输出
NAME                     READY     STATUS    RESTARTS   AGE
secret-envars-test-pod   1/1       Running   0          4m

# 在 Pod 中运行的容器中获取一个 shell
kubectl exec -it secret-envars-test-pod -- /bin/bash

# 在 shell 中，显示环境变量
root@secret-envars-test-pod:/# printenv

# 输出包括用户名和密码
...
SECRET_USERNAME=user
...
SECRET_PASSWORD=123456
```

# 参考

- [使用 Secret 安全地分发凭证](https://kubernetes.io/zh/docs/tasks/inject-data-application/distribute-credentials-secure/)