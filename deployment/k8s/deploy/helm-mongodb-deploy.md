---
title: Helm 部署 mongodb
date: 2020-03-11 21:30:00
tags: '部署'
categories:
  - ['部署', 'k8s']
permalink: helm-mongodb-deploy
photo:
---

> 使用 helm 后部署 k8s 应用就方便多了, 有很多已经配置好的配置文件, 只需要进行参数设置后带上即可, 当然不同环境的部署还是有一定区别的

## 获取

```sh
helm search repo mongo
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
stable/mongodb                          7.8.6           4.2.3           NoSQL document-oriented database that stores JS...
stable/mongodb-replicaset               3.11.5          3.6             NoSQL document-oriented database that stores JS...
stable/prometheus-mongodb-exporter      2.4.0           v0.10.0         A Prometheus exporter for MongoDB metrics
stable/unifi                            0.6.1           5.11.50         Ubiquiti Network's Unifi Controller
```

这里选择最简单的 `stable/mongodb` APP 即可

## 安装

```sh
helm install mongo stable/mongodb -n default
```

这里安装了一份 release 为 `my-mongo`, 默认命名空间的 `stable/mongodb` 应用, 因为新版本的 helm 的新特性, 支持不同命名空间下面相同的 release, 所有可以在其他命名空间继续安装 `helm install mongo stable/mongodb -n mongodb`

<!-- more -->

## 检查

