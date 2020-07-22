---
title: MySQL Insert 加锁死锁
date: 2020-07-18 17:00:00
tags: 'MySQL'
categories:
  - ['使用说明', '软件']
permalink: insert-lock
photo:
---

## Insert 加锁

简单的 insert 会在 insert 的行对应的索引记录上加一个排它锁, 这是一个 record lock, 并没有 gap, 所以并不会阻塞其他 session 在 gap 间隙里插入记录

不过在 insert 操作之前, 还会加一种锁, 官方文档称它为 insertion intention gap lock, 也就是意向的 gap 锁, 这个意向 gap 锁的作用就是预示着当多事务并发插入相同的 gap 空隙时, 只要插入的记录不是 gap 间隙中的相同位置, 则无需等待其他 session 就可完成, 这样就使得 insert 操作无须加真正的 gap lock

有了插入意向锁, 就可以减少 gap lock 的使用, 总体上提高数据插入的并发能力

假设有一个记录索引包含键值 4 和 7, 不同的事务分别插入 5 和 6, 每个事务都会产生一个加在 4-7 之间的插入意向锁, 获取在插入行上的排它锁, 但是不会被互相锁住, 因为数据行并不冲突

假设发生了一个唯一键冲突错误, 那么将会在重复的索引记录上加读锁, 当有多个 session 同时插入相同的行记录时, 如果另外一个 session 已经获得该行的排它锁, 那么将会导致死锁

<!-- more -->

## Insert 死锁场景分析

### duplicate key error 引发的死锁

在两个以上的事务同时进行唯一键值相同的记录插入操作, 高并发场景中程序没有生成唯一的主键

#### 表结构

