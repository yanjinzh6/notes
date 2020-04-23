---
title: MySQL binlog 简介
date: 2020-1-10 12:00:00
tags: 'MySQL'
categories:
  - ['使用说明', '软件']
permalink: mysql-binlog-introduction
photo:
---

## 简介

该文档仅记录相应的描述, 说明文档

MySQL 中一般有以下几种日志:

| 日志类型              | 写入日志的信息                                                      |
| --------------------- | ------------------------------------------------------------------- |
| 错误日志              | 记录在启动, 运行或停止 mysqld 时遇到的问题                          |
| 通用查询日志          | 记录建立的客户端连接和执行的语句                                    |
| 二进制日志            | 记录更改数据的语句                                                  |
| 中继日志              | 从复制主服务器接收的数据更改                                        |
| 慢查询日志            | 记录所有执行时间超过 long_query_time 秒的所有查询或不使用索引的查询 |
| DDL 日志 (元数据日志) | 元数据操作由 DDL 语句执行                                           |

Mysql 5.0 以后, 支持通过 binary log (二进制日志) 以支持主从复制. 复制允许将来自一个 MySQL 数据库服务器 (master) 的数据复制到一个或多个其他 MySQL 数据库服务器 (slave), 以实现灾难恢复, 水平扩展, 统计分析, 远程数据分发等功能. 它记录了所有的 DDL 和 DML 语句 (除了数据查询语句 select, show 等) , 以事件形式记录, 还包含语句所执行的消耗的时间, MySQL 的二进制日志是事务安全型的. binlog 的主要目的是复制和恢复.

## 使用场景

* 最典型的场景就是通过 Mysql 主从之间通过 binlog 复制来实现横向扩展, 实现读写分离
  * 主库 Master 负责所有的更新操作
  * 同时会有多个 Slave, 每个 Slave 都连接到 Master 上, 获取 binlog 在本地回放, 实现数据复制.
  * 在应用层面, 需要对执行的 sql 进行判断. 所有的更新操作都通过 Master(Insert, Update, Delete 等), 而查询操作(Select 等)都在 Slave 上进行. 由于存在多个 slave, 所以我们可以在 slave 之间做负载均衡. 通常业务都会借助一些数据库中间件, 如 tddl, sharding-jdbc 等来完成读写分离功能.
* 数据恢复: 通过使用 mysqlbinlog 工具来使恢复数据
* 数据最终一致性: 通过解析 binlog 的信息, 去异步的更新缓存, 索引或者发送 MQ 消息, 保证数据库与其他组件中数据的最终一致. 例如: linkedin 的 databus, 阿里巴巴的 canal, 美团点评的 puma
* 通常索引分为全量索引和增量索引. 对于增量索引的部分, 可以通过监听 binlog 变化, 根据 binlog 中包含的信息, 转换成 es 语法, 进行实时索引更新

<!-- more -->

## 配置

### 基本设置

> MySQL 5.7 镜像通过 `/etc/mysql/my.cnf` 指向 `/etc/mysql/mysql.cnf`, 实际上是包括了同目录下的 `/etc/mysql/conf.d/` 和 `/etc/mysql/mysql.conf.d/` 文件夹, 所以只需要在文件夹中添加个配置就可以了

```cnf
[mysqld]
log-bin=my-binlog-name
```

也可以通过 SET SQL_LOG_BIN=1 命令来启用 binlog, 通过 SET SQL_LOG_BIN=0 命令停用 binlog. 启用 binlog 之后须重启 MySQL 才能生效.

`max_binlog_size` : 用于控制一个 binlog 文件的大小, 默认是 1G
`expire_logs_days` : 配置项, 可以控制 binlog 文件保留天数, 默认是 0, 也就是永久保留

在实际生产环境中, 一般无法保留所有的历史 binlog. 因为一条记录可能会变更多次, 记录依然是一条, 但是对应的 binlog 事件就会有多个. 在数据变更比较频繁的情况下, 就会产生大量的 binlog 文件. 此时, 则无法保留所有的历史 binlog 文件.

