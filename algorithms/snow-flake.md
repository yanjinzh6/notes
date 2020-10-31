---
title: 雪花算法
date: 2020-10-25 15:00:00
tags: '算法'
categories:
  - ['算法', '哈希']
permalink: snow-flake
photo:
mathjax: false
---

## 简介

主要定义三个区域, 时间戳 + 数据中心编号 + 序列编号

<!-- more -->

## 实现

```java
/**
  * 开始时间
  * Wed Jan 01 2020 08:00:00 GMT+0800 (中国标准时间)
  */
private final static long START_TIME = 1577836800000L;
/**
  * 数据中心编号所占位数
  */
private final static long DATA_CENTER_BITS = 10L;
/**
  * 最大数据中心编号
  *
  * 2^10 = 1024
  */
private final static long MAX_DATA_CENTER_ID = 1023;
/**
  * 序列编号占位位数
  */
private final static long SEQUENCE_BIT = 12L;
/**
  * 数据中心编号向左移 12 位
  */
private final static long DATA_CENTER_SHIFT = SEQUENCE_BIT;
/**
  * 时间戳向左移 22 位 (10+12)
  */
private final static long TIMESTAMP_SHIFT
        = DATA_CENTER_BITS + DATA_CENTER_SHIFT;
/**
  * 最大生成序列号
  *
  * 2^12 = 4096
  */
private final static long MAX_SEQUENCE = 4095;
/**
  * 数据中心 ID (0~1023)
  */
private long dataCenterId;
/**
  * 毫秒内序列 (0~4095)
  */
private long sequence = 0L;
/**
  * 上次生成 ID 的时间戳
  */
private long lastTimestamp = -1L;

/**
  * 因为当前微服务和分布式趋向于去中心化，所以不存在受理机器编号，
  * 10 位二进制全部用于数据中心
  *
  * @param dataCenterId 数据中心 ID [0~1023]
  */
public Test(long dataCenterId) {
    // 验证数据中心编号的合法性
    if (dataCenterId > MAX_DATA_CENTER_ID) {
        String msg = "数据中心编号[" + dataCenterId
                + "]超过最大允许值[" + MAX_DATA_CENTER_ID + "]";
        throw new RuntimeException(msg);
    }
    if (dataCenterId < 0) {
        String msg = "数据中心编号[" + dataCenterId + "]不允许小于 0";
        throw new RuntimeException(msg);
    }
    this.dataCenterId = dataCenterId;
}

/**
  * 获得下一个 ID (为了避免多线程环境产生的错误，这里方法是线程安全的)
  *
  * @return ID
  */
public synchronized long nextId() {
    // 获取当前时间
    long timestamp = System.currentTimeMillis();
    // 如果是同一个毫秒时间戳的处理
    if (timestamp == lastTimestamp) {
        // 序号 +1
        sequence += 1;
        // 是否超过允许的最大序列
        if (sequence > MAX_SEQUENCE) {
            sequence = 0;
            // 等待到下一毫秒
            timestamp = tilNextMillis(timestamp);
        }
    } else {
        // 修改时间戳
        lastTimestamp = timestamp;
        // 序号重新开始
        sequence = 0;
    }
    // 二进制的位运算，其中 "<<" 代表二进制左移，"|" 代表或运算
    return ((timestamp - START_TIME) << TIMESTAMP_SHIFT)
            | (this.dataCenterId << DATA_CENTER_SHIFT)
            | sequence;
}

/**
  * 阻塞到下一毫秒，直到获得新的时间戳
  *
  * @param lastTimestamp 上次生成 ID 的时间戳
  * @return 当前时间戳
  */
protected long tilNextMillis(long lastTimestamp) {
    long timestamp;
    do {
        timestamp = System.currentTimeMillis();
    } while (timestamp > lastTimestamp);
    return timestamp;
}
```

这里通过实例保存一个时间戳和当前序号, 只要当前时间与时间戳一致, 就增加序号, 直到序号超过 4095 便自旋进入下一秒, 序号也重置为 0

然后将三部分数据通过二进制运算返回

## 应用

实际测试中, 普通的机器毫秒级的运算只有几百, 也就是 [0, 4095] 序列中只使用了一部分而已, 所以可以通过减少位数来降低计算量, 或者通过压缩位数为新的业务数据添加字段

如果发号和数据存储是分开的, 那就可以将 10 位的数据中心和 12 位序列号设置为 8 位数据中心和 8 为序列号, 然后剩余的 6 位可以作为发号机器的标识, 这样每个发号机器可以为每个数据中心提供每秒生成最多 512 000 个 ID, 这是符合当前业务的
