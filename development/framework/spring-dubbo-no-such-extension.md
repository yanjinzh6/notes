---
title: spring-dubbo-no-such-extension
date: 2021-01-31 12:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: spring-dubbo-no-such-extension
---

```
Error starting ApplicationContext. To display the conditions report re-run your application with 'debug' enabled.
2018-08-28 09:01:19.390 ERROR 8816 --- [           main] o.s.boot.SpringApplication               : Application run failed

org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'ServiceBean: alarmServiceImpl: com.mc.alarm.api.service.AlarmService': Error setting property values; nested exception is org.springframework.beans.PropertyBatchUpdateException; nested PropertyAccessExceptions (1) are:
PropertyAccessException 1: org.springframework.beans.MethodInvocationException: Property 'filter' threw exception; nested exception is java.lang.IllegalStateException: No such extension traceIdFilter for filter/com.alibaba.dubbo.rpc.Filter
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyPropertyValues(AbstractAutowireCapableBeanFactory.java:1648) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1363) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:580) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:503) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.lambda$doGetBean$0(AbstractBeanFactory.java:317) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:222) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:315) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:199) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.preInstantiateSingletons(DefaultListableBeanFactory.java:760) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.finishBeanFactoryInitialization(AbstractApplicationContext.java:869) ~[spring-context-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:550) ~[spring-context-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.boot.web.servlet.context.ServletWebServerApplicationContext.refresh(ServletWebServerApplicationContext.java:140) ~[spring-boot-2.0.3.RELEASE.jar:2.0.3.RELEASE]
	at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:759) [spring-boot-2.0.3.RELEASE.jar:2.0.3.RELEASE]
	at org.springframework.boot.SpringApplication.refreshContext(SpringApplication.java:395) [spring-boot-2.0.3.RELEASE.jar:2.0.3.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:327) [spring-boot-2.0.3.RELEASE.jar:2.0.3.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1255) [spring-boot-2.0.3.RELEASE.jar:2.0.3.RELEASE]
	at org.springframework.boot.SpringApplication.run(SpringApplication.java:1243) [spring-boot-2.0.3.RELEASE.jar:2.0.3.RELEASE]
	at com.hcycom.mc.alarm.provider.McAlarmProviderApplication.main(McAlarmProviderApplication.java:10) [classes/: na]
Caused by: org.springframework.beans.PropertyBatchUpdateException: Failed properties: Property 'filter' threw exception; nested exception is java.lang.IllegalStateException: No such extension traceIdFilter for filter/com.alibaba.dubbo.rpc.Filter
	at org.springframework.beans.AbstractPropertyAccessor.setPropertyValues(AbstractPropertyAccessor.java:122) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.beans.AbstractPropertyAccessor.setPropertyValues(AbstractPropertyAccessor.java:77) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyPropertyValues(AbstractAutowireCapableBeanFactory.java:1644) ~[spring-beans-5.0.7.RELEASE.jar:5.0.7.RELEASE]
	... 17 common frames omitted

2018-08-28 09:01:19.398  INFO 8816 --- [bboShutdownHook] com.alibaba.dubbo.config.AbstractConfig  :  [DUBBO] Run shutdown hook now., dubbo version: 2.6.2, current host: 192.168.60.1
2018-08-28 09:01:19.399  INFO 8816 --- [bboShutdownHook] c.a.d.r.support.AbstractRegistryFactory  :  [DUBBO] Close all registries [], dubbo version: 2.6.2, current host: 192.168.60.1
```

在 springboot 集成 dubbo 时自定义日志跟踪 Filter: TraceIDFilter, 启动报 No such extension traceIdFilter for filter/com.alibaba.dubbo.rpc.Filter 原因如下:

1, 在 resources 目录下未创建 META-INF/dubbo/com.alibaba.dubbo.rpc.Filter 文件指定自定义的 Filter;

2, 在创建 com.alibaba.dubbo.rpc.Filter 文件时, 把.Filter 作为了文件的后缀, 实际上你创建的文件是名是 com.alibaba.dubbo.rpc, 而并非是 com.alibaba.dubbo.rpc.Filter, 所以启动时报找不到 com.alibaba.dubbo.rpc.Filter 文件.

解决方法:

  自定义的 Filter (TraceIDFilter) 不是 spring 的 bean. 而需要在 META-INF/dubbo/com.alibaba.dubbo.rpc.Filter 文件中配置如下文件内容:

```
traceIdFilter= com.mc.log.filter.TraceIDFilter
```

**欢迎关注**

! [](https://img-blog.csdnimg.cn/20200419195035807.jpg? x-oss-process= image/watermark, type_ZmFuZ3poZW5naGVpdGk, shadow_10, text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3R5cDE4MDU=, size_16, color_FFFFFF, t_70)
