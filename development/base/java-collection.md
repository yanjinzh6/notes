---
title: 集合简介
date: 2019-12-21 19:28:54
tags: 'Java'
categories: '开发技术'
permalink: java-collection
---

# 集合

- List: 存储一组不唯一, 有序的对象, 可以有多个元素引用相同的对象.
- Set: 不允许重复的集合, 不会有多个元素引用相同的对象.
- Map: 使用键值对存储.两个 Key 可以引用相同的对象, 但 Key 不能重复.

<!-- more -->

## 集合框架底层数据结构总结

- List
  - Arraylist: 数组 (查询快,增删慢 线程不安全,效率高)
  - Vector: 数组 (查询快,增删慢 线程安全,效率低)
  - LinkedList: 链表 (查询慢,增删快 线程不安全,效率高)
- Set
  - HashSet (无序, 唯一) :哈希表或者叫散列集(hash table)
  - LinkedHashSet: 链表和哈希表组成 . 由链表保证元素的排序 , 由哈希表证元素的唯一性
  - TreeSet (有序, 唯一) : 红黑树 (自平衡的排序二叉树.)
- Map
  - HashMap: 基于哈希表的 Map 接口实现 (哈希表对键进行散列, Map 结构即映射表存放键值对)
  - LinkedHashMap:HashMap 的基础上加上了链表数据结构
  - HashTable:哈希表
  - TreeMap:红黑树 (自平衡的排序二叉树)

## 参考书籍

《Head first java 》第二版 推荐阅读真心不错 (适合基础较差的)

《Java 核心技术卷 1》推荐阅读真心不错 (适合基础较好的)

《算法》第四版 (适合想对数据结构的 Java 实现感兴趣的)

## Arraylist 与 LinkedList 区别

两者都是线程不安全的

Arraylist 底层使用的是数组 (存读数据效率高, 插入删除特定位置效率低) , LinkedList 底层使用的是双向链表数据结构 (插入, 删除效率特别高) (JDK1.6 之前为循环链表, JDK1.7 取消了循环. 注意双向链表和双向循环链表的区别. 采用链表存储, 插入, 删除元素时间复杂度不受元素位置的影响, 都是近似 O (1) 而数组为近似 O (n) , 因此当数据特别多, 而且经常需要插入删除元素时建议选用 LinkedList. 一般数据量都不会蛮大, 用 Arraylist.

## ArrayList 与 Vector 区别

Vector 类的所有方法都是同步的

## HashMap 和 HashTable 的区别

1. HashMap 是非线程安全的, HashTable 是线程安全的；HashTable 内部的方法基本都经过 synchronized 修饰.
2. 因为线程安全的问题, HashMap 要比 HashTable 效率高一点, HashTable 基本被淘汰.
3. HashMap 允许有 null 值的存在, 而在 HashTable 中 put 进的键值只要有一个 null, 直接抛出 NullPointerException.

使用 ConcurrentHashMap 代替 HashTable

## HashMap 和 ConcurrentHashMap 的区别

- ConcurrentHashMap 把 Map 分成了 N 个 Segment, put 和 get 的时候, 都是现根据 key.hashCode()算出放到哪个 Segment 中, 然后在每一个分段上都用 lock 锁进行保护, 相对于 HashTable 的 synchronized 锁的粒度更精细了一些, 并发性能更好, 默认分成 16 段. 而 HashMap 没有锁机制, 不是线程安全的.(JDK1.8 之后 ConcurrentHashMap 启用了一种全新的方式实现,利用 CAS 算法)
- HashMap 的键值对允许有 null, ConCurrentHashMap 不允许.

## 检查重复

当你把对象加入 HashSet 时, HashSet 会先计算对象的 hashcode 值来判断对象加入的位置, 同时也会与其他加入的对象的 hashcode 值作比较, 如果没有相符的 hashcode, HashSet 会假设对象没有重复出现. 但是如果发现有相同 hashcode 值的对象, 这时会调用 equals () 方法来检查 hashcode 相等的对象是否真的相同. 如果两者相同, HashSet 就不会让加入操作成功. (摘自《Head fist java》第二版)

### hashCode () 与 equals () 的相关规定:

1. 如果两个对象相等, 则 hashcode 一定也是相同的
2. 两个对象相等,对两个 equals 方法返回 true
3. 两个对象有相同的 hashcode 值, 它们也不一定是相等的
4. equals 方法被覆盖过, 则 hashCode 方法也必须被覆盖
5. hashCode()的默认行为是对堆上的对象产生独特值. 如果没有重写 hashCode(), 则该 class 的两个对象无论如何都不会相等 (即使这两个对象指向相同的数据)

### ==与 equals 的区别

1. ==是判断两个变量或实例是不是指向同一个内存空间 equals 是判断两个变量或实例所指向的内存空间的值是不是相同
2. ==是指对内存地址进行比较 equals()是对字符串的内容进行比较
3. ==指引用是否相同 equals()指的是值是否相同
