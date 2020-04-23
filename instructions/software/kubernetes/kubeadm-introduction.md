---
title: KubeAdm 部署
date: 2020-1-8 11:00:00
tags: 'kubernetes'
categories:
  - ['使用说明', '软件']
permalink: kubeadm-introduction
photo:
---

## 简介

[Kubernetes](https://kubernetes.io/zh/) 是用于自动部署, 扩展和管理容器化应用程序的开源系统. 它将组成应用程序的容器组合成逻辑单元, 以便于管理和服务发现. Kubernetes 源自 Google 15 年生产环境的运维经验, 同时凝聚了社区的最佳创意和实践.

[Kubeadm](https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm/) 是一个工具, 它提供了 kubeadm init 以及 kubeadm join 这两个命令作为快速创建 kubernetes 集群的最佳实践.

<!-- more -->

## 前提

### 关闭 Swap

1. 编辑 <code>/etc/fstab</code> 文件, 注释掉引用 <code>swap</code> 的行
2. 输入命令 <code>sudo swapoff -a</code>
3. 参考 [Kubelet/Kubernetes should work with Swap Enabled](https://github.com/kubernetes/kubernetes/issues/53533)

### 安装 Docker

#### 安装参考 [安装 Docker-阿里云](https://www.alibabacloud.com/help/zh/doc-detail/60742.htm)

```sh
## step 1: 安装必要的一些系统工具
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
## step 2: 安装GPG证书
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
## Step 3: 写入软件源信息
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
## Step 4: 更新并指定版本安装Docker-CE  (这里安装 18.06.1~ce~3-0~ubuntu)
sudo apt-get -y update
apt-cache madison docker-ce
sudo apt-get -y install docker-ce
```

#### 配置镜像加速

参考 [Docker 镜像加速器](https://yq.aliyun.com/articles/29941)

```sh
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://registry.docker-cn.com", "https://[阿里云分配的私有地址].mirror.aliyuncs.com"]
}
EOF
```

#### 配置 Docker 的 proxy

Kubernetes 的一些 docker 镜像是需要借助梯子才能拉取到的, 为此需要为 Docker 配置 Proxy.

参考 [httphttps-proxy](https://docs.docker.com/config/daemon/systemd/#httphttps-proxy)

配置文件 <code>/etc/systemd/system/docker.service.d/http-proxy.conf</code>

```sh
[Service]
Environment="HTTP_PROXY=http://192.168.137.1:1080/"
Environment="HTTPS_PROXY=http://192.168.137.1:1080/"
Environment="NO_PROXY=localhost,127.0.0.0/8,10.0.0.0/8,172.16.0.0/16,192.168.0.0/16,https://registry.docker-cn.com,https://[阿里云分配的私有地址].mirror.aliyuncs.com"
```

这里的 NO_PROXY 需要上面配置的镜像加速地址添加进去

重启 Docker

```sh
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 部署

### 安装 kubeadm

#### 参考 [Installing kubeadm](https://kubernetes.io/docs/setup/independent/install-kubeadm/)

这里使用 [kubeadm](https://kubernetes.io/zh/docs/) 简化集群部署

```sh
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
## 写入文件
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

需要设置系统代理

#### 使用国内源

```sh
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
```

? 目前 kubernetes 还没有 Ubuntu18.04 的编好的版本, 用的 16.04 xenial 的二进制文件

### 配置

#### 配置 master 节点

##### 使用 [kubeadm init 初始化 master 节点](https://kubernetes.io/zh/docs/reference/setup-tools/kubeadm/kubeadm-init/)

```sh
kubeadm init --pod-network-cidr=10.244.0.0/16
```

init 命令常用主要参数

- --apiserver-advertise-address 类型: string, API Server 将要广播的监听地址, 如指定为 `0.0.0.0` 将使用缺省的网卡地址, 不指定的话会自动检测 IP
- --ignore-preflight-errors 类型: stringSlice, 忽视检查项错误列表, 列表中的每一个检查项如发生错误将被展示输出为警告, 而非错误, 例如: 'IsPrivilegedUser,Swap'. 如填写为 'all' 则将忽视所有的检查项错误
- --kubernetes-version 类型: string, 缺省值: "stable-1", 为 control plane 选择一个特定的 Kubernetes 版本
- --pod-network-cidr 类型: string, 指明 pod 网络可以使用的 IP 地址段, 如果设置了这个参数, control plane 将会为每一个节点自动分配 CIDRs, 使用 flannel 网络, 需要指定为 10.244.0.0/16

kubeadm init 输出的 token 用于 master 和加入节点间的身份认证, token 是机密的, 需要保证它的安全, 因为拥有此标记的人都可以随意向集群中添加节点

配置完成后会自动从 gcr.io 拉取镜像, 成功后输出

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

按照要求将 kubectl 配置文件设置好

##### 获取 nodes 状态

```sh
kubectl get nodes
```

这时会出现一个 master 节点并显示状态 notReady

#### 安装网络插件

[flannel](https://github.com/coreos/flannel)

```
For Kubernetes v1.7+ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

等待拉取镜像完成后再次获取 nodes 状态, 部署成功后会显示 Ready

其他网络插件 [addons](https://kubernetes.io/zh/docs/concepts/cluster-administration/addons/)

#### 添加从节点

节点安装配置完 docker 后, 使用 kubeadm join 加入集群, token 24 小时后过期, 可以使用集群内的机器发送命令 <code>kubeadm token create</code> 生成新的 token

```sh
yanjin@yanjin-ubuntu-master:~$ kubectl get nodes
NAME                   STATUS   ROLES    AGE     VERSION
yanjin-ubuntu-master   Ready    master   3d23h   v1.13.4
yanjin-ubuntu-node     Ready    <none>   3d22h   v1.13.4
yanjin-ubuntu-node2    Ready    <none>   43h     v1.13.4
```

#### 部署 [nginx ingress](https://kubernetes.github.io/ingress-nginx/deploy/)

##### 安装

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml
```

官方的脚本已经内置有基于角色的访问控制的 ServiceAccount

##### 验证安装命令

```sh
kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx --watch
yanjin@yanjin-ubuntu-master:~$ kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx --watch
NAMESPACE       NAME                                        READY   STATUS    RESTARTS   AGE
ingress-nginx   nginx-ingress-controller-797b884cbc-dxl99   1/1     Running   0          7m11s
```

通过 <code>Ctrl + c</code> 退出

##### 检查安装版本

进入到相应 pod 中运行 <code>/nginx-ingress-controller --version</code> 命令

```sh
POD_NAMESPACE=ingress-nginx
POD_NAME=$(kubectl get pods -n $POD_NAMESPACE -l app.kubernetes.io/name=ingress-nginx -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD_NAME -n $POD_NAMESPACE -- /nginx-ingress-controller --version

nginx-ingress-controller --version
-------------------------------------------------------------------------------
NGINX Ingress controller
  Release:    0.23.0
  Build:      git-be1329b22
  Repository: https://github.com/kubernetes/ingress-nginx
-------------------------------------------------------------------------------
```

##### 暴露服务

用不同云服务商的就找一下提供的专用版本, 云服务商有提供负载均衡服务, 如果是个人机器部署需要创建一个 ingress service 将 nginx-ingress-controller 暴露到集群外

可以采用 NodePort 和 externalIPs 的方式暴露服务

```yaml
## ingress-service.yaml
## 通过 externalIPs 提供集群外访问
apiVersion: v1
kind: Service
metadata:
  name: ingress-nginx
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  externalIPs:
    # 集群机器
    - 192.168.137.200
    - 192.168.137.201
    - 192.168.137.202
  ports:
    - name: http
      port: 80
      targetPort: 80
      protocol: TCP
    - name: https
      port: 443
      targetPort: 443
      protocol: TCP
  selector:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
```

##### 使用 tls https

[配置 tls](https://github.com/kubernetes/contrib/blob/master/ingress/controllers/nginx/examples/tls/README.md)
使用 Cloudflare 可以自动生成证书和 https 转接服务

这里使用两个不同的证书

1. 通过在父目录中创建 rc 目录来部署控制器
2. 为 foo.bar.com 创建密钥
3. 创建 rc-ssl.yaml 文件

```sh
## 为 foo.bar.com 主机创建SSL证书
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /tmp/tls.key -out /tmp/tls.crt -subj "/CN=foo.bar.com"
## 将 SSL 证书存储在一个密钥中, 为了安全 ingress 所有的资源 (凭证, 路由, 服务) 必须在同一命名空间下面
kubectl create secret tls foo-secret --key /tmp/tls.key --cert /tmp/tls.crt
```

```yaml
## 创建一个 Ingress tls 规则
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: foo
  namespace: default
  annotations:
    # 默认情况需要 HTTPS
    nginx.ingress.kubernetes.io/secure-backends: 'true'
    # v0.21.0 后上面替换为
    nginx.ingress.kubernetes.io/backend-protocol: 'HTTPS'
spec:
  tls:
    - hosts:
        - foo.bar.com
      secretName: foo-secret
  rules:
    - host: foo.bar.com
      http:
        paths:
          - backend:
              serviceName: echoheaders-x
              servicePort: 443
            path: /
```

```sh
kubectl create -f rc-ssl.yaml
```

#### 安装 dashboard UI

##### 部署 dashboard UI

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```

##### 使用 ingress 来暴露 dashboard

```yaml
## dashboard-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: dashboard-ingress
  namespace: kube-system
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: 'true'
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/secure-backends: 'true'
    nginx.ingress.kubernetes.io/backend-protocol: 'HTTPS'
spec:
  tls:
    - hosts:
        - www.yj.com
      secretName: kubernetes-dashboard-certs
  rules:
    - host: www.yj.com
      http:
        paths:
          - path: /
            backend:
              serviceName: kubernetes-dashboard
              servicePort: 443
```

主机通过指定 hosts 文件来访问 [url](https://www.yj.com) 即可访问 kubernetes-dashboard 服务

```sh
externalIPs www.yj.com
```

##### 创建简单的用户权限访问

参考 [admin-privileges](https://github.com/kubernetes/dashboard/wiki/Access-control#admin-privileges)

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: dashboard-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: dashboard-admin
    namespace: kube-system
```

这里创建了一个 dashboard-admin 的 ClusterRoleBinding
通过获取 dashboard-admin 的 token 来登录 dashboard UI

```sh
## 获取 kube-system 命名空间下的 dashboard-admin secret 然后输出该 secret 的详情, 里面有 token 信息, 复制 token 信息登录 dashboard UI
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep dashboard-admin | awk '{print $1}')
```

##### 更新

官方文档说 dashboard 安装后不会自动更新, 如果要更新需要先删除所有已经部署的 pods, 然后 k8s 就会自动用新的配置重新部署

```sh
$ kubectl -n kube-system delete $(kubectl -n kube-system get pod -o name | grep dashboard)
pod "kubernetes-dashboard-3313488171-7706x" deleted
pod "kubernetes-dashboard-3313488171-ddkqd" deleted
pod "kubernetes-dashboard-3313488171-dpf9t" deleted
pod "kubernetes-dashboard-3313488171-jdz1n" deleted
pod "kubernetes-dashboard-3313488171-sxc9n" deleted
```

### 卸载集群

首先要排除节点, 并确保在关闭节点之前要清空节点

在主节点上运行

```sh
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
```

然后在需要移除的节点上, 重置 kubeadm 的安装状态

```sh
kubeadm reset
```
