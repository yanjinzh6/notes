---
title: Java FutureTask 类和 Callable 接口
date: 2020-06-07 10:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '并发']
permalink: java-future-task-callball
---

## 简介

Java 在 1.5 版本之后提供了一种新的多线程的创建方式 FutureTask, 可以返回线程执行的结果, 其中最为重要的是 FutureTask 类和 Callable 接口

<!-- more -->

## Callable 接口

为了解决 Runnable 接口 run 方法没有返回值的问题，Java 定义了一个新的和 Runnable 类似的 Callable 接口。并且将其中的代表业务处理的方法命名为 call，call 方法带返回值

```java
// java.util.concurrent.Callable
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

与 Runnable 接口差别:

- call 方法带返回值, run 方法不带返回值
- Runnable 实例作为 Thread 线程实例的 target 来使用, Callable 实例作为 FutureTask 实例的 target 来使用

## Future 接口

Future 接口主要是对并发任务的执行及获取其结果的一些操作

- 判断并发任务是否执行完成
- 获取并发的任务完成后的结果
- 取消并发执行中的任务

```java
// java.util.concurrent.Future
public interface Future<V> {

    /**
     * Attempts to cancel execution of this task.  This attempt will
     * fail if the task has already completed, has already been cancelled,
     * or could not be cancelled for some other reason. If successful,
     * and this task has not started when {@code cancel} is called,
     * this task should never run.  If the task has already started,
     * then the {@code mayInterruptIfRunning} parameter determines
     * whether the thread executing this task should be interrupted in
     * an attempt to stop the task.
     *
     * <p>After this method returns, subsequent calls to {@link #isDone} will
     * always return {@code true}.  Subsequent calls to {@link #isCancelled}
     * will always return {@code true} if this method returned {@code true}.
     *
     * @param mayInterruptIfRunning {@code true} if the thread executing this
     * task should be interrupted; otherwise, in-progress tasks are allowed
     * to complete
     * @return {@code false} if the task could not be cancelled,
     * typically because it has already completed normally;
     * {@code true} otherwise
     */
    boolean cancel(boolean mayInterruptIfRunning);

    /**
     * Returns {@code true} if this task was cancelled before it completed
     * normally.
     *
     * @return {@code true} if this task was cancelled before it completed
     */
    boolean isCancelled();

    /**
     * Returns {@code true} if this task completed.
     *
     * Completion may be due to normal termination, an exception, or
     * cancellation -- in all of these cases, this method will return
     * {@code true}.
     *
     * @return {@code true} if this task completed
     */
    boolean isDone();

    /**
     * Waits if necessary for the computation to complete, and then
     * retrieves its result.
     *
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     */
    V get() throws InterruptedException, ExecutionException;

    /**
     * Waits if necessary for at most the given time for the computation
     * to complete, and then retrieves its result, if available.
     *
     * @param timeout the maximum time to wait
     * @param unit the time unit of the timeout argument
     * @return the computed result
     * @throws CancellationException if the computation was cancelled
     * @throws ExecutionException if the computation threw an
     * exception
     * @throws InterruptedException if the current thread was interrupted
     * while waiting
     * @throws TimeoutException if the wait timed out
     */
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

- V get()：获取并发任务执行的结果。注意，这个方法是阻塞性的。如果并发任务没有执行完成，调用此方法的线程会一直阻塞，直到并发任务执行完成
- V get（Long timeout，TimeUnit unit）：获取并发任务执行的结果。也是阻塞性的，但是会有阻塞的时间限制，如果阻塞时间超过设定的 timeout 时间，该方法将抛出异常
- booleanisDone()：获取并发任务的执行状态。如果任务执行结束，则返回 true
- booleanisCancelled()：获取并发任务的取消状态。如果任务完成前被取消，则返回 true
- boolean cancel（booleanmayInterruptRunning）：取消并发任务的执行

## FutureTask 类

FutureTask 类的构造函数的参数为 Callable 类型，实际上是对 Callable 类型的二次封装，可以执行 Callable 的 call 方法。FutureTask 类间接地继承了 Runnable 接口，从而可以作为 Thread 实例的 target 执行目标

构造函数

```java
// java.util.concurrent.FutureTask#FutureTask(java.util.concurrent.Callable<V>)
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```

执行过程:

- 创建 FutureTask 实例时将 Callable 实例赋值给成员 callable
- Thread 线程实例执行 FutureTask 的 run 方法
- FutureTask 的 run 方法执行成员 callable 的 call 方法, 并将结果保存到 outcome 属性
- 通过调用 FutureTask 的 get 方法获得结果

## 实现

```java
@Slf4j
public class FutureDemo {
    public static final int SLEEP = 1000;

    public static String getCurThreadName() {
        return Thread.currentThread().getName();
    }

    static class CallbaleA implements Callable<Boolean> {

        @Override
        public Boolean call() throws Exception {
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
            return true;
        }
    }

    static class CallbaleB implements Callable<Boolean> {

        @Override
        public Boolean call() throws Exception {
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
            return true;
        }
    }

    public static void main(String[] args) {
        CallbaleA callbaleA = new CallbaleA();
        CallbaleB callbaleB = new CallbaleB();

        FutureTask<Boolean> futureTaskA = new FutureTask<>(callbaleA);
        FutureTask<Boolean> futureTaskB = new FutureTask<>(callbaleB);

        Thread threadA = new Thread(futureTaskA, "I am Thread-A");
        Thread threadB = new Thread(futureTaskB, "I am Thread-B");

        Thread.currentThread().setName("I am MainThread");
        log.info("启动线程 A");
        threadA.start();
        log.info("启动线程 B");
        threadB.start();

        try {
            Boolean a = futureTaskA.get();
            log.info("线程 A 结果 {}", a);
            Boolean b = futureTaskB.get();
            log.info("线程 B 结果 {}", b);

            if (a && b) {
                log.info("动作 1");
                Thread.sleep(SLEEP);
                log.info("动作 2");
                Thread.sleep(SLEEP);
            } else if (!a && b) {
                log.info("执行线程 A 出错");
            } else if (!b && a) {
                log.info("执行线程 A 出错");
            } else {
                log.info("执行线程 A B 出错");
            }
        } catch (InterruptedException e) {
            e.printStackTrace();
            log.error(e.getLocalizedMessage());
        } catch (ExecutionException e) {
            e.printStackTrace();
            log.error(e.getLocalizedMessage());
        }
        log.info("运行结束");
    }
}
```

```sh
[I am MainThread] INFO reactor.simple.FutureDemo - 启动线程 A
[I am MainThread] INFO reactor.simple.FutureDemo - 启动线程 B
[I am Thread-A] INFO reactor.simple.FutureDemo - 执行线程 A 动作
[I am Thread-A] INFO reactor.simple.FutureDemo - 动作 A1
[I am Thread-B] INFO reactor.simple.FutureDemo - 执行线程 B 动作
[I am Thread-B] INFO reactor.simple.FutureDemo - 动作 B1
[I am Thread-A] INFO reactor.simple.FutureDemo - 动作 A2
[I am Thread-B] INFO reactor.simple.FutureDemo - 动作 B2
[I am MainThread] INFO reactor.simple.FutureDemo - 线程 A 结果 true
[I am MainThread] INFO reactor.simple.FutureDemo - 线程 B 结果 true
[I am MainThread] INFO reactor.simple.FutureDemo - 动作 1
[I am MainThread] INFO reactor.simple.FutureDemo - 动作 2
[I am MainThread] INFO reactor.simple.FutureDemo - 运行结束
```
