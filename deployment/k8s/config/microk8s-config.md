---
title: Microk8s 配置
date: 2020-01-28 22:30:00
tags: '配置'
categories:
  - ['部署', 'k8s']
permalink: microk8s-config
photo:
---

# 安装

安装 Ubuntu service 18.04 的时候已经有 snap 的安装选项, 直接选择了 microk8s, 没想到现在部署一套 kubernetes 本地的集群已经这么方便了, 以前的 kubeadm 已经算是很简配了

<!-- more -->

## snap 安装 microk8s

`snap` 是 **canonical** 公司给出的更高级的包管理的解决方案, 可通过下面命令简单安装

```sh
snap install microk8s --classic --channel=1.13/stable
Download snap "microk8s" (581) from channel "1.13/stable"
```

## snap 代理配置

安装过程比较缓慢, 所以需要进行设置, 由于 snap 不会读取系统环境配置, 所以必须修改配置文件

使用下面的命令可以方便的修改 snap 的环境变量

```sh
systemctl edit snapd.service
```

这里可以先更新编辑器为 `Vim`

```sh
sudo update-alternatives --install "$(which editor)" editor "$(which vim)" 15
sudo update-alternatives --config editor
```

交互式终端需要手动输入数字, 然后按下回车确认

```sh
There are 5 choices for the alternative editor (providing /usr/bin/editor).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /bin/nano            40        auto mode
  1            /bin/ed             -100       manual mode
  2            /bin/nano            40        manual mode
  3            /usr/bin/vim         15        manual mode
  4            /usr/bin/vim.basic   30        manual mode
  5            /usr/bin/vim.tiny    15        manual mode

Press <enter> to keep the current choice[*], or type selection number: 5
update-alternatives: using /usr/bin/vim.tiny to provide /usr/bin/editor (editor) in manual mode
```

再执行编辑配置文件的命令, 设置相应的配置

```sh
systemctl edit snapd.service
```

```conf
[Service]
Environment="HTTP_PROXY=socks5://127.0.0.1:1080/"
Environment="HTTPS_PROXY=socks5://127.0.0.1:1080/"
Environment="NO_PROXY=localhost,127.0.0.0/8,10.0.0.0/8,172.16.0.0/16,192.168.0.0/16"
```

修改完成后理论上配置会生效, 最好还是重启一下

```sh
systemctl daemon-reload
systemctl restart snap
```

## microk8s 配置

```sh
➜  WorkSpace snap list
Name      Version    Rev   Tracking  Publisher   Notes
core      16-2.42.5  8268  stable    canonical✓  core
microk8s  v1.17.2    1173  stable    canonical✓  classic
```

默认的版本已经到了 `v1.17.2` 了, 从 `v1.14` 后默认的容器采用的是 `containerd`, 已经自带了 ctr 客户端了

```sh
➜  WorkSpace microk8s.ctr version
Client:
  Version:  v1.2.5
  Revision: bb71b10fd8f58240ca47fbb579b9d1028eea7c84

Server:
  Version:  v1.2.5
  Revision: bb71b10fd8f58240ca47fbb579b9d1028eea7c84
```

客户端的命令与 docker cli 比较类似

```sh
➜  WorkSpace microk8s.ctr --help
NAME:
   ctr - 
        __
  _____/ /______
 / ___/ __/ ___/
/ /__/ /_/ /
\___/\__/_/

containerd CLI


USAGE:
   ctr [global options] command [command options] [arguments...]

VERSION:
   v1.2.5

COMMANDS:
     plugins, plugin           provides information about containerd plugins
     version                   print the client and server versions
     containers, c, container  manage containers
     content                   manage content
     events, event             display containerd events
     images, image, i          manage images
     leases                    manage leases
     namespaces, namespace     manage namespaces
     pprof                     provide golang pprof outputs for containerd
     run                       run a container
     snapshots, snapshot       manage snapshots
     tasks, t, task            manage tasks
     install                   install a new package
     shim                      interact with a shim directly
     cri                       interact with cri plugin
     help, h                   Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --debug                      enable debug output in logs
   --address value, -a value    address for containerd's GRPC server (default: "/run/containerd/containerd.sock")
   --timeout value              total timeout for ctr commands (default: 0s)
   --connect-timeout value      timeout for connecting to containerd (default: 0s)
   --namespace value, -n value  namespace to use with commands (default: "default") [$CONTAINERD_NAMESPACE]
   --help, -h                   show help
   --version, -v                print the version
```

