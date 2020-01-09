---
title: MySQL 数据库
date: 2019-12-21 18:26:54
tags: 'MySQL'
categories:
  - ['使用说明', '软件']
permalink: mysql-introduction
photo:
---

# MySQL

## 简介

MySQL 是一种关联数据库管理系统, 关联数据库将数据保存在不同的表中, 而不是将所有数据放在一个大仓库内, 这样就增加了速度并提高了灵活性. MySQL 所使用的 SQL 语言是用于访问数据库的最常用标准化语言. MySQL 软件采用了双授权政策, 它分为社区版和商业版, 由于其体积小, 速度快, 总体拥有成本低, 尤其是开放源码这一特点, 一般中小型网站的开发都选择 MySQL 作为网站数据库.

<!-- more -->

## 安装

推荐使用 docker 容器.

```sh
$ docker run -it --network some-network --rm mysql mysql -hsome-mysql -uexample-user -p
```

```yml
# Use root/example as user/password credentials
version: '3.1'

services:

  db:
    image: mysql
    command: --default-authentication-plugin=mysql_native_password
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example

  adminer:
    image: adminer
    restart: always
    ports:
      - 8080:8080
```

## 基础命令

* 创建用户 `mysql>create user test identified by 'BaC321@#';`
* 修改密码
  * ##5.5 版本及以前的命令 `mysql>set password for test=passowrd('!1A@2#3');`
  * ##5.6 及以上命令 `mysql>update mysql.user set authentication_string=password('A1b2c3#!@') where user='test';`
* 创建用户并授权 `mysql>grant select,insert,update on student.* to test@localhost identified by 'A1b2c3#!@';`
* 查看授权 `mysql> show grants for test@localhost;`
* 移除权限 `mysql> revoke insert,update on student.* from test@localhost;`
* 创建库 `mysql> create database student;`
* 创建表 `mysql> create table T1 (name varchar(10) not null,sex varchar(10) not null);`
  * ##通过现有的表创建新表 `mysql> create table T2 as select * from T1;`
* 创建主键 `mysql> alter table T1 add primary key(name);`

### mysql 程序命令

mysql 是数据库管理命令, 通过 mysql --help 来查看相关参数及使用说明

* mysql --help #mysql 数据库管理命令
* Usage: mysql [OPTIONS] [database] #语法格式
* --help #查看帮助文档
* --auto-rehash #自动补全功能
* -A, --no-auto-rehash #不需自动补全
* -B, --batch #不使用历史文件, 禁用交互
* --character-sets-dir=name #字符集安装目录
* -C, --compress #客户端与服务端传递信息时压缩
* -#--debug [=#] #调用功能
* -D, --database=name #使用数据库
* --default-character-set=name #设置默认字符集
* -e, --execute=name #执行 sql 语句
* -E, --vertical #垂直打印输出信息
* -f, --force #跳过错误, 执行下面的命令
* -G, --named-commands #查询结果按列打印
* -i, --ignore-spaces #忽略空格
* -h, --host=name #设置连接服务器的地址与 IP
* --line-numbers #显示有错误的行号
* -L, --skip-line-numbers #忽略有错误的行号
* -n, --unbuffered #每次执行 sql 后刷新缓存
* --column-names #查询时显示列信息
* -N, --skip-column-names #不显示列信息
* -p, --password [=name] #输入密码信息
* -P, --port=# #设置端口信息
  * --prompt=name #设置 mysql 提示符
  * --protocol=name #设置使用协议
* -s, --silent #一行一行输出, tab 间隔
* -S, --socket=name #连接服务器使用 socket 文件
* -t, --table #以表格的格式输出
* -u, --user=name #连接服务器的用户名
* -v, --verbose #打印 sql 执行的命令
* -V, --version #输出版本信息
* -w, --wait #服务器停机后等待重启的时间
* --connect-timeout=# #连接前要等待的时间
* --max-allowed-packet=# #服务器发送与接收包的最大长度
* --show-warnings #显示警告信息

### mysqldump 数据备份命令 (逻辑备份)

日常使用最为频繁的命令之一, 也是中小企业或者说数据量不大的情况下常用的数据库备份命令, 非常实用.

* mysqldump --help #mysql 数据库备份命令 (逻辑备份)
* Usage: mysqldump [OPTIONS] database [tables]
* mysqldump [OPTIONS] --databases [OPTIONS] DB1 [DB2 DB3...]
* mysqldump [OPTIONS] --all-databases [OPTIONS] #备份命令格式
* --print-defaults #打印默认的程序参数列表
* --no-defaults #不输出默认选项参数
* --defaults-file=# #设置指定的选项参数文件
* -A, --all-databases #所有数据库
* --add-drop-database #创建数据之前添加 drop 数据库语句
* --add-locks #每个表导出之前增加 lock tables 并且之后 unlock tables
* --character-sets-dir #字符集文件目录
* --compact #导出更少的输出信息
* -B --databases #指定数据库
* --debug-info #输出调试信息并退出
* --default-character-set #设置默认字符集, 默认为 utf8
* --dump-slave #将主 binlog 位置和文件名追加到导出的数据文件中
* --events,-E #备份事件信息
* --flush-logs,-F #备份后刷新日志
* -p, --password [=name] #连接数据库密码
* -P, --port=# #设置端口信息
* -S, --socket=name #连接服务器使用 socket 文件
* -V, --version #输出版本信息
* -u, --user=name #连接服务器的用户名

### mysqlbinlog 命令

mysqlbinlog 是用来查看 binlog 二进制日志文件信息的命令, 也是日常经常使用的命令之一, 通常在恢复数据库数据时使用.

* mysqlbinlog --help #查看 mysql 的 binlog 日志文件记录的信息
* Usage: mysqlbinlog [options] log-files #语法格式
* --character-sets-dir=name #指定字符集文件目录
* -d, --database=name #查看指定数据库的日志文件
* -h, --host=name #查看指定主机上的日志文件
* --start-position=953 #起始 pos 点
* --stop-position=1437 #结束 pos 点
* --start-datetime= #起始时间点
* --stop-datetime= #结束时间点
* --database= #指定只恢复数据库

### SQL 语句

#### SHOW

```sql
SHOW PROCESSLIST -- 显示哪些线程正在运行
SHOW VARIABLES -- 显示系统变量信息
```

#### 数据库操作

```sql
-- 查看当前数据库
SELECT DATABASE();

-- 显示当前时间, 用户名, 数据库版本
SELECT now(), user(), version();

-- 创建库
CREATE DATABASE [IF NOT EXISTS] 数据库名 数据库选项
-- 数据库选项:
  -- CHARACTER SET charset_name
  -- COLLATE collation_name

-- 查看已有库
SHOW DATABASES [LIKE 'PATTERN']

-- 查看当前库信息
SHOW CREATE DATABASE 数据库名

-- 修改库的选项信息
ALTER DATABASE 库名 选项信息

-- 删除库
DROP DATABASE [IF EXISTS] 数据库名
    -- 同时删除该数据库相关的目录及其目录内容
```

#### 表的操作

```sql
-- 创建表
CREATE [TEMPORARY] TABLE [IF NOT EXISTS] [库名.] 表名 ( 表的结构定义 )[表选项]
-- 每个字段必须有数据类型
-- 最后一个字段后不能有逗号
-- TEMPORARY 临时表, 会话结束时表自动消失
-- 对于字段的定义: 字段名 数据类型 [NOT NULL | NULL] [DEFAULT default_value] [AUTO_INCREMENT] [UNIQUE [KEY] | [PRIMARY] KEY] [COMMENT 'string']
-- 表选项
  -- 字符集
  CHARSET = charset_name
  -- 如果表没有设定, 则使用数据库字符集
  -- 存储引擎
  ENGINE = engine_name
  -- 表在管理数据时采用的不同的数据结构, 结构不同会导致处理方式, 提供的特性操作等不同
  -- 常见的引擎: InnoDB MyISAM Memory/Heap BDB Merge Example CSV MaxDB Archive
  -- 不同的引擎在保存表的结构和数据时采用不同的方式
  -- MyISAM 表文件含义: .frm 表定义, .MYD 表数据, .MYI 表索引
  -- InnoDB 表文件含义: .frm 表定义, 表空间数据和日志文件
  SHOW ENGINES -- 显示存储引擎的状态信息
  SHOW ENGINE 引擎名 {LOGS|STATUS} -- 显示存储引擎的日志或状态信息
    -- 自增起始数
    AUTO_INCREMENT = 行数
    -- 数据文件目录
    DATA DIRECTORY = '目录'
    -- 索引文件目录
    INDEX DIRECTORY = '目录'
    -- 表注释
    COMMENT = 'string'
    -- 分区选项
    PARTITION BY ... (详细见手册)

-- 查看所有表
SHOW TABLES [LIKE 'pattern']
SHOW TABLES FROM 表名

-- 查看表机构
SHOW CREATE TABLE 表名 (信息更详细)
DESC 表名 / DESCRIBE 表名 / EXPLAIN 表名 / SHOW COLUMNS FROM 表名 [LIKE 'PATTERN']
SHOW TABLE STATUS [FROM db_name] [LIKE 'pattern']

-- 修改表
-- 修改表本身的选项
ALTER TABLE 表名 表的选项
-- eg: ALTER TABLE 表名 ENGINE=MYISAM;
-- 对表进行重命名
RENAME TABLE 原表名 TO 新表名
RENAME TABLE 原表名 TO 库名.表名 (可将表移动到另一个数据库)
-- RENAME 可以交换两个表名
-- 修改表的字段机构 (13.1.2. ALTER TABLE 语法)
   ALTER TABLE 表名 操作名
   -- 操作名
      ADD [COLUMN] 字段定义 -- 增加字段
        AFTER 字段名 -- 表示增加在该字段名后面
        FIRST -- 表示增加在第一个
        ADD PRIMARY KEY(字段名) -- 创建主键
        ADD UNIQUE [索引名] (字段名)-- 创建唯一索引
        ADD INDEX [索引名] (字段名) -- 创建普通索引
        DROP [COLUMN] 字段名 -- 删除字段
        MODIFY [COLUMN] 字段名 字段属性 -- 支持对字段属性进行修改, 不能修改字段名(所有原有属性也需写上)
        CHANGE [COLUMN] 原字段名 新字段名 字段属性 -- 支持对字段名修改
        DROP PRIMARY KEY -- 删除主键(删除主键前需删除其 AUTO_INCREMENT 属性)
        DROP INDEX 索引名 -- 删除索引
        DROP FOREIGN KEY 外键 -- 删除外键
-- 删除表
DROP TABLE [IF EXISTS] 表名 ...

-- 清空表数据
TRUNCATE [TABLE] 表名

-- 复制表结构
CREATE TABLE 表名 LIKE 要复制的表名

-- 复制表结构和数据
CREATE TABLE 表名 [AS] SELECT * FROM 要复制的表名

-- 检查表是否有错误
CHECK TABLE tbl_name [, tbl_name] ... [option] ...

-- 优化表
OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...

-- 修复表
REPAIR [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ... [QUICK] [EXTENDED] [USE_FRM]

-- 分析表
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
```

