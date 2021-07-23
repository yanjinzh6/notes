---
title: kubectl-rollout
date: 2021-02-01 12:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubectl-rollout
photo:
---

https://blog.csdn.net/liumiaocn/article/details/104214812

kubectl rollout 可以对 Deployment, DaemonSet 和 StatefulSet 进行控制, 这篇文章以 Deployment 为例, 对控制方式进行说明.

# rollout 常见操作

| 子命令 | 功能说明 |
| --- | --- |
| history | 查看 rollout 操作历史 |
| pause | 将提供的资源设定为暂停状态 |
| restart | 重启某资源 |
| resume | 将某资源从暂停状态恢复正常 |
| status | 查看 rollout 操作状态 |
| undo | 回滚前一 rollout |

# 事前准备

使用如下 YAML 文件创建 Deployment

```
[root@host131 Deployment]# cat v1.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: busybox-v1
  template:
    metadata:
      labels:
        app: busybox-v1
    spec:
      containers:
      - name: busybox-host
        image: busybox:1.31.1
        command: ["sleep"]
        args: ["1000"]
...
[root@host131 Deployment]#
```

创建 Deployment 并确认

```
[root@host131 Deployment]# kubectl create -f v1.yaml
deployment.apps/busybox-deployment-v1 created
[root@host131 Deployment]# kubectl get deployment -o wide
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS     IMAGES           SELECTOR
busybox-deployment-v1   3/3     3            3           7s    busybox-host   busybox:1.31.1   app= busybox-v1
[root@host131 Deployment]# kubectl get pods -o wide
NAME                                     READY   STATUS    RESTARTS   AGE   IP             NODE              NOMINATED NODE   READINESS GATES
busybox-deployment-v1-7bfdbd9656-ch65w   1/1     Running   0          14s   10.254.152.6   192.168.163.131   <none>           <none>
busybox-deployment-v1-7bfdbd9656-f68s2   1/1     Running   0          14s   10.254.152.5   192.168.163.131   <none>           <none>
busybox-deployment-v1-7bfdbd9656-s6f6j   1/1     Running   0          14s   10.254.152.7   192.168.163.131   <none>           <none>
[root@host131 Deployment]# kubectl exec -it busybox-deployment-v1-7bfdbd9656-ch65w sh
/ # ps -ef
PID   USER     TIME  COMMAND
    1 root      0:00 sleep 1000
    6 root      0:00 sh
   11 root      0:00 ps -ef
/ # busybox | grep BusyBox
BusyBox v1.31.1 (2019-12-23 19:20:27 UTC) multi-call binary.
BusyBox is copyrighted by many authors between 1998-2015.
	BusyBox is a multi-call binary that combines many common Unix
	link to busybox for each function they wish to use and BusyBox
/ #
```

# 操作之: history

| 子命令 | 功能说明 |
| --- | --- |
| history | 查看 rollout 操作历史 |

```
[root@host131 Deployment]# kubectl rollout history deployment busybox-deployment-v1
deployment.apps/busybox-deployment-v1
REVISION  CHANGE-CAUSE
1         <none>

[root@host131 Deployment]#
```

# 操作之: status

| 子命令 | 功能说明 |
| --- | --- |
| status | 查看 rollout 操作状态 |

```
[root@host131 Deployment]# kubectl rollout status deployment busybox-deployment-v1
deployment "busybox-deployment-v1" successfully rolled out
[root@host131 Deployment]#
```

# 操作之: pause

| 子命令 | 功能说明 |
| --- | --- |
| pause | 将提供的资源设定为暂停状态 |

```
[root@host131 Deployment]# kubectl rollout pause deployment busybox-deployment-v1
deployment.apps/busybox-deployment-v1 paused
[root@host131 Deployment]#
```

可以通过 describe 命令确认到 Progressing 的状态:

```
[root@host131 Deployment]# kubectl describe deployment busybox-deployment-v1 | grep Progressing
  Progressing    Unknown  DeploymentPaused
[root@host131 Deployment]#
```

# 操作之: resume

| 子命令 | 功能说明 |
| --- | --- |
| resume | 将某资源从暂停状态恢复正常 |

```
[root@host131 Deployment]# kubectl rollout resume deployment busybox-deployment-v1
deployment.apps/busybox-deployment-v1 resumed
[root@host131 Deployment]#
```

可以通过 describe 命令确认到 Progressing 的状态:

```
[root@host131 Deployment]# kubectl describe deployment busybox-deployment-v1 | grep Progressing
  Progressing    True    NewReplicaSetAvailable
[root@host131 Deployment]#
```

# 操作之: restart

| 子命令 | 功能说明 |
| --- | --- |
| restart | 重启某资源 |