在 mysql 的 percona 分支上, 还提供了 max_binlog_files 配置项, 用于设置可以保留的 binlog 文件数量, 以便我们更精确的控制 binlog 文件占用的磁盘空间. 这是一个非常有用的配置, 笔者曾经遇到一个库, 大约 10 分钟就会产生一个 binlog 文件, 也就是 1G, 按照这种增长速度, 1 天下来产生的 binlog 文件, 就会占用大概 144G 左右的空间, 磁盘空间可能很快就会被使用完. 通过此配置, 我们可以显示的控制 binlog 文件的数量, 例如指定 50, binlog 文件最多只会占用 50G 左右的磁盘空间. 在更高版本的 mysql 中, 支持按照秒级精度, 来控制 binlog 文件的保留时间.

### 常用命令

```sql
## 是否启用 binlog 日志
show variables like 'log_bin';

## 查看详细的日志配置信息
show global variables like '%log%';

## mysql 数据存储目录
show variables like '%dir%';

## 查看 binlog 的目录
show global variables like "%log_bin%";

## 查看当前服务器使用的 biglog 文件及大小
show binary logs;

## 查看主服务器使用的 biglog 文件及大小

## 查看最新一个 binlog 日志文件名称和 Position
show master status;

## 事件查询命令
## IN 'log_name' : 指定要查询的 binlog 文件名(不指定就是第一个 binlog 文件)
## FROM pos : 指定从哪个 pos 起始点开始查起(不指定就是从整个文件首个 pos 点开始算)
## LIMIT [offset,] : 偏移量(不指定就是 0)
## row_count : 查询总条数(不指定就是所有行)
show binlog events [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count];

## 查看 binlog 内容
show binlog events;

## 查看具体一个 binlog 文件的内容  (in 后面为 binlog 的文件名)
show binlog events in 'master.000003';

## 设置 binlog 文件保存事件, 过期删除, 单位天
set global expire_log_days=3;

## 删除当前的 binlog 文件
reset master;

## 删除 slave 的中继日志
reset slave;

## 删除指定日期前的日志索引中 binlog 日志文件
purge master logs before '2019-03-09 14:00:00';

## 删除指定日志文件
purge master logs to 'master.000003';
```

### mysqlbinlog 命令

```sh
## mysqlbinlog 的执行格式
mysqlbinlog [options] log_file ...

## 查看 bin-log 二进制文件 (shell 方式)
mysqlbinlog -v --base64-output=decode-rows /var/lib/mysql/master.000003

## 查看 bin-log 二进制文件 (带查询条件)
mysqlbinlog -v --base64-output=decode-rows /var/lib/mysql/master.000003 \
  --start-datetime="2019-03-01 00:00:00"  \
  --stop-datetime="2019-03-10 00:00:00"   \
  --start-position="5000"    \
  --stop-position="20000"
```

### sync_binlog 说明

对支持事务的引擎如 InnoDB 而言, 必须要提交了事务才会记录 binlog. binlog 什么时候刷新到磁盘跟参数 sync_binlog 相关.

* 如果设置为 0, 则表示 MySQL 不控制 binlog 的刷新, 由文件系统去控制它缓存的刷新;
* 如果设置为不为 0 的值, 则表示每 sync_binlog 次事务, MySQL 调用文件系统的刷新操作刷新 binlog 到磁盘中.
* 设为 1 是最安全的, 在系统故障时最多丢失一个事务的更新, 但是会对性能有所影响.

在 MySQL 5.7.7 之前, 默认值 sync_binlog 是 0, MySQL 5.7.7 和更高版本使用默认值 1, 这是最安全的选择

## 解析

### binlog 文件

binlog 日志包括两类文件:

* 二进制日志索引文件 (文件名后缀为.index) 用于记录所有有效的的二进制文件
* 二进制日志文件 (文件名后缀为.00000*) 记录数据库所有的 DDL 和 DML 语句事件