#### 数据操作

```sql
-- 增
INSERT [INTO] 表名 [(字段列表)] VALUES (值列表)[, (值列表), ...]
  -- 如果要插入的值列表包含所有字段并且顺序一致, 则可以省略字段列表.
  -- 可同时插入多条数据记录！
  -- REPLACE 与 INSERT 完全一样, 可互换.
  INSERT [INTO] 表名 SET 字段名=值 [, 字段名=值, ...]

-- 删
DELETE FROM 表名 [删除条件子句]
  -- 没有条件子句, 则会删除全部

-- 改
UPDATE 表名 SET 字段名=新值 [, 字段名=新值] [更新条件]

-- 查
SELECT 字段列表 FROM 表名 [其他子句]
  -- 可来自多个表的多个字段
  -- 其他子句可以不使用
  -- 字段列表可以用*代替, 表示所有字段
```

#### 字符集编码

```sql
-- MySQL, 数据库, 表, 字段均可设置编码
-- 数据编码与客户端编码不需一致
SHOW VARIABLES LIKE 'character_set_%' -- 查看所有字符集编码项
  -- character_set_client 客户端向服务器发送数据时使用的编码
  -- character_set_results 服务器端将结果返回给客户端所使用的编码
  -- character_set_connection 连接层编码
SET 变量名 = 变量值
  SET character_set_client = gbk;
  SET character_set_results = gbk;
  SET character_set_connection = gbk;
SET NAMES GBK; -- 相当于完成以上三个设置

-- 校对集
  -- 校对集用以排序
  SHOW CHARACTER SET [LIKE 'pattern']/SHOW CHARSET [LIKE 'pattern'] -- 查看所有字符集
  SHOW COLLATION [LIKE 'pattern'] -- 查看所有校对集
  CHARSET 字符集编码 -- 设置字符集编码
  COLLATE 校对集编码 -- 设置校对集编码
```

#### 数据类型 (列类型)

##### 整型

| 类型      | 字节   | 范围 (有符号位)              |
| --------- | ------ | ---------------------------- |
| tinyint   | 1 字节 | -128 ~ 127 无符号位: 0 ~ 255 |
| smallint  | 2 字节 | -32768 ~ 32767               |
| mediumint | 3 字节 | -8388608 ~ 8388607           |
| int       | 4 字节 |                              |
| bigint    | 8 字节 |                              |

* int(M) M 表示总位数
* 默认存在符号位, unsigned 属性修改
* 显示宽度, 如果某个数不够定义字段时设置的位数, 则前面以 0 补填, zerofill 属性修改, 例: int(5) 插入一个数'123', 补填后为'00123'
* 在满足要求的情况下, 越小越好.
* 1 表示 bool 值真, 0 表示 bool 值假. MySQL 没有布尔类型, 通过整型 0 和 1 表示. 常用 tinyint(1)表示布尔型.

##### 浮点型

| 类型           | 字节   | 范围 |
| -------------- | ------ | ---- |
| float(单精度)  | 4 字节 |      |
| double(双精度) | 8 字节 |      |

* 浮点型既支持符号位 unsigned 属性, 也支持显示宽度 zerofill 属性.
  * 不同于整型, 前后均会补填 0.
* 定义浮点型时, 需指定总位数和小数位数.
  * float(M, D) double(M, D)
  * M 表示总位数, D 表示小数位数.
  * M 和 D 的大小会决定浮点数的范围. 不同于整型的固定范围.
  * M 既表示总位数 (不包括小数点和正负号) , 也表示显示宽度 (所有显示符号均包括) .
  * 支持科学计数法表示.
  * 浮点数表示近似值.

##### 定点数

* decimal -- 可变长度
* decimal(M, D) M 也表示总位数, D 表示小数位数.
* 保存一个精确的数值, 不会发生数据的改变, 不同于浮点数的四舍五入.
* 将浮点数转换为字符串来保存, 每 9 位数字保存为 4 个字节.

##### 字符串

| 类型      | 描述                           |
| --------- | ------------------------------ |
| char      | 定长字符串, 速度快, 但浪费空间 |
| varchar   | 变长字符串, 速度慢, 但节省空间 |
| binary    | 定长二进制字符串               |
| varbinary | 变长二进制字符串               |
| blob      | 二进制字符串 (字节字符串)      |
| text      | 非二进制字符串 (字符字符串)    |

* char: 最多 255 个字符, 与编码无关.
* varchar: 最多 65535 字符, 与编码有关.
  * 一条有效记录最大不能超过 65535 个字节.
  * utf8 最大为 21844 个字符, gbk 最大为 32766 个字符, latin1 最大为 65532 个字符
  * varchar 是变长的, 需要利用存储空间保存 varchar 的长度, 如果数据小于 255 个字节, 则采用一个字节来保存长度, 反之需要两个字节来保存.
  * varchar 的最大有效长度由最大行大小和使用的字符集确定.
  * 最大有效长度是 65532 字节, 因为在 varchar 存字符串时, 第一个字节是空的, 不存在任何数据, 然后还需两个字节来存放字符串的长度, 所以有效长度是 64432-1-2=65532 字节.
  * 例: 若一个表定义为 `CREATE TABLE tb(c1 int, c2 char(30), c3 varchar(N)) charset=utf8;` 问 N 的最大值是多少? 答: (65535-1-2-4-30*3)/3
* binary: 类似于 char, 用于保存二进制字符串
* varbinary: 类似于 varchar, 用于保存二进制字符串
* blob: tinyblob, blob, mediumblob, longblob
* text: tinytext, text, mediumtext, longtext
  * text 在定义时, 不需要定义长度, 也不会计算总长度.
  * text 类型在定义时, 不可给 default 值
* char, varchar, text 对应 binary, varbinary, blob

##### 日期时间类型

一般用整型保存时间戳, 方便多语言多系统使用.

| 类型      | 字节   | 表示       | 范围                                       |
| --------- | ------ | ---------- | ------------------------------------------ |
| datetime  | 8 字节 | 日期及时间 | 1000-01-01 00:00:00 到 9999-12-31 23:59:59 |
| date      | 3 字节 | 日期       | 1000-01-01 到 9999-12-31                   |
| timestamp | 4 字节 | 时间戳     | 19700101000000 到 2038-01-19 03:14:07      |
| time      | 3 字节 | 时间       | -838:59:59 到 838:59:59                    |
| year      | 1 字节 | 年份       | 1901 - 2155                                |

* datetime
  * YYYY-MM-DD hh:mm:ss
* timestamp
  * YY-MM-DD hh:mm:ss
  * YYYYMMDDhhmmss
  * YYMMDDhhmmss
  * YYYYMMDDhhmmss
  * YYMMDDhhmmss
* date
  * YYYY-MM-DD
  * YY-MM-DD
  * YYYYMMDD
  * YYMMDD
  * YYYYMMDD
  * YYMMDD
* time
  * hh:mm:ss
  * hhmmss
  * hhmmss
* year
  * YYYY
  * YY
  * YYYY
  * YY

##### 枚举和集合

* 枚举: `enum(val1, val2, val3...)`
  * 在已知的值中进行单选. 最大数量为 65535.
  * 枚举值在保存时, 以 2 个字节的整型(smallint)保存. 每个枚举值, 按保存的位置顺序, 从 1 开始逐一递增.
  * 表现为字符串类型, 存储却是整型.
  * NULL 值的索引是 NULL.
  * 空字符串错误值的索引值是 0.
* 集合: `set(val1, val2, val3...)`
  * `create table tab ( gender set('男', '女', '无') );`
  * `insert into tab values ('男, 女');`
  * 最多可以有 64 个不同的成员. 以 bigint 存储, 共 8 个字节. 采取位运算的形式.
  * 当创建表时, SET 成员值的尾部空格将自动被删除.

#### 列属性 (列约束)

1. PRIMARY 主键
   * 能唯一标识记录的字段, 可以作为主键.
   * 一个表只能有一个主键.
   * 主键具有唯一性.
   * 声明字段时, 用 primary key 标识.
   * 也可以在字段列表之后声明, 例: `create table tab ( id int, stu varchar(10), primary key (id));`
   * 主键字段的值不能为 null.
   * 主键可以由多个字段共同组成. 此时需要在字段列表后声明的方法. 例: `create table tab ( id int, stu varchar(10), age int, primary key (stu, age));`
2. UNIQUE 唯一索引 (唯一约束)
   * 使得某字段的值也不能重复.
3. NULL 约束
   * null 不是数据类型, 是列的一个属性.
   * 表示当前列是否可以为 null, 表示什么都没有.
   * null, 允许为空. 默认.
   * not null, 不允许为空.
   * `insert into tab values (null, 'val'); -- 此时表示将第一个字段的值设为 null, 取决于该字段是否允许为 null`
4. DEFAULT 默认值属性
   * 当前字段的默认值.
   * `insert into tab values (default, 'val'); -- 此时表示强制使用默认值. `
   * `create table tab ( add_time timestamp default current_timestamp ); -- 表示将当前时间的时间戳设为默认值. current_date, current_time`
5. AUTO_INCREMENT 自动增长约束
   * 自动增长必须为索引 (主键或 unique)
   * 只能存在一个字段为自动增长.
   * 默认为 1 开始自动增长. 可以通过表属性 `auto_increment = x`进行设置, 或 `alter table tbl auto_increment = x;`
6. COMMENT 注释
   * 例: `create table tab ( id int ) comment '注释内容';`
7. FOREIGN KEY 外键约束
* 用于限制主表与从表数据完整性.
* `alter table t1 add constraint t1_t2_fk foreign key (t1_id) references t2(id);`
  * -- 将表 t1 的 t1_id 外键关联到表 t2 的 id 字段.
  * -- 每个外键都有一个名字, 可以通过 constraint 指定
* 存在外键的表, 称之为从表 (子表) , 外键指向的表, 称之为主表 (父表) .
* 作用: 保持数据一致性, 完整性, 主要目的是控制存储在外键表 (从表) 中的数据.
* MySQL 中, 可以对 InnoDB 引擎使用外键约束:
  * foreign key (外键字段) references 主表名 (关联字段) [主表记录删除时的动作] [主表记录更新时的动作]
  * 此时需要检测一个从表的外键需要约束为主表的已存在的值. 外键在没有关联的情况下, 可以设置为 null.前提是该外键列, 没有 not null.
  * 可以不指定主表记录更改或更新时的动作, 那么此时主表的操作被拒绝.
  * 如果指定了 on update 或 on delete: 在删除或更新时, 有如下几个操作可以选择:
    1. cascade, 级联操作. 主表数据被更新 (主键值更新) , 从表也被更新 (外键值更新) . 主表记录被删除, 从表相关记录也被删除.
    2. set null, 设置为 null. 主表数据被更新 (主键值更新) , 从表的外键被设置为 null. 主表记录被删除, 从表相关记录外键被设置成 null. 但注意, 要求该外键列, 没有 not null 属性约束.
    3. restrict, 拒绝父表删除和更新.
