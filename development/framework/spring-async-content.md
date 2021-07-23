---
title: spring-async-content
date: 2021-03-21 12:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: spring-async-content
---

https://juejin.cn/post/6844903904539312141

https://stackoverflow.com/questions/23732089/how-to-enable-request-scope-in-async-task-executor

[! [](data: image/png; base64, iVBORw0KGgoAAAANSUhEUgAAADwAAAA8CAYAAAA6/NlyAAAAAXNSR0IArs4c6QAAAERlWElmTU0AKgAAAAgAAYdpAAQAAAABAAAAGgAAAAAAA6ABAAMAAAABAAEAAKACAAQAAAABAAAAPKADAAQAAAABAAAAPAAAAACL3+ lcAAAAV0lEQVRoBe3QMQEAAADCoPVP7WkJiEBhwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGDBgwIABAwYMGPjAADh8AAFUUgPVAAAAAElFTkSuQmCC)](/user/3491704659786455)

2019 年 08 月 01 日 阅读 3703

关注

# 如何在 Spring 异步调用中传递上下文

### 什么是异步调用?

异步调用是相对于同步调用而言的, 同步调用是指程序按预定顺序一步步执行, 每一步必须等到上一步执行完后才能执行, 异步调用则无需等待上一步程序执行完即可执行. 异步调用指, 在程序在执行时, 无需等待执行的返回值即可继续执行后面的代码. 在我们的应用服务中, 有很多业务逻辑的执行操作不需要同步返回 (如发送邮件, 冗余数据表等), 只需要异步执行即可.

本文将介绍 Spring 应用中, 如何实现异步调用. 在异步调用的过程中, 会出现线程上下文信息的丢失, 我们该如何解决线程上下文信息的传递.

### Spring 应用中实现异步

Spring 为任务调度与异步方法执行提供了注解支持. 通过在方法或类上设置 `@Async` 注解, 可使得方法被异步调用. 调用者会在调用时立即返回, 而被调用方法的实际执行是交给 Spring 的 `TaskExecutor` 来完成的. 所以被注解的方法被调用的时候, 会在新的线程中执行, 而调用它的方法会在原线程中执行, 这样可以避免阻塞, 以及保证任务的实时性.

#### 引入依赖

```
 <dependency>
            <groupId> org.springframework.boot</groupId>
            <artifactId> spring-boot-starter-web</artifactId>
        </dependency>
复制代码
```

引入 Spring 相关的依赖即可.

#### 入口类

```
@SpringBootApplication
@EnableAsync
public class AsyncApplication {
    public static void main(String[] args) {
        SpringApplication.run(AsyncApplication.class, args);
    }

复制代码
```

入口类增加了 `@EnableAsync` 注解, 主要是为了扫描范围包下的所有 `@Async` 注解.

#### 对外的接口

这里写了一个简单的接口:

```
@RestController
@Slf4j
public class TaskController {

    @Autowired
    private TaskService taskService;

    @GetMapping("/task")
    public String taskExecute() {
        try {
            taskService.doTaskOne();
            taskService.doTaskTwo();
            taskService.doTaskThree();
        } catch (Exception e) {
           log.error("error executing task for {}", e.getMessage());
        }
        return "ok";
    }
}
复制代码
```

调用 `TaskService` 执行三个异步方法.

#### Service 方法

```
@Component
@Slf4j
//@Async
public class TaskService {

    @Async
    public void doTaskOne() throws Exception {
        log.info(" 开始做任务一 ");
        long start = System.currentTimeMillis();
        Thread.sleep(1000);
        long end = System.currentTimeMillis();
        log.info(" 完成任务一, 耗时:" + (end - start) + " 毫秒 ");
    }

    @Async
    public void doTaskTwo() throws Exception {
        log.info(" 开始做任务二 ");
        long start = System.currentTimeMillis();
        Thread.sleep(1000);
        long end = System.currentTimeMillis();
        log.info(" 完成任务二, 耗时:" + (end - start) + " 毫秒 ");
    }

    @Async
    public void doTaskThree() throws Exception {
        log.info(" 开始做任务三 ");
        long start = System.currentTimeMillis();
        Thread.sleep(1000);
        long end = System.currentTimeMillis();
        log.info(" 完成任务三, 耗时:" + (end - start) + " 毫秒 ");
    }
}
复制代码
```