```sh
kubectl get pod mongo-mongodb-545c588f78-xkqhq
NAME                             READY   STATUS    RESTARTS   AGE
mongo-mongodb-545c588f78-xkqhq   1/1     Running   0          13m

kubectl logs mongo-mongodb-545c588f78-xkqhq
 14:00:53.21
 14:00:53.21 Welcome to the Bitnami mongodb container
 14:00:53.22 Subscribe to project updates by watching https://github.com/bitnami/bitnami-docker-mongodb
 14:00:53.22 Submit issues and feature requests at https://github.com/bitnami/bitnami-docker-mongodb/issues
 14:00:53.23 Send us your feedback at containers@bitnami.com
 14:00:53.23
 14:00:53.23 INFO  ==> ** Starting MongoDB setup **
 14:00:53.25 INFO  ==> Validating settings in MONGODB_* env vars...
 14:00:53.25 INFO  ==> Initializing MongoDB...
 14:00:53.26 INFO  ==> No injected configuration files found. Creating default config files...
 14:00:53.27 INFO  ==> Deploying MongoDB from scratch...
 14:00:54.16 INFO  ==> Creating users...
 14:00:54.16 INFO  ==> Creating root user...
 14:00:54.32 INFO  ==> Users created
 14:00:54.32 INFO  ==> Stopping MongoDB...
 14:00:55.35 INFO  ==> Enabling authentication...
 14:00:55.36 INFO  ==>
 14:00:55.36 INFO  ==> ########################################################################
 14:00:55.36 INFO  ==>  Installation parameters for MongoDB:
 14:00:55.37 INFO  ==> (Passwords are not shown for security reasons)
 14:00:55.37 INFO  ==> ########################################################################
 14:00:55.37 INFO  ==>
 14:00:55.38 INFO  ==> Loading custom scripts...
 14:00:55.39 INFO  ==> ** MongoDB setup finished! **

 14:00:55.42 INFO  ==> ** Starting MongoDB **
2020-03-11T14:00:55.454+0000 I  CONTROL  [main] ***** SERVER RESTARTED *****
2020-03-11T14:00:55.456+0000 I  CONTROL  [main] Automatically disabling TLS 1.0, to force-enable TLS 1.0 specify --sslDisabledProtocols 'none'
2020-03-11T14:00:55.460+0000 I  CONTROL  [initandlisten] MongoDB starting : pid=1 port=27017 dbpath=/bitnami/mongodb/data/db 64-bit host=mongo-mongodb-545c588f78-xkqhq
2020-03-11T14:00:55.461+0000 I  CONTROL  [initandlisten] db version v4.2.3
2020-03-11T14:00:55.461+0000 I  CONTROL  [initandlisten] git version: 6874650b362138df74be53d366bbefc321ea32d4
2020-03-11T14:00:55.461+0000 I  CONTROL  [initandlisten] OpenSSL version: OpenSSL 1.1.1d  10 Sep 2019
2020-03-11T14:00:55.461+0000 I  CONTROL  [initandlisten] allocator: tcmalloc
2020-03-11T14:00:55.461+0000 I  CONTROL  [initandlisten] modules: none
2020-03-11T14:00:55.461+0000 I  CONTROL  [initandlisten] build environment:
2020-03-11T14:00:55.461+0000 I  CONTROL  [initandlisten]     distmod: debian10
2020-03-11T14:00:55.461+0000 I  CONTROL  [initandlisten]     distarch: x86_64
2020-03-11T14:00:55.461+0000 I  CONTROL  [initandlisten]     target_arch: x86_64
2020-03-11T14:00:55.461+0000 I  CONTROL  [initandlisten] options: { config: "/opt/bitnami/mongodb/conf/mongodb.conf", net: { bindIp: "*", ipv6: false, port: 27017, unixDomainSocket: { enabled: true, pathPrefix: "/opt/bitnami/mongodb/tmp" } }, processManagement: { fork: false, pidFilePath: "/opt/bitnami/mongodb/tmp/mongodb.pid" }, security: { authorization: "enabled" }, setParameter: { enableLocalhostAuthBypass: "false" }, storage: { dbPath: "/bitnami/mongodb/data/db", directoryPerDB: false, journal: { enabled: true } }, systemLog: { destination: "file", logAppend: true, logRotate: "reopen", path: "/opt/bitnami/mongodb/logs/mongodb.log", quiet: false, verbosity: 0 } }
2020-03-11T14:00:55.461+0000 I  STORAGE  [initandlisten] Detected data files in /bitnami/mongodb/data/db created by the 'wiredTiger' storage engine, so setting the active storage engine to 'wiredTiger'.
2020-03-11T14:00:55.462+0000 I  STORAGE  [initandlisten]
2020-03-11T14:00:55.462+0000 I  STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
2020-03-11T14:00:55.462+0000 I  STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
2020-03-11T14:00:55.462+0000 I  STORAGE  [initandlisten] wiredtiger_open config: create,cache_size=6646M,cache_overflow=(file_max=0M),session_max=33000,eviction=(threads_min=4,threads_max=4),config_base=false,statistics=(fast),log=(enabled=true,archive=true,path=journal,compressor=snappy),file_manager=(close_idle_time=100000,close_scan_interval=10,close_handle_minimum=250),statistics_log=(wait=0),verbose=[recovery_progress,checkpoint_progress],
2020-03-11T14:00:56.102+0000 I  STORAGE  [initandlisten] WiredTiger message [1583935256:102234][1:0x7fb982596d40], txn-recover: Recovering log 1 through 2
2020-03-11T14:00:56.191+0000 I  STORAGE  [initandlisten] WiredTiger message [1583935256:191611][1:0x7fb982596d40], txn-recover: Recovering log 2 through 2
2020-03-11T14:00:56.296+0000 I  STORAGE  [initandlisten] WiredTiger message [1583935256:296164][1:0x7fb982596d40], txn-recover: Main recovery loop: starting at 1/28928 to 2/256
2020-03-11T14:00:56.431+0000 I  STORAGE  [initandlisten] WiredTiger message [1583935256:431801][1:0x7fb982596d40], txn-recover: Recovering log 1 through 2
2020-03-11T14:00:56.499+0000 I  STORAGE  [initandlisten] WiredTiger message [1583935256:499096][1:0x7fb982596d40], txn-recover: Recovering log 2 through 2
2020-03-11T14:00:56.560+0000 I  STORAGE  [initandlisten] WiredTiger message [1583935256:560415][1:0x7fb982596d40], txn-recover: Set global recovery timestamp: (0, 0)
2020-03-11T14:00:56.586+0000 I  RECOVERY [initandlisten] WiredTiger recoveryTimestamp. Ts: Timestamp(0, 0)
2020-03-11T14:00:56.590+0000 I  STORAGE  [initandlisten] Timestamp monitor starting
2020-03-11T14:00:56.594+0000 I  CONTROL  [initandlisten]
2020-03-11T14:00:56.594+0000 I  CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2020-03-11T14:00:56.594+0000 I  CONTROL  [initandlisten] **        We suggest setting it to 'never'
2020-03-11T14:00:56.594+0000 I  CONTROL  [initandlisten]
2020-03-11T14:00:56.597+0000 I  SHARDING [initandlisten] Marking collection local.system.replset as collection version: <unsharded>
2020-03-11T14:00:56.599+0000 I  STORAGE  [initandlisten] Flow Control is enabled on this deployment.
2020-03-11T14:00:56.599+0000 I  SHARDING [initandlisten] Marking collection admin.system.roles as collection version: <unsharded>
2020-03-11T14:00:56.599+0000 I  SHARDING [initandlisten] Marking collection admin.system.version as collection version: <unsharded>
2020-03-11T14:00:56.601+0000 I  SHARDING [initandlisten] Marking collection local.startup_log as collection version: <unsharded>
2020-03-11T14:00:56.601+0000 I  FTDC     [initandlisten] Initializing full-time diagnostic data capture with directory '/bitnami/mongodb/data/db/diagnostic.data'
2020-03-11T14:00:56.603+0000 I  SHARDING [LogicalSessionCacheRefresh] Marking collection config.system.sessions as collection version: <unsharded>
2020-03-11T14:00:56.604+0000 I  SHARDING [LogicalSessionCacheReap] Marking collection config.transactions as collection version: <unsharded>
2020-03-11T14:00:56.604+0000 I  NETWORK  [listener] Listening on /opt/bitnami/mongodb/tmp/mongodb-27017.sock
2020-03-11T14:00:56.604+0000 I  NETWORK  [listener] Listening on 0.0.0.0
2020-03-11T14:00:56.604+0000 I  NETWORK  [listener] waiting for connections on port 27017
2020-03-11T14:00:57.002+0000 I  SHARDING [ftdc] Marking collection local.oplog.rs as collection version: <unsharded>
2020-03-11T14:01:03.394+0000 I  NETWORK  [listener] connection accepted from 127.0.0.1:33162 #1 (1 connection now open)
2020-03-11T14:01:03.394+0000 I  NETWORK  [conn1] received client metadata from 127.0.0.1:33162 conn1: { application: { name: "MongoDB Shell" }, driver: { name: "MongoDB Internal Client", version: "4.2.3" }, os: { type: "Linux", name: "PRETTY_NAME="Debian GNU/Linux 10 (buster)"", architecture: "x86_64", version: "Kernel 4.19.84-microsoft-standard" } }
2020-03-11T14:01:03.401+0000 I  NETWORK  [conn1] end connection 127.0.0.1:33162 (0 connections now open)
2020-03-11T14:01:13.391+0000 I  NETWORK  [listener] connection accepted from 127.0.0.1:33224 #2 (1 connection now open)
2020-03-11T14:01:13.391+0000 I  NETWORK  [conn2] received client metadata from 127.0.0.1:33224 conn2: { application: { name: "MongoDB Shell" }, driver: { name: "MongoDB Internal Client", version: "4.2.3" }, os: { type: "Linux", name: "PRETTY_NAME="Debian GNU/Linux 10 (buster)"", architecture: "x86_64", version: "Kernel 4.19.84-microsoft-standard" } }
```

