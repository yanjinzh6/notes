---
title: java-thread-local
date: 2021-01-31 22:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '并发']
permalink: java-thread-local
---

https://www.liaoxuefeng.com/wiki/1252599548343744/1306581251653666

#### 使用 ThreadLocal

阅读: 702731 [编辑](/manage/wiki/wikipage_update? id=1306581251653666)

* * *

多线程是 Java 实现多任务的基础,`Thread` 对象代表一个线程, 我们可以在代码中调用 `Thread.currentThread()` 获取当前线程. 例如, 打印日志时, 可以同时打印出当前线程的名字:

```
// Thread
----
public class Main {
    public static void main(String[] args) throws Exception {
        log("start main...");
        new Thread(() -> {
            log("run task...");
        }).start();
        new Thread(() -> {
            log("print...");
        }).start();
        log("end main.");
    }

    static void log(String s) {
        System.out.println(Thread.currentThread().getName() + ": " + s);
    }
}
```

对于多任务, Java 标准库提供的线程池可以方便地执行这些任务, 同时复用线程.Web 应用程序就是典型的多任务应用, 每个用户请求页面时, 我们都会创建一个任务, 类似:

```
public void process(User user) {
    checkPermission();
    doWork();
    saveStatus();
    sendResponse();
}
```

然后, 通过线程池去执行这些任务.

观察 `process()` 方法, 它内部需要调用若干其他方法, 同时, 我们遇到一个问题: 如何在一个线程内传递状态?

`process()` 方法需要传递的状态就是 `User` 实例. 有的童鞋会想, 简单地传入 `User` 就可以了:

```
public void process(User user) {
    checkPermission(user);
    doWork(user);
    saveStatus(user);
    sendResponse(user);
}
```

但是往往一个方法又会调用其他很多方法, 这样会导致 `User` 传递到所有地方:

```
void doWork(User user) {
    queryStatus(user);
    checkStatus();
    setNewStatus(user);
    log();
}
```

这种在一个线程中, 横跨若干方法调用, 需要传递的对象, 我们通常称之为上下文 (Context), 它是一种状态, 可以是用户身份, 任务信息等.

给每个方法增加一个 context 参数非常麻烦, 而且有些时候, 如果调用链有无法修改源码的第三方库,`User` 对象就传不进去了.

Java 标准库提供了一个特殊的 `ThreadLocal`, 它可以在一个线程中传递同一个对象.

`ThreadLocal` 实例通常总是以静态字段初始化如下:

```
static ThreadLocal<User> threadLocalUser = new ThreadLocal<> ();
```

它的典型使用方式如下:

```
void processUser(user) {
    try {
        threadLocalUser.set(user);
        step1();
        step2();
    } finally {
        threadLocalUser.remove();
    }
}
```

通过设置一个 `User` 实例关联到 `ThreadLocal` 中, 在移除之前, 所有方法都可以随时获取到该 `User` 实例:

```
void step1() {
    User u = threadLocalUser.get();
    log();
    printUser();
}

void log() {
    User u = threadLocalUser.get();
    println(u.name);
}

void step2() {
    User u = threadLocalUser.get();
    checkUser(u.id);
}
```

注意到普通的方法调用一定是同一个线程执行的, 所以,`step1()`,`step2()` 以及 `log()` 方法内,`threadLocalUser.get()` 获取的 `User` 对象是同一个实例.

实际上, 可以把 `ThreadLocal` 看成一个全局 `Map<Thread, Object>`: 每个线程获取 `ThreadLocal` 变量时, 总是使用 `Thread` 自身作为 key:

```
Object threadLocalValue = threadLocalMap.get(Thread.currentThread());
```

因此,`ThreadLocal` 相当于给每个线程都开辟了一个独立的存储空间, 各个线程的 `ThreadLocal` 关联的实例互不干扰.

最后, 特别注意 `ThreadLocal` 一定要在 `finally` 中清除:

```
try {
    threadLocalUser.set(user);
    ...
} finally {
    threadLocalUser.remove();
}
```

这是因为当前线程执行完相关代码后, 很可能会被重新放入线程池中, 如果 `ThreadLocal` 没有被清除, 该线程执行其他代码时, 会把上一次的状态带进去.

为了保证能释放 `ThreadLocal` 关联的实例, 我们可以通过 `AutoCloseable` 接口配合 `try (resource) {...}` 结构, 让编译器自动为我们关闭. 例如, 一个保存了当前用户名的 `ThreadLocal` 可以封装为一个 `UserContext` 对象:

```
public class UserContext implements AutoCloseable {

    static final ThreadLocal<String> ctx = new ThreadLocal<> ();

    public UserContext(String user) {
        ctx.set(user);
    }

    public static String currentUser() {
        return ctx.get();
    }

    @Override
    public void close() {
        ctx.remove();
    }
}
```

使用的时候, 我们借助 `try (resource) {...}` 结构, 可以这么写:

```
try (var ctx = new UserContext("Bob")) {
    // 可任意调用 UserContext.currentUser():
    String currentUser = UserContext.currentUser();
} // 在此自动调用 UserContext.close() 方法释放 ThreadLocal 关联对象
```

这样就在 `UserContext` 中完全封装了 `ThreadLocal`, 外部代码在 `try (resource) {...}` 内部可以随时调用 `UserContext.currentUser()` 获取当前线程绑定的用户名.

### 练习

[ThreadLocal 练习](https://gitee.com/liaoxuefeng/learn-java/raw/master/practices/Java%E6%95%99%E7%A8%8B/130.%E5%A4%9A%E7%BA%BF%E7%A8%8B.1255943750561472/200.%E4%BD%BF%E7%94%A8ThreadLocal.1306581251653666/thread-threadlocal.zip)

### 小结

`ThreadLocal` 表示线程的 " 局部变量 ", 它确保每个线程的 `ThreadLocal` 变量都是各自独立的;

`ThreadLocal` 适合在一个线程的处理流程中保持上下文 (避免了同一参数在所有方法中传递);

使用 `ThreadLocal` 要用 `try ... finally` 结构, 并在 `finally` 中清除.

* * *