* 注意, 外键只被 InnoDB 存储引擎所支持. 其他引擎是不支持的.

#### SELECT 查询

```sql
SELECT [ALL|DISTINCT] select_expr FROM -> WHERE -> GROUP BY [合计函数] -> HAVING -> ORDER BY -> LIMIT
-- a. select_expr
  -- 可以用 * 表示所有字段.
  select * from tb;
  -- 可以使用表达式 (计算公式, 函数调用, 字段也是个表达式)
  select stu, 29+25, now() from tb;
  -- 可以为每个列使用别名. 适用于简化列标识, 避免多个列标识符重复.
  -- 使用 as 关键字, 也可省略 as.
  select stu+10 as add10 from tb;

-- b. FROM 子句
  -- 用于标识查询来源.
  -- 可以为表起别名. 使用 as 关键字.
  SELECT * FROM tb1 AS tt, tb2 AS bb;
  -- from 子句后, 可以同时出现多个表.
  -- 多个表会横向叠加到一起, 而数据会形成一个笛卡尔积.
  SELECT * FROM tb1, tb2;
  -- 向优化符提示如何选择索引
  USE INDEX, IGNORE INDEX, FORCE INDEX
  SELECT * FROM table1 USE INDEX (key1,key2) WHERE key1=1 AND key2=2 AND key3=3;
  SELECT * FROM table1 IGNORE INDEX (key3) WHERE key1=1 AND key2=2 AND key3=3;

-- c. WHERE 子句
  -- 从 from 获得的数据源中进行筛选.
  -- 整型 1 表示真, 0 表示假.
  -- 表达式由运算符和运算数组成.
  -- 运算数: 变量 (字段) , 值, 函数返回值
  -- 运算符:
  =, <=>, <>, !=, <=, <, >=, >, !, &&, ||,
  in (not) null, (not) like, (not) in, (not) between and, is (not), and, or, not, xor
  -- is/is not 加上 ture/false/unknown, 检验某个值的真假
  -- <=>与<>功能相同, <=>可用于 null 比较

-- d. GROUP BY 子句, 分组子句
  GROUP BY 字段/别名 [排序方式]
  -- 分组后会进行排序. 升序: ASC, 降序: DESC
  -- 以下 [合计函数] 需配合 GROUP BY 使用:
  -- count 返回不同的非 NULL 值数目 count(*), count(字段)
  -- sum 求和
  -- max 求最大值
  -- min 求最小值
  -- avg 求平均值
  -- group_concat 返回带有来自一个组的连接的非 NULL 值的字符串结果. 组内字符串连接.

-- e. HAVING 子句, 条件子句
  -- 与 where 功能, 用法相同, 执行时机不同.
  -- where 在开始时执行检测数据, 对原数据进行过滤.
  -- having 对筛选出的结果再次进行过滤.
  -- having 字段必须是查询出来的, where 字段必须是数据表存在的.
  -- where 不可以使用字段的别名, having 可以. 因为执行 WHERE 代码时, 可能尚未确定列值.
  -- where 不可以使用合计函数. 一般需用合计函数才会用 having
  -- SQL 标准要求 HAVING 必须引用 GROUP BY 子句中的列或用于合计函数中的列.

-- f. ORDER BY 子句, 排序子句
  order by 排序字段/别名 排序方式 [,排序字段/别名 排序方式]...
  -- 升序: ASC, 降序: DESC
  -- 支持多个字段的排序.

-- g. LIMIT 子句, 限制结果数量子句
  -- 仅对处理好的结果进行数量限制. 将处理好的结果的看作是一个集合, 按照记录出现的顺序, 索引从 0 开始.
  limit 起始位置, 获取条数
  -- 省略第一个参数, 表示从索引 0 开始. limit 获取条数

-- h. DISTINCT, ALL 选项
  distinct 去除重复记录
  -- 默认为 all, 全部记录
```

#### UNION

```sql
-- 将多个 select 查询的结果组合成一个结果集合.
SELECT ... UNION [ALL|DISTINCT] SELECT ...
-- 默认 DISTINCT 方式, 即所有返回的行都是唯一的
-- 建议, 对每个 SELECT 查询加上小括号包裹.
-- ORDER BY 排序时, 需加上 LIMIT 进行结合.
-- 需要各 select 查询的字段数量一样.
-- 每个 select 查询的字段列表(数量, 类型)应一致, 因为结果中的字段名以第一条 select 语句为准.
```

#### 子查询

```sql
-- 子查询需用括号包裹.
-- from 型
  -- from 后要求是一个表, 必须给子查询结果取个别名.
  -- 简化每个查询内的条件.
  -- from 型需将结果生成一个临时表格, 可用以原表的锁定的释放.
  -- 子查询返回一个表, 表型子查询.
  select * from (select * from tb where id>0) as subfrom where id>1;

-- where 型
  -- 子查询返回一个值, 标量子查询.
  -- 不需要给子查询取别名.
  -- where 子查询内的表, 不能直接用以更新.
  select * from tb where money = (select max(money) from tb);

  -- 列子查询
    -- 如果子查询结果返回的是一列.
    -- 使用 in 或 not in 完成查询
    -- exists 和 not exists 条件
    -- 如果子查询返回数据, 则返回 1 或 0. 常用于判断条件.
    select column1 from t1 where exists (select * from t2);

  -- 行子查询
    -- 查询条件是一个行.
    select * from t1 where (id, gender) in (select id, gender from t2);
    -- 行构造符: (col1, col2, ...) 或 ROW(col1, col2, ...)
    -- 行构造符通常用于与对能返回两个或两个以上列的子查询进行比较.

  -- 特殊运算符
  != all() -- 相当于 not in
  = some() -- 相当于 in. any 是 some 的别名
  != some() -- 不等同于 not in, 不等于其中某一个.
  -- all, some 可以配合其他运算符一起使用.
```

#### 连接查询(join)

```sql
-- 将多个表的字段进行连接, 可以指定连接条件.
-- 内连接(inner join)
  -- 默认就是内连接, 可省略 inner.
  -- 只有数据存在时才能发送连接. 即连接结果不能出现空行.
  -- on 表示连接条件. 其条件表达式与 where 类似. 也可以省略条件 (表示条件永远为真)
  -- 也可用 where 表示连接条件.
  -- 还有 using, 但需字段名相同. using(字段名)

-- 交叉连接 cross join 即, 没有条件的内连接.
  select * from tb1 cross join tb2;

-- 外连接(outer join)
  -- 如果数据不存在, 也会出现在连接结果中.
  -- 左外连接 left join
    -- 如果数据不存在, 左表记录会出现, 而右表为 null 填充
  -- 右外连接 right join
    -- 如果数据不存在, 右表记录会出现, 而左表为 null 填充

-- 自然连接(natural join)
  -- 自动判断连接条件完成连接.
  -- 相当于省略了 using, 会自动查找相同字段名.
  natural join
  natural left join
  natural right join

select info.id, info.name, info.stu_num, extra_info.hobby, extra_info.sex from info, extra_info where info.stu_num = extra_info.stu_id;
```

#### 导出

```sql
select * into outfile 文件地址 [控制格式] from 表名; -- 导出表数据

load data [local] infile 文件地址 [replace|ignore] into table 表名 [控制格式]; -- 导入数据
  -- 生成的数据默认的分隔符是制表符
  -- local 未指定, 则数据文件必须在服务器上
  -- replace 和 ignore 关键词控制对现有的唯一键记录的重复的处理

-- 控制格式
-- fields 控制字段格式
-- 默认: fields terminated by '\t' enclosed by '' escaped by '\\'
  terminated by 'string' -- 终止
  enclosed by 'char' -- 包裹
  escaped by 'char' -- 转义

  -- 示例:
  SELECT a,b,a+b INTO OUTFILE '/tmp/result.text'
  FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
  LINES TERMINATED BY '\n'
  FROM test_table;
-- lines 控制行格式
-- 默认: lines terminated by '\n'
  terminated by 'string' -- 终止
```

#### INSERT

```sql
-- select 语句获得的数据可以用 insert 插入.
-- 可以省略对列的指定, 要求 values () 括号内, 提供给了按照列顺序出现的所有字段的值. 或者使用 set 语法.
INSERT INTO tbl_name SET field=value,...；

-- 可以一次性使用多个值, 采用(), (), ();的形式.
INSERT INTO tbl_name VALUES (), (), ();

-- 可以在列值指定时, 使用表达式.
INSERT INTO tbl_name VALUES (field_value, 10+10, now());

-- 可以使用一个特殊值 DEFAULT, 表示该列使用默认值.
INSERT INTO tbl_name VALUES (field_value, DEFAULT);

-- 可以通过一个查询的结果, 作为需要插入的值.
INSERT INTO tbl_name SELECT ...;

-- 可以指定在插入的值出现主键 (或唯一索引) 冲突时, 更新其他非主键列的信息.
INSERT INTO tbl_name VALUES/SET/SELECT ON DUPLICATE KEY UPDATE 字段=值, …;
```

#### DELETE

```sql
DELETE FROM tbl_name [WHERE where_definition] [ORDER BY ...] [LIMIT row_count]
-- 按照条件删除. where
-- 指定删除的最多记录数. limit
-- 可以通过排序条件删除. order by + limit
-- 支持多表删除, 使用类似连接语法.
-- delete from 需要删除数据多表 1, 表 2 using 表连接操作 条件.
```

#### TRUNCATE

```sql
TRUNCATE [TABLE] tbl_name
-- 清空数据
-- 删除重建表
-- 区别:
-- 1, truncate 是删除表再创建, delete 是逐条删除
-- 2, truncate 重置 auto_increment 的值. 而 delete 不会
-- 3, truncate 不知道删除了几条, 而 delete 知道.
-- 4, 当被用于带分区的表时, truncate 会保留分区

```

#### 备份与还原

```sql
-- 备份, 将数据的结构与表内数据保存起来.
-- 利用 mysqldump 指令完成.

-- 导出
mysqldump [options] db_name [tables]
mysqldump [options] ---database DB1 [DB2 DB3...]
mysqldump [options] --all--database
-- 1. 导出一张表
　　mysqldump -u 用户名 -p 密码 库名 表名 > 文件名(D:/a.sql)
-- 2. 导出多张表
　　mysqldump -u 用户名 -p 密码 库名 表 1 表 2 表 3 > 文件名(D:/a.sql)
-- 3. 导出所有表
　　mysqldump -u 用户名 -p 密码 库名 > 文件名(D:/a.sql)
-- 4. 导出一个库
　　mysqldump -u 用户名 -p 密码 --lock-all-tables --database 库名 > 文件名(D:/a.sql)
-- 可以-w 携带 WHERE 条件

-- 导入
-- 1. 在登录 mysql 的情况下:
　　source 备份文件
-- 2. 在不登录的情况下
　　mysql -u 用户名 -p 密码 库名 < 备份文件
```

