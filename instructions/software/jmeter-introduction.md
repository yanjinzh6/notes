---
title: JMeter 使用
date: 2020-12-05 20:00:00
tags: 'JMeter'
categories:
  - ['使用说明', '软件']
permalink: jmeter-introduction
---

## 简介

JMeter 是一款常用的基于 Java 的多线程压力测试工具

<!-- more -->

## 线程组概念

Jmeter 自带的线程组控件 (Thread Group) 中有三个重要的属性, 分别是 Number of Threads, Ramp-Up Period, 和 Loop Count, 用于控制线程组的行为

这是一个在压力测试中比较重要的概念

Thread Group 三个属性 No. of Threads, Ramp-Up Period, 和 Loop Count 默认都为1, 表示仅有一个用户, 在整个测试过程中仅发起一次请求, 且请求会在1秒钟内发出

可以这么认为

Jmeter 将在 Ramp-Up Period 时间范围内, 启动 Number of Threads 个用户 (线程) , 并且使每个用户 (线程) 重复发出 Loop Count 次请求 (采样)

## 使用方法

### GUI 模式

暂无

### 命令行模式

通过 GUI 生成 jmx 配置文件, 使用命令行模式运行测试能够大大缩减所需要的系统资源

```sh
jmeter -n -t <testplan filename> -l <listener filename>
# eg.
jmeter -n -t testplan.jmx -l test.jtl
```

#### 参数

```sh
-h, –help -> prints usage information and exit
-n, –nongui -> run JMeter in nongui mode
-t, –testfile <argument> -> the jmeter test(.jmx) file to run
-l, –logfile <argument> -> the file to log samples to
-r, –runremote -> Start remote servers (as defined in remote_hosts)
-H, –proxyHost <argument> -> Set a proxy server for JMeter to use
-P, –proxyPort <argument> -> Set proxy server port for JMeter to use
```

- -h 帮助 : 打印出有用的信息并退出
- -n 非 GUI 模式 : 在非 GUI 模式下运行 JMeter
- -t 测试文件 : 要运行的 JMeter 测试脚本文件
- -l 日志文件 : 记录结果的文件
- -r 远程执行 : 在Jmter.properties文件中指定的所有远程服务器
- -H 代理主机 : 设置 JMeter 使用的代理主机
- -P 代理端口 : 设置 JMeter 使用的代理主机的端口号
