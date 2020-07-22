---
title: MySQL Lock wait timeout exceeded 错误
date: 2020-07-18 10:00:00
tags: 'MySQL'
categories:
  - ['使用说明', '软件']
permalink: lock-wait-timeout-exceeded
photo:
---

## 简介

某天出现了如下的日志

```sh
Caused by: org.springframework.dao.CannotAcquireLockException:
### Error updating database.  Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLTransactionRollbackException: Lock wait timeout exceeded; try restarting transaction
### The error may involve com...
### The error occurred while setting parameters
### SQL: INSERT INTO C_MSC_MESSAGE(   UUID,   CODE,   SEND_USER_TYPE,   SEND_USER,   SEND_USER_NAME,   MS_TYPE,   TITLE,   CONTENT,   WANT_SEND_TIME,   BUSINESS_CODE,   SERVER_CODE,   CREATE_DATE,   APP_CODE,   ATTA_INFO1,   ATTA_INFO2,   SING_TYPE,   TOPARTY,   TOTAG,   AGENTID,   MSGTYPE   ) VALUES (   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?,   ?   )
### Cause: com.mysql.jdbc.exceptions.jdbc4.MySQLTransactionRollbackException: Lock wait timeout exceeded; try restarting transaction
; SQL []; Lock wait timeout exceeded; try restarting transaction; nested exception is com.mysql.jdbc.exceptions.jdbc4.MySQLTransactionRollbackException: Lock wait timeout exceeded; try restarting transaction
  at org.springframework.jdbc.support.SQLErrorCodeSQLExceptionTranslator.doTranslate(SQLErrorCodeSQLExceptionTranslator.java:259)
  at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:73)
  at org.mybatis.spring.MyBatisExceptionTranslator.translateExceptionIfPossible(MyBatisExceptionTranslator.java:75)
  at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:447)
  at com.sun.proxy.$Proxy45.insert(Unknown Source)
  at org.mybatis.spring.SqlSessionTemplate.insert(SqlSessionTemplate.java:279)
  at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:56)
  at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:53)
  at com.sun.proxy.$Proxy54.insert(Unknown Source)
```

看这异常的提示是超过了锁等待的时间, 尝试重启事务, 查了下资料, 发现这个问题很普遍, 但是造成异常的 SQL 是 INSERT 语句的很少

Mysql 造成锁的情况有很多

- 执行 DML 操作没有 commit, 再执行删除操作就会锁表
- 在同一事务内先后对同一条数据进行插入和更新操作
- 表索引设计不当, 导致数据库出现死锁
- 长事物, 阻塞 DDL, 继而阻塞所有同表的后续操作

锁等待时间超时并不一定就是死锁, MySQL 的死锁异常是 Dead Lock, 两者的区别是

- Lock wait timeout exceeded: 后提交的事务等待前面处理的事务释放锁, 但是在等待的时候超过了 mysql 的锁等待时间, 就会引发这个异常
- Dead Lock: 两个事务互相等待对方释放相同资源的锁, 从而造成的死循环, 就会引发这个异常

<!-- more -->

## 详细处理

### 紧急处理

通过 `show full processlist;` 查询出有问题的进程 ID, 并使用 `kill` 命令将其结束, 有的时候通过 processlist 是看不出哪里有锁等待的, 当两个事务都在 commit 阶段是无法体现在 processlist 上

`show full processlist` 参数

- id: ID 标识, 要 kill 一个语句的时候很有用
- use: 当前连接用户
- host: 显示这个连接从哪个 ip 的哪个端口上发出
- db: 数据库名
- command: 连接状态, 一般是休眠（sleep）, 查询（query）, 连接（connect）
- time: 连接持续时间, 单位是秒
- state: 显示当前 sql 语句的状态
- info: 显示这个 sql 语句

state 参数列表