```
[root@host131 Deployment]# kubectl get deployment -o wide
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS     IMAGES           SELECTOR
busybox-deployment-v1   3/3     3            3           10m   busybox-host   busybox:1.31.1   app= busybox-v1
[root@host131 Deployment]# kubectl get pods -o wide
NAME                                     READY   STATUS    RESTARTS   AGE   IP             NODE              NOMINATED NODE   READINESS GATES
busybox-deployment-v1-7bfdbd9656-ch65w   1/1     Running   0          10m   10.254.152.6   192.168.163.131   <none>           <none>
busybox-deployment-v1-7bfdbd9656-f68s2   1/1     Running   0          10m   10.254.152.5   192.168.163.131   <none>           <none>
busybox-deployment-v1-7bfdbd9656-s6f6j   1/1     Running   0          10m   10.254.152.7   192.168.163.131   <none>           <none>
[root@host131 Deployment]# kubectl rollout history deployment busybox-deployment-v1
deployment.apps/busybox-deployment-v1
REVISION  CHANGE-CAUSE
1         <none>

[root@host131 Deployment]#
[root@host131 Deployment]# kubectl rollout restart deployment busybox-deployment-v1
deployment.apps/busybox-deployment-v1 restarted
[root@host131 Deployment]# kubectl rollout status deployment busybox-deployment-v1
Waiting for deployment "busybox-deployment-v1" rollout to finish: 1 old replicas are pending termination...
Waiting for deployment "busybox-deployment-v1" rollout to finish: 1 old replicas are pending termination...
deployment "busybox-deployment-v1" successfully rolled out
[root@host131 Deployment]# kubectl rollout history deployment busybox-deployment-v1
deployment.apps/busybox-deployment-v1
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

[root@host131 Deployment]# kubectl get deployment -o wide
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS     IMAGES           SELECTOR
busybox-deployment-v1   3/3     3            3           11m   busybox-host   busybox:1.31.1   app= busybox-v1
[root@host131 Deployment]# kubectl get pods -o wide
NAME                                     READY   STATUS        RESTARTS   AGE   IP              NODE              NOMINATED NODE   READINESS GATES
busybox-deployment-v1-7bfdbd9656-ch65w   1/1     Terminating   0          11m   10.254.152.6    192.168.163.131   <none>           <none>
busybox-deployment-v1-7bfdbd9656-f68s2   1/1     Terminating   0          11m   10.254.152.5    192.168.163.131   <none>           <none>
busybox-deployment-v1-7bfdbd9656-s6f6j   1/1     Terminating   0          11m   10.254.152.7    192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-kjt5z    1/1     Running       0          25s   10.254.152.10   192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-r9xjk    1/1     Running       0          29s   10.254.152.8    192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-vnnhx    1/1     Running       0          27s   10.254.152.9    192.168.163.131   <none>           <none>
[root@host131 Deployment]#
```

# 操作之: undo

| 子命令 | 功能说明 |
| --- | --- |
| undo | 回滚前一 rollout |

*   回滚前状态确认

```
[root@host131 Deployment]# kubectl get pods -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP              NODE              NOMINATED NODE   READINESS GATES
busybox-deployment-v1-7c6899456-kjt5z   1/1     Running   0          66s   10.254.152.10   192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-r9xjk   1/1     Running   0          70s   10.254.152.8    192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-vnnhx   1/1     Running   0          68s   10.254.152.9    192.168.163.131   <none>           <none>
[root@host131 Deployment]# kubectl exec -it busybox-deployment-v1-7c6899456-kjt5z sh
/ # ps -ef
PID   USER     TIME  COMMAND
    1 root      0:00 sleep 1000
    6 root      0:00 sh
   11 root      0:00 ps -ef
/ # busybox | grep BusyBox
BusyBox v1.31.1 (2019-12-23 19:20:27 UTC) multi-call binary.
BusyBox is copyrighted by many authors between 1998-2015.
	BusyBox is a multi-call binary that combines many common Unix
	link to busybox for each function they wish to use and BusyBox
/ #
```

*   修改配置文件
    修改启动命令的 sleep 时间以及 busybox 的版本, 详细如下所示

```
[root@host131 Deployment]# cat v1.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox-deployment-v1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: busybox-v1
  template:
    metadata:
      labels:
        app: busybox-v1
    spec:
      containers:
      - name: busybox-host
        image: busybox:1.30.1
        command: ["sleep"]
        args: ["10000"]
...
[root@host131 Deployment]#
```

*   修改当前 Deployment 和 Pod 信息

