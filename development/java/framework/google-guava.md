---
title: Google guava 异步回调框架
date: 2020-06-13 10:00:00
tags: 'guava'
categories:
  - ['开发', 'Java', ' 框架']
permalink: google-guava
---

## 简介

Guava 包 `com.google.common.util.concurrent` 主要对 `java.util.concurrent` 能力的扩展和增强

- 引入了一个新的接口 ListenableFuture, 继承了 Java 的 Future 接口, 使得 Java 的 Future 异步任务, 在 Guava 中能被监控和获得非阻塞异步执行的结果
- 引入了一个新的接口 FutureCallback, 这是一个独立的新接口, 该接口的目的, 是在异步任务执行完成后, 根据异步结果, 完成不同的回调处理, 并且可以处理异步结果

<!-- more -->

## FutureCallback 接口

FutureCallback 用来定义异步任务执行完后的监听逻辑

- onSuccess 方法, 在异步任务执行成功后被回调；调用时, 异步任务的执行结果, 作为 onSuccess 方法的参数被传入
- onFailure 方法, 在异步任务执行过程中, 抛出异常时被回调；调用时, 异步任务所抛出的异常, 作为 onFailure 方法的参数被传入

```java
// com.google.common.util.concurrent.FutureCallback
@GwtCompatible
public interface FutureCallback<V> {
    void onSuccess(@Nullable V var1);

    void onFailure(Throwable var1);
}
```

与 Callable 接口的区别

- Java 的 Callable 接口, 代表的是异步执行的逻辑
- Guava 的 FutureCallback 接口, 代表的是 Callable 异步逻辑执行完成之后, 根据成功或者异常两种情况, 所需要执行的善后工作

Guava 是对 Java Future 异步回调的增强, 使用 Guava 异步回调, 也需要用到 Java 的 Callable 接口, 简单地说, 只有在 Java 的 Callable 任务执行的结果出来之后, 才可能执行 Guava 中的 FutureCallback 结果回调

## ListenableFuture 接口

Guava 的 ListenableFuture 接口是对 Java 的 Future 接口的扩展

```java
// com.google.common.util.concurrent.ListenableFuture
public interface ListenableFuture<V> extends Future<V> {
    void addListener(Runnable var1, Executor var2);
}
```

addListener 方法: 将 FutureCallback 善后回调工作, 封装成一个内部的 Runnable 异步回调任务, 在 Callable 异步任务完成后, 回调 FutureCallback 进行善后处理, 方法只在 Guava 内部调用

使用 Guava 的 Futures 工具类 addCallback 静态方法将 FutureCallback 的回调实例绑定到 ListenableFuture 异步任务

## 线程池

Guava 线程池, 是对 Java 线程池的一种装饰, 创建 Guava 线程池步骤

- 创建 Java 线程池
- 以 Java 线程池作为 Guava 线程池的参数, 构造一个 Guava 线程池
- 通过 submit 方法提交任务
- 获取 ListenableFuture 异步任务实例
- 通过 Futures.addCallback 方法, 将 FutureCallback 回调逻辑的实例绑定到 ListenableFuture 异步任务实例, 实现异步执行完成后的回调

```java
// java 线程池
ExecutorService jPool = Executors.newFixedThreadPool(10);
// Guava 线程池
ListeningExecutorService gPool = MoreExecutors.listeningDecorator(jPool);
//调用 submit 方法来提交任务, 返回异步任务实例
ListenableFuture<Boolean> listenableFuture = gPool.submit(hJob);
// 绑定回调实例
Futures.addCallback(listenableFuture, newFutureCallback<Boolean>()
{
    // 实现回调方法
    // onSuccess
    // onFailure
});
```

## Guava 异步回调的流程

- 实现 Java 的 Callable 接口, 创建异步执行逻辑, 还有一种情况, 如果不需要返回值, 异步执行逻辑也可以实现 Java 的 Runnable 接口
- 创建 Guava 线程池
- 将第 1 步创建的 Callable/Runnable 异步执行逻辑的实例, 通过 submit 提交到 Guava 线程池, 从而获取 ListenableFuture 异步任务实例
- 创建 FutureCallback 回调实例, 通过 Futures.addCallback 将回调实例绑定到 ListenableFuture 异步任务上
- Callable/Runnable 异步执行逻辑完成后, 就会回调异步回调实例 FutureCallback 的回调方法 onSuccess/onFailure

## 实现