```sql
CREATE TABLE `aa` (
  `id` int(10) unsigned NOT NULL COMMENT '主键',
  `name` varchar(20) NOT NULL DEFAULT '' COMMENT '姓名',
  `age` int(11) NOT NULL DEFAULT '0' COMMENT '年龄',
  `stage` int(11) NOT NULL DEFAULT '0' COMMENT '关卡数',
  PRIMARY KEY (`id`),
  UNIQUE KEY `udx_name` (`name`),
  KEY `idx_stage` (`stage`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 表数据

```sh
mysql> select * from aa;
+----+------+-----+-------+
| id | name | age | stage |
+----+------+-----+-------+
| 1 | yst | 11 | 8 |
| 2 | dxj | 7 | 4 |
| 3 | lb | 13 | 7 |
| 4 | zsq | 5 | 7 |
| 5 | lxr | 13 | 4 |
+----+------+-----+-------+
```

#### 事务执行时序

| T1(36727) | T2(36728) | T3(36729) |
| -- | -- | -- |
| begin; | begin; | begin; |
| insert into aa values(6, 'test', 12, 3); |   |   |
|   | insert into aa values(6, 'test', 12, 3); |   |
|   |   | insert into aa values(6, 'test', 12, 3); |
| rollback; |   |   |
|   |   | ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction |
|   | Query OK, 1 row affected (13.10 sec) |   |

如果 T1 未 rollback, 而是 commit 的话, T2 和 T3 会报唯一键冲突: `ERROR 1062 (23000): Duplicate entry '6' for key 'PRIMARY'`

#### 事务锁占用情况

T1 rollback 前, 各事务锁占用情况:

```sh
mysql> select * from information_schema.innodb_locks;
+--------------+-------------+-----------+-----------+-------------+------------+------------+-----------+----------+-----------+
| lock_id | lock_trx_id | lock_mode | lock_type | lock_table | lock_index | lock_space | lock_page | lock_rec | lock_data |
+--------------+-------------+-----------+-----------+-------------+------------+------------+-----------+----------+-----------+
| 36729:24:3:7 | 36729 | S | RECORD | `test`.`aa` | PRIMARY | 24 | 3 | 7 | 6 |
| 36727:24:3:7 | 36727 | X | RECORD | `test`.`aa` | PRIMARY | 24 | 3 | 7 | 6 |
| 36728:24:3:7 | 36728 | S | RECORD | `test`.`aa` | PRIMARY | 24 | 3 | 7 | 6 |
+--------------+-------------+-----------+-----------+-------------+------------+------------+-----------+----------+-----------+
```

注: **mysql 有自己的一套规则来决定 T2 与 T3 哪个进行回滚, 这里不做讨论**

#### 死锁日志

```sh
------------------------
LATEST DETECTED DEADLOCK
------------------------
2016-07-21 19:34:23 700000a3f000
*** (1) TRANSACTION:
TRANSACTION 36728, ACTIVE 199 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1184, 2 row lock(s)
MySQL thread id 13, OS thread handle 0x700000b0b000, query id 590 localhost root update
insert into aa values(6, 'test', 12, 3)
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 24 page no 3 n bits 80 index `PRIMARY` of table `test`.`aa` trx id 36728 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
0: len 8; hex 73757072656d756d; asc supremum;;
*** (2) TRANSACTION:
TRANSACTION 36729, ACTIVE 196 sec inserting
mysql tables in use 1, locked 1
4 lock struct(s), heap size 1184, 2 row lock(s)
MySQL thread id 14, OS thread handle 0x700000a3f000, query id 591 localhost root update
insert into aa values(6, 'test', 12, 3)
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 24 page no 3 n bits 80 index `PRIMARY` of table `test`.`aa` trx id 36729 lock mode S
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
0: len 8; hex 73757072656d756d; asc supremum;;
*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 24 page no 3 n bits 80 index `PRIMARY` of table `test`.`aa` trx id 36729 lock_mode X insert intention waiting
Record lock, heap no 1 PHYSICAL RECORD: n_fields 1; compact format; info bits 0
0: len 8; hex 73757072656d756d; asc supremum;;
*** WE ROLL BACK TRANSACTION (2)
```

#### 死锁原因

- 事务 T1 成功插入记录, 并获得索引 id=6 上的排他记录锁(LOCK_X | LOCK_REC_NOT_GAP)
- 事务 T2、T3 也开始插入记录, 请求排他插入意向锁(LOCK_X | LOCK_GAP | LOCK_INSERT_INTENTION)；但由于发生重复唯一键冲突, 各自请求的排他记录锁(LOCK_X | LOCK_REC_NOT_GAP)转成共享记录锁(LOCK_S | LOCK_REC_NOT_GAP)
- T1 回滚释放索引 id=6 上的排他记录锁(LOCK_X | LOCK_REC_NOT_GAP), T2 和 T3 都要请求索引 id=6 上的排他记录锁(LOCK_X | LOCK_REC_NOT_GAP)
- 由于 X 锁与 S 锁互斥, T2 和 T3 都等待对方释放 S 锁
- 死锁产生

如果此场景下, 只有两个事务 T1 与 T2 或者 T1 与 T3, 则不会引发如上死锁情况产生

### GAP 与 Insert Intention 冲突引发的死锁

#### 表 2 结构

```sql
CREATE TABLE `t` (
  `a` int(11) NOT NULL,
  `b` int(11) DEFAULT NULL,
  PRIMARY KEY (`a`),
  KEY `idx_b` (`b`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

#### 表 2 数据

```sh
mysql> select * from t;
+----+------+
| a | b |
+----+------+
| 1 | 2 |
| 2 | 3 |
| 3 | 4 |
| 11 | 22 |
+----+------+
```

#### 事务 2 执行时序

| T1(36831) | T2(36832) |
| -- | -- |
| begin; | begin; |
| select * from t where b = 6 for update; |   |
|   | select * from t where b = 8 for update; |
| insert into t values (4,5); |   |
|   | insert into t values (4,5); |
|   | ERROR 1213 (40001): Deadlock found when trying to get lock; try restarting transaction |
| Query OK, 1 row affected (5.45 sec) |   |

#### 事务 2 锁占用情况

T2 insert 前, 各事务锁占用情况:

```sh
mysql> select * from information_schema.innodb_locks;
+--------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+-----------+
| lock_id | lock_trx_id | lock_mode | lock_type | lock_table | lock_index | lock_space | lock_page | lock_rec | lock_data |
+--------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+-----------+
| 36831:25:4:5 | 36831 | X,GAP | RECORD | `test`.`t` | idx_b | 25 | 4 | 5 | 22, 11 |
| 36832:25:4:5 | 36832 | X,GAP | RECORD | `test`.`t` | idx_b | 25 | 4 | 5 | 22, 11 |
+--------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+-----------+
```

#### 死锁日志 2

```sh
------------------------
LATEST DETECTED DEADLOCK
------------------------
2016-07-28 12:28:34 700000a3f000
*** (1) TRANSACTION:
TRANSACTION 36831, ACTIVE 17 sec inserting
mysql tables in use 1, locked 1
LOCK WAIT 4 lock struct(s), heap size 1184, 3 row lock(s), undo log entries 1
MySQL thread id 38, OS thread handle 0x700000b0b000, query id 953 localhost root update
insert into t values (4,5)
*** (1) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 25 page no 4 n bits 72 index `idx_b` of table `test`.`t` trx id 36831 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 5 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
0: len 4; hex 80000016; asc ;;
1: len 4; hex 8000000b; asc ;;
*** (2) TRANSACTION:
TRANSACTION 36832, ACTIVE 13 sec inserting
mysql tables in use 1, locked 1
3 lock struct(s), heap size 360, 2 row lock(s)
MySQL thread id 39, OS thread handle 0x700000a3f000, query id 954 localhost root update
insert into t values (4,5)
*** (2) HOLDS THE LOCK(S):
RECORD LOCKS space id 25 page no 4 n bits 72 index `idx_b` of table `test`.`t` trx id 36832 lock_mode X locks gap before rec
Record lock, heap no 5 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
0: len 4; hex 80000016; asc ;;
1: len 4; hex 8000000b; asc ;;
*** (2) WAITING FOR THIS LOCK TO BE GRANTED:
RECORD LOCKS space id 25 page no 3 n bits 72 index `PRIMARY` of table `test`.`t` trx id 36832 lock mode S locks rec but not gap waiting
Record lock, heap no 5 PHYSICAL RECORD: n_fields 4; compact format; info bits 0
0: len 4; hex 80000004; asc ;;
1: len 6; hex 000000008fdf; asc ;;
2: len 7; hex 8d000001d00110; asc ;;
3: len 4; hex 80000005; asc ;;
*** WE ROLL BACK TRANSACTION (2)
```

#### 死锁原因 2

- 事务 T1 执行查询语句, 在索引 b=6 上加排他 Next-key 锁(LOCK_X | LOCK_ORDINARY), 会锁住 idx_b 索引范围(4, 22)
- 事务 T2 执行查询语句, 在索引 b=8 上加排他 Next-key 锁(LOCK_X | LOCK_ORDINARY), 会锁住 idx_b 索引范围(4, 22), 由于请求的 GAP 与已持有的 GAP 是兼容的, 因此, 事务 T2 在 idx_b 索引范围(4, 22)也能加锁成功
- 事务 T1 执行插入语句, 会先加排他 Insert Intention 锁, 由于请求的 Insert Intention 锁与已有的 GAP 锁不兼容, 则事务 T1 等待 T2 释放 GAP 锁
- 事务 T2 执行插入语句, 也会等待 T1 释放 GAP 锁
- 死锁产生

注: `LOCK_ORDINARY 拥有 LOCK_GAP 一部分特性`

## 引用

- [mysql insert 锁机制(insert 死锁)](https://blog.csdn.net/varyall/article/details/80219459)
