---
title: nginx-udp-stream
date: 2021-02-01 18:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: nginx-udp-stream
photo:
---

https://blog.csdn.net/kid551/article/details/88703414

在 spring 的注解 `@RequestMapping` 之下可以直接获取 `HttpServletRequest` 来获得诸如 request header 等重要的请求信息:

```
@Slf4j
@RestController
@RequestMapping("/test")
public class TestController {

    private static final String HEADER = "app-version";

    @RequestMapping(value = "/async", method = RequestMethod.GET)
    public void test(HttpServletRequest request) {
				request.getHeader(HEADER);
    }
}
```

往往, 这些重要的信息也会在异步线程中被使用到. 于是, 一个很自然的想法是, 那不如直接把这里获取到的 request 当做参数传到其它 spawn 出的子线程里, 比如:

```
@Slf4j
@RestController
@RequestMapping("/test")
public class TestController {

    private static final String HEADER = "app-version";

    @RequestMapping(value = "/async", method = RequestMethod.GET)
    public void test(HttpServletRequest request) {
        log.info("Main thread: " + request.getHeader(HEADER));
		new Thread(() -> {
            log.info("Child thread: " + request.getHeader(HEADER));
        }).start();
    }
}
```

在 header 中设置 "app-version" 为 1.0.1 后发送 `<base_url>/test/async` 请求, 可以看到结果:

> Main thread: 1.0.1
>
> Child thread: 1.0.1

但是, 坑, 也就此出现了.

由于 `HttpServletRequest` 不是线程安全的 (后知后觉), 当主线程完成自己的工作返回 response 后, 相应的诸如 `HttpServletRequest` 等对象就会被销毁. 为了看到这个现象, 我们可以在子线程中多等待一段时间来保证主线程先于子线程结束.

```
@Slf4j
@RestController
@RequestMapping("/test")
public class TestController {

    private static final String HEADER = "app-version";
    private static final long CHILD_THREAD_WAIT_TIME = 5000;

    @RequestMapping(value = "/async", method = RequestMethod.GET)
    public void test(HttpServletRequest request) {
        log.info("Main thread: " + request.getHeader(HEADER));

        new Thread(() -> {
            try {
                Thread.sleep(CHILD_THREAD_WAIT_TIME);
            } catch (Throwable e) {

            }
            log.info("Child thread: " + request.getHeader(HEADER));
        }).start();
    }
}
```

在 header 中设置 "app-version" 为 1.0.1 后发送 `<base_url>/test/async` 请求, 可以看到结果:

> Main thread: 1.0.1
>
> Child thread: null

显然, 谁也没办法保证自己 spawn 出来的子线程会先于主线程结束, 所以直接传递 `HttpServletRequest` 参数给子线程是不可行的.

网上有一种方法是通过 spring 框架自带的 `RequestContextHolder` 来获取 request, 这对异步线程来讲是不可行的. 因为只有在负责 request 处理的线程才能调用到 `RequestContextHolder` 对象, 其它线程中它会直接为空.

那么, 一个可以想到的笨办法是将 request 的值取出来, 注入到自定义的对象中, 然后将这个对象作为参数传递给子线程:

```
@Slf4j
@RestController
@RequestMapping("/test")
public class TestController {

    private static final String HEADER = "app-version";
    private static final long MAIN_THREAD_WAIT_TIME = 0;
    private static final long CHILD_THREAD_WAIT_TIME = 5000;

    @RequestMapping(value = "/async", method = RequestMethod.GET)
    public void test(HttpServletRequest request) {
        log.info("Main thread: " + request.getHeader(HEADER));
        TestVo testVo = new TestVo(request.getHeader(HEADER));

        new Thread(() -> {
            try {
                Thread.sleep(CHILD_THREAD_WAIT_TIME);
            } catch (Throwable e) {

            }
            log.info("Child thread: " + request.getHeader(HEADER) + ", testVo = " + testVo.getAppVersion());
        }).start();

        try {
            Thread.sleep(MAIN_THREAD_WAIT_TIME);
        } catch (Throwable e) {

        }
    }

    @Data
    @AllArgsConstructor
    public static class TestVo {
        private String appVersion;
    }
}
```

