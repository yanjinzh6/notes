---
title: Nacos 自动注册
date: 2020-09-27 19:00:00
tags: 'Nacos'
categories:
  - ['开发', 'Java', '框架']
permalink: nacos-auto-registration
---

## 简介

使用 nacos 提供的 starter 可以很方便的开发

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>2.2.1.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>2.2.1.RELEASE</version>
</dependency>
```

但是打包后部署到服务器上后并不会自动进行注册

<!-- more -->

## 分析

Nacos 自动注册类 `NacosAutoServiceRegistration` 继承了 spring cloud 中的 `AbstractAutoServiceRegistration`, 监听了 `WebServerInitializedEvent` 事件

```java
// org/springframework/cloud/spring-cloud-commons/2.2.2.RELEASE/spring-cloud-commons-2.2.2.RELEASE-sources.jar!/org/springframework/cloud/client/serviceregistry/AbstractAutoServiceRegistration.java:86
@Override
@SuppressWarnings("deprecation")
public void onApplicationEvent(WebServerInitializedEvent event) {
  bind(event);
}

@Deprecated
public void bind(WebServerInitializedEvent event) {
  ApplicationContext context = event.getApplicationContext();
  if (context instanceof ConfigurableWebServerApplicationContext) {
    if ("management".equals(((ConfigurableWebServerApplicationContext) context)
        .getServerNamespace())) {
      return;
    }
  }
  this.port.compareAndSet(0, event.getWebServer().getPort());
  this.start();
}
```

### spring-boot 上下文

`ServletWebServerApplicationContext` 是 2.0 中 spring 框架的 web 应用上下文的子类, 主要控制着 web 容器

```java
// org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext#finishRefresh
@Override
protected void finishRefresh() {
  super.finishRefresh();
  WebServer webServer = startWebServer();
  if (webServer != null) {
    publishEvent(new ServletWebServerInitializedEvent(webServer, this));
  }
}
```

可以看到, 在启动 `WebServer` 后会发送一个 `ServletWebServerInitializedEvent` 事件, 通过源码可以发现该事件是 `WebServerInitializedEvent` 事件的子类, 这样就可以了解 nacos 的自动注册逻辑

问题出现在, 当使用外部容器启动时, 这里的 `webServer` 为空, 于是没办法完成监听事件并自动注册

## 解决

最简单的解决方法就是在应用启动后手动进行 nacos 注册

```java
@Component
public class ManualNacosRegistration implements ApplicationRunner, DisposableBean {
    @Autowired
    private NacosRegistration nacosRegistration;
    @Autowired
    private NacosServiceRegistry nacosServiceRegistry;

    @Override
    public void run(ApplicationArguments args) throws Exception {
        nacosRegistration.setPort(8080);
        Object status = nacosServiceRegistry.getStatus(nacosRegistration);
        if (!"UP".equals(status)) {
            nacosServiceRegistry.register(nacosRegistration);
        }
    }

    @Override
    public void destroy() throws Exception {
        if (nacosServiceRegistry != null) {
            nacosServiceRegistry.close();
        }
    }
}
```

需要注意, 自动注册时根据 `event.getWebServer().getPort()` 获取内嵌容器的端口, 使用外部容器时需要指定端口, 或者读取配置文件定义的端口

这里简单根据当前应用的状态进行注册, 当然应用启动后注册的方法有很多, 也可以通过实现 `ApplicationListener`, 通过实现 `ApplicationReadyEvent` 事件在应用启动后进行注册
