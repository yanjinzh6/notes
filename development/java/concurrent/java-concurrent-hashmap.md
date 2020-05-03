---
title: Java ConcurrentHashMap
date: 2020-05-01 13:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '并发']
permalink: java-concurrent-hashmap
---

## 简介

ConcurrentHashMap 和 HashMap 的实现方式类似, 不同的是它采用分段锁的思想支持并发操作, 所以是线程安全的

ConcurrentHashMap 在内部细分为若干个小的 HashMap, 叫作数据段 (Segment) , 在默认情况下, 一个 ConcurrentHashMap 被细分为 16 个数据段, 对每个数据段的数据都单独进行加锁操作, Segment 的个数为锁的并发度

ConcurrentHashMap 是由 Segment 数组和 HashEntry 数组组成的, Segment 继承了可重入锁 (ReentrantLock) , 它在 ConcurrentHashMap 里扮演锁的角色, HashEntry 则用于存储键值对数据

在每一个 ConcurrentHashMap 里都包含一个 Segment 数组, Segment 的结构和 HashMap 类似, 是数组和链表结构, 在每个 Segment 里都包含一个 HashEntry 数组, 每个 HashEntry 都是一个链表结构的数据, 每个 Segment 都守护一个 HashEntry 数组里的元素, 在对 HashEntry 数组的数据进行修改时, 必须首先获得它对应的 Segment 锁

在操作 ConcurrentHashMap 时, 如果需要在其中添加一个新的数据, 则并不是将整个 HashMap 加锁, 而是先根据 HashCode 查询该数据应该被存放在哪个段, 然后对该段加锁并完成 put 操作, 在多线程环境下, 如果多个线程同时进行 put 操作, 则只要加入的数据被存放在不同的段中, 在线程间就可以做到并行的线程安全
