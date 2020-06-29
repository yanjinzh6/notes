---
title: Linux journalctl 命令
date: 2020-06-14 13:30:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: linux-journalctl-command
---

## 简介

journalctl 用来查询 systemd-journald 服务收集到的日志, systemd-journald 服务是 systemd init 系统提供的收集系统日志的服务

## 命令

```sh
journalctl [OPTIONS…] [MATCHES…]
```

## 常用

```sh
# 获取帮助
man journalctl
journalctl -h

# 输出所有日志
journalctl

# 查看所有可匹配的字段
man 7 systemd.journal-fields

# 字段匹配, 通过 FIELD=VALUE 的格式来匹配日志
journalctl _SYSTEMD_UNIT=sshd.service

# 多字段 与 匹配
journalctl _SYSTEMD_UNIT=sshd.service PRIORITY=6

# 多字段使用 + 号进行 与 匹配
journalctl _SYSTEMD_UNIT=sshd.service + _PID=28097

# 查询字段可选值
journalctl -F PRIORITY

# 同字段多条件 或 匹配
journalctl _SYSTEMD_UNIT=sshd.service _SYSTEMD_UNIT=cron.service

# 查看日志占用的磁盘空间
journalctl --disk-usage

# 设置日志最大占用空间 1G
journalctl --vacuum-size=1G

# 设置日志最大的保存时间范围 1 年
journalctl --vacuum-time=1years

# 查看指定时间段的日志
journalctl --since "2020-06-17 15:30:00" --until "2020-06-17 15:40:00"

# 按 unit 过滤日志, 可以获得多个服务日志
journalctl -u nginx.service -u php-fpm.service

# 显示最新 20 行, 默认 10
journalctl -n 20

# 使用标准输出到文件
journalctl --no-pager > file.log

# 实时显示日志
journalctl -f

# 过滤可执行文件路径
journalctl /usr/lib/systemd/systemd
journalctl /usr/bin/bash

# 查看内核日志
journalctl -k
```

<!-- more -->

## 匹配

匹配主要用于筛选, 每个日志都是一个对象, 里面很多参数, 如下

```json
{
  "__CURSOR": "s=258934bd652b422a9f509f9ee998a2ae;i=363da7f;b=577c6a853f214160af459992490b5fad;m=b127c4b0833;t=5a844232534d5;x=162cedac6a756190",
  "__REALTIME_TIMESTAMP": "1592385484436693",
  "__MONOTONIC_TIMESTAMP": "12174022608947",
  "_BOOT_ID": "577c6a853f214160af459992490b5fad",
  "_UID": "0",
  "_GID": "0",
  "_CAP_EFFECTIVE": "3fffffffff",
  "_SYSTEMD_SLICE": "system.slice",
  "_MACHINE_ID": "c74619868c104cfbb10536e85ec81d93",
  "_HOSTNAME": "ubuntu",
  "_TRANSPORT": "stdout",
  "PRIORITY": "6",
  "SYSLOG_FACILITY": "3",
  "SYSLOG_IDENTIFIER": "microk8s.daemon-containerd",
  "_COMM": "containerd",
  "_EXE": "/snap/microk8s/1378/bin/containerd",
  "_CMDLINE": "/snap/microk8s/1378/bin/containerd --config /var/snap/microk8s/1378/args/containerd.toml --root /var/snap/microk8s/common/var/lib/containerd --state /var/snap/microk8s/common/run/containerd --address /var/snap/microk8s/common/run/containerd.sock",
  "_SELINUX_CONTEXT": "snap.microk8s.daemon-containerd (complain)\n",
  "_SYSTEMD_CGROUP": "/system.slice/snap.microk8s.daemon-containerd.service",
  "_SYSTEMD_UNIT": "snap.microk8s.daemon-containerd.service",
  "_STREAM_ID": "2bccced224ef4b33bf84e998609df686",
  "_PID": "11748",
  "_SYSTEMD_INVOCATION_ID": "c78d0f089a4e4317b43141191ea4db3c",
  "MESSAGE": "time=\"2020-06-17T17:18:04.436655034+08:00\" level=info msg=\"Finish piping \"stdout\" of container exec \"a15e4cd4647e5e217f319a143d027a9ed64919f5eeba49b9caa0b72d068f1bd6\"\""
}
```