由于是 microk8s 自带的容器, 所以不能像修改 snap 配置一样, 必须修改位于 microk8s 中的配置文件

```sh
➜  WorkSpace vi /var/snap/microk8s/current/args/containerd-env

# 添加配置
HTTP_PROXY=socks5://127.0.0.1:1080
HTTPS_PROXY=socks5://127.0.0.1:1080
NO_PROXY=localhost,127.0.0.0/8,10.0.0.0/8,172.16.0.0/16,192.168.0.0/16,https://mirror.ccs.tencentyun.com,https://docker.mirrors.ustc.edu.cn/
```

## 开启插件

microk8s 自带很多插件, 我们需要开启相应的配置

```sh
microk8s.enable dns registry storage
```

```sh
➜  WorkSpace microk8s.status 
microk8s is running
addons:
cilium: disabled
dashboard: disabled
dns: enabled
fluentd: disabled
gpu: disabled
helm3: disabled
helm: disabled
ingress: enabled
istio: disabled
jaeger: disabled
juju: disabled
knative: disabled
kubeflow: disabled
linkerd: disabled
metallb: disabled
metrics-server: disabled
prometheus: disabled
rbac: disabled
registry: enabled
storage: enabled
```

```sh
➜  WorkSpace microk8s.kubectl get pod -n kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-7b67f9f8c-9b8nz                 1/1     Running   0          12h
hostpath-provisioner-6d744c4f7c-g85m2   1/1     Running   11         273d
```

# dashboard

`Kubernetes Dashboard` 是 Kubernetes 集群的基于 Web 的通用 UI. 它允许用户管理在群集中运行的应用程序并对其进行故障排除, 以及管理群集本身

默认情况下, dashboard 部署完会自动生成一个证书, 但是这个证书貌似不是很好使, 一般浏览器不放行, 只有火狐浏览器能忽略风险继续访问, 因此需要用自己的证书替换, 只需要在启用 dashboard 前, 在与 dashboard 相同命名空间下创建名为
`kubernetes-dashboard-certs` 的密钥

## 生成证书

可以用 [Let’s Encrypt](https://letsencrypt.org/) 生成证书, 或者使用自签名证书, 参考 [GitHub 文档](https://github.com/kubernetes/dashboard/blob/master/docs/user/certificate-management.md)

以下使用 openssl 生成证书, 当提示输入密码时直接回车跳过即可

```sh
# 生成dashboard.key私钥和dashboard.csr文件
openssl genrsa -des3 -passout pass:123456 -out dashboard.pass.key 2048
openssl rsa -passin pass:123456 -in dashboard.pass.key -out dashboard.key
rm dashboard.pass.key
openssl req -new -key dashboard.key -out dashboard.csr
# SSL证书
openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt
rm dashboard.csr
mkdir certs && mv dashboard.key dashboard.crt certs
```

## 创建密钥

自定义证书必须存储在与 `Kubernetes Dashboard` 相同的命名空间中的 `kubernetes-dashboard-certs` 密钥中. 将 `dashboard.crt` 和 `dashboard.key` 文件存储在 `certs` 目录下, 然后用这些文件的内容创建密钥
参考 [GitHub 文档](https://github.com/kubernetes/dashboard/blob/master/docs/user/installation.md#recommended-setup)

```sh
microk8s.kubectl create secret generic kubernetes-dashboard-certs --from-file=./certs -n kube-system
```

## 部署并访问

启用插件, `microk8s.enable dns ingress dashboard`

修改 `dashboar dservice` 中 `spec-type` 为 `NodePort`: `microk8s.kubectl edit svc kubernetes-dashboard -n kube-system`

使用命令 `kubectl describe svc kubernetes-dashboard -n kube-system` 查看 `NodePort`, 该值则为主机端口, 访问 `https+主机 ip+NodePort` 即可

登录之前需要获取 `token`, 使用如下命令获取: 

```sh
microk8s.kubectl -n kube-system describe secret $(microk8s.kubectl -n kube-system get secret | grep default-token | cut -d " " -f1) | grep token: | awk '{print $2;}'
```

# 引用

- [通过 MicroK8s 搭建你的 K8s 环境](https://juejin.im/post/5d7456d05188250a98581fbf)
- [使用 microk8s 安装单节点 k8s 集群](https://www.imooc.com/article/291860)