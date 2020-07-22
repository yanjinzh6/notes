---
title: MySQL 查看事务和锁等待状态命令
date: 2020-07-18 20:00:00
tags: 'MySQL'
categories:
  - ['使用说明', '软件']
permalink: show-tx-lock-command
photo:
---

## 查看数据库版本

```sql
select version();
```

## 查看数据库引擎

```sql
show variables like '%engine%';
```

## 查看 gap 锁开启状态

```sql
show variables like 'innodb_locks_unsafe_for_binlog';
```

默认值为 0, 即启用 gap lock

作用就是控制 innodb 是否对 gap 加锁

这一设置变更并不影响外键和唯一索引（含主键）对 gap 进行加锁的需要

开启 innodb_locks_unsafe_for_binlog 的 REPEATABLE-READ 事务隔离级别, 很大程度上已经蜕变成了 READ-COMMITTED

## 查看事务隔离级别

```sql
SELECT @@global.tx_isolation;
SELECT @@session.tx_isolation;
SELECT @@tx_isolation;
```

## 设置隔离级别

```sql
SET [SESSION | GLOBAL] TRANSACTION ISOLATION LEVEL {READ UNCOMMITTED | READ COMMITTED | REPEATABLE READ | SERIALIZABLE}
-- 例如: set session transaction isolation level read uncommitted;
```

## 查看 autocommit

```sql
SHOW VARIABLES LIKE 'autocommit';
-- 关闭
set autocommit=0;
```

## 查看自增锁模式

```sql
show variables like 'innodb_autoinc_lock_mode';
```

innodb_autoinc_lock_mode 有 3 种配置模式: 0, 1, 2, 分别对应 "传统模式" , "连续模式" , "交错模式"

- 传统模式: 涉及 auto-increment 列的插入语句加的表级 AUTO-INC 锁, 只有插入执行结束后才会释放锁, 这是一种兼容 MySQL 5.1 之前版本的策略
- 连续模式: 可以事先确定插入行数的语句(包括单行和多行插入), 分配连续的确定的 auto-increment 值；对于插入行数不确定的插入语句, 仍加表锁, 这种模式下, 事务回滚, auto-increment 值不会回滚, 换句话说, 自增列内容会不连续
- 交错模式: 同一时刻多条 SQL 语句产生交错的 auto-increment 值

## 查看锁等待超时时间

```sql
show variables like 'innodb_lock_wait_timeout';
```

## 查看表状态

```sql
show table status like 'plan_branch'\G;
show table status from test like 'plan_branch'\G;
```

## 查看 SQL 性能

```sql
show profiles
show profile for query 1;
```

## 查看当前最新事务 ID

```sql
-- 每开启一个新事务, 记录当前最新事务的 id, 可用于后续死锁分析
show engine innodb status\G;
```

## 查看事务锁等待状态情况

```sql
select from information_schema.innodb_locks;
select from information_schema.innodb_lock_waits;
select * from information_schema.innodb_trx;
```

## 查看 innodb 状态(包含最近的死锁日志)

```sql
show engine innodb status;
```

## 查询锁表情况

```sql
show OPEN TABLES where In_use > 0;
```