```java
@Slf4j
public class GuavaDemo {
    public static final int SLEEP = 1000;

    public static String getCurThreadName() {
        return Thread.currentThread().getName();
    }

    static class JobA implements Callable<Boolean> //①
    {

        @Override
        public Boolean call() throws Exception //②
        {

            try {
                log.info("执行线程 A 动作");
                log.info("动作 A1");
                Thread.sleep(SLEEP);
                log.info("动作 A2");
                Thread.sleep(SLEEP);
            } catch (InterruptedException e) {
                e.printStackTrace();
                log.error(e.getLocalizedMessage());
                return false;
            }
            log.info("线程 A 运行结束.");

            return true;
        }
    }

    static class JobB implements Callable<Boolean> //①
    {

        @Override
        public Boolean call() throws Exception //②
        {

            try {
                log.info("执行线程 B 动作");
                log.info("动作 B1");
                Thread.sleep(SLEEP);
                log.info("动作 B2");
                Thread.sleep(SLEEP);
            } catch (InterruptedException e) {
                e.printStackTrace();
                log.error(e.getLocalizedMessage());
                return false;
            }
            log.info("线程 B 运行结束.");

            return true;
        }
    }

    static class MainJob implements Runnable {

        boolean aBoolean = false;
        boolean bBoolean = false;

        @Override
        public void run() {
            while (!doMain(aBoolean, bBoolean)) {
                try {
                    log.info("等待中......");
                    Thread.sleep(SLEEP);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    log.error(e.getLocalizedMessage());
                }
            }
        }


        public boolean doMain(Boolean aOk, Boolean bOk) {
            if (aOk && bOk) {
                log.info("完成主线程");
                return true;
            } else if (!aOk) {
                log.info("线程 A 未就绪");
            } else if (!bOk) {
                log.info("线程 B 未就绪");
            }

            return false;
        }
    }

    public static void main(String args[]) {

        //新起一个线程，作为泡茶主线程
        MainJob mainJob = new MainJob();
        Thread mainThread = new Thread(mainJob);
        mainThread.setName("主线程");
        mainThread.start();

        // 业务逻辑任务
        Callable<Boolean> jobA = new JobA();
        // 业务逻辑任务
        Callable<Boolean> jobB = new JobB();

        // 创建java 线程池
        ExecutorService jPool =
                Executors.newFixedThreadPool(10);

        // 包装 java 线程池, 构造 guava 线程池
        ListeningExecutorService gPool =
                MoreExecutors.listeningDecorator(jPool);

        // 提交线程 A 业务逻辑, 取到异步任务
        ListenableFuture<Boolean> futureA = gPool.submit(jobA);
        // 绑定任务执行完成后的回调到异步任务
        Futures.addCallback(futureA, new FutureCallback<Boolean>() {
            @Override
            public void onSuccess(Boolean r) {
                log.info("线程 A onSuccess 调用值 {}", r);
                if (r) {
                    log.info("完成线程 A 操作");
                    mainJob.aBoolean = true;
                }
            }

            @Override
            public void onFailure(Throwable t) {
                log.info("线程 A 失败");
            }
        }, gPool);

        // 提交清洗的业务逻辑, 取到异步任务
        ListenableFuture<Boolean> futureB = gPool.submit(jobB);
        // 绑定任务执行完成后的回调到异步任务
        Futures.addCallback(futureB, new FutureCallback<Boolean>() {
            @Override
            public void onSuccess(Boolean r) {
                log.info("线程 B onSuccess 调用值 {}", r);
                if (r) {
                    log.info("完成线程 B 操作");
                    mainJob.bBoolean = true;
                }
            }

            @Override
            public void onFailure(Throwable t) {
                log.info("线程 B 失败");
            }
        }, gPool);
    }

}
```

```sh
[主线程] INFO reactor.simple.GuavaDemo - 线程 A 未就绪
[主线程] INFO reactor.simple.GuavaDemo - 等待中......
[pool-1-thread-1] INFO reactor.simple.GuavaDemo - 执行线程 A 动作
[pool-1-thread-1] INFO reactor.simple.GuavaDemo - 动作 A1
[pool-1-thread-2] INFO reactor.simple.GuavaDemo - 执行线程 B 动作
[pool-1-thread-2] INFO reactor.simple.GuavaDemo - 动作 B1
[主线程] INFO reactor.simple.GuavaDemo - 线程 A 未就绪
[主线程] INFO reactor.simple.GuavaDemo - 等待中......
[pool-1-thread-1] INFO reactor.simple.GuavaDemo - 动作 A2
[pool-1-thread-2] INFO reactor.simple.GuavaDemo - 动作 B2
[主线程] INFO reactor.simple.GuavaDemo - 线程 A 未就绪
[主线程] INFO reactor.simple.GuavaDemo - 等待中......
[pool-1-thread-1] INFO reactor.simple.GuavaDemo - 线程 A 运行结束.
[pool-1-thread-3] INFO reactor.simple.GuavaDemo - 线程 A onSuccess 调用值 true
[pool-1-thread-3] INFO reactor.simple.GuavaDemo - 完成线程 A 操作
[pool-1-thread-2] INFO reactor.simple.GuavaDemo - 线程 B 运行结束.
[pool-1-thread-4] INFO reactor.simple.GuavaDemo - 线程 B onSuccess 调用值 true
[pool-1-thread-4] INFO reactor.simple.GuavaDemo - 完成线程 B 操作
[主线程] INFO reactor.simple.GuavaDemo - 完成主线程
```

## 小结

通过主线程的不断轮询可以实现获取回调状态, 但是最好是使用阻塞队列来实现
