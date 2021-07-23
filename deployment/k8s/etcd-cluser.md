---
title: etcd-cluser
date: 2021-03-21 16:00:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: etcd-cluser
photo:
---

[etcd-cluser](https://segmentfault.com/a/1190000022817355)

# 6, etcd 集群搭建

etcd 是一个 key-value 存储的分布式系统, 还提供共享配置及服务发现, 使用 go 编写, 实现了 `Raft 协议 `.

## 6.1 安装, 准备证书

```
wget 'https://github.com/etcd-io/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz'
tar xf etcd-v3.4.9-linux-amd64.tar.gz -C /opt
#群发证书
for host in '11' '12' '21' '22'; do
    ssh root@10.4.7.$host 'ln -s /opt/etcd-v3.4.9-linux-amd64/ /opt/etcd'
    ssh root@10.4.7.$host 'mkdir -p /data/etcd/etcd-server /data/logs/etcd-server /opt/certs'
    scp /opt/certs/etcd-peer* root@10.4.7.$host:/opt/certs/
    scp /opt/certs/ca.pem root@10.4.7.$host:/opt/certs/
done
```

## 6.2 启动脚本

ETCD 3.4 版本手册 [https://etcd.io/docs/v3.4.0/op-guide/clustering/](https://etcd.io/docs/v3.4.0/op-guide/clustering/)

```
#启动脚本
[10.4.7.12]# vi /opt/etcd/etcd-server-startup.sh
ETCD_CLUSTER='etcd-server-7-12= https://10.4.7.12:2380, etcd-server-7-21= https://10.4.7.21:2380, etcd-server-7-22= https://10.4.7.22:2380'
/opt/etcd/etcd --data-dir=/data/etcd/etcd-server --name etcd-server-7-12 \
  --initial-advertise-peer-urls https://10.4.7.12:2380 \
  --listen-peer-urls https://10.4.7.12:2380 \
  --listen-client-urls https://10.4.7.11:2379, http://10.4.7.12:2379, http://127.0.0.1:2379 \
  --advertise-client-urls https://10.4.7.12:2379, http://10.4.7.12:2379 \
  --initial-cluster-token etcd-cluster-1 \
  --initial-cluster $ETCD_CLUSTER \
  --initial-cluster-state new --logger= zap --log-level= warn --log-outputs stdout \
  --client-cert-auth --trusted-ca-file=/opt/certs/ca.pem \
  --cert-file=/opt/certs/etcd-peer.pem --key-file=/opt/certs/etcd-peer-key.pem \
  --peer-client-cert-auth --peer-trusted-ca-file=/opt/certs/ca.pem \
  --peer-cert-file=/opt/certs/etcd-peer.pem --peer-key-file=/opt/certs/etcd-peer-key.pem
#--ca-file 为不支持选项
#--log-outputs stdout 复数
```

## 6.3 守护进程软件 Supervisor

```
useradd -M -s /sbin/nologin etcd
chown etcd: etcd -R /data/etcd /data/logs/etcd-server /opt/etcd*
chmod + r -R /opt/certs/
```

#### Supervisor 是用 Python 开发的一套通用的进程管理程序, 能将一个普通的命令行进程变为后台 daemon, 并监控进程状态, 异常退出时能自动重启.

```
yum install supervisor -y
systemctl start supervisord
systemctl enable supervisord
#守护脚本配置 https://www.cnblogs.com/kevin-ying/p/12343699.html
[10.4.7.12]# vi /etc/supervisord.d/etcd-server.ini
[program: etcd-server-7-12]
command=/opt/etcd/etcd-server-startup.sh
numprocs=1
directory=/opt/etcd
autostart= true
autorestart= true
startsecs=30
startretries=3
exitcodes=0,2
stopsignal= QUIT
stopwaitsecs=10
user= etcd
redirect_stderr= true
stdout_logfile=/data/logs/etcd-server/etcd.stdout.log
stdout_logfile_maxbytes=64MB
stdout_logfile_backups=4
stdout_capture_maxbytes=1MB
stdout_events_enabled= false
```

启动脚本

```
chmod + x /opt/etcd/etcd-server-startup.sh
supervisorctl update
#supervisorctl start| stop| restart etcd-server-7-12
```

## 6.4 yml 文件版启动配置

##### 官方 [参考文件](https://github.com/etcd-io/etcd/blob/master/etcd.conf.yml.sample)

```
# 执行命令
/opt/etcd/etcd --config-file /opt/etcd/conf.yml
#yml 配置: 缺少参考实例
[10.4.7.12]# vi /opt/etcd/conf.yml
name: etcd-server-7-11
data-dir: /data/etcd/etcd-server
listen-client-urls: https://10.4.7.12:2379, http://10.4.7.12:2379, http://127.0.0.1:2379
advertise-client-urls: https://10.4.7.12:2379, http://10.4.7.12:2379
listen-peer-urls: https://10.4.7.12:2380
initial-advertise-peer-urls: https://10.4.7.12:2380
initial-cluster: etcd-server-7-12= https://10.4.7.12:2380, etcd-server-7-21= https://10.4.7.21:2380, etcd-server-7-22= https://10.4.7.22:2380
initial-cluster-token: etcd-cluster-token
initial-cluster-state: new
logger: zap
log-level: warn
client-transport-security:
  cert-file: /opt/certs/etcd-peer.pem
  key-file: /opt/certs/etcd-peer-key.pem
  trusted-ca-file: /opt/certs/ca.pem
peer-transport-security:
  cert-file: /opt/certs/etcd-peer.pem
  key-file: /opt/certs/etcd-peer-key.pem
  trusted-ca-file: /opt/certs/ca.pem
```

该版启动提升缺少认证文件...

## 6.5 查看状态

```
yum install net-tools -y
netstat -lunpt| grep 23
ln -s /opt/etcd/etcd /usr/local/bin
ln -s /opt/etcd/etcdctl /usr/local/bin
```

##### [etcdctl 命令手册](https://github.com/etcd-io/etcd/tree/master/etcdctl)

```
[root@hdss1-21 ~]# etcdctl endpoint health
127.0.0.1:2379 is healthy: successfully committed proposal: took = 2.031575ms
[root@hdss1-21 ~]# etcdctl endpoint --cluster health -w table
{"level":"warn","ts":"2020-06-02T08:55:49.189+0800","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"endpoint://client-3d1487c6-471e-4874-bcd7-62540a2b3483/10.4.7.22:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: all SubConns are in TransientFailure, latest connection error: connection error: desc = \"transport: authentication handshake failed: x509: certificate signed by unknown authority\""}
{"level":"warn","ts":"2020-06-02T08:55:49.189+0800","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"endpoint://client-b2c7540f-2786-48fd-b5a6-0d5b1a1e2d5b/10.4.7.21:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: all SubConns are in TransientFailure, latest connection error: connection error: desc = \"transport: authentication handshake failed: x509: certificate signed by unknown authority\""}
{"level":"warn","ts":"2020-06-02T08:55:49.191+0800","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"endpoint://client-ddaf67c1-75f4-4b6c-ae1b-57484ae0b202/10.4.7.12:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: all SubConns are in TransientFailure, latest connection error: connection error: desc = \"transport: authentication handshake failed: x509: certificate signed by unknown authority\""}
+------------------------+--------+--------------+---------------------------+
|        ENDPOINT        | HEALTH |     TOOK     |           ERROR           |
+------------------------+--------+--------------+---------------------------+
|  http://10.4.7.12:2379 |   true |  10.379832ms |                           |
|  http://10.4.7.21:2379 |   true |  11.786675ms |                           |
|  http://10.4.7.22:2379 |   true |  10.240377ms |                           |
| https://10.4.7.22:2379 |  false |  5.00034375s | context deadline exceeded |
| https://10.4.7.21:2379 |  false | 5.000158814s | context deadline exceeded |
| https://10.4.7.12:2379 |  false | 5.000885212s | context deadline exceeded |
+------------------------+--------+--------------+---------------------------+
[root@hdss1-21 ~]# etcdctl endpoint status -w table
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT    |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 127.0.0.1:2379 | 41f77afc31d598a9 |   3.4.9 |   20 kB |      true |      false |       168 |         20 |                 20 |        |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
[root@hdss1-12 ~]# etcdctl endpoint status -w table
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|    ENDPOINT    |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| 127.0.0.1:2379 | 4cc0e9e701b89995 |   3.4.9 |   20 kB |     false |      false |       168 |         20 |                 20 |        |
+----------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
[root@hdss1-12 ~]# etcdctl -w table endpoint --cluster status
{"level":"warn","ts":"2020-06-02T09:35:33.456+0800","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"passthrough:///https://10.4.7.21:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: connection error: desc = \"transport: authentication handshake failed: x509: certificate signed by unknown authority\""}
Failed to get the status of endpoint https://10.4.7.21:2379 (context deadline exceeded)
{"level":"warn","ts":"2020-06-02T09:35:38.458+0800","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"passthrough:///https://10.4.7.12:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: connection error: desc = \"transport: authentication handshake failed: x509: certificate signed by unknown authority\""}
Failed to get the status of endpoint https://10.4.7.12:2379 (context deadline exceeded)
{"level":"warn","ts":"2020-06-02T09:35:43.461+0800","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"passthrough:///https://10.4.7.22:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: connection error: desc = \"transport: authentication handshake failed: x509: certificate signed by unknown authority\""}
Failed to get the status of endpoint https://10.4.7.22:2379 (context deadline exceeded)
+-----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
|       ENDPOINT        |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
+-----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
| http://10.4.7.21:2379 | 41f77afc31d598a9 |   3.4.9 |   20 kB |      true |      false |       168 |         20 |                 20 |        |
| http://10.4.7.12:2379 | 4cc0e9e701b89995 |   3.4.9 |   20 kB |     false |      false |       168 |         20 |                 20 |        |
| http://10.4.7.22:2379 | 9dbbb56f94b8e356 |   3.4.9 |   20 kB |     false |      false |       168 |         20 |                 20 |        |
+-----------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
```

https 认证失败, 暂未找到问题.

## 6.6 加入新节点, 参考 [etcd 集群添加节点](https://www.cnblogs.com/ilifeilong/p/11625151.html)

##### etcdctl member add

```
[root@hdss1-11 opt]# etcdctl member add etcd-server-7-200 --peer-urls= https://10.4.7.200:2380
Member aa3fa18813730f87 added to cluster 95d119deb8fc35c5

ETCD_NAME="etcd-server-7-200"
ETCD_INITIAL_CLUSTER="etcd-server-7-22= https://10.4.7.22:2380, etcd-server-7-11= https://10.4.7.11:2380, etcd-server-7-21= https://10.4.7.21:2380, etcd-server-7-200= https://10.4.7.200:2380, etcd-server-7-12= https://10.4.7.12:2380"
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://10.4.7.200:2380"
ETCD_INITIAL_CLUSTER_STATE="existing"
#将各节点 etcd.conf 配置文件的变量 ETCD_INITIAL_CLUSTER 添加新节点信息, 然后依次重启??. 没变化
#加入是正常的, 但 etcd-server-7-200 的结果 name 显示还是问号?, 日志显示是握手失败
```

##### etcdctl member update/remove

```
etcdctl member update aa3fa18813730f87 --peer-urls= https://10.4.7.200:2380
etcdctl member remove aa3fa18813730f87 #200
etcdctl member remove 1ba3960d0c371211 #11
curl 127.0.0.1:2379/health #{"health":"true"}
```

##### etcdctl --cacert=/opt/certs/ca.pem member list

```
[root@hdss1-22 ~]# etcdctl --cacert=/opt/certs/ca.pem --cert=/opt/certs/etcd-peer.pem --key=/opt/certs/etcd-peer-key.pem member list
{"level":"warn","ts":"2020-06-01T20:31:11.155+0800","caller":"clientv3/retry_interceptor.go:62","msg":"retrying of unary invoker failed","target":"endpoint://client-c46b8cf1-a3d8-400f-b0c2-feaf0d9730fd/127.0.0.1:2379","attempt":0,"error":"rpc error: code = DeadlineExceeded desc = latest balancer error: all SubConns are in TransientFailure, latest connection error: connection error: desc = \"transport: authentication handshake failed: EOF\""}
Error: context deadline exceeded
```

结果显示的是等效的:

> etcdctl --insecure-skip-tls-verify --cacert /opt/certs/ca.pem --cert /opt/certs/etcd-peer.pem --key /opt/certs/etcd-peer-key.pem member list

##### 自动 https 认证 [TLS 手册](https://etcd.io/docs/v3.4.0/op-guide/security/)

```
vi /opt/etcd/conf.yml:
auto-tls: true #--auto-tls
client-cert-auth: false
peer-auto-tls: true
peer-client-cert-auth: false
```

自动设置设置依然是 https 握手失败, 保留代码

```
for host in '11' '12' '21' '22'; do
    echo "ssh root@10.4.7.$host 'mv /opt/etcd/etcd-server-startup.sh /opt/etcd/etcd-server-startup.sh.yml'"
    echo "ssh root@10.4.7.$host 'mv /o/opt/etcd/etcd-server-startup.sh.cli /opt/etcd/etcd-server-startup.sh'"
done
```

所以, 这还是一个 http 版的集群.

到此, etcd 集群搭建完毕, 可惜是个 http 版的集群: shell 启动, yml 配置文件启动, auto-tls 版的 shell 和 yml 测试的均未成功.