这里需要注意的是 Windows 环境下是需要开启 `volumePermissions` 参数的, 不然会导致无权限的错误, 另外看到配置文件中已经开启了管理员密码, 而且获取的就是名称为 `RELEASE-NAME-mongodb` 的 secret 配置中的 `mongodb-root-password` 值, 上面的例子就是 `mongo-mongodb`

```sh
kubectl get secret mongo-mongodb
NAME            TYPE     DATA   AGE
mongo-mongodb   Opaque   1      4d10h
```

之前并没有提前配置好, 看了一下配置, 里面已经默认生成了, 只需要通过解码 base64 就可以得到密码, 在安装过后有相应的提示, 这里贴一下格式化后的说明

```md
**Please be patient while the chart is being deployed**

MongoDB can be accessed via port 27017 on the following DNS name from within your cluster:

`mongo-mongodb.default.svc.cluster.local`

To get the root password run:

`export MONGODB_ROOT_PASSWORD=$(kubectl get secret --namespace default mongo-mongodb -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)`

To connect to your database run the following command:

`kubectl run --namespace default mongo-mongodb-client --rm --tty -i --restart='Never' --image docker.io/bitnami/mongodb:4.2.3-debian-10-r31 --command -- mongo admin --host mongo-mongodb --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD`

To connect to your database from outside the cluster execute the following commands:

`kubectl port-forward --namespace default svc/mongo-mongodb 27017:27017 & mongo --host 127.0.0.1 --authenticationDatabase admin -p $MONGODB_ROOT_PASSWORD`
```

可以通过相应的命令来进行操作
