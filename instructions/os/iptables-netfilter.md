---
title: netfilter/iptables 防火墙
date: 2020-12-05 11:00:00
tags: 'Linux'
categories:
  - ['使用说明', '操作系统']
permalink: iptables-netfilter
---

## 简介

netfilter 是防火墙

## x

```sh
iptables -nL
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```

```sh
iptables -t filter -L INPUT
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
```

```sh
iptables -nvL --line-number
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination
```

```
*filter

# Allows all loopback (lo0) traffic and drop all traffic to 127/8 that doesn't use lo0
-A INPUT -i lo -j ACCEPT
-A INPUT ! -i lo -d 127.0.0.0/8 -j REJECT

# Accepts all established inbound connections
-A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT

# Allows all outbound traffic
# You could modify this to only allow certain traffic
-A OUTPUT -j ACCEPT

# Allows HTTP and HTTPS connections from anywhere (the normal ports for websites)
-A INPUT -p tcp --dport 80 -j ACCEPT
-A INPUT -p tcp --dport 443 -j ACCEPT
-A INPUT -p tcp --dport 8443 -j ACCEPT

# Allows SSH connections
# The --dport number is the same as in /etc/ssh/sshd_config
-A INPUT -p tcp -m state --state NEW --dport 22 -j ACCEPT

# Now you should read up on iptables rules and consider whether ssh access
# for everyone is really desired. Most likely you will only allow access from certain IPs.

# Allow ping
# note that blocking other types of icmp packets is considered a bad idea by some
# remove -m icmp --icmp-type 8 from this line to allow all kinds of icmp:
# https://security.stackexchange.com/questions/22711
-A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT

# log iptables denied calls (access via 'dmesg' command)
-A INPUT -m limit --limit 5/min -j LOG --log-prefix "iptables denied: " --log-level 7

# Reject all other inbound - default deny unless explicitly allowed policy:
-A INPUT -j REJECT
-A FORWARD -j REJECT
COMMIT
```

```
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
# http
-I INPUT -p tcp --dport 80 -j ACCEPT
# https
-I INPUT -p tcp --dport 443 -j ACCEPT
# shadowsocks kcptun
-I INPUT -p udp --dport 443 -j ACCEPT
# shadowsocks
-I INPUT -p tcp --dport 8443 -j ACCEPT
# test
-I INPUT -p tcp --dport 8000 -j ACCEPT
-I INPUT -p tcp --dport 8080 -j ACCEPT
# Kubernetes API 服务器: 所有组件
-I INPUT -p tcp --dport 6443 -j ACCEPT
# etcd server client API: kube-apiserver, etcd
-I INPUT -p tcp --dport 2379:2380 -j ACCEPT
# Kubelet API: kubelet 自身、控制平面组件
-I INPUT -p tcp --dport 10250 -j ACCEPT
# kube-scheduler: kube-scheduler 自身
-I INPUT -p tcp --dport 10251 -j ACCEPT
# kube-controller-manager: kube-controller-manager 自身
-I INPUT -p tcp --dport 10252 -j ACCEPT
# NodePort 服务: 所有组件
-I INPUT -p tcp --dport 30000:32767 -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

```
*filter
:INPUT ACCEPT [0:0]
:FORWARD ACCEPT [0:0]
:OUTPUT ACCEPT [0:0]
# http
-I INPUT -p tcp --dport 80 -j ACCEPT
# https
-I INPUT -p tcp --dport 443 -j ACCEPT
# shadowsocks kcptun
-I INPUT -p udp --dport 443 -j ACCEPT
# shadowsocks
-I INPUT -p tcp --dport 8443 -j ACCEPT
# test
-I INPUT -p tcp --dport 8000 -j ACCEPT
-I INPUT -p tcp --dport 8080 -j ACCEPT
# Kubelet API: kubelet 自身、控制平面组件
-I INPUT -p tcp --dport 10250 -j ACCEPT
# NodePort 服务: 所有组件
-I INPUT -p tcp --dport 30000:32767 -j ACCEPT
-A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
-A INPUT -p icmp -j ACCEPT
-A INPUT -i lo -j ACCEPT
-A INPUT -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
-A INPUT -j REJECT --reject-with icmp-host-prohibited
-A FORWARD -j REJECT --reject-with icmp-host-prohibited
COMMIT
```

```sh
iptables-restore < /etc/iptables.test.rules
iptables-save > /etc/iptables.up.rules
```

添加启动配置

```sh
cat << EOF | tee /etc/network/if-pre-up.d/iptables
#!/bin/sh
/sbin/iptables-restore < /etc/iptables.up.rules
EOF
```

```sh
chmod +x /etc/network/if-pre-up.d/iptables
```