binlog 是一个二进制文件集合, 每个 binlog 文件以一个 4 字节的魔数开头, 接着是一组 Events:

* 魔数: 0xfe62696e 对应的是 0xfebin;
* Event: 每个 Event 包含 header 和 data 两个部分; header 提供了 Event 的创建时间, 哪个服务器等信息, data 部分提供的是针对该 Event 的具体信息, 如具体数据的修改;
* 第一个 Event 用于描述 binlog 文件的格式版本, 这个格式就是 event 写入 binlog 文件的格式;
* 其余的 Event 按照第一个 Event 的格式版本写入;
* 最后一个 Event 用于说明下一个 binlog 文件;
* binlog 的索引文件是一个文本文件, 其中内容为当前的 binlog 文件列表

当遇到以下 3 种情况时, MySQL 会重新生成一个新的日志文件, 文件序号递增:

* MySQL 服务器停止或重启时
* 使用 flush logs 命令;
* 当 binlog 文件大小超过 max_binlog_size 变量的值时;

### binlog 模式

记录在二进制日志中的事件的格式取决于二进制记录格式. 支持三种格式类型:

* STATEMENT: 基于 SQL 语句的复制 (statement-based replication, SBR) , 每一条会修改数据的 sql 都会记录在 binlog 中
  * 优点: 不需要记录每一行的变化, 减少了 binlog 日志量, 节约了 IO, 提高了性能.
  * 缺点: 由于记录的只是执行语句, 为了这些语句能在 slave 上正确运行, 因此还必须记录每条语句在执行的时候的一些相关信息, 以保证所有语句能在 slave 得到和在 master 端执行的时候相同的结果. 另外 mysql 的复制, 像一些特定函数的功能, slave 与 master 要保持一致会有很多相关问题.
* ROW: 基于行的复制 (row-based replication, RBR) , 5.1.5 版本的 MySQL 才开始支持 row level 的复制,它不记录 sql 语句上下文相关信息, 仅保存哪条记录被修改.
  * 优点:  binlog 中可以不记录执行的 sql 语句的上下文相关的信息, 仅需要记录那一条记录被修改成什么了. 所以 row 的日志内容会非常清楚的记录下每一行数据修改的细节. 而且不会出现某些特定情况下的存储过程, 或 function, 以及 trigger 的调用和触发无法被正确复制的问题.
  * 缺点:所有的执行的语句当记录到日志中的时候, 都将以每行记录的修改来记录, 这样可能会产生大量的日志内容.
* MIXED: 混合模式复制 (mixed-based replication, MBR) , 从 5.1.8 版本开始, MySQL 提供了 Mixed 格式, 实际上就是 Statement 与 Row 的结合.
  * 在 Mixed 模式下, 一般的语句修改使用 statment 格式保存 binlog, 如一些函数, statement 无法完成主从复制的操作, 则采用 row 格式保存 binlog, MySQL 会根据执行的每一条具体的 sql 语句来区分对待记录的日志形式, 也就是在 Statement 和 Row 之间选择一种.

在 MySQL 5.7.7 之前, 默认的格式是 STATEMENT, 在 MySQL 5.7.7 及更高版本中, 默认值是 ROW. 日志格式通过 binlog-format 指定, 如 binlog-format=STATEMENT, binlog-format=ROW, binlog-format=MIXED.

### 格式