筛选规则

- 多个条件为 与 操作
- 单个条件多个不同值为 或 操作
- 使用 `+` 号为 或 操作

多条件筛选

```sh
journalctl _SYSTEMD_UNIT=snap.microk8s.daemon-containerd.service _PID=11748 + _SYSTEMD_UNIT=nginx.service
```

这里的结果是查询 `(_SYSTEMD_UNIT=snap.microk8s.daemon-containerd.service AND _PID=11748) OR _SYSTEMD_UNIT=nginx.service`

## 保存日志到文件中

systemd-journald 服务收集到的日志默认保存在 `/run/log` 目录中, 重启系统会丢掉以前的日志信息

可以将所有的日志都保存到文件中, 这样重新启动后就不会丢掉以前的日志

### 通过创建 journal 目录的方式

```sh
# 创建方法
mkdir /var/log/journal
# 配置权限
chown root:systemd-journal /var/log/journal
chmod 2775 /var/log/journal
# 重启服务
systemctl restart systemd-journald.service
```

重启后日志文件被保存在 `/var/log/journal` 目录下

### 通过修改配置文件的方式

- 修改配置文件 `/etc/systemd/journald.conf`, 修改 `Storage=persistent` 配置
- 重启服务 `systemctl restart systemd-journald.service`

## 限定日志所能占用的最高容量

可以通过修改 `/etc/systemd/journald.conf` 文件以下参数来限定日志数据可以占用的最大存储数量和日志数据体积的膨胀速度

- SystemMaxUse: 指定 journal 所能使用的最高持久存储容量
- SystemKeepFree: 指定 journal 在添加新条目时需要保留的剩余空间
- SystemMaxFileSize: 控制单 `一journal` 文件大小, 符合要求方可被转为持久存储
- RuntimeMaxUse: 指定易失性存储中的最大可用磁盘容量 (`/run` 文件系统之内)
- RuntimeKeepFree: 指定向易失性存储内写入数据时为其它应用保留的空间量 (`/run` 文件系统之内)
- RuntimeMaxFileSize: 指定单 `一journal` 文件可占用的最大易失性存储容量 (`/run` 文件系统之内)

## 查看某次启动后的日志

默认情况下 systemd-journald 服务只保存本次启动后的日志, 当配置了日志保存到文件中之后, 就可以通过下面的命令查看系统的重启记录

```sh
journalctl --list-boots
# 结果
0 577c6a853f214160af459992490b5fad Mon 2020-06-15 03:59:05 CST—Fri 2020-06-19 09:14:40 CST
# 获取某次运行日志, 默认获取当前运行日志
journalctl -b 0
# 或
journalctl -b 577c6a853f214160af459992490b5fad
```

## 查看指定时间段的日志

利用 `--since` 与 `--until` 选项设定时间段, 如果日期部分未填写, 则会直接显示当前日期, 如果时间部分未填写, 则缺省使用 "00:00:00"(午夜), 可以使用 "yesterday", "today", "tomorrow" 或者 "now" 等

```sh
# 获取一个小时内的日志
journalctl --since "1 hour ago"
```

## 按 unit 过滤日志

systemd 把几乎所有的任务都抽象成了 unit, 可以方便的使用 `-u` 选项通过 unit 的名称来过滤器日志记录, 可以使用多个 `-u` 选项同时获得多个 unit 的日志

```sh
journalctl -u nginx.service
```

## 通过日志级别进行过滤

除了通过 `PRIORITY=` 的方式, 还可以通过 `-p` 选项来过滤日志的级别, 可以指定的优先级如下:

- 0: emerg
- 1: alert
- 2: crit
- 3: err
- 4: warning
- 5: notice
- 6: info
- 7: debug

## 结果重定向到标准输出

默认情况下, journalctl 会在 pager 内显示输出结果, 可以通过 `--no-pager` 选项使用标准输出, 可以使用 `-o` 选项指定输出格式

## 选项

