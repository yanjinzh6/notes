---
title: 安装 kubeadm
date: 2020-12-12 13:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: kubeadm-install
photo:
---

https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

## 简介

允许 iptables 检查桥接流量
确保 br_netfilter 模块被加载。这一操作可以通过运行 lsmod | grep br_netfilter 来完成。若要显式加载该模块，可执行 sudo modprobe br_netfilter。

为了让你的 Linux 节点上的 iptables 能够正确地查看桥接流量，你需要确保在你的 sysctl 配置中将 net.bridge.bridge-nf-call-iptables 设置为 1。例如：

```sh
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
```

```sh
cat <<EOF | tee /etc/modules-load.d/k8s.conf
br_netfilter
EOF

cat <<EOF | tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system
```

```sh
apt-get update && apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat << EOF | tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl
apt-mark hold kubelet kubeadm kubectl

kubeadm init --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint=107.173.34.138 --apiserver-advertise-address=159.75.102.161
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
export KUBECONFIG=/etc/kubernetes/admin.conf
kubectl get nodes
kubectl get pods -A

kubeadm token list
kubeadm token create
```

```sh
kubeadm join 107.173.34.138:6443 --token lctouu.cfxd4iblpesmm1np \
    --discovery-token-ca-cert-hash sha256:b476403dd38655b89ab5b5436b8395eedc9e87a81d7ce32e113d741961265b1a
```

```sh
kubectl get nodes
NAME                 STATUS   ROLES                  AGE   VERSION
vr-la-a.361163.xyz   Ready    control-plane,master   46s   v1.20.0
```

```sh
kubectl get pods -A
NAMESPACE     NAME                                         READY   STATUS    RESTARTS   AGE
kube-system   coredns-74ff55c5b-jt6qn                      1/1     Running   0          84s
kube-system   coredns-74ff55c5b-qlgvj                      1/1     Running   0          83s
kube-system   etcd-vr-la-a.361163.xyz                      1/1     Running   0          88s
kube-system   kube-apiserver-vr-la-a.361163.xyz            1/1     Running   0          88s
kube-system   kube-controller-manager-vr-la-a.361163.xyz   1/1     Running   0          88s
kube-system   kube-flannel-ds-49v7m                        1/1     Running   0          29s
kube-system   kube-proxy-fj27q                             1/1     Running   0          84s
kube-system   kube-scheduler-vr-la-a.361163.xyz            1/1     Running   0          88s
```

```sh
kubectl get nodes
NAME                 STATUS   ROLES                  AGE   VERSION
vr-la-a.361163.xyz   Ready    control-plane,master   5m    v1.20.0
vr-ny-b.361163.xyz   Ready    <none>                 4s    v1.20.0
```

https://kubernetes.io/zh/docs/setup/production-environment/tools/kubeadm/create-cluster-kubeadm/