`@Async` 可以用于类上, 标识该类的所有方法都是异步方法, 也可以单独用于某些方法. 每个方法都会 sleep 1000 ms.

#### 结果展示

运行结果如下:

可以看到 `TaskService` 中的三个方法是异步执行的, 接口的结果快速返回, 日志信息异步输出. 异步调用, 通过开启新的线程调用的方法, 不影响主线程. 异步方法实际的执行交给了 Spring 的 `TaskExecutor` 来完成.

### Future: 获取异步执行的结果

在上面的测试中我们也可以发现主调用方法并没有等到调用方法执行完就结束了当前的任务. 如果想要知道调用的三个方法全部执行完该怎么办呢, 下面就可以用到异步回调.

异步回调就是让每个被调用的方法返回一个 Future 类型的值, Spring 中提供了一个 Future 接口的子类:`AsyncResult`, 所以我们可以返回 `AsyncResult` 类型的值.

```
public class AsyncResult<V> implements ListenableFuture<V> {

	private final V value;

	private final ExecutionException executionException;
	//...
}
复制代码
```

`AsyncResult` 实现了 `ListenableFuture` 接口, 该对象内部有两个属性: 返回值和异常信息.

```
public interface ListenableFuture<T> extends Future<T> {
    void addCallback(ListenableFutureCallback<? super T> var1);

    void addCallback(SuccessCallback<? super T> var1, FailureCallback var2);
}
复制代码
```

`ListenableFuture` 接口继承自 Future, 在此基础上增加了回调方法的定义.Future 接口定义如下:

```
public interface Future<V> {
	// 是否可以打断当前正在执行的任务
    boolean cancel(boolean mayInterruptIfRunning);

    // 任务取消的结果
    boolean isCancelled();

	// 异步方法中最后返回的那个对象中的值
	V get() throws InterruptedException, ExecutionException;
	// 用来判断该异步任务是否执行完成, 如果执行完成, 则返回 true, 如果未执行完成, 则返回 false
    boolean isDone();
	// 与 get() 一样, 只不过这里参数中设置了超时时间
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
复制代码
```

`#get()` 方法, 在执行的时候是需要等待回调结果的, 阻塞等待. 如果不设置超时时间, 它就阻塞在那里直到有了任务执行完成. 我们设置超时时间, 就可以在当前任务执行太久的情况下中断当前任务, 释放线程, 这样就不会导致一直占用资源.

`#cancel(boolean)` 方法, 参数是一个 boolean 类型的值, 用来传入是否可以打断当前正在执行的任务. 如果参数是 true 且当前任务没有执行完成 , 说明可以打断当前任务, 那么就会返回 true; 如果当前任务还没有执行, 那么不管参数是 true 还是 false, 返回值都是 true; 如果当前任务已经完成, 那么不管参数是 true 还是 false, 那么返回值都是 false; 如果当前任务没有完成且参数是 false, 那么返回值也是 false. 即:

1.  如果任务还没执行, 那么如果想取消任务, 就一定返回 true, 与参数无关.
2.  如果任务已经执行完成, 那么任务一定是不能取消的, 所以此时返回值都是 false, 与参数无关.
3.  如果任务正在执行中, 那么此时是否取消任务就看参数是否允许打断 (true/false).

#### 获取异步方法返回值的实现

```
 public Future<String> doTaskOne() throws Exception {
        log.info(" 开始做任务一 ");
        long start = System.currentTimeMillis();
        Thread.sleep(1000);
        long end = System.currentTimeMillis();
        log.info(" 完成任务一, 耗时:" + (end - start) + " 毫秒 ");
        return new AsyncResult<> (" 任务一完成, 耗时 " + (end - start) + " 毫秒 ");
    }
    //... 其他两个方法类似, 省略
复制代码
```

我们将 task 方法的返回值改为 `Future<String>`, 将执行的时间拼接为字符串返回.

```
 @GetMapping("/task")
    public String taskExecute() {
        try {
            Future<String> r1 = taskService.doTaskOne();
            Future<String> r2 = taskService.doTaskTwo();
            Future<String> r3 = taskService.doTaskThree();
            while (true) {
                if (r1.isDone() && r2.isDone() && r3.isDone()) {
                    log.info("execute all tasks");
                    break;
                }
                Thread.sleep(200);
            }
            log.info("\n" + r1.get() + "\n" + r2.get() + "\n" + r3.get());
        } catch (Exception e) {
           log.error("error executing task for {}", e.getMessage());
        }

        return "ok";
    }
复制代码
```

