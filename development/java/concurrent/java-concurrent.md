---
title: Java 并发关键字
date: 2020-05-01 11:30:00
tags: 'Java'
categories:
  - ['开发', 'Java', '并发']
permalink: java-concurrent
---

## CountDownLatch

CountDownLatch 类位于 `java.util.concurrent` 包下, 是一个同步工具类, 允许一个或多个线程一直等待其他线程的操作执行完后再执行相关操作

CountDownLatch 基于线程计数器来实现并发访问控制, 主要用于主线程等待其他子线程都执行完毕后执行相关操作, 其使用过程为：在主线程中定义 CountDownLatch, 并将线程计数器的初始值设置为子线程的个数, 多个子线程并发执行, 每个子线程在执行完毕后都会调用 countDown 函数将计数器的值减 1, 直到线程计数器为 0, 表示所有的子线程任务都已执行完毕, 此时在 CountDownLatch 上等待的主线程将被唤醒并继续执行

主线程调用 await 方法阻塞等待, 在所有线程都执行完成并调用 countDown 方法后,会主动唤醒主线程并开始执行主线程的业务逻辑

<!-- more -->

## CyclicBarrier

CyclicBarrier (循环屏障) 是一个同步工具, 可以实现让一组线程等待至某个状态之后再全部同时执行, 在所有等待线程都被释放之后, CyclicBarrier 可以被重用, CyclicBarrier 的运行状态叫作 Barrier 状态, 在调用 await 方法后, 线程就处于 Barrier 状态

CyclicBarrier 中最重要的方法是 await 方法, 它有两种实现

- `public int await()`：挂起当前线程直到所有线程都为 Barrier 状态再同时执行后续的任务
- `public int await(long timeout, TimeUnit unit)` ：设置一个超时时间, 在超时时间过后, 如果还有线程未达到 Barrier 状态, 则不再等待, 让达到 Barrier 状态的线程继续执行后续的任务

CyclicBarrier 主要作用是让一组线程等待到达某个状态再一起执行, 执行过程中, 所有线程实例都持有 CyclicBarrier 实例, 并且在执行线程逻辑后调用 `cyclicBarrier.await` 方法等待其他线程也完成后, 继续执行主线程逻辑

## Semaphore

Semaphore 指信号量, 用于控制同时访问某些资源的线程个数, 具体做法为通过调用 `acquire()` 获取一个许可, 如果没有许可, 则等待, 在许可使用完毕后通过 `release()` 释放该许可, 以便其他线程使用, 常被用于多个线程需要共享有限资源的情况

Semaphore 对锁的申请和释放和 ReentrantLock 类似, 通过 acquire 方法和 release 方法来获取和释放许可信号资源, `Semaphone.acquire` 方法默认和 `ReentrantLock.lockInterruptibly` 方法的效果一样, 为可响应中断锁, 也就是说在等待许可信号资源的过程中可以被 `Thread.interrupt` 方法中断而取消对许可信号的申请, Semaphore 也实现了可轮询的锁请求, 定时锁的功能, 以及公平锁与非公平锁的机制, 对公平与非公平锁的定义在构造函数中设定

Semaphore 可以用于实现一些对象池, 资源池的构建, 比如静态全局对象池, 数据库连接池等

- `public void acquire()`：以阻塞的方式获取一个许可, 在有可用许可时返回该许可, 在没有可用许可时阻塞等待, 直到获得许可
- `public void acquire(int permits)` ：同时获取 permits 个许可
- `public void release()`：释放某个许可
- `public void release(int permits)` ：释放 permits 个许可
- `public boolean tryAcquire()`：以非阻塞方式获取一个许可, 在有可用许可时获取该许可并返回 `true`, 否则返回 `false`, 不会等待
- `public boolean tryAcquire(long timeout,TimeUnit unit)` ：如果在指定的时间内获取到可用许可, 则返回 `true`, 否则返回 `false`
- `public boolean tryAcquire(int permits)` ：如果成功获取 permits 个许可, 则返回 `true`, 否则立即返回 `false`
- `public boolean tryAcquire(int permits,long timeout,TimeUnit unit)` ：如果在指定的时间内成功获取 permits 个许可, 则返回 `true`, 否则返回 `false`
- `availablePermits()`：查询可用的许可数量

## volatile

volatile 变量具备两种特性

- 保证该变量对所有线程可见, 在一个线程修改了变量的值后, 新的值对于其他线程是可以立即获取的
- 禁止指令重排, 即 volatile 变量不会被缓存在寄存器中或者对其他处理器不可见的地方, 因此在读取 volatile 类型的变量时总会返回最新写入的值

因为在访问 volatile 变量时不会执行加锁操作, 也就不会执行线程阻塞, 因此 volatile 变量是一种比 synchronized 关键字更轻量级的同步机制, volatile 主要适用于一个变量被多个线程共享, 多个线程均可针对这个变量执行赋值或者读取的操作

在有多个线程对普通变量进行读写时, 每个线程都首先需要将数据从内存中复制变量到 CPU 缓存中, 如果计算机有多个 CPU, 则线程可能都在不同的 CPU 中被处理, 这意味着每个线程都需要将同一个数据复制到不同的 CPU Cache 中, 这样在每个线程都针对同一个变量的数据做了不同的处理后就可能存在数据不一致的情况, 如果将变量声明为 volatile, JVM 就能保证每次读取变量时都直接从内存中读取, 跳过 CPU Cache 这一步, 有效解决了多线程数据同步的问题

volatile 关键字可以严格保障变量的单次读, 写操作的原子性, 但并不能保证像 i++ 这种操作的原子性, 因为 i++ 在本质上是读, 写两次操作, volatile 在某些场景下可以代替 synchronized, 但是 volatile 不能完全取代 synchronized 的位置, 只有在一些特殊场景下才适合使用 volatile, 必须同时满足下面两个条件才能保证并发环境的线程安全

- 对变量的写操作不依赖于当前值 (比如 `i++`) , 或者说是单纯的变量赋值 (`boolean flag=true`)
- 该变量没有被包含在具有其他变量的不变式中, 也就是说在不同的 volatile 变量之间不能互相依赖, 只有在状态真正独立于程序内的其他内容时才能使用 volatile

## 小结

CountDownLatch, CyclicBarrier, Semaphore 区别

- CountDownLatch 和 CyclicBarrier 都用于实现多线程之间的相互等待, 但二者的关注点不同, CountDownLatch 主要用于主线程等待其他子线程任务均执行完毕后再执行接下来的业务逻辑单元, 而 CyclicBarrier 主要用于一组线程互相等待大家都达到某个状态后, 再同时执行接下来的业务逻辑单元, 此外, CountDownLatch 是不可以重用的, 而 CyclicBarrier 是可以重用的
- Semaphore 和 Java 中的锁功能类似, 主要用于控制资源的并发访问
