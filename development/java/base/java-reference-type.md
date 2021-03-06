---
title: Java 引用类型
date: 2020-04-22 22:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '基础']
permalink: java-reference-type
---

## 为什么是引用

Java 是面向对象语言, 对象的操作是通过该对象的引用（Reference）实现的

## 4 种引用类型

- 强引用：最普遍存在。在把一个对象赋给一个引用变量时，这个引用变量就是一个强引用。有强引用的对象一定为可达性状态，所以不会被垃圾回收机制回收。因此，强引用是造成 Java 内存泄漏（Memory Link）的主要原因
- 软引用：软引用通过 SoftReference 类实现。如果一个对象只有软引用，则在系统内存空间不足时该对象将被回收
- 弱引用：弱引用通过 WeakReference 类实现，如果一个对象只有弱引用，则在垃圾回收过程中一定会被回收
- 虚引用：虚引用通过 PhantomReference 类实现，虚引用和引用队列联合使用，主要用于跟踪对象的垃圾回收状态