在调用异步方法之后, 可以通过循环判断异步方法是否执行完成. 结果正如我们所预期, future 所 get 到的是 AsyncResult 返回的字符串.

### 配置线程池

前面是最简单的使用方法, 使用默认的 `TaskExecutor`. 如果想使用自定义的 Executor, 可以结合 `@Configuration` 注解的配置方式.

```
 import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;

import java.util.concurrent.Executor;
import java.util.concurrent.ThreadPoolExecutor;

@Configuration
public class TaskPoolConfig {

    @Bean("taskExecutor") // bean 的名称, 默认为首字母小写的方法名
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10); // 核心线程数 (默认线程数)
        executor.setMaxPoolSize(20); // 最大线程数
        executor.setQueueCapacity(200); // 缓冲队列数
        executor.setKeepAliveSeconds(60); // 允许线程空闲时间 (单位: 默认为秒)
        executor.setThreadNamePrefix("taskExecutor-"); // 线程池名前缀
        // 线程池对拒绝任务的处理策略
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        return executor;
    }
}
复制代码
```

线程池的配置很灵活, 对核心线程数, 最大线程数等属性进行配置. 其中, rejection-policy, 当线程池已经达到最大线程数的时候, 如何处理新任务. 可选策略有 CallerBlocksPolicy, CallerRunsPolicy 等.CALLER\_RUNS: 不在新线程中执行任务, 而是由调用者所在的线程来执行. 我们验证下, 线程池的设置是否生效, 在 TaskService 中, 打印当前的线程名称:

```
 public Future<String> doTaskOne() throws Exception {
        log.info(" 开始做任务一 ");
        long start = System.currentTimeMillis();
        Thread.sleep(1000);
        long end = System.currentTimeMillis();
        log.info(" 完成任务一, 耗时:" + (end - start) + " 毫秒 ");
        log.info(" 当前线程为 {}", Thread.currentThread().getName());
        return new AsyncResult<> (" 任务一完成, 耗时 " + (end - start) + " 毫秒 ");
    }
复制代码
```

通过结果可以看到, 线程池配置的线程名前缀已经生效. 在 Spring @Async 异步线程使用过程中, 需要注意的是以下的用法会使 `@Async` 失效:

*   异步方法使用 static 修饰;
*   异步类没有使用 @Component 注解 (或其他注解) 导致 Spring 无法扫描到异步类;
*   异步方法不能与被调用的异步方法在同一个类中;
*   类中需要使用 @Autowired 或 @Resource 等注解自动注入, 不能手动 new 对象;
*   如果使用 Spring Boot 框架必须在启动类中增加 `@EnableAsync` 注解.

### 线程上下文信息传递

很多时候, 在微服务架构中的一次请求会涉及多个微服务. 或者一个服务中会有多个处理方法, 这些方法有可能是异步方法. 有些线程上下文信息, 如请求的路径, 用户唯一的 userId, 这些信息会一直在请求中传递. 如果不做任何处理, 我们看下是否能够正常获取这些信息.

```
@GetMapping("/task")
    public String taskExecute() {
        try {
            Future<String> r1 = taskService.doTaskOne();
            Future<String> r2 = taskService.doTaskTwo();
            Future<String> r3 = taskService.doTaskThree();

            ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
            HttpServletRequest request = requestAttributes.getRequest();
            log.info(" 当前线程为 {}, 请求方法为 {}, 请求路径为: {}", Thread.currentThread().getName(), request.getMethod(), request.getRequestURL().toString());
            while (true) {
                if (r1.isDone() && r2.isDone() && r3.isDone()) {
                    log.info("execute all tasks");
                    break;
                }
                Thread.sleep(200);
            }
            log.info("\n" + r1.get() + "\n" + r2.get() + "\n" + r3.get());
        } catch (Exception e) {
            log.error("error executing task for {}", e.getMessage());
        }

        return "ok";
    }
复制代码
```

在 Spring Boot Web 中我们可以通过 `RequestContextHolder` 很方便的获取 request. 在接口方法中, 输出请求的方法和请求的路径.