```sh
-no-full, --full, -l
如果字段内容超长 则以省略号(…)截断以适应列宽, 默认显示完整的字段内容 (超长的部分换行显示或者被分页工具截断)

老旧的 `-l/--full` 选项 仅用于撤销已有的 `--no-full` 选项, 除此之外没有其他用处

-a, --all
完整显示所有字段内容, 即使其中包含非打印字符或者字段内容超长, 默认情况下, 包含非打印字符的字段将被缩写为"blob data"(二进制数据), 注意, 分页程序可能会再次转义非打印字符

-f, --follow
只显示最新的日志项, 并且不断显示新生成的日志项, 此选项隐含了 `-n` 选项

-e, --pager-end
在分页工具内 立即跳转到日志的尾部, 此选项隐含了 `-n 1000` 以确保分页工具不必缓存太多的日志行, 不过这个隐含的行数可以被明确设置的 `-n` 选项覆盖, 注意, 此选项仅可用于 `less(1)` 分页器

-n, --lines=
限制显示最新的日志行数, `--pager-end` 与 `--follow` 隐含了此选项, 此选项的参数: 若为正整数则表示最大行数； 若为 "all" 则表示不限制行数； 若不设参数则表示默认值10行

--no-tail
显示所有日志行, 也就是用于撤销已有的 `--lines=` 选项(即使与 `-f` 连用)

-r, --reverse
反转日志行的输出顺序, 也就是最先显示最新的日志

-o, --output=
控制日志的 输出格式, 可以使用如下选项:

  short
  这是默认值, 其输出格式与传统的 syslog 文件的格式相似, 每条日志一行

  short-full
  与 short 类似, 只是将时间戳字段 按照 `--since=` 与 `--until=` 接受的格式显示, 与 short 的不同之处在于, 输出的时间戳中还包含星期, 年份, 时区信息, 并且与系统的本地化设置无关

  short-iso
  与 short 类似, 只是将时间戳字段以 ISO 8601 格式 显示

  short-iso-precise
  与 short-iso 类似, 只是将时间戳字段的秒数 精确到了微秒级别(百万分之一秒)

  short-precise
  与 short 类似, 只是将时间戳字段的秒数 精确到了微秒级别(百万分之一秒)

  short-monotonic
  与 short 类似, 只是将时间戳字段的零值 从内核启动时开始计算

  short-unix
  与 short 类似, 只是将时间戳字段显示为从 "UNIX时间原点" (1970-1-1 00:00:00 UTC)以来的秒数, 精确到微秒级别

  verbose
  以结构化的格式显示 每条日志的所有字段

  export
  将日志序列化为二进制字节流 (大部分依然是文本), 以适用于备份与网络传输(详见 Journal Export Format 文档), 亦可使用 `systemd-journal-remote.service(8)` 工具将二进制字节流转换为本地 journald 格式

  json
  将日志项格式化为 JSON 对象, 并用换行符分隔(也就是每条日志一行, 详见 Journal JSON Format 文档), 字段值通常按照 JSON 字符串规范进行编码, 但是如下三种情况例外:

  大于4096字节的字段将被编码为 null 值(可以使用 `--all` 选项关闭此特性, 但是这样做会导致生成又大又长的 JSON 对象)

  日志中允许存在同名字段, 但是在 JSON 对象中不允许, 因此, 同名字段的多个值在 JSON 对象中将会被 编码为一个数组

  包含非打印字符或非UTF8字符的字段, 将被编码为包含原始二进制字节的数组(其中的每个字节都视为一个无符号整数)

  注意, 这种编码方式是可逆的(存在空间大小的限制)

  json-pretty
  将日志项按照JSON数据结构格式化, 但是每个字段一行, 以便于人类阅读

  json-sse
  将日志项按照JSON结构格式化, 每条日志一行, 但是用大括号包围, 以符合 Server-Sent Events 的要求

  json-seq
  将日志项按照JSON结构格式化, 同时为每条日志加上一个ASCII记录分隔符(0x1E)前缀以及一个ASCII换行符(0x0A)后缀, 以符合 JavaScript Object Notation (JSON) Text Sequences ("application/json-seq") 的要求

  cat
  仅显示日志的实际内容, 而不显示与此日志相关的 任何元数据(包括时间戳)

  with-unit
  与 short-full 类似, 但是在日志项前缀中使用单元名称代替传统的 syslog 标识, 这对于从模板实例化而来的单元比较有意义, 因为会在单元名称中包含实例化参数

--output-fields=
一个逗号分隔的字段名称列表, 表示仅输出列表中的字段, 仅影响默认输出全部字段的输出格式 (verbose, export, json, json-pretty, json-sse, json-seq), 注意, "__CURSOR", "__REALTIME_TIMESTAMP", "__MONOTONIC_TIMESTAMP", "_BOOT_ID" 字段永远输出, 不能被排除

--utc
以世界统一时间 (UTC) 表示时间

--no-hostname
不显示来源于本机的日志消息的主机名字段, 此选项仅对 short 系列输出格式(见上文)有效

-x, --catalog
在日志的输出中 增加一些解释性的短文本, 以帮助进一步说明 日志的含义,  问题的解决方案, 支持论坛,  开发文档, 以及其他任何内容, 并非所有日志都有 这些额外的帮助文本, 详见 Message Catalog Developer Documentation 文档

注意, 如果要将日志输出用于bug报告, 请不要使用 此选项

-q, --quiet
安静模式, 也就是当以普通用户身份运行时, 不显示任何警告信息与提示信息, 例如:  "-- Logs begin at …", "-- Reboot --"

-m, --merge
混合显示 包括来自于其他远程主机的日志在内的所有可见日志

-b [ID][±offset], --boot=[ID][±offset]
显示特定于某次启动的日志, 这相当于添加了一个 "_BOOT_ID=" 匹配条件

如果未指定 ID 与 `±offset` 参数, 则表示仅显示本次启动的日志

如果省略了 ID , 那么当 `±offset` 是正数的时候, 将从日志头开始正向查找, 否则(也就是为负数或零)将从日志尾开始反响查找, 举例来说, "-b 1" 表示按时间顺序排列最早的那次启动, "-b 2" 则表示在时间上第二早的那次启动； "-b -0" 表示最后一次启动, "-b -1" 表示在时间上第二近的那次启动, 以此类推, 如果 `±offset` 也省略了, 那么相当于"-b -0", 除非本次启动不是最后一次启动(例如用 --directory 指定了 另外一台主机上的日志目录)

如果指定了32字符的 ID , 那么表示以此 ID 所代表的那次启动为基准 计算偏移量 (±offset), 计算方法同上, 换句话说, 省略 ID 表示以本次启动为基准 计算偏移量 (±offset)

--list-boots
列出每次启动的 序号(也就是相对于本次启动的偏移量), 32 字符的 ID,  第一条日志的时间戳, 最后一条日志的时间戳

-k, --dmesg
仅显示内核日志, 隐含了 `-b` 选项以及 "_TRANSPORT=kernel" 匹配项

-t, --identifier=SYSLOG_IDENTIFIER
仅显示 `syslog` 识别符为 `SYSLOG_IDENTIFIER` 的日志项

可以多次使用该选项以 指定多个识别符

-u, --unit=UNIT|PATTERN
仅显示 属于特定单元的日志, 也就是单元名称正好等于 UNIT 或者符合 PATTERN 模式的单元, 这相当于添加了一个 "_SYSTEMD_UNIT=UNIT" 匹配项(对于 UNIT 来说), 或一组匹配项(对于 PATTERN 来说)

可以多次使用此选项以添加多个并列的匹配条件(相当于用 "OR" 逻辑连接)

--user-unit=
仅显示 属于特定用户会话单元的日志, 相当于同时添加了 "_SYSTEMD_USER_UNIT=" 与 "_UID=" 两个匹配条件

可以多次使用此选项以添加多个并列的匹配条件(相当于用 "OR" 逻辑连接)

-p, --priority=
根据 日志等级(包括等级范围) 过滤输出结果, 日志等级数字与其名称之间的 对应关系如下 (参见 syslog(3)):  "emerg" (0), "alert" (1), "crit" (2), "err" (3), "warning" (4), "notice" (5), "info" (6), "debug" (7) , 若设为一个单独的数字或日志等级名称, 则表示仅显示小于或等于此等级的日志(也就是重要程度等于或高于此等级的日志), 若使用 FROM..TO.. 设置一个范围, 则表示仅显示指定的等级范围内(含两端)的日志, 此选项相当于添加了 "PRIORITY=" 匹配条件

-g, --grep=
使用指定的正则表达式对 MESSAGE= 字段进行过滤, 仅输出匹配的日志项, 必须使用 PERL 兼容的正则表达式, 详细语法参见 pcre2pattern(3) 手册

如果正则表达式中仅含小写字母, 那么将自动进行大小写无关的匹配, 否则将使用大小写敏感的匹配, 是否大小写敏感可以使用下面的 `--case-sensitive` 选项强制指定

--case-sensitive[=BOOLEAN]
是否对正则表达式进行大小写敏感的匹配

-c, --cursor=
从指定的游标 (cursor)开始显示日志, [提示]每条日志都有一个"__CURSOR"字段, 类似于该条日志的指纹

--after-cursor=
从指定的游标 (cursor) 之后开始显示日志, 如果使用了 `--show-cursor` 选项, 则也会显示游标本身

--show-cursor
在最后一条日志之后显示游标, 类似下面这样, 以"--"开头:

-- cursor: s=0639…
游标的具体格式是私有的(也就是没有公开的规范), 并且会变化

-S, --since=, -U, --until=
显示晚于指定时间(--since=)的日志, 显示早于指定时间(--until=)的日志, 参数的格式类似 "2012-10-30 18:17:16" 这样, 如果省略了"时:分:秒"部分, 则相当于设为 "00:00:00" , 如果仅省略了"秒"的部分则相当于设为 ":00" , 如果省略了"年-月-日"部分, 则相当于设为当前日期, 除了"年-月-日 时:分:秒"格式, 参数还可以进行如下设置:  (1)设为 "yesterday", "today", "tomorrow" 以表示那一天的零点(00:00:00), (2)设为 "now" 以表示当前时间, (3)可以在"年-月-日 时:分:秒"前加上 "-"(前移) 或 "+"(后移) 前缀以表示相对于当前时间的偏移, 关于时间与日期的详细规范, 参见 `systemd.time(7)` 注意, `--output=short-full` 将会一丝不苟的严格按照上述格式输出时间戳

-F, --field=
显示所有日志中指定字段的所有可能值, [译者注]类似于SQL语句: "SELECT DISTINCT 指定字段 FROM 全部日志"

-N, --fields
输出所有日志字段的名称

--system, --user
仅显示 系统服务与内核的日志 (--system),  仅显示当前用户的日志 (--user), 如果两个选项都未指定, 则显示当前用户的所有可见日志

-M, --machine=
显示来自于正在运行的, 特定名称的本地容器的日志, 参数必须是一个本地容器的名称

-D DIR, --directory=DIR
仅显示 特定目录中的日志, 而不是 默认的运行时和系统日志目录中的日志

--file=GLOB
GLOB 是一个 可以包含 "?" 与 "*" 的文件路径匹配模式, 表示仅显示来自与指定的 GLOB 模式匹配的文件中的日志, 而不是默认的运行时和系统日志目录中的日志, 可以多次使用此选项 以指定多个匹配模式(多个模式之间用 "OR" 逻辑连接)

--root=ROOT
在对日志进行操作时, 将 ROOT 视为系统的根目录, 例如 `--update-catalog` 将会创建 `ROOT/var/lib/systemd/catalog/database` 文件, 并且将会显示 `ROOT/run/journal` 或 `ROOT/var/log/journal` 目录中的日志

--header
此选项并不用于显示日志内容, 而是用于显示 日志文件内部的头信息(类似于元数据)

--disk-usage
此选项并不用于显示日志内容, 而是用于显示 所有日志文件(归档文件与活动文件)的磁盘占用总量

--vacuum-size=, --vacuum-time=, --vacuum-files=
这些选项并不用于显示日志内容, 而是用于清理过期的日志归档文件(并不清理活动的日志文件), 以释放磁盘空间, `--vacuum-size=` 可用于限制归档文件的最大磁盘使用量(可以使用 "K", "M", "G", "T" 后缀)； `--vacuum-time=` 可用于清除指定时间之前的归档(可以使用 "s", "m", "h", "days", "weeks", "months", "years" 后缀)； `--vacuum-files=` 可用于限制日志归档文件的最大数量, 注意, `--vacuum-size=` 对 -`-disk-usage` 的输出仅有间接效果, 因为 `--disk-usage` 输出的是归档日志与活动日志的总量, 同样, `--vacuum-files=` 也未必一定会减少日志文件的总数, 因为它同样仅作用于归档文件而不会删除活动的日志文件

此三个选项可以同时使用, 以同时从三个维度去限制归档文件, 若将某选项设为零, 则表示取消此选项的限制

此三个选项还可以和 `--rotate` 一起使用, 表示首先滚动所有日志归档, 然后再执行清理操作, 这样可以确保首先完成所有活动日志的归档, 进而使得清理操作变得非常高效

--list-catalog [128-bit-ID…]
简要列出日志分类信息, 其中包括对分类信息的简要描述

如果明确指定了分类ID(128-bit-ID), 那么仅显示指定的分类

--dump-catalog [128-bit-ID…]
详细列出 日志分类信息 (格式与 .catalog 文件相同)

如果明确指定了分类 ID(128-bit-ID), 那么仅显示指定的分类

--update-catalog
更新 日志分类索引二进制文件, 每当安装, 删除, 更新了分类文件, 都需要执行一次此动作

--setup-keys
此选项并不用于显示日志内容, 而是用于生成一个新的 FSS(Forward Secure Sealing) 密钥对, 此密钥对包含一个 "sealing key"  与一个 "verification key", "sealing key" 保存在本地日志目录中, 而 "verification key" 则必须保存在其他地方, 详见 journald.conf(5) 中的 `Seal=` 选项

--force
与 `--setup-keys` 连用, 表示即使已经配置了FSS(Forward Secure Sealing)密钥对, 也要强制重新生成

--interval=
与 `--setup-keys` 连用, 指定"sealing key"的变化间隔, 较短的时间间隔会导致占用更多的CPU资源, 但是能够减少未检测的日志变化时间, 默认值是 15min

--verify
检查日志文件的内在一致性, 如果日志文件在生成时开启了FSS特性, 并且使用 `--verify-key=` 指定了 FSS 的 "verification key", 那么, 同时还将验证日志文件的真实性

--verify-key=
与 `--verify` 选项连用, 指定 FSS 的 "verification key"

--sync
要求日志守护进程 将所有未写入磁盘的日志数据刷写到磁盘上, 并且一直阻塞到刷写操作实际完成之后才返回, 因此该命令可以保证当它返回的时候, 所有在调用此命令的时间点之前的日志, 已经全部安全的刷写到了磁盘中

--flush
要求日志守护进程 将 `/run/log/journal` 中的日志数据 刷写到 `/var/log/journal` 中 (如果持久存储设备当前可用的话), 此操作会一直阻塞到操作完成之后才会返回, 因此可以确保在该命令返回时, 数据转移确实已经完成, 注意, 此命令仅执行一个单独的, 一次性的转移动作, 若没有数据需要转移, 则此命令什么也不做, 并且也会返回一个 表示操作已正确完成的返回值

--rotate
要求日志守护进程滚动日志文件, 此命令会一直阻塞到滚动操作完成之后才会返回, 日志滚动可以确保所有活动的日志文件都被关闭, 并被重命名以完成归档, 同时, 新的空白日志文件将被创建, 并成为新的活动日志文件, 此选项可以与 `--vacuum-size=`, `--vacuum-time=`, `--vacuum-file=` 一起使用, 以提高日志清理的效率

-h, --help
显示简短的帮助信息并退出

--version
显示简短的版本信息并退出

--no-pager
不将程序的输出内容管道 (pipe) 给分页程序
```

## 引用

- [linux journalctl 命令](https://www.cnblogs.com/sparkdev/p/8795141.html)
- [journalctl 中文手册](http://www.jinbuguo.com/systemd/journalctl.html)