#### 视图

```sql
-- 什么是视图:
-- 视图是一个虚拟表, 其内容由查询定义. 同真实的表一样, 视图包含一系列带有名称的列和行数据. 但是, 视图并不在数据库中以存储的数据值集形式存在. 行和列数据来自由定义视图的查询所引用的表, 并且在引用视图时动态生成.
-- 视图具有表结构文件, 但不存在数据文件.
-- 对其中所引用的基础表来说, 视图的作用类似于筛选. 定义视图的筛选可以来自当前或其它数据库的一个或多个表, 或者其它视图. 通过视图进行查询没有任何限制, 通过它们进行数据修改时的限制也很少.
-- 视图是存储在数据库中的查询的 sql 语句, 它主要出于两种原因: 安全原因, 视图可以隐藏一些数据, 如: 社会保险基金表, 可以用视图只显示姓名, 地址, 而不显示社会保险号和工资数等, 另一原因是可使复杂的查询易于理解和使用.

-- 创建视图
CREATE [OR REPLACE] [ALGORITHM = {UNDEFINED | MERGE | TEMPTABLE}] VIEW view_name [(column_list)] AS select_statement
  -- 视图名必须唯一, 同时不能与表重名.
  -- 视图可以使用 select 语句查询到的列名, 也可以自己指定相应的列名.
  -- 可以指定视图执行的算法, 通过 ALGORITHM 指定.
  -- column_list 如果存在, 则数目必须等于 SELECT 语句检索的列数

-- 查看结构
SHOW CREATE VIEW view_name

-- 删除视图
-- 删除视图后, 数据依然存在.
-- 可同时删除多个视图.
DROP VIEW [IF EXISTS] view_name ...

-- 修改视图结构
-- 一般不修改视图, 因为不是所有的更新视图都会映射到表上.
ALTER VIEW view_name [(column_list)] AS select_statement

-- 视图作用
-- 1. 简化业务逻辑
-- 2. 对客户端隐藏真实的表结构

-- 视图算法(ALGORITHM)
  -- MERGE 合并
  -- 将视图的查询语句, 与外部查询需要先合并再执行！
  -- TEMPTABLE 临时表
  -- 将视图执行完毕后, 形成临时表, 再做外层查询！
  -- UNDEFINED 未定义(默认), 指的是 MySQL 自主去选择相应的算法.
```

#### 事务(transaction)

```sql
-- 事务是指逻辑上的一组操作, 组成这组操作的各个单元, 要不全成功要不全失败.
-- 支持连续 SQL 的集体成功或集体撤销.
-- 事务是数据库在数据晚自习方面的一个功能.
-- 需要利用 InnoDB 或 BDB 存储引擎, 对自动提交的特性支持完成.
-- InnoDB 被称为事务安全型引擎.

-- 事务开启
  START TRANSACTION; 或者 BEGIN;
  -- 开启事务后, 所有被执行的 SQL 语句均被认作当前事务内的 SQL 语句.

-- 事务提交
  COMMIT;

-- 事务回滚
  ROLLBACK;
  -- 如果部分操作发生问题, 映射到事务开启前.

-- 事务的特性
  -- 1. 原子性 (Atomicity)
    -- 事务是一个不可分割的工作单位, 事务中的操作要么都发生, 要么都不发生.
  -- 2. 一致性 (Consistency)
    -- 事务前后数据的完整性必须保持一致.
    -- 事务开始和结束时, 外部数据一致
    -- 在整个事务过程中, 操作是连续的
  -- 3. 隔离性 (Isolation)
    -- 多个用户并发访问数据库时, 一个用户的事务不能被其它用户的事物所干扰, 多个并发事务之间的数据要相互隔离.
  -- 4. 持久性 (Durability)
    -- 一个事务一旦被提交, 它对数据库中的数据改变就是永久性的.

-- 事务的实现
  -- 1. 要求是事务支持的表类型
  -- 2. 执行一组相关的操作前开启事务
  -- 3. 整组操作完成后, 都成功, 则提交；如果存在失败, 选择回滚, 则会回到事务开始的备份点.

-- 事务的原理
  -- 利用 InnoDB 的自动提交(autocommit)特性完成.
  -- 普通的 MySQL 执行语句后, 当前的数据提交操作均可被其他客户端可见.
  -- 而事务是暂时关闭“自动提交”机制, 需要 commit 提交持久化数据操作.
-- 注意
  -- 1. 数据定义语言 (DDL) 语句不能被回滚, 比如创建或取消数据库的语句, 和创建, 取消或更改表或存储的子程序的语句.
  -- 2. 事务不能被嵌套

-- 保存点
  SAVEPOINT 保存点名称 -- 设置一个事务保存点
  ROLLBACK TO SAVEPOINT 保存点名称 -- 回滚到保存点
  RELEASE SAVEPOINT 保存点名称 -- 删除保存点

-- InnoDB 自动提交特性设置
  SET autocommit = 0|1; 0 表示关闭自动提交, 1 表示开启自动提交.
  -- 如果关闭了, 那普通操作的结果对其他客户端也不可见, 需要 commit 提交后才能持久化数据操作.
  -- 也可以关闭自动提交来开启事务. 但与 START TRANSACTION 不同的是,
    -- SET autocommit 是永久改变服务器的设置, 直到下次再次修改该设置. (针对当前连接)
    -- 而 START TRANSACTION 记录开启前的状态, 而一旦事务提交或回滚后就需要再次开启事务. (针对当前事务)
```

#### 锁表

```sql
-- 表锁定只用于防止其它客户端进行不正当地读取和写入
-- MyISAM 支持表锁, InnoDB 支持行锁
-- 锁定
  LOCK TABLES tbl_name [AS alias]
-- 解锁
  UNLOCK TABLES
```

#### 触发器

```sql
-- 触发程序是与表有关的命名数据库对象, 当该表出现特定事件时, 将激活该对象.
-- 监听: 记录的增加, 修改, 删除.

-- 创建触发器
CREATE TRIGGER trigger_name trigger_time trigger_event ON tbl_name FOR EACH ROW trigger_stmt
  -- 参数:
  -- trigger_time 是触发程序的动作时间. 它可以是 before 或 after, 以指明触发程序是在激活它的语句之前或之后触发.
  -- trigger_event 指明了激活触发程序的语句的类型
    -- INSERT: 将新行插入表时激活触发程序
    -- UPDATE: 更改某一行时激活触发程序
    -- DELETE: 从表中删除某一行时激活触发程序
  -- tbl_name: 监听的表, 必须是永久性的表, 不能将触发程序与 TEMPORARY 表或视图关联起来.
  -- trigger_stmt: 当触发程序激活时执行的语句. 执行多个语句, 可使用 BEGIN...END 复合语句结构

-- 删除
DROP TRIGGER [schema_name.] trigger_name
-- 可以使用 old 和 new 代替旧的和新的数据
  -- 更新操作, 更新前是 old, 更新后是 new.
  -- 删除操作, 只有 old.
  -- 增加操作, 只有 new.
-- 注意
  -- 1. 对于具有相同触发程序动作时间和事件的给定表, 不能有两个触发程序.
-- 字符连接函数
concat(str1,str2,...])
concat_ws(separator,str1,str2,...)

-- 分支语句
if 条件 then
  -- 执行语句
elseif 条件 then
  -- 执行语句
else
  -- 执行语句
end if;

-- 修改最外层语句结束符
delimiter 自定义结束符号
  -- SQL 语句
-- 自定义结束符号
delimiter ; -- 修改回原来的分号

-- 语句块包裹
begin
  -- 语句块
end

-- 特殊的执行
-- 1. 只要添加记录, 就会触发程序.
-- 2. Insert into on duplicate key update 语法会触发:
  -- 如果没有重复记录, 会触发 before insert, after insert;
  -- 如果有重复记录并更新, 会触发 before insert, before update, after update;
  -- 如果有重复记录但是没有发生更新, 则触发 before insert, before update
-- 3. Replace 语法 如果有记录, 则执行 before insert, before delete, after delete, after insert
```

#### SQL 编程