```
 public Future<String> doTaskOne() throws Exception {
        log.info(" 开始做任务一 ");
        long start = System.currentTimeMillis();
        Thread.sleep(1000);
        long end = System.currentTimeMillis();
        log.info(" 完成任务一, 耗时:" + (end - start) + " 毫秒 ");
        ServletRequestAttributes requestAttributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = requestAttributes.getRequest();
        log.info(" 当前线程为 {}, 请求方法为 {}, 请求路径为: {}", Thread.currentThread().getName(), request.getMethod(), request.getRequestURL().toString());
        return new AsyncResult<> (" 任务一完成, 耗时 " + (end - start) + " 毫秒 ");
    }
复制代码
```

同时在 TaskService 中, 验证是不是也能输出请求的信息. 运行程序, 结果如下:

在 TaskService 中, 每个异步线程的方法获取 `RequestContextHolder` 中的请求信息时, 报了空指针异常. 这说明了请求的上下文信息未传递到异步方法的线程中.`RequestContextHolder` 的实现, 里面有两个 ThreadLocal 保存当前线程下的 request.

```
 // 得到存储进去的 request
    private static final ThreadLocal<RequestAttributes> requestAttributesHolder =
            new NamedThreadLocal<RequestAttributes> ("Request attributes");
    // 可被子线程继承的 request
    private static final ThreadLocal<RequestAttributes> inheritableRequestAttributesHolder =
            new NamedInheritableThreadLocal<RequestAttributes> ("Request context");
复制代码
```

再看 `#getRequestAttributes()` 方法, 相当于直接获取 ThreadLocal 里面的值, 这样就使得每一次获取到的 Request 是该请求的 request. 如何将上下文信息传递到异步线程呢? Spring 中的 `ThreadPoolTaskExecutor` 有一个配置属性 `TaskDecorator`,`TaskDecorator` 是一个回调接口, 采用装饰器模式. 装饰模式是动态的给一个对象添加一些额外的功能, 就增加功能来说, 装饰模式比生成子类更为灵活. 因此 `TaskDecorator` 主要用于任务的调用时设置一些执行上下文, 或者为任务执行提供一些监视 / 统计.

```
public interface TaskDecorator {

	Runnable decorate(Runnable runnable);
}
复制代码
```

`#decorate` 方法, 装饰给定的 Runnable, 返回包装的 Runnable 以供实际执行.

下面我们定义一个线程上下文拷贝的 `TaskDecorator`.

```
import org.springframework.core.task.TaskDecorator;
import org.springframework.web.context.request.RequestAttributes;
import org.springframework.web.context.request.RequestContextHolder;

public class ContextDecorator implements TaskDecorator {
    @Override
    public Runnable decorate(Runnable runnable) {
        RequestAttributes context = RequestContextHolder.currentRequestAttributes();
        return () -> {
            try {
                RequestContextHolder.setRequestAttributes(context);
                runnable.run();
            } finally {
                RequestContextHolder.resetRequestAttributes();
            }
        };
    }
}
复制代码
```

实现较为简单, 将当前线程的 context 装饰到指定的 Runnable, 最后重置当前线程上下文.

在线程池的配置中, 增加回调的 TaskDecorator 属性的配置:

```
 @Bean("taskExecutor")
    public Executor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setQueueCapacity(200);
        executor.setKeepAliveSeconds(60);
        executor.setThreadNamePrefix("taskExecutor-");
        executor.setWaitForTasksToCompleteOnShutdown(true);
        executor.setAwaitTerminationSeconds(60);
        // 增加 TaskDecorator 属性的配置
        executor.setTaskDecorator(new ContextDecorator());
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }
复制代码
```

经过如上配置, 我们再次运行服务, 并访问接口, 控制台日志信息如下:

由结果可知, 线程的上下文信息传递成功.

### 小结

本文结合示例讲解了 Spring 中实现异步方法, 获取异步方法的返回值. 并介绍了配置 Spring 线程池的方式. 最后介绍如何在异步多线程中传递线程上下文信息. 线程上下文传递在分布式环境中会经常用到, 比如分布式链路追踪中需要一次请求涉及到的 TraceId, SpanId. 简单来说, 需要传递的信息能够在不同线程中. 异步方法是我们在日常开发中用来多线程处理业务逻辑, 这些业务逻辑不需要严格的执行顺序. 用好异步解决问题的同时, 更要用对异步多线程的方式.

[源码地址](https://github.com/keets2012/Spring-Boot-Samples)

#### 推荐阅读

[微服务合集](http://blueskykong.com/categories/%E5%BE%AE%E6%9C%8D%E5%8A%A1/)

#### 订阅最新文章, 欢迎关注我的公众号
