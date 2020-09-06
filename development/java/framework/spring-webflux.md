---
title: Spring 响应式 Web 控制层简介
date: 2020-08-22 10:00:00
tags: 'Spring'
categories:
  - ['开发', 'Java', '框架']
permalink: spring-webflux
---

## 简介

在 Servlet 3.1 之前, Web 容器都是基于阻塞机制开发的, 而在 Servlet 3.1 (包含)之后, 就开始了非阻塞的规范, 对于高并发网站, 使用函数式的编程就显得更为直观和简易, 所以它十分适合那些需要高并发和大量请求的互联网的应用, 特别是那些需要高速响应而对业务逻辑要求并不十分严格的网站, 如游戏, 视频, 新闻浏览网站等

NodeJS 的 web 开发就是使用非阻塞 IO, 使得在高并发上性能非常好

在 Java 8 发布之后, 引入了 Lambda 表达式和 Functional 接口等新特性, 使得 Java 的语法更为丰富, Spring 5 推出了 Spring WebFlux 新一代的 Web 响应式编程框架, Spring WebFlux 需要的是能够支持 Servlet 3.1+ 的容器, 如 Tomcat, Jetty 和 Undertow 等, 在 Spring Boot 中对 Spring WebFlux 的 starter 中默认是依赖于 Netty 库

在 Spring WebFlux 中, 存在两种开发方式, 一种是类似于 Spring MVC 的模式, 另一种则是函数功能性的编程

<!-- more -->

## 使用

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
```

这里如果依赖 `spring-boot-starter-web` 的话会只加载 Spring MVC

### Spring MVC 模式控制层

使用这种模式开发的控制层只是将返回的方式类型转换为 Flux 和 Mono

```java
@GetMapping
public Flux<Demo> find() {}

@GetMapping("/details")
public Mono<Demo> findById() {}
```

### 函数式风格 API

```java
@Configuration
public class Router {
  @Bean
  public RouterFunction<?> demoRouter() {
    return RouterFunctions.route().GET("/demo", request -> {
      return ServerResponse.ok().bodyValue(new Demo());
    }).build();
  }
}
```

可以通过 `andRoute()` 方法来声明另外的映射关系

### 使用 RxJava 类型

Spring WebFlux 使用 Reactive 库默认使用 Flux 和 Mono, 但也可以使用 RxJava 类型 Observable 和 Single

```java
@GetMapping
public Observable<Demo> find() {}

@GetMapping("/details")
public Single<Demo> findById() {}
```

### 使用客户端

Spring WebFlux 提供了 WebClient 类方便微服务间的调用, 类似 ResTemplate, 可以通过它请求后端的服务

```java
@Bean
public WebClient getWebClient() {
  // 注入设置请求基础路径的对象
  return WebClient.create("http://localhost:8080");
}

@Autowire
private WebClient client;

private void insertUser(User newUser) {
    // 注意, 这只是定义一个发送器, 并不会发送请求
    Mono<UserVo> userMono =
        // 定义 POST 请求
        client.post()
            // 设置请求 URI
            .uri("/user")
            // 请求体为 JSON 数据流
            .contentType(MediaType.APPLICATION_STREAM_JSON)
            // 请求体内容
            .body(Mono.just(newUser), User.class)
            // 接收请求结果类型
            .accept(MediaType.APPLICATION_STREAM_JSON)
            // 设置请求结果检索规则
            .retrieve()
            // 将结果体转换为一个 Mono 封装的数据流
            .bodyToMono(UserVo.class);
    // 获取服务器发布的数据流, 此时才会发送请求
    UserVo user = userMono.block();
}
```