```sql
--// 局部变量 ----------
-- 变量声明
  declare var_name [,...] type [default value]
  -- 这个语句被用来声明局部变量. 要给变量提供一个默认值, 请包含一个 default 子句. 值可以被指定为一个表达式, 不需要为一个常数. 如果没有 default 子句, 初始值为 null.
-- 赋值
  -- 使用 set 和 select into 语句为变量赋值.
  -- 注意: 在函数内是可以使用全局变量 (用户自定义的变量)


--// 全局变量 ----------
-- 定义, 赋值
-- set 语句可以定义并为变量赋值.
set @var = value;
-- 也可以使用 select into 语句为变量初始化并赋值. 这样要求 select 语句只能返回一行, 但是可以是多个字段, 就意味着同时为多个变量进行赋值, 变量的数量需要与查询的列数一致.
-- 还可以把赋值语句看作一个表达式, 通过 select 执行完成. 此时为了避免=被当作关系运算符看待, 使用:=代替. (set 语句可以使用= 和 :=) .
select @var:=20;
select @v1:=id, @v2=name from t1 limit 1;
select * from tbl_name where @var:=30;
-- select into 可以将表中查询获得的数据赋给变量.
select max(height) into @max_height from tb;

-- 自定义变量名
-- 为了避免 select 语句中, 用户自定义的变量与系统标识符 (通常是字段名) 冲突, 用户自定义变量在变量名前使用@作为开始符号.
@var=10;
  -- 变量被定义后, 在整个会话周期都有效 (登录到退出)


--// 控制结构 ----------
-- if 语句
if search_condition then
  statement_list
[elseif search_condition then
  statement_list]
...
[else
  statement_list]
end if;
-- case 语句
CASE value WHEN [compare-value] THEN result
[WHEN [compare-value] THEN result ...]
[ELSE result]
END
-- while 循环
[begin_label:] while search_condition do
  statement_list
end while [end_label];
-- 如果需要在循环内提前终止 while 循环, 则需要使用标签；标签需要成对出现.
  -- 退出循环
    -- 退出整个循环 leave
    -- 退出当前循环 iterate
    -- 通过退出的标签决定退出哪个循环


--// 内置函数 ----------
-- 数值函数
abs(x) -- 绝对值 abs(-10.9) = 10
format(x, d) -- 格式化千分位数值 format(1234567.456, 2) = 1,234,567.46
ceil(x) -- 向上取整 ceil(10.1) = 11
floor(x) -- 向下取整 floor (10.1) = 10
round(x) -- 四舍五入去整
mod(m, n) -- m%n m mod n 求余 10%3=1
pi() -- 获得圆周率
pow(m, n) -- m^n
sqrt(x) -- 算术平方根
rand() -- 随机数
truncate(x, d) -- 截取 d 位小数

-- 时间日期函数
now(), current_timestamp(); -- 当前日期时间
current_date(); -- 当前日期
current_time(); -- 当前时间
date('yyyy-mm-dd hh:ii:ss'); -- 获取日期部分
time('yyyy-mm-dd hh:ii:ss'); -- 获取时间部分
date_format('yyyy-mm-dd hh:ii:ss', '%d %y %a %d %m %b %j'); -- 格式化时间
unix_timestamp(); -- 获得 unix 时间戳
from_unixtime(); -- 从时间戳获得时间

-- 字符串函数
length(string) -- string 长度, 字节
char_length(string) -- string 的字符个数
substring(str, position [,length]) -- 从 str 的 position 开始,取 length 个字符
replace(str ,search_str ,replace_str) -- 在 str 中用 replace_str 替换 search_str
instr(string ,substring) -- 返回 substring 首次在 string 中出现的位置
concat(string [,...]) -- 连接字串
charset(str) -- 返回字串字符集
lcase(string) -- 转换成小写
left(string, length) -- 从 string2 中的左边起取 length 个字符
load_file(file_name) -- 从文件读取内容
locate(substring, string [,start_position]) -- 同 instr,但可指定开始位置
lpad(string, length, pad) -- 重复用 pad 加在 string 开头,直到字串长度为 length
ltrim(string) -- 去除前端空格
repeat(string, count) -- 重复 count 次
rpad(string, length, pad) --在 str 后用 pad 补充,直到长度为 length
rtrim(string) -- 去除后端空格
strcmp(string1 ,string2) -- 逐字符比较两字串大小

-- 流程函数
case when [condition] then result [when [condition] then result ...] [else result] end 多分支
if(expr1,expr2,expr3) 双分支.

-- 聚合函数
count()
sum();
max();
min();
avg();
group_concat()

-- 其他常用函数
md5();
default();

--// 存储函数, 自定义函数 ----------
-- 新建
  CREATE FUNCTION function_name (参数列表) RETURNS 返回值类型
    -- 函数体
    -- 函数名, 应该合法的标识符, 并且不应该与已有的关键字冲突.
    -- 一个函数应该属于某个数据库, 可以使用 db_name.funciton_name 的形式执行当前函数所属数据库, 否则为当前数据库.
    -- 参数部分, 由"参数名"和"参数类型"组成. 多个参数用逗号隔开.
    -- 函数体由多条可用的 mysql 语句, 流程控制, 变量声明等语句构成.
    -- 多条语句应该使用 begin...end 语句块包含.
    -- 一定要有 return 返回值语句.

-- 删除
DROP FUNCTION [IF EXISTS] function_name;

-- 查看
SHOW FUNCTION STATUS LIKE 'partten'
SHOW CREATE FUNCTION function_name;

-- 修改
ALTER FUNCTION function_name 函数选项

--// 存储过程, 自定义功能 ----------
-- 定义
-- 存储存储过程 是一段代码 (过程) , 存储在数据库中的 sql 组成.
-- 一个存储过程通常用于完成一段业务逻辑, 例如报名, 交班费, 订单入库等.
-- 而一个函数通常专注与某个功能, 视为其他程序服务的, 需要在其他语句中调用函数才可以, 而存储过程不能被其他调用, 是自己执行 通过 call 执行.

-- 创建
CREATE PROCEDURE sp_name (参数列表)
  -- 过程体
  -- 参数列表: 不同于函数的参数列表, 需要指明参数类型
  -- IN, 表示输入型
  -- OUT, 表示输出型
  -- INOUT, 表示混合型
  -- 注意, 没有返回值.


/* 存储过程 */ ------------------
-- 存储过程是一段可执行性代码的集合. 相比函数, 更偏向于业务逻辑.
-- 调用:
CALL 过程名
-- 注意
  -- 没有返回值.
  -- 只能单独调用, 不可夹杂在其他语句中

-- 参数
IN|OUT|INOUT 参数名 数据类型
IN 输入: 在调用过程中, 将数据输入到过程体内部的参数
OUT 输出: 在调用过程中, 将过程体处理完的结果返回到客户端
INOUT 输入输出: 既可输入, 也可输出

-- 语法
CREATE PROCEDURE 过程名 (参数列表)
BEGIN
  -- 过程体
END
```

#### 用户和权限管理

```sql
-- root 密码重置
-- 1. 停止 MySQL 服务
-- 2. [Linux] /usr/local/mysql/bin/safe_mysqld --skip-grant-tables &
-- [Windows] mysqld --skip-grant-tables
-- 3. use mysql;
-- 4. UPDATE `user` SET PASSWORD=PASSWORD("密码") WHERE `user` = "root";
-- 5. FLUSH PRIVILEGES;
-- 用户信息表: mysql.user

-- 刷新权限
FLUSH PRIVILEGES;

-- 增加用户
CREATE USER 用户名 IDENTIFIED BY [PASSWORD] 密码(字符串)
  -- 必须拥有 mysql 数据库的全局 CREATE USER 权限, 或拥有 INSERT 权限.
  -- 只能创建用户, 不能赋予权限.
  -- 用户名, 注意引号: 如 'user_name'@'192.168.1.1'
  -- 密码也需引号, 纯数字密码也要加引号
  -- 要在纯文本中指定密码, 需忽略 PASSWORD 关键词. 要把密码指定为由 PASSWORD()函数返回的混编值, 需包含关键字 PASSWORD

-- 重命名用户
RENAME USER old_user TO new_user

-- 设置密码
SET PASSWORD = PASSWORD('密码') -- 为当前用户设置密码
SET PASSWORD FOR 用户名 = PASSWORD('密码') -- 为指定用户设置密码

-- 删除用户
DROP USER 用户名

-- 分配权限/添加用户
GRANT 权限列表 ON 表名 TO 用户名 [IDENTIFIED BY [PASSWORD] 'password']
  -- all privileges 表示所有权限
  -- *.* 表示所有库的所有表
  -- 库名.表名 表示某库下面的某表
  GRANT ALL PRIVILEGES ON `pms`.* TO 'pms'@'%' IDENTIFIED BY 'pms0817';

-- 查看权限
SHOW GRANTS FOR 用户名
  -- 查看当前用户权限
  SHOW GRANTS; 或 SHOW GRANTS FOR CURRENT_USER; 或 SHOW GRANTS FOR CURRENT_USER();

-- 撤消权限
REVOKE 权限列表 ON 表名 FROM 用户名
REVOKE ALL PRIVILEGES, GRANT OPTION FROM 用户名 -- 撤销所有权限

-- 权限层级
-- 要使用 GRANT 或 REVOKE, 您必须拥有 GRANT OPTION 权限, 并且您必须用于您正在授予或撤销的权限.
-- 全局层级: 全局权限适用于一个给定服务器中的所有数据库, mysql.user
GRANT ALL ON *.*和 REVOKE ALL ON *.*只授予和撤销全局权限.
-- 数据库层级: 数据库权限适用于一个给定数据库中的所有目标, mysql.db, mysql.host
GRANT ALL ON db_name.*和 REVOKE ALL ON db_name.*只授予和撤销数据库权限.
-- 表层级: 表权限适用于一个给定表中的所有列, mysql.talbes_priv
GRANT ALL ON db_name.tbl_name 和 REVOKE ALL ON db_name.tbl_name 只授予和撤销表权限.
-- 列层级: 列权限适用于一个给定表中的单一列, mysql.columns_priv
  -- 当使用 REVOKE 时, 您必须指定与被授权列相同的列.

-- 权限列表
ALL [PRIVILEGES] -- 设置除 GRANT OPTION 之外的所有简单权限
ALTER -- 允许使用 ALTER TABLE
ALTER ROUTINE -- 更改或取消已存储的子程序
CREATE -- 允许使用 CREATE TABLE
CREATE ROUTINE -- 创建已存储的子程序
CREATE TEMPORARY TABLES -- 允许使用 CREATE TEMPORARY TABLE
CREATE USER -- 允许使用 CREATE USER, DROP USER, RENAME USER 和 REVOKE ALL PRIVILEGES.
CREATE VIEW -- 允许使用 CREATE VIEW
DELETE -- 允许使用 DELETE
DROP -- 允许使用 DROP TABLE
EXECUTE -- 允许用户运行已存储的子程序
FILE -- 允许使用 SELECT...INTO OUTFILE 和 LOAD DATA INFILE
INDEX -- 允许使用 CREATE INDEX 和 DROP INDEX
INSERT -- 允许使用 INSERT
LOCK TABLES -- 允许对您拥有 SELECT 权限的表使用 LOCK TABLES
PROCESS -- 允许使用 SHOW FULL PROCESSLIST
REFERENCES -- 未被实施
RELOAD -- 允许使用 FLUSH
REPLICATION CLIENT -- 允许用户询问从属服务器或主服务器的地址
REPLICATION SLAVE -- 用于复制型从属服务器 (从主服务器中读取二进制日志事件)
SELECT -- 允许使用 SELECT
SHOW DATABASES -- 显示所有数据库
SHOW VIEW -- 允许使用 SHOW CREATE VIEW
SHUTDOWN -- 允许使用 mysqladmin shutdown
SUPER -- 允许使用 CHANGE MASTER, KILL, PURGE MASTER LOGS 和 SET GLOBAL 语句, mysqladmin debug 命令；允许您连接 (一次) , 即使已达到 max_connections.
UPDATE -- 允许使用 UPDATE
USAGE -- “无权限”的同义词
GRANT OPTION -- 允许授予权限
```

#### 表维护

```sql
-- 分析和存储表的关键字分布
ANALYZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE 表名 ...

-- 检查一个或多个表是否有错误
CHECK TABLE tbl_name [, tbl_name] ... [option] ...
option = {QUICK | FAST | MEDIUM | EXTENDED | CHANGED}

-- 整理数据文件的碎片
OPTIMIZE [LOCAL | NO_WRITE_TO_BINLOG] TABLE tbl_name [, tbl_name] ...
```

#### 杂项