| 状态  | 描述 |
| -- | -- |
| Checking table | 正在检查数据表（这是自动的） |
| Closing tables | 正在将表中修改的数据刷新到磁盘中, 同时正在关闭已经用完的表, 这是一个很快的操作, 如果不是这样的话, 就应该确认磁盘空间是否已经满了或者磁盘是否正处于重负中 |
| Connect Out | 复制从服务器正在连接主服务器 |
| Copying to tmp table on disk | 由于临时结果集大于 tmp_table_size, 正在将临时表从内存存储转为磁盘存储以此节省内存 |
| Creating tmp table | 正在创建临时表以存放部分查询结果 |
| deleting from main table | 服务器正在执行多表删除中的第一部分, 刚删除第一个表 |
| deleting from reference tables | 服务器正在执行多表删除中的第二部分, 正在删除其他表的记录 |
| Flushing tables | 正在执行 FLUSH TABLES, 等待其他线程关闭数据表 |
| Killed | 发送了一个 kill 请求给某线程, 那么这个线程将会检查 kill 标志位, 同时会放弃下一个 kill 请求, MySQL 会在每次的主循环中检查 kill 标志位, 不过有些情况下该线程可能会过一小段才能死掉, 如果该线程程被其他线程锁住了, 那么 kill 请求会在锁释放时马上生效 |
| Locked | 被其他查询锁住了 |
| Sending data | 正在处理 SELECT 查询的记录, 同时正在把结果发送给客户端 |
| Sorting for group | 正在为 GROUP BY 做排序 |
| Sorting for order | 正在为 ORDER BY 做排序 |
| Opening tables | 这个过程应该会很快, 除非受到其他因素的干扰, 例如, 在执 ALTER TABLE 或 LOCK TABLE 语句行完以前, 数据表无法被其他线程打开, 正尝试打开一个表 |
| Removing duplicates | 正在执行一个 SELECT DISTINCT 方式的查询, 但是 MySQL 无法在前一个阶段优化掉那些重复的记录, 因此, MySQL 需要再次去掉重复的记录, 然后再把结果发送给客户端 |
| Reopen table | 获得了对一个表的锁, 但是必须在表结构修改之后才能获得这个锁, 已经释放锁, 关闭数据表, 正尝试重新打开数据表 |
| Repair by sorting | 修复指令正在排序以创建索引 |
| Repair with keycache | 修复指令正在利用索引缓存一个一个地创建新索引, 它会比 Repair by sorting 慢些 |
| Searching rows for update | 正在讲符合条件的记录找出来以备更新, 它必须在 UPDATE 要修改相关的记录之前就完成了 |
| Sleeping | 正在等待客户端发送新请求. |
| System lock | 正在等待取得一个外部的系统锁, 如果当前没有运行多个 mysqld 服务器同时请求同一个表, 那么可以通过增加-- skip-external-locking 参数来禁止外部系统锁 |
| Upgrading lock | INSERT DELAYED 正在尝试取得一个锁表以插入新记录 |
| Updating | 正在搜索匹配的记录, 并且修改它们 |
| User Lock | 正在等待 GET_LOCK() |
| Waiting for tables | 该线程得到通知, 数据表结构已经被修改了, 需要重新打开数据表以取得新的结构, 然后, 为了能的重新打开数据表, 必须等到所有其他线程关闭这个表, 以下几种情况下会产生这个通知：FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE, REPAIR TABLE, ANALYZE TABLE,或 OPTIMIZE TABLE |
| waiting for handler insert | INSERT DELAYED 已经处理完了所有待处理的插入操作, 正在等待新的请求 |

### 修改数据库参数

查询 MySQL `innodb_lock_wait_timeout` 配置, 该参数指的是事务等待获取资源等待的最长时间, 超过这个时间还未分配到资源则会返回应用失败, 当锁等待超过设置时间的时候, 就会报如下的错误, `ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction`, 其参数的时间单位是秒, 最小可设置为 1s (一般不会设置得这么小), 最大可设置 1073741824 秒, 默认安装时这个值是 50s (默认参数设置), 有可能是因为修改了配置导致超时时间太短

```sql
-- 查看
SHOW VARIABLES LIKE 'innodb_lock_wait_timeout';
-- 通过命令修改
set innodb_lock_wait_timeout=100;
-- global 的修改对当前线程是不生效的, 只有建立新的连接才生效
set global innodb_lock_wait_timeout=100;
```

通过修改配置文件可以实现全局修改

修改参数文件 `/etc/my.cnf` 添加 `innodb_lock_wait_timeout = 100`

注意与参数 lock_wait_timeout 的区别

- innodb_lock_wait_timeout: innodb 的 dml 操作的行级锁的等待时间
- lock_wait_timeout: 数据结构 ddl 操作的锁的等待时间

### 查询事务记录表并结束事务线程

下面几张表是 innodb 的事务和锁的信息表

innodb_trx: 当前运行的所有事务
innodb_locks: 当前出现的锁
innodb_lock_waits: 锁等待的对应关系

innodb_trx 表

