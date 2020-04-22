---
title: Java 运行时内存
date: 2020-04-22 21:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '基础']
permalink: java-runtime-memory
---

## 新生代

JVM 新创建的对象 (除了大对象外) 会被存放在新生代, 默认占 `1/3` 堆内存空间. 由于 JVM 会频繁创建对象, 所以新生代会频繁触发 MinorGC 进行垃圾回收

- Eden 区: Java 新创建的对象首先会被存放在 Eden 区, 如果新创建的对象属于大对象, 则直接将其分配到老年代
  - 大对象的定义和具体的 JVM 版本、堆大小和垃圾回收策略有关, 一般为 `2KB - 128KB`, 可通过 `XX:PretenureSizeThreshold` 设置其大小
  - Eden 区的内存空间不足时会触发 MinorGC
- ServivorTo 区: 保留上一次 MinorGC 时的幸存者
- ServivorFrom 区: 将上一次 MinorGC 时的幸存者作为这一次 MinorGC 的被扫描者

新生代的 GC 过程叫作 MinorGC, 采用复制算法实现

- 把在 Eden 区和 ServivorFrom 区中存活的对象复制到 ServivorTo 区. 如果某对象的年龄达到老年代的标准 (对象晋升老年代的标准由 `XX:MaxTenuringThreshold` 设置, 默认为 `15`) , 则将其复制到老年代, 同时把这些对象的年龄加 `1`. 如果 ServivorTo 区的内存空间不够, 则也直接将其复制到老年代. 如果对象属于大对象 (大小为 `2KB - 128KB` 的对象属于大对象, 例如通过 `XX:PretenureSizeThreshold=2097152` 设置大对象为 `2MB`, `1024 × 1024 × 2Byte = 2097152Byte = 2MB`) , 则也直接将其复制到老年代
- 清空 Eden 区和 ServivorFrom 区中的对象
- 将 ServivorTo 区和 ServivorFrom 区互换, 原来的 ServivorTo 区成为下一次 GC 时的 ServivorFrom 区

## 老年代

老年代主要存放有长生命周期的对象和大对象. GC 过程称为 MajorGC, 在进行 MajorGC 前, JVM 会进行一次 MinorGC, 在 MinorGC 过后仍然出现老年代空间不足或无法找到足够大的连续空间分配给新创建的大对象时, 会触发 MajorGC, MajorGC 的标记清除算法容易产生内存碎片. 在老年代没有内存空间可分配时, 会抛出 `Out Of Memory` 异常

## 永久代

永久代主要存放 Class 和 Meta (元数据) Class 在类加载时被放入永久代, 内存会随着加载的 Class 文件的增加而增加, 在加载的 Class 文件过多时会抛出 `Out Of Memory` 异常, 在 Java 8 中永久代已经被元数据区 (也叫作元空间) 取代, 在 Java 8 中, JVM 将类的元数据放入本地内存 (Native Memory) 中, 将常量池和类的静态变量放入 Java 堆中, 这样 JVM 能够加载多少元数据信息由操作系统的实际可用内存空间决定