再按照 "app-version" 为 1.0.1 发送请求后得到:

> Main thread: 1.0.1
>
> Child thread: null, testVo = 1.0.1

嗯, 终于成功了.

故事似乎到此就结束了, 但如果仔细考察细节的话, 有几个问题是值得思考的:

*   如果 child thread 中的 request 已经被销毁了, 为什么没有报 null exception, 而只是获取到空的 "app-version" 的值?
*   如果 request 被销毁了, TestVo 这个同样在主线程中创建的 object 为什么没有被销毁?
*   主线程真的可以销毁对象吗? 销毁对象不是 GC 负责的吗, 为什么总是可以在 child thread 中得到 null 的结果?

一个合理的推理是: 主线程结束时, 调用了一个 `destroy()` 方法, 这个方法主动将 `HttpServletRequest` 中的资源释放, 例如调用了存放 header 的 map 对应的 `clear()` 方法. 如此, 在子线程中便无法获得之前的 "app-version" 所对应的 value 了. 而 TestVo 由于是用户自己创建, 必然不可能实现在 `destroy()` 方法中写出释放资源的代码. 它的值也就保存下来了.

另外, 无论主线程是否调用了 `destroy()` 方法, 真正回收的时候还是 GC 的工作, 这也就解释了在子线程中不是报 null exception, 而只是取不到特定的 key 所对应的值.

进一步, 我们还可以思考的问题是, 为什么在主线程的 `destoy()` 方法中, 不直接将 request 对象赋值为 null 呢?

这个问题看似有些蹊跷, 而实则根本不成立. 因为就算你把主线程的 request 变量赋值为 null 时, 子线程中的另一个变量已经指向了这个 request 对应的内存, 依旧可以拿到相应的值. 例如:

```
@Slf4j
@RestController
@RequestMapping("/test")
public class TestController {

    private static final String HEADER = "app-version";
    private static final long MAIN_THREAD_WAIT_TIME = 5000;
    private static final long CHILD_THREAD_WAIT_TIME = 3000;

    @RequestMapping(value = "/async", method = RequestMethod.GET)
    public void test(HttpServletRequest request) {
        log.info("Main thread: " + request.getHeader(HEADER));
        TestVo testVo = new TestVo(request);

        new Thread(() -> {
            try {
                Thread.sleep(CHILD_THREAD_WAIT_TIME);
            } catch (Throwable e) {

            }
            log.info("Child thread: " + testVo.getRequest().getHeader(HEADER));
        }).start();

        request = null;

        try {
            Thread.sleep(MAIN_THREAD_WAIT_TIME);
        } catch (Throwable e) {

        }
    }

    @Data
    @AllArgsConstructor
    public static class TestVo {
        private HttpServletRequest request;
    }
}
```

按照 "app-version" 为 1.0.1 发送请求后得到:

> Main thread: 1.0.1
>
> Child thread: 1.0.1

这里让子线程等待 3 秒, 以便主线程有充分的时间将 request 赋值为 null. 但 child 线程依旧可以拿到对应的值.

所以, 将 request 变量赋值为 null 根本无法做到释放资源. 所以对 request 里保存 header 的 map 来讲, 将变量赋值为 null 无法保证其它地方的引用也会一并消失. 最直接有效的方法是调用 `clear()` 让 map 中的每一个元素失效.

所以总结起来是:

*   主线程的 request 和 testVo, 由于都有子线程的变量指向, 也即是两个对象上的 reference count 不为 0, GC 便不会真正回收这两部分对应的内存.
*   但是, 由于 request 很可能在主线程中在 `destroy()` 方法被调用了内部 map 的 `clear()` 方法, 导致无法获取到 header 的值.
*   testVo 是用户创建的对象, 无法事先被放到 `destroy()` 方法中被释放, 所以还能继续保持原有的值.