1. 可用反引号 (`) 为标识符 (库名, 表名, 字段名, 索引, 别名) 包裹, 以避免与关键字重名！中文也可以作为标识符！
2. 每个库目录存在一个保存当前数据库的选项文件 db.opt.
3. 注释:
   * 单行注释 # 注释内容
   * 多行注释 /* 注释内容 */
   * 单行注释 -- 注释内容 (标准 SQL 注释风格, 要求双破折号后加一空格符 (空格, TAB, 换行等) )
4. 模式通配符:
   * _ 任意单个字符
   * % 任意多个字符, 甚至包括零字符
   * 单引号需要进行转义 \'
5. CMD 命令行内的语句结束符可以为 ";", "\G", "\g", 仅影响显示结果. 其他地方还是用分号结束. delimiter 可修改当前对话的语句结束符.
6. SQL 对大小写不敏感
7. 清除已有语句: \c

## 核心

### 存储引擎

在 MySQL 中的存储引擎有很多种, 可以通过“SHOW ENGINES”语句来查看.

| 特性                                                   | InnoDB | MyISAM | MEMORY | ARCHIVE |
| ------------------------------------------------------ | ------ | ------ | ------ | ------- |
| 存储限制(Storage limits)                               | 64TB   | No     | YES    | No      |
| 支持事物(Transactions)                                 | Yes    | No     | No     | No      |
| 锁机制(Locking granularity)                            | 行锁   | 表锁   | 表锁   | 行锁    |
| B 树索引(B-tree indexes)                               | Yes    | Yes    | Yes    | No      |
| T 树索引(T-tree indexes)                               | No     | No     | No     | No      |
| 哈希索引(Hash indexes)                                 | Yes    | No     | Yes    | No      |
| 全文索引(Full-text indexes)                            | Yes    | Yes    | No     | No      |
| 集群索引(Clustered indexes)                            | Yes    | No     | No     | No      |
| 数据缓存(Data caches)                                  | Yes    | No     | N/A    | No      |
| 索引缓存(Index caches)                                 | Yes    | Yes    | N/A    | No      |
| 数据可压缩(Compressed data)                            | Yes    | Yes    | No     | Yes     |
| 加密传输(Encrypted data [1])                           | Yes    | Yes    | Yes    | Yes     |
| 集群数据库支持(Cluster databases support)              | No     | No     | No     | No      |
| 复制支持(Replication support [2])                      | Yes    | No     | No     | Yes     |
| 外键支持(Foreign key support)                          | Yes    | No     | No     | No      |
| 存储空间消耗(Storage Cost)                             | 高     | 低     | N/A    | 非常低  |
| 内存消耗(Memory Cost)                                  | 高     | 低     | N/A    | 低      |
| 数据字典更新(Update statistics for data dictionary)    | Yes    | Yes    | Yes    | Yes     |
| 备份/时间点恢复(backup/point-in-time recovery [3])     | Yes    | Yes    | Yes    | Yes     |
| 多版本并发控制(Multi-Version Concurrency Control/MVCC) | Yes    | No     | No     | No      |
| 批量数据写入效率(Bulk insert speed)                    | 慢     | 快     | 快     | 非常快  |
| 地理信息数据类型(Geospatial datatype support)          | Yes    | Yes    | No     | Yes     |
| 地理信息索引(Geospatial indexing support [4])          | Yes    | Yes    | No     | Yes     |

1. 在服务器中实现 (通过加密功能) . 在其他表空间加密数据在 MySQL 5.7 或更高版本兼容.
2. 在服务中实现的, 而不是在存储引擎中实现的.
3. 在服务中实现的, 而不是在存储引擎中实现的.
4. 地理位置索引, InnoDB 支持可 mysql5.7.5 或更高版本兼容

#### InnoDB 存储引擎

InnoDB 给 MySQL 的表提供了事务处理, 回滚, 崩溃修复能力和多版本并发控制的事务安全. 在 MySQL 从 3.23.34a 开始包含 InnnoDB. 它是 MySQL 上第一个提供外键约束的表引擎. 而且 InnoDB 对事务处理的能力, 也是其他存储引擎不能比拟的. 靠后版本的 MySQL 的默认存储引擎就是 InnoDB.

InnoDB 存储引擎总支持 AUTO_INCREMENT. 自动增长列的值不能为空, 并且值必须唯一. MySQL 中规定自增列必须为主键. 在插入值的时候, 如果自动增长列不输入值, 则插入的值为自动增长后的值；如果输入的值为 0 或空 (NULL) , 则插入的值也是自动增长后的值；如果插入某个确定的值, 且该值在前面没有出现过, 就可以直接插入.

InnoDB 还支持外键 (FOREIGN KEY) . 外键所在的表叫做子表, 外键所依赖 (REFERENCES) 的表叫做父表. 父表中被字表外键关联的字段必须为主键. 当删除, 更新父表中的某条信息时, 子表也必须有相应的改变, 这是数据库的参照完整性规则.

InnoDB 中, 创建的表的表结构存储在.frm 文件中 (我觉得是 frame 的缩写吧) . 数据和索引存储在 innodb_data_home_dir 和 innodb_data_file_path 定义的表空间中.

InnoDB 的优势在于提供了良好的事务处理, 崩溃修复能力和并发控制. 缺点是读写效率较差, 占用的数据空间相对较大.

#### MyISAM 存储引擎

MyISAM 是 MySQL 中常见的存储引擎, 曾经是 MySQL 的默认存储引擎. MyISAM 是基于 ISAM 引擎发展起来的, 增加了许多有用的扩展.

MyISAM 的表存储成 3 个文件. 文件的名字与表名相同. 拓展名为 frm, MYD, MYI. 其实, frm 文件存储表的结构；MYD 文件存储数据, 是 MYData 的缩写；MYI 文件存储索引, 是 MYIndex 的缩写.

基于 MyISAM 存储引擎的表支持 3 种不同的存储格式. 包括静态型, 动态型和压缩型. 其中, 静态型是 MyISAM 的默认存储格式, 它的字段是固定长度的；动态型包含变长字段, 记录的长度不是固定的；压缩型需要用到 myisampack 工具, 占用的磁盘空间较小.

MyISAM 的优势在于占用空间小, 处理速度快. 缺点是不支持事务的完整性和并发性.

#### MEMORY 存储引擎

MEMORY 是 MySQL 中一类特殊的存储引擎. 它使用存储在内存中的内容来创建表, 而且数据全部放在内存中. 这些特性与前面的两个很不同.

每个基于 MEMORY 存储引擎的表实际对应一个磁盘文件. 该文件的文件名与表名相同, 类型为 frm 类型. 该文件中只存储表的结构. 而其数据文件, 都是存储在内存中, 这样有利于数据的快速处理, 提高整个表的效率. 值得注意的是, 服务器需要有足够的内存来维持 MEMORY 存储引擎的表的使用. 如果不需要了, 可以释放内存, 甚至删除不需要的表.

MEMORY 默认使用哈希索引. 速度比使用 B 型树索引快. 当然如果你想用 B 型树索引, 可以在创建索引时指定.

注意, MEMORY 用到的很少, 因为它是把数据存到内存中, 如果内存出现异常就会影响数据. 如果重启或者关机, 所有数据都会消失. 因此, 基于 MEMORY 的表的生命周期很短, 一般是一次性的.

### 索引数据结构

#### B-树

B-树中有两种节点类型: 索引节点和叶子节点. 叶子节点是用来存储数据的, 而索引节点则用来告诉用户存储在叶子节点中的数据顺序, 并帮助用户找到相应的数据.

B-树的搜索, 从根节点开始, 对节点内的关键字有序进行二分查找, 如果命中则结束, 否则进入查询关键字所属范围的儿子节点, 重复. 直到所对应的儿子指针为空, 或已经是叶子节点.

B-树是一种多路搜索树:

1. 定义任意非叶子节点最多有 M 个儿子, 且 M>2;
2. 根节点的儿子数为 [2,M];
3. 除根节点以外的非叶子节点的儿子数为 [M/2,M];
4. 每个节点存放至少 M/2-1(取上整)和至多 M-1 个关键字;
5. 非叶子节点的关键字个数=指向儿子节点的指针的个数-1;
6. 非叶子节点的关键字: k [i]<k [i+1];
7. 非叶子节点的指针: p [1], p [2], ·····, p [M]；其中 p [1] 指向的关键字小于 k [1] 的子树, p [M] 指向的关键字大于 K [m-1] 的子树;
8. 所有的叶子节点位于同一层;

#### B+树

B+树数据结构是 B-树实现的增强版本. 尽管 B+树支持 B-树索引的所有特性, 它们之间最显著的不同点在于 B+树中底层数据是根据被提及的索引列进行排序的. B+树还通过叶子节点之间的附加引用来优化扫描性能.

B+搜索和 B-搜索不同, 区别是 B+树只有达到叶子节点才命中 (B-树可以在非叶子节点命中) , 其性能等价于关键字全集做一次二分搜索.

B+树的特性:

1. 所有关键字都出现在叶子节点的链表中, 叶子节点相当于存储数据的数据层.
2. 不可能在非叶子节点上命中.
3. 非叶子节点相当于是叶子节点的索引, 叶子节点相当于数据层.

#### 散列

散列表数据结构是一种很简单的概念, 它将一种算法应用到给定值中以在底层数据存储系统中返回一个唯一的指针或位置. 散列表的优点是始终以线性时间复杂度找到需要读取的行的位置, 而不像 B-树那样需要横跨多层节点来确定位置.

#### 通信 R-树

R-树数据结构支持基于数据类型对几何数据进行管理. 目前只有 MyISAM 使用 R-树实现支持空间索引, 使用空间索引也有很多限制, 比如只支持唯一的 NOT NULL 列等.

#### 全文本

全文本结构也是一种 MySQL 采用的基本数据结构. 这种数据结构目前只有当前版本 MySQL 中的 MyISAM 存储引擎支持. 5.6 版本将要在 InnoDB 存储引擎中加入全文本功能. 全文本索引在大型系统中并没有什么实用的价值, 因为大规模系统有很多专门的文件检索产品. 所以不用在介绍.

#### MyISAM 的 B-树

MyISAM 存储引擎使用 B-树数据结构来实现主码索引, 唯一索引以及非主码索引. 在 MyISAM 实现数据目录和数据库模式子目录中, 用户可以找到和每个 MySQL 表对应的.MYD 和.MYI 文件. 数据库表上定义的索引信息就存储在 MYI 文件中, 该文件的块大小是 1024 字节. 这个大小是可以通过 myisam-block-size 系统变量分配.

在 MyISAM 中, 非主码索引的 B-树结构存储索引值和一个指向主码数据的指针, 这是 MyISAM 和 InnoDB 的一个显著区别. 这一点导致了两个存储引擎的索引的不同工作方式.

MyISAM 索引是在内存的一个公共缓存中管理的, 这个缓存的大小可以通过 key_buffer_size 或者其他命名键缓存来定义. 这是根据统计和规划的表索引的大小来设定缓存大小时主要的考虑因素.

#### InnoDB 的 B+树聚簇主码

InnoDB 存储引擎在它的主码索引 (也被称为聚簇主码) 中使用了 B+树, 这种结构把所有数据都和对应的主码组织在一起, 并且在叶子节点这一层上添加额外的向前和向后的指针, 这样就可以更方便地进行范围扫描.

在文件系统层面, 所有 InnoDB 数据和索引信息都默认在公共 InnoDB 表空间中管理, 否则管理员就通过 innodb_data_file_path 这个变量指定文件路径. 这是一个叫 ibdatal 文件.

由于 InnoDB 用聚簇主码存储数据, 底层信息占用的磁盘空间的大小很大程度上取决于页面的填充因子. 对于按序排列的主码, InnoDB 会用 16K 页面的 15/16 作为填充因子. 对于不是按序排列的主码, 默认情况下 InnoDB 会插入初始数据的时候为每一个页面分配 50%作为填充因子.

在改索引的实现方式中 B+树的叶子节点上是 data 就是数据本身, key 为主键, 如果是一般索引的话, data 便会指向对应的主索引. 在 B+树的每一个叶子节点上面增加一个指向相邻叶子节点的指针, 就形成了带有顺序访问指针的 B+树. 其目的是提高区间访问的性能.

#### InnoDB 的 B-树非主码

InnoDB 中的非主码索引使用了 B-树数据结构, 但 InnoDB 中的 B-树结构实现和 MyISAM 中并不一样. 在 InnoDB 中, 非主码索引存储的是主码的实际值. 而 MyISAM 中, 非主码索引存储的包含主码值的数据指针. 这一点很重要. 首先, 当定义很大的主码的时候, InnoDB 的非主码索引可能回更大, 随着非主码索引数量的增加, 索引之间大小差别可能会变得很大. 另一个不同点在于非主码索引当前可以包含主键的值, 并且可以不是索引必须有的部分.

#### 内存散列索引

在默认 MySQL 的引擎索引中, 只有 MEMORY 引擎支持散列数据结构, 散列结构的强度可以表示为直接键查找的简单性, 散列索引的相似度模式匹配查询比直接查询慢. 也可以为 MEMORY 引擎指定一个 B-树索引实现.

#### 内存 B-树索引

对于大型 MEMORY 表来说, 使用散列索引进行索引范围搜索的效率很低, B-树索引在执行直接键查询时确实比使用默认的散列索引快. 根据 B-树的不同深度, B-树索引在个别操作中的确可能比散列算法快.

#### InnoDB 内部散列索引

InnoDB 存储引擎在聚簇 B+树索引中存储主码: 但在 InnoDB 内部还是使用内存中的散列表来更高效地进行主码查询. 这个机制有 InnoDB 存储引擎来管理, 用户只能通过 innodb_adaptive_hash_index 配置项来选择是否启用这个唯一的配置选项.

## 优化

### Explain 执行计划

在 select 语句之前增加 explain 关键字, MySQL 会在查询上设置一个标记, 执行查询时, 会返回执行计划的信息, 而不是执行这条 SQL (如果 from 中包含子查询, 仍会执行该子查询, 将结果放入临时表中) .

```sh
mysql> explain select * from mysql.user;

+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+

| id | select_type | table | partitions | type |possible_keys | key  | key_len | ref  | rows | filtered | Extra |

+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+

|  1 |SIMPLE      | user  | NULL      | ALL  | NULL          | NULL | NULL    | NULL |  26 |   100.00 | NULL  |

+----+-------------+-------+------------+------+---------------+------+---------+------+------+----------+-------+

1 row in set, 1 warning (0.00 sec)
```

explain 有两个变种:

* explain extended: 会在 explain  的基础上额外提供一些查询优化的信息. 紧随其后通过 showwarnings 命令可以 得到优化后的查询语句, 从而看出优化器优化了什么. 额外还有 filtered 列, 是一个半分比的值, rows * filtered/100 可以估算出将要和 explain 中前一个表进行连接的行数 (前一个表指 explain 中的 id 值比当前表 id 值小的表) .
* explain partitions: 相比 explain 多了个 partitions 字段, 如果查询是基于分区表的话, 会显示查询将访问的分区.

#### id

id 列的编号是 select 的序列号, 有几个 select 就有几个 id, 并且 id 的顺序是按 select 出现的顺序增长的. MySQL 将 select 查询分为简单查询和复杂查询. 复杂查询分为三类: 简单子查询, 派生表 (from 语句中的子查询) , union 查询.

union 结果总是放在一个匿名临时表中, 临时表不在 SQL 总出现, 因此它的 id 是 NULL.

1. id 相同: 执行顺序由上至下
1. id 不同: 如果是子查询, id 的序号会递增, id 值越大优先级越高, 越先被执行. 理解是 SQL 执行的顺利的标识,SQL 从大到小的执行,先执行的语句编号大;
1. id 相同又不同 (两种情况同时存在) : id 如果相同, 可以认为是一组, 从上往下顺序执行；在所有组中, id 值越大, 优先级越高, 越先执行

#### select_type

查询的类型, 主要是用于区分普通查询, 联合查询, 子查询等复杂的查询.

* SIMPLE: 表示此查询不包含 UNION 查询或子查询
* PRIMARY: 表示此查询是最外层的查询
* SUBQUERY: 子查询中的第一个 SELECT
* UNION: 表示此查询是 UNION 的第二或随后的查询
* DEPENDENT UNION: UNION 中的第二个或后面的查询语句, 取决于外面的查询
* UNION RESULT, UNION 的结果
* DEPENDENT SUBQUERY: 子查询中的第一个 SELECT, 取决于外面的查询. 即子查询依赖于外层查询的结果.
* DERIVED: 衍生, 表示导出表的 SELECT (FROM 子句的子查询)

#### table

table 表示查询涉及的表或衍生的表:

当 from 子句中有子查询时, table 列是 <derivenN> 格式, 表示当前查询依赖 id=N 的查询, 于是先执行 id=N 的查询. 当有 union 时, UNION RESULT 的 table 列的值为 <union1,2>, 1 和 2 表示参与 union 的 select 行 id.

#### type

type 字段比较重要, 它提供了判断查询是否高效的重要依据依据. 通过 type 字段, 我们判断此次查询是 全表扫描 还是 索引扫描等.

type 常用的取值有:

* NULL: mysql 能够在优化阶段分解查询语句, 在执行阶段用不着再访问表或索引. 例如: 在索引列中选取最小值, 可以单独查找索引来完成, 不需要在执行时访问表
* system: 表中只有一条数据, 这个类型是特殊的 const 类型.
* const: 针对主键或唯一索引的等值查询扫描, 最多只返回一行数据. const 查询速度非常快, 因为它仅仅读取一次即可. 例如下面的这个查询, 它使用了主键索引, 因此 type 就是 const 类型的: explain select * from user_info where id = 2；
* eq_ref: 此类型通常出现在多表的 join 查询, 表示对于前表的每一个结果, 都只能匹配到后表的一行结果. 并且查询的比较操作通常是 =, 查询效率较高. 例如: explain select * from user_info, order_info where user_info.id = order_info.user_id;
* ref: 此类型通常出现在多表的 join 查询, 针对于非唯一或非主键索引, 或者是使用了 最左前缀 规则索引的查询. 例如下面这个例子中, 就使用到了 ref 类型的查询: explain select * from user_info, order_info where user_info.id = order_info.user_id AND order_info.user_id = 5
* ref_or_null: 类似 ref, 但是可以搜索值为 NULL 的行.
* index_merge: 表示使用了索引合并的优化方法. 例如下表: id 是主键, tenant_id 是普通索引. or 的时候没有用 primary key, 而是使用了 primary key(id) 和 tenant_id 索引
* range: 表示使用索引范围查询, 通过索引字段范围获取表中部分数据记录. 这个类型通常出现在 =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, IN() 操作中. 例如下面的例子就是一个范围查询: explain select * from user_info where id between 2 and 8；
* index: 表示全索引扫描(full index scan), 和 ALL 类型类似, 只不过 ALL 类型是全表扫描, 而 index 类型则仅仅扫描所有的索引, 而不扫描数据. index 类型通常出现在: 所要查询的数据直接在索引树中就可以获取到, 而不需要扫描数据. 当是这种情况时, Extra 字段 会显示 Using index.
* ALL: 表示全表扫描, 这个类型的查询是性能最差的查询之一. 通常来说, 我们的查询不应该出现 ALL 类型的查询, 因为这样的查询在数据量大的情况下, 对数据库的性能是巨大的灾难. 如一个查询是 ALL 类型查询, 那么一般来说可以对相应的字段添加索引来避免.
* fulltext: 全文索引检索, 要注意, 全文索引的优先级很高, 若全文索引和普通索引同时存在时, mysql 不管代价, 优先选择使用全文索引.

通常来说, 不同的 type 类型的性能关系: ALL < index < range ~ index_merge < ref < eq_ref < const < system

ALL 类型因为是全表扫描, 因此在相同的查询条件下, 它是速度最慢的. 而 index 类型的查询虽然不是全表扫描, 但是它扫描了所有的索引, 因此比 ALL 类型的稍快.后面的几种类型都是利用了索引来查询数据, 因此可以过滤部分或大部分数据, 因此查询效率就比较高了.

#### possible_keys

这一列显示查询可能使用哪些索引来查找.  

explain 时可能出现 possible_keys 有列, 而 key 显示 NULL 的情况, 这种情况是因为表中数据不多, mysql 认为索引对此查询帮助不大, 选择了全表查询.  

如果该列是 NULL, 则没有相关的索引. 在这种情况下, 可以通过检查 where 子句看是否可以创造一个适当的索引来提高查询性能, 然后用 explain 查看效果.

#### key

这一列显示 mysql 实际采用哪个索引来优化对该表的访问.

如果没有使用索引, 则该列是 NULL. 如果想强制 mysql 使用或忽视 possible_keys 列中的索引, 在查询中使用 force index, ignore index.
查询中如果使用了覆盖索引, 则该索引仅出现在 key 列表中.

#### key_len

表示查询优化器使用了索引的字节数, 这个字段可以评估组合索引是否完全被使用. 查询中使用的索引的长度 (最大可能长度) , 并非实际使用长度, 理论上长度越短越好. key_len 是根据表定义计算而得的, 计算规则如下:

* 字符串
  * char(n): n 字节长度
  * varchar(n): 2 字节存储字符串长度, 如果是 utf-8, 则长度 3n + 2
* 数值类型
  * tinyint: 1 字节
  * smallint: 2 字节
  * int: 4 字节
  * bigint: 8 字节　　
* 时间类型　
  * date: 3 字节
  * timestamp: 4 字节
  * datetime: 8 字节
* 如果字段允许为 NULL, 需要 1 字节记录是否为 NULL

索引最大长度是 768 字节, 当字符串过长时, mysql 会做一个类似左前缀索引的处理, 将前半部分的字符提取出来做索引.

#### ref

显示索引的那一列被使用了, 如果可能, 是一个常量 const.

这一列显示了在 key 列记录的索引中, 表查找值所用到的列或常量, 常见的有: const (常量) , func, NULL, 字段名 (例: film.id) .

#### rows

mysql 查询优化器根据统计信息, 估算 sql 要查找到结果集需要扫描读取的数据行数, 这个值非常直观的显示 sql 效率好坏, 原则上 rows 越少越好.

#### extra

重要的额外信息

| 类型                         | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Using filesort               | MySQL 有两种方式可以生成有序的结果, 通过排序操作或者使用索引, 当 Extra 中出现了 Using filesort 说明 MySQL 使用了后者, 但注意虽然叫 filesort 但并不是说明就是用了文件来进行排序, 只要可能排序都是在内存里完成的. 大部分情况下利用索引排序更快, 所以一般这时也要考虑优化查询了. 使用文件完成排序操作, 这是可能是 ordery by, group by 语句的结果, 这可能是一个 CPU 密集型的过程, 可以通过选择合适的索引来改进性能, 用索引来为查询结果排序. |
| Using temporary              | 用临时表保存中间结果, 常用于 GROUP BY 和 ORDER BY 操作中, 一般看到它说明查询需要优化了, 就算避免不了临时表的使用也要尽量避免硬盘临时表的使用.                                                                                                                                                                                                                                                                                           |
| Not exists                   | MYSQL 优化了 LEFT JOIN, 一旦它找到了匹配 LEFT JOIN 标准的行, 就不再搜索了.                                                                                                                                                                                                                                                                                                                                                              |
| Using index                  | 说明查询是覆盖了索引的, 不需要读取数据文件, 从索引树 (索引文件) 中即可获得信息. 如果同时出现 using where, 表明索引被用来执行索引键值的查找, 没有 using where, 表明索引用来读取数据而非执行查找动作. 这是 MySQL 服务层完成的, 但无需再回表查询记录.                                                                                                                                                                                      |
| Using index condition        | 这是 MySQL 5.6 出来的新特性, 叫做“索引条件推送”. 简单说一点就是 MySQL 原来在索引上是不能执行如 like 这样的操作的, 但是现在可以了, 这样减少了不必要的 IO 操作, 但是只能用在二级索引上.                                                                                                                                                                                                                                                   |
| Using where                  | 使用了 WHERE 从句来限制哪些行将与下一张表匹配或者是返回给用户. 注意: Extra 列出现 Using where 表示 MySQL 服务器将存储引擎返回服务层以后再应用 WHERE 条件过滤.                                                                                                                                                                                                                                                                           |
| Using join buffer            | 使用了连接缓存: Block Nested Loop, 连接算法是块嵌套循环连接;Batched Key Access, 连接算法是批量索引连接                                                                                                                                                                                                                                                                                                                                  |
| impossible where             | where 子句的值总是 false, 不能用来获取任何元组                                                                                                                                                                                                                                                                                                                                                                                          |
| select tables optimized away | 在没有 GROUP BY 子句的情况下, 基于索引优化 MIN/MAX 操作, 或者对于 MyISAM 存储引擎优化 COUNT(*)操作, 不必等到执行阶段再进行计算, 查询执行计划生成的阶段即完成优化.                                                                                                                                                                                                                                                                       |
| distinct                     | 优化 distinct 操作, 在找到第一匹配的元组后即停止找同样值的动作                                                                                                                                                                                                                                                                                                                                                                          |

### 设计规范

#### 数据库设计

1. 使用 Innodb 存储引擎: 没有特殊要求 (即 Innodb 无法满足的功能如: 列存储, 存储空间数据等) 的情况下, 所有表必须使用 Innodb 存储引擎 (mysql5.5 之前默认使用 Myisam, 5.6 以后默认的为 Innodb) Innodb 支持事务, 支持行级锁, 更好的恢复性, 高并发下性能更好
2. 字符集统一使用 UTF8mb4: 兼容性更好, 统一字符集可以避免由于字符集转换产生的乱码, 不同的字符集进行比较前需要进行转换会造成索引失效
3. 所有表和字段都需要添加注释: 使用 comment 从句添加表和列的备注 从一开始就进行数据字典的维护
4. 尽量控制单表数据量的大小, 建议控制在 500 万以内: 500 万并不是 MySQL 数据库的限制, 过大会造成修改表结构, 备份, 恢复都会有很大的问题. 可以用历史数据归档 (应用于日志数据) , 分库分表 (应用于业务数据) 等手段来控制数据量大小
5. 谨慎使用 MySQL 分区表: 分区表在物理上表现为多个文件, 在逻辑上表现为一个表 谨慎选择分区键, 跨分区查询效率可能更低 建议采用物理分表的方式管理大数据
6. 尽量做到冷热数据分离, 减小表的宽度: MySQL 限制每个表最多存储 4096 列, 并且每一行数据的大小不能超过 65535 字节 减少磁盘 IO,保证热数据的内存缓存命中率 (表越宽, 把表装载进内存缓冲池时所占用的内存也就越大,也会消耗更多的 IO) 更有效的利用缓存, 避免读入无用的冷数据 经常一起使用的列放到一个表中 (避免更多的关联操作)
7. 禁止在表中建立预留字段: 预留字段的命名很难做到见名识义 预留字段无法确认存储的数据类型, 所以无法选择合适的类型 对预留字段类型的修改, 会对表进行锁定
8. 禁止在数据库中存储图片, 文件等大的二进制数据: 通常文件很大, 会短时间内造成数据量快速增长, 数据库进行数据库读取时, 通常会进行大量的随机 IO 操作, 文件很大时, IO 操作很耗时 通常存储于文件服务器, 数据库只存储文件地址信息

#### 表字段设计

1. 优先选择符合存储需要的最小的数据类型: 列的字段越大, 建立索引时所需要的空间也就越大, 这样一页中所能存储的索引节点的数量也就越少也越少, 在遍历时所需要的 IO 次数也就越多, 索引的性能也就越差
   1. 将字符串转换成数字类型存储, 如: 将 IP 地址转换成整形数据. inet_aton 把 ip 转为无符号整型(4-8 位), inet_ntoa 把整型的 ip 转为地址
   2. 对于非负型的数据 (如自增 ID, 整型 IP) 来说, 要优先使用无符号整型来存储, 因为无符号相对于有符号可以多出一倍的存储空间, SIGNED INT -2147483648~2147483647, UNSIGNED INT 0~4294967295
   3. VARCHAR(N)中的 N 代表的是字符数, 而不是字节数, 使用 UTF8 存储 255 个汉字 Varchar(255)=765 个字节. 过大的长度会消耗更多的内存
2. 避免使用 TEXT, BLOB 数据类型, 最常见的 TEXT 类型可以存储 64k 的数据
   1. 建议把 BLOB 或是 TEXT 列分离到单独的扩展表中: Mysql 内存临时表不支持 TEXT, BLOB 这样的大数据类型, 如果查询中包含这样的数据, 在排序等操作时, 就不能使用内存临时表, 必须使用磁盘临时表进行. 而且对于这种数据, Mysql 还是要进行二次查询, 会使 sql 性能变得很差, 但是不是说一定不能使用这样的数据类型. 如果一定要使用, 建议把 BLOB 或是 TEXT 列分离到单独的扩展表中, 查询时一定不要使用 select * 而只需要取出必要的列, 不需要 TEXT 列的数据时不要对该列进行查询.
   2. TEXT 或 BLOB 类型只能使用前缀索引: 因为 MySQL 对索引字段长度是有限制的, 所以 TEXT 类型只能使用前缀索引, 并且 TEXT 列上是不能有默认值的.
3. 避免使用 ENUM 类型
   1. 修改 ENUM 值需要使用 ALTER 语句
   2. ENUM 类型的 ORDER BY 操作效率低, 需要额外操作
   3. 禁止使用数值作为 ENUM 的枚举值
4. 尽可能把所有列定义为 NOT NULL
   1. 索引 NULL 列需要额外的空间来保存, 所以要占用更多的空间；
   2. 进行比较和计算时要对 NULL 值做特别的处理
5. 使用 TIMESTAMP (4 个字节) 或 DATETIME 类型 (8 个字节) 存储时间
   1. TIMESTAMP 存储的时间范围 1970-01-01 00:00:01 ~ 2038-01-19-03:14:07.
   2. TIMESTAMP 占用 4 字节和 INT 相同, 但比 INT 可读性高
   3. 超出 TIMESTAMP 取值范围的使用 DATETIME 类型存储
6. 字符串存储日期型的数据无法用日期函数进行计算和比较, 用字符串存储日期要占用更多的空间
7. 同财务相关的金额类数据必须使用 decimal 类型: Decimal 类型为精准浮点数, 在计算时不会丢失精度. 占用空间由定义的宽度决定, 每 4 个字节可以存储 9 位数字, 并且小数点要占用一个字节. 可用于存储比 bigint 更大的整型数据.

### 索引设计

* 出现在 SELECT, UPDATE, DELETE 语句的 WHERE 从句中的列
* 包含在 ORDER BY, GROUP BY, DISTINCT 中的字段
* 不要将符合 1 和 2 中的字段的列都建立一个索引, 通常将 1, 2 中的字段建立联合索引效果更好
* 多表 join 的关联列

#### 联合索引

建立索引的目的是: 希望通过索引进行数据查找, 减少随机 IO, 增加查询性能 , 索引能过滤出越少的数据, 则从磁盘中读入的数据也就越少.

* 区分度最高的放在联合索引的最左侧 (区分度=列中不同值的数量/列的总行数) ；
* 尽量把字段长度小的列放在联合索引的最左侧 (因为字段长度越小, 一页能存储的数据量越大, IO 性能也就越好) ；
* 使用最频繁的列放到联合索引的左侧 (这样可以比较少的建立一些索引) .

#### 覆盖索引

对于频繁的查询优先考虑使用覆盖索引. 就是包含了所有查询字段(where,select,ordery by,group by 包含的字段)的索引

覆盖索引的好处:

* 避免 Innodb 表进行索引的二次查询: Innodb 是以聚集索引的顺序来存储的, 对于 Innodb 来说, 二级索引在叶子节点中所保存的是行的主键信息, 如果是用二级索引查询数据的话, 在查找到相应的键值后, 还要通过主键进行二次查询才能获取我们真实所需要的数据. 而在覆盖索引中, 二级索引的键值中可以获取所有的数据, 避免了对主键的二次查询 , 减少了 IO 操作, 提升了查询效率.
* 可以把随机 IO 变成顺序 IO 加快查询效率: 由于覆盖索引是按键值的顺序存储的, 对于 IO 密集型的范围查找来说, 对比随机从磁盘读取每一行的数据 IO 要少的多, 因此利用覆盖索引在访问时也可以把磁盘的随机读取的 IO 转变成索引查找的顺序 IO.

#### 尽量避免使用外键约束

* 不建议使用外键约束 (foreign key) , 但一定要在表与表之间的关联键上建立索引；
* 外键可用于保证数据的参照完整性, 但建议在业务端实现；
* 外键会影响父表和子表的写操作从而降低性能.

# 引用

* [数据库存储引擎](https://github.com/jaywcjlove/mysql-tutorial/blob/master/chapter3/3.5.md)
* [吐血总结｜史上最全的 MySQL 学习资料！！](https://juejin.im/post/5c91ac636fb9a071012a0c28)
* [explain 执行计划详解](https://blog.csdn.net/eagle89/article/details/80433723)