```sql
## at 56154447
#200109  9:19:31 server id 1921  end_log_pos 56154478 CRC32 0x4b37d090   Xid = 196250
COMMIT/*!*/;
## at 56154478
#200109  9:28:54 server id 1921  end_log_pos 56154543 CRC32 0x81f576af   Anonymous_GTID  last_committed=275  sequence_number=276  rbr_only=yes
/*!50718 SET TRANSACTION ISOLATION LEVEL READ COMMITTED*//*!*/;
SET @@SESSION.GTID_NEXT= 'ANONYMOUS'/*!*/;
## at 56154543
#200109  9:28:54 server id 1921  end_log_pos 56154615 CRC32 0x71663570   Query  thread_id=25  exec_time=0  error_code=0
SET TIMESTAMP=1578562134/*!*/;
BEGIN
/*!*/;
## at 56154615
#200109  9:28:54 server id 1921  end_log_pos 56154704 CRC32 0x7069bc01   Table_map: `test`.`mc_org_area` mapped to number 113
## at 56154704
#200109  9:28:54 server id 1921  end_log_pos 56154962 CRC32 0x05b35d8b   Update_rows: table id 113 flags: STMT_END_F
### UPDATE `test`.`mc_org_area`
### WHERE
###   @1='aO4DozlHLxx9uAU7Rwt'
###   @2='3uLeyI1Wossi9SetQPq'
###   @3='阿层才'
###   @4='123444533'
###   @5='03'
###   @6='333444'
###   @7='1233'
###   @8=5
###   @9='0'
###   @10='2020-01-09 17:09:59.511'
###   @11='admin'
###   @12='2020-01-09 17:17:01.453'
###   @13='admin'
### SET
###   @1='aO4DozlHLxx9uAU7Rwt'
###   @2='3uLeyI1Wossi9SetQPq'
###   @3='阿层才'
###   @4='123444533'
###   @5='03'
###   @6='333444'
###   @7='1233'
###   @8=5
###   @9='1'
###   @10='2020-01-09 17:09:59.511'
###   @11='admin'
###   @12='2020-01-09 17:28:54.373'
###   @13='admin'
```

这里是一个 ROW 类型的 binlog

* `# at 56154447`: 文件中的位置, 说明事件记录从文件第 4 个字节开始
* `200109  9:19:31`: 事件时间戳
* `server id 1921`: master 配置的 ID
* `end_log_pos 56154478`: 当前事件结束位置
* `Query`: 事件名称
* `Table_map`: 当前的库表名

### 事件类型

binlog 事件的结构主要有 3 个版本:

* v1: 在 MySQL 3.23 中使用
* v3: 在 MySQL 4.0.2 到 4.1 中使用
* v4: 在 MySQL 5.0 及以上版本中使用

现在一般不会使用 MySQL5.0 以下版本, 所以下面仅介绍 v4 版本的 binlog 事件类型