- trx_id: 事务 ID
- trx_state: 事务状态, 有以下几种状态: RUNNING, LOCK WAIT, ROLLING BACK 和 COMMITTING
- trx_started: 事务开始时间
- trx_requested_lock_id: 事务当前正在等待锁的标识, 可以和 INNODB_LOCKS 表 JOIN 以得到更多详细信息
- trx_wait_started: 事务开始等待的时间
- trx_weight: 事务的权重
- trx_mysql_thread_id: 事务线程 ID, 可以和 PROCESSLIST 表 JOIN
- trx_query: 事务正在执行的 SQL 语句
- trx_operation_state: 事务当前操作状态
- trx_tables_in_use: 当前事务执行的 SQL 中使用的表的个数
- trx_tables_locked: 当前执行 SQL 的行锁数量
- trx_lock_structs: 事务保留的锁数量
- trx_lock_memory_bytes: 事务锁住的内存大小, 单位为 BYTES
- trx_rows_locked: 事务锁住的记录数, 包含标记为 DELETED, 并且已经保存到磁盘但对事务不可见的行
- trx_rows_modified: 事务更改的行数
- trx_concurrency_tickets: 事务并发票数
- trx_isolation_level: 当前事务的隔离级别
- trx_unique_checks: 是否打开唯一性检查的标识
- trx_foreign_key_checks: 是否打开外键检查的标识
- trx_last_foreign_key_error: 最后一次的外键错误信息
- trx_adaptive_hash_latched: 自适应散列索引是否被当前事务锁住的标识
- trx_adaptive_hash_timeout: 是否立刻放弃为自适应散列索引搜索 LATCH 的标识

innodb_locks 表

- lock_id: 锁 ID
- lock_trx_id: 拥有锁的事务 ID, 可以和 INNODB_TRX 表 JOIN 得到事务的详细信息
- lock_mode: 锁的模式, 有如下锁类型: 行级锁包括: S, X, IS, IX, 分别代表: 共享锁, 排它锁, 意向共享锁, 意向排它锁, 表级锁包括: S_GAP, X_GAP, IS_GAP, IX_GAP 和 AUTO_INC, 分别代表共享间隙锁, 排它间隙锁, 意向共享间隙锁, 意向排它间隙锁和自动递增锁
- lock_type: 锁的类型, RECORD 代表行级锁, TABLE 代表表级锁
- lock_table: 被锁定的或者包含锁定记录的表的名称
- lock_index: 当 LOCK_TYPE=’RECORD’ 时, 表示索引的名称, 否则为 NULL
- lock_space: 当 LOCK_TYPE=’RECORD’ 时, 表示锁定行的表空间 ID, 否则为 NULL
- lock_page: 当 LOCK_TYPE=’RECORD’ 时, 表示锁定行的页号, 否则为 NULL
- lock_rec: 当 LOCK_TYPE=’RECORD’ 时, 表示一堆页面中锁定行的数量, 亦即被锁定的记录号, 否则为 NULL
- lock_data: 当 LOCK_TYPE=’RECORD’ 时, 表示锁定行的主键, 否则为 NULL

innodb_lock_waits 表

- requesting_trx_id: 请求事务的 ID
- requested_lock_id: 事务所等待的锁定的 ID, 可以和 INNODB_LOCKS 表 JOIN
- blocking_trx_id: 阻塞事务的 ID
- blocking_lock_id: 某一事务的锁的 ID, 该事务阻塞了另一事务的运行, 可以和 INNODB_LOCKS 表 JOIN

```sql
-- 查看 innodb_lock_waits 表
SELECT * FROM innodb_lock_waits;

-- innodb_locks 表和 innodb_lock_waits 表结合:
SELECT * FROM innodb_locks WHERE lock_trx_id IN (SELECT blocking_trx_id FROM innodb_lock_waits);

-- innodb_locks 表 JOIN innodb_lock_waits 表:
SELECT innodb_locks.* FROM innodb_locks JOIN innodb_lock_waits ON (innodb_locks.lock_trx_id = innodb_lock_waits.blocking_trx_id);

-- 查询 innodb_trx 表: trx_mysql_thread_id 即 kill 掉事务线程 ID
SELECT trx_id, trx_requested_lock_id, trx_mysql_thread_id, trx_query FROM innodb_trx WHERE trx_state = 'LOCK WAIT';

-- 查询状态
SHOW ENGINE INNODB STATUS;

-- 查询进程列表
SHOW PROCESSLIST;
-- KILL 掉发生锁等待的线程
kill ID;
```

## 小结

当前只是进行 MySQL 的处理, 还需要对程序进行排查

- 优化 SQL, 创建索引
- 是否存在过长的事务, 或者事务中包含了第三方等待超长的逻辑
- 是否多个事务在并发过程中容易出现资源竞争
- 如果是并发数太高那就需要降低并发或者增加集群操作

## 参考

- [MySQL 事务锁等待超时 Lock wait timeout exceeded; try restarting transaction](https://juejin.im/post/5e5b7935518825492d4de463)
