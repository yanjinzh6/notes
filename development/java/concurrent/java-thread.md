---
title: Java 线程
date: 2020-05-01 9:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '并发']
permalink: java-thread
---

## 简介

常见的 Java 线程的 4 种创建方式

- 继承 Thread 类: 创建一个类并继承 Thread 接口，然后实例化线程对象并调用 start 方法启动线程。start 方法是一个 native 方法，通过在操作系统上启动一个新线程，并最终执行 run 方法来启动一个线程。run 方法内的代码是线程类的具体实现逻辑
- 实现 Runnable 接口: 将一个实现了 Runnable 的线程实例 target 给 Thread 后，Thread 的 run 方法在执行时就会调用 target.run 方法并执行该线程具体的实现逻辑
- 通过 ExecutorService 和 `Callable<Class>` 实现有返回值的线程: 创建一个类并实现 Callable 接口，在 call 方法中实现具体的运算逻辑并返回计算结果。具体的调用过程为：创建一个线程池、一个用于接收返回结果的 Future List 及 Callable 线程实例，使用线程池提交任务并将线程执行之后的结果保存在 Future 中，在线程执行结束后遍历 Future List 中的 Future 对象，在该对象上调用 get 方法就可以获取 Callable 线程任务返回的数据并汇总结果
- 基于线程池