| 事件类型                 | 说明                                                                                                                                                                                                                                                  |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| UNKNOWN_EVENT            | 此事件从不会被触发, 也不会被写入 binlog 中; 发生在当读取 binlog 时, 不能被识别其他任何事件, 那被视为 UNKNOWN_EVENT                                                                                                                                    |
| START_EVENT_V3           | 每个 binlog 文件开始的时候写入的事件, 此事件被用在 MySQL3.23 – 4.1, MYSQL5.0 以后已经被 FORMAT_DESCRIPTION_EVENT 取代                                                                                                                                 |
| QUERY_EVENT              | 执行更新语句时会生成此事件, 包括: create, insert, update, delete;                                                                                                                                                                                     |
| STOP_EVENT               | 当 mysqld 停止时生成此事件                                                                                                                                                                                                                            |
| ROTATE_EVENT             | 当 mysqld 切换到新的 binlog 文件生成此事件, 切换到新的 binlog 文件可以通过执行 flush logs 命令或者 binlog 文件大于 max_binlog_size 参数配置的大小;                                                                                                    |
| INTVAR_EVENT             | 当 sql 语句中使用了 AUTO_INCREMENT 的字段或者 LAST_INSERT_ID()函数; 此事件没有被用在 binlog_format 为 ROW 模式的情况下                                                                                                                                |
| LOAD_EVENT               | 执行 LOAD DATA INFILE 语句时产生此事件, 在 MySQL 3.23 版本中使用                                                                                                                                                                                      |
| SLAVE_EVENT              | 未使用                                                                                                                                                                                                                                                |
| CREATE_FILE_EVENT        | 执行 LOAD DATA INFILE 语句时产生此事件, 在 MySQL4.0 和 4.1 版本中使用                                                                                                                                                                                 |
| APPEND_BLOCK_EVENT       | 执行 LOAD DATA INFILE 语句时产生此事件, 在 MySQL4.0 版本中使用                                                                                                                                                                                        |
| EXEC_LOAD_EVENT          | 执行 LOAD DATA INFILE 语句时产生此事件, 在 MySQL4.0 和 4.1 版本中使用                                                                                                                                                                                 |
| DELETE_FILE_EVENT        | 执行 LOAD DATA INFILE 语句时产生此事件, 在 MySQL4.0 版本中使用                                                                                                                                                                                        |
| NEW_LOAD_EVENT           | 执行 LOAD DATA INFILE 语句时产生此事件, 在 MySQL4.0 和 4.1 版本中使用                                                                                                                                                                                 |
| RAND_EVENT               | 执行包含 RAND()函数的语句产生此事件, 此事件没有被用在 binlog_format 为 ROW 模式的情况下                                                                                                                                                               |
| USER_VAR_EVENT           | 执行包含了用户变量的语句产生此事件, 此事件没有被用在 binlog_format 为 ROW 模式的情况下                                                                                                                                                                |
| FORMAT_DESCRIPTION_EVENT | 描述事件, 被写在每个 binlog 文件的开始位置, 用在 MySQL5.0 以后的版本中, 代替了 START_EVENT_V3                                                                                                                                                         |
| XID_EVENT                | 支持 XA 的存储引擎才有, 本地测试的数据库存储引擎是 innodb, 所有上面出现了 XID_EVENT; innodb 事务提交产生了 QUERY_EVENT 的 BEGIN 声明, QUERY_EVENT 以及 COMMIT 声明, 如果是 myIsam 存储引擎也会有 BEGIN 和 COMMIT 声明, 只是 COMMIT 类型不是 XID_EVENT |
| BEGIN_LOAD_QUERY_EVENT   | 执行 LOAD DATA INFILE 语句时产生此事件, 在 MySQL5.0 版本中使用                                                                                                                                                                                        |
| EXECUTE_LOAD_QUERY_EVENT | 执行 LOAD DATA INFILE 语句时产生此事件, 在 MySQL5.0 版本中使用                                                                                                                                                                                        |
| TABLE_MAP_EVENT          | 用在 binlog_format 为 ROW 模式下, 将表的定义映射到一个数字, 在行操作事件之前记录 (包括: WRITE_ROWS_EVENT, UPDATE_ROWS_EVENT, DELETE_ROWS_EVENT)                                                                                                       |
| PRE_GA_WRITE_ROWS_EVENT  | 已过期, 被 WRITE_ROWS_EVENT 代替                                                                                                                                                                                                                      |
| PRE_GA_UPDATE_ROWS_EVENT | 已过期, 被 UPDATE_ROWS_EVENT 代替                                                                                                                                                                                                                     |
| PRE_GA_DELETE_ROWS_EVENT | 已过期, 被 DELETE_ROWS_EVENT 代替                                                                                                                                                                                                                     |
| WRITE_ROWS_EVENT         | 用在 binlog_format 为 ROW 模式下, 对应 insert 操作                                                                                                                                                                                                    |
| UPDATE_ROWS_EVENT        | 用在 binlog_format 为 ROW 模式下, 对应 update 操作                                                                                                                                                                                                    |
| DELETE_ROWS_EVENT        | 用在 binlog_format 为 ROW 模式下, 对应 delete 操作                                                                                                                                                                                                    |
| INCIDENT_EVENT           | 主服务器发生了不正常的事件, 通知从服务器并告知可能会导致数据处于不一致的状态                                                                                                                                                                          |
| HEARTBEAT_LOG_EVENT      | 主服务器告诉从服务器, 主服务器还活着, 不写入到日志文件中                                                                                                                                                                                              |

## 参考文档

* [MySQL Binlog 介绍](https://laijianfeng.org/2019/03/MySQL-Binlog-%E4%BB%8B%E7%BB%8D/)
* [Mysql binlog 应用场景与原理深度剖析](https://www.cnblogs.com/caicz/p/11009400.html)