```
[root@host131 Deployment]# kubectl apply -f v1.yaml
Warning: kubectl apply should be used on resource created by either kubectl create --save-config or kubectl apply
deployment.apps/busybox-deployment-v1 configured
[root@host131 Deployment]#
[root@host131 Deployment]# kubectl get deployment -o wide
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS     IMAGES           SELECTOR
busybox-deployment-v1   3/3     3            3           15m   busybox-host   busybox:1.30.1   app= busybox-v1
[root@host131 Deployment]# kubectl get pods -o wide
NAME                                     READY   STATUS        RESTARTS   AGE     IP              NODE              NOMINATED NODE   READINESS GATES
busybox-deployment-v1-5fd9b98ccc-9h6gd   1/1     Running       0          17s     10.254.152.6    192.168.163.131   <none>           <none>
busybox-deployment-v1-5fd9b98ccc-l67nw   1/1     Running       0          15s     10.254.152.7    192.168.163.131   <none>           <none>
busybox-deployment-v1-5fd9b98ccc-vfkdv   1/1     Running       0          19s     10.254.152.5    192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-kjt5z    1/1     Terminating   0          4m25s   10.254.152.10   192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-r9xjk    1/1     Terminating   0          4m29s   10.254.152.8    192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-vnnhx    1/1     Terminating   0          4m27s   10.254.152.9    192.168.163.131   <none>           <none>
[root@host131 Deployment]#
[root@host131 Deployment]# kubectl get pods -o wide
NAME                                     READY   STATUS    RESTARTS   AGE   IP             NODE              NOMINATED NODE   READINESS GATES
busybox-deployment-v1-5fd9b98ccc-9h6gd   1/1     Running   0          45s   10.254.152.6   192.168.163.131   <none>           <none>
busybox-deployment-v1-5fd9b98ccc-l67nw   1/1     Running   0          43s   10.254.152.7   192.168.163.131   <none>           <none>
busybox-deployment-v1-5fd9b98ccc-vfkdv   1/1     Running   0          47s   10.254.152.5   192.168.163.131   <none>           <none>
[root@host131 Deployment]# kubectl exec -it busybox-deployment-v1-5fd9b98ccc-9h6gd sh
/ # ps -ef
PID   USER     TIME  COMMAND
    1 root      0:00 sleep 10000
    6 root      0:00 sh
   11 root      0:00 ps -ef
/ # busybox | grep BusyBox
BusyBox v1.30.1 (2019-05-09 01:23:43 UTC) multi-call binary.
BusyBox is copyrighted by many authors between 1998-2015.
	BusyBox is a multi-call binary that combines many common Unix
	link to busybox for each function they wish to use and BusyBox
/ #
```

查看历史信息

```
[root@host131 Deployment]# kubectl rollout history deployment busybox-deployment-v1
deployment.apps/busybox-deployment-v1
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
3         <none>

[root@host131 Deployment]#
```

*   回滚
    此处使用不带参数的回滚, 表示回滚至上一个版本, 也可以指定回滚的具体版本

```
[root@host131 Deployment]# kubectl rollout undo deployment busybox-deployment-v1
deployment.apps/busybox-deployment-v1 rolled back
[root@host131 Deployment]# kubectl rollout status deployment busybox-deployment-v1
deployment "busybox-deployment-v1" successfully rolled out
[root@host131 Deployment]# kubectl rollout history deployment busybox-deployment-v1
deployment.apps/busybox-deployment-v1
REVISION  CHANGE-CAUSE
1         <none>
3         <none>
4         <none>

[root@host131 Deployment]#
```

*   回滚结果确认

```
[root@host131 Deployment]# kubectl get deployment -o wide
NAME                    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS     IMAGES           SELECTOR
busybox-deployment-v1   3/3     3            3           19m   busybox-host   busybox:1.31.1   app= busybox-v1
[root@host131 Deployment]#
[root@host131 Deployment]# kubectl get pods -o wide
NAME                                     READY   STATUS        RESTARTS   AGE     IP              NODE              NOMINATED NODE   READINESS GATES
busybox-deployment-v1-5fd9b98ccc-l67nw   0/1     Terminating   0          4m10s   10.254.152.7    192.168.163.131   <none>           <none>
busybox-deployment-v1-5fd9b98ccc-vfkdv   0/1     Terminating   0          4m14s   <none>          192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-2qbgc    1/1     Running       0          36s     10.254.152.9    192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-cwpw7    1/1     Running       0          34s     10.254.152.10   192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-qzh44    1/1     Running       0          39s     10.254.152.8    192.168.163.131   <none>           <none>
[root@host131 Deployment]#
[root@host131 Deployment]# kubectl get pods -o wide
NAME                                    READY   STATUS    RESTARTS   AGE   IP              NODE              NOMINATED NODE   READINESS GATES
busybox-deployment-v1-7c6899456-2qbgc   1/1     Running   0          59s   10.254.152.9    192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-cwpw7   1/1     Running   0          57s   10.254.152.10   192.168.163.131   <none>           <none>
busybox-deployment-v1-7c6899456-qzh44   1/1     Running   0          62s   10.254.152.8    192.168.163.131   <none>           <none>
[root@host131 Deployment]# kubectl exec -it busybox-deployment-v1-7c6899456-2qbgc sh
/ # ps -ef
PID   USER     TIME  COMMAND
    1 root      0:00 sleep 1000
    6 root      0:00 sh
   11 root      0:00 ps -ef
/ # busybox | grep BusyBox
BusyBox v1.31.1 (2019-12-23 19:20:27 UTC) multi-call binary.
BusyBox is copyrighted by many authors between 1998-2015.
	BusyBox is a multi-call binary that combines many common Unix
	link to busybox for each function they wish to use and BusyBox
/ #
```
