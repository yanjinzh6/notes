---
title: Canal 使用
date: 2020-1-9 18:00:00
tags: 'MySQL'
categories:
  - ['使用说明', '软件']
permalink: canal-introduction
photo:
---

## 简介

**[canal](https://github.com/alibaba/canal) [kə'næl]**, 译意为水道/管道/沟渠, 主要用途是基于 MySQL 数据库增量日志解析, 提供增量数据订阅和消费

早期阿里巴巴因为杭州和美国双机房部署, 存在跨机房同步的业务需求, 实现方式主要是基于业务 trigger 获取增量变更。从 2010 年开始, 业务逐步尝试数据库日志解析获取增量变更进行同步, 由此衍生出了大量的数据库增量订阅和消费业务。

基于日志增量订阅和消费的业务包括

- 数据库镜像
- 数据库实时备份
- 索引构建和实时维护(拆分异构索引, 倒排索引等)
- 业务 cache 刷新
- 带业务逻辑的增量数据处理

当前的 canal 支持源端 MySQL 版本包括 5.1.x , 5.5.x , 5.6.x , 5.7.x , 8.0.x

<!-- more -->

## 工作原理

- MySQL 主备复制原理
  - MySQL master 将数据变更写入二进制日志( binary log, 其中记录叫做二进制日志事件binary log events, 可以通过 show binlog events 进行查看 )
  - MySQL slave 将 master 的 binary log events 拷贝到它的中继日志( relay log )
  - MySQL slave 重放 relay log 中事件, 将数据变更反映它自己的数据
- canal 工作原理
  - canal 模拟 MySQL slave 的交互协议, 伪装自己为 MySQL slave , 向 MySQL master 发送dump 协议
  - MySQL master 收到 dump 请求, 开始推送 binary log 给 slave ( 即 canal )
  - canal 解析 binary log 对象( 原始为 byte 流 )

## 部署

### 使用镜像部署

canal docker hub 上最新的版本是 1.1.4

```sh
docker pull canal/canal-server:v1.1.4
Trying to pull repository docker.io/canal/canal-server ...
v1.1.4: Pulling from docker.io/canal/canal-server
1c8f9aa56c90: Already exists
c5e21c824d1c: Already exists
4ba7edb60123: Already exists
80d8e8fac1be: Pull complete
33468ce45688: Pull complete
6e50db081ec0: Pull complete
8b35456bff69: Pull complete
Digest: sha256:40ae71b5d96baf6c408877ce9cd4261a5e5a085ab370ec47e6037000579c58c4
Status: Downloaded newer image for docker.io/canal/canal-server:v1.1.4
```

#### docker-compose 配置文件

```yml
version: '3.1'
services:
  mysql-db:
    # 构建mysql镜像
    image: mysql:5.7
    container_name: mysql-db # 容器名
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci --lower_case_table_names=1 # 设置utf8字符集
    restart: always
    # networks:
    #   - dev-net
    environment:
      MYSQL_ROOT_PASSWORD: root # root管理员用户密码
      MYSQL_USER: test   # 创建test用户
      MYSQL_PASSWORD: test  # 设置test用户的密码
    ports:
      - '3306:3306'  # host物理直接映射端口为6606
    volumes:
      # mysql数据库挂载到host物理机目录/e/docker/mysql/data/db
      - /Users/xxx/Dev/mysql:/var/lib/mysql
      - /Users/xxx/Dev/mysql-files:/var/lib/mysql-files
      # 容器的配置目录挂载到host物理机目录
      - /Users/xxx/Dev/mysql-conf/canal.cnf:/etc/mysql/conf.d/canal.cnf

  canal-server:
    image: canal/canal-server
    container_name: canal-server
    # 2222 sys , 8000 debug , 11111 canal , 11112 metrics
    ports:
      - 2222:2222
      - 28000:8000
      - 11111:11111
      - 11112:11112
    depends_on:
      - mysql-db
    environment:
      - canal.instance.mysql.slaveId=1324
      - canal.instance.master.address=mysql-db:3306
      - canal.instance.dbUsername=canal
      - canal.instance.dbPassword=canal
      - canal.instance.connectionCharset=UTF-8
      - canal.instance.filter.regex=.\*\\\\..\*
    volumes:
      # - /Users/xxx/Dev/canal-server/canal.properties:/home/admin/canal-server/conf/canal.properties
      - /Users/xxx/Dev/canal-server/logs:/home/admin/canal-server/logs
```

简单说一下, canal 可以通过设置环境变量来进行配置, 这里主要设置了从库 ID, 主库地址还有用户名密码, 使用正则表达式匹配所有的库和表

具体配置参考[官方 GitHub](https://github.com/alibaba/canal/wiki/AdminGuide)

#### 开启 MySQL binlog

MySQL 5.7 镜像通过 `/etc/mysql/my.cnf` 指向 `/etc/mysql/mysql.cnf`, 实际上是包括了同目录下的 `/etc/mysql/conf.d/` 和 `/etc/mysql/mysql.conf.d/` 文件夹, 所以只需要在文件夹中添加个配置就可以了

```cnf
[mysqld]
server_id=1918
log-bin=mysql-bin
binlog-format=Row
```

这里设置了 master ID, 避免与其他 slave 起冲突

然后通过添加 canal 用户并且授权用户使用 slave 权限

```sql
CREATE USER canal IDENTIFIED BY 'canal';
GRANT SELECT, REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'canal'@'%';
-- GRANT ALL PRIVILEGES ON *.* TO 'canal'@'%' ;
FLUSH PRIVILEGES;
```

#### 通过 docker-compose 启动容器

```sh
$ docker-compose up -d

$ docker logs canal-server
DOCKER_DEPLOY_TYPE=VM
==> INIT /alidata/init/02init-sshd.sh
==> EXIT CODE: 0
==> INIT /alidata/init/fix-hosts.py
==> EXIT CODE: 0
==> INIT DEFAULT
Generating SSH1 RSA host key: [  OK  ]
Starting sshd: [  OK  ]
Starting crond: [  OK  ]
==> INIT DONE
==> RUN /home/admin/app.sh
==> START ...
start canal ...
```
