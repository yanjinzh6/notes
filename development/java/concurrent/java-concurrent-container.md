---
title: Java 并发容器
date: 2020-01-17 17:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '并发']
permalink: java-concurrent-container
---

# 简介

由于并行程序与串行程序的不同特点，适用于串行程序的一些数据结构可能无法直接在并发环境下正常工作，这是因为这些数据结构不是线程安全的。这里简单介绍多线程环境下的数据结构，包括并发 List，并发 Set，并发 Map， 并发 Queue，并发 Deque

<!-- more -->

# 并发 List

Vector 或者 CopyOnWriteArrayList 是两个线程安全的List实现，ArrayList 不是线程安全的

> 在读多写少的高并发环境中，使用 CopyOnWriteArrayList 可以提高系统的性能，但是，在写多读少的场合，CopyOnWriteArrayList 的性能可能不如 Vector

## Vector

![Vector 依赖](/java-concurrent-container/vector)

```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

Vector 是一个矢量队列，它的依赖关系跟 ArrayList  是一致的，因此它具有一下功能：

- Serializable：支持对象实现序列化，虽然成员变量没有使用 transient 关键字修饰，Vector 还是实现了 writeObject() 方法进行序列化。
- Cloneable：重写了 clone（）方法，通过 Arrays.copyOf（） 拷贝数组。
- RandomAccess：提供了随机访问功能，我们可以通过元素的序号快速获取元素对象。
- AbstractList：继承了AbstractList ，说明它是一个列表，拥有相应的增，删，查，改等功能。
- List：留一个疑问，为什么继承了 AbstractList 还需要 实现List 接口？

### 序列化

`writeObject` 方法注释中 `This method performs synchronization to ensure the consistency of the serialized data.`

## CopyOnWriteArrayList

Copy-On-Write 就是 CopyOnWriteArrayList 的实现机制。即当对象进行写操作时，复制该对象；若进行的读操作，则直接返回结果，操作过程中不需要进行同步

CopyOnWriteArrayList 很好地利用了对象的不变性，在没有对对象进行写操作前，由于对象未发生改变，因此不需要加锁。而在试图改变对象时，总是先获取对象的一个副本，然后对副本进行修改，最后将副本写回

这种实现方式的核心思想是减少锁竞争，从而提高在高并发时的读取性能，但是它却在一定程度上牺牲了写的性能

## Collections.synchronizedList(List list)

应该尽量避免在多线程环境中使用 ArrayList。如果因为某些原因必须使用的，则需要使用 Collections.synchronizedList(List list) 进行包装

```java
List list = Collections.synchronizedList(new ArrayList());
// ...
synchronized (list) {
    // 必须在同步块中
    Iterator i = list.iterator();
    while (i.hasNext())
        foo(i.next());
}
```

# 并发 Set

# 并发 Map

# 并发 Queue

# 并发 Deque

# 参考

- [百密一疏之 Vector](https://juejin.im/post/5aedcc80518825673c61ccd6)
- [高并发下的 Java 数据结构 (List、Set、Map、Queue)](https://www.cnblogs.com/yueshutong/p/9696216.html#)