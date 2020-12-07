---
title: Spring 4.x 对动态代理接口使用注解 aop 不生效
date: 2020-06-20 8:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: spring-4-proxy-annotation-aop
---

## 描述

最近修改一个旧项目, 需要对特定服务接口的实体配置相应的字段, 一开始通过注解的方式处理, 但是不生效, 采用配置文件的方式就可以了, 于是就了解一下 spring 对 rpc 框架动态实现的类怎么处理的

环境应该是比较古老的, 具体如下

```
spring: 4.0.8
dubbo: 2.5.8
```

## 具体原因

参考文章 [接口方法上的注解无法被 @Aspect 声明的切面拦截的原因分析](https://my.oschina.net/guangshan/blog/1808373?p=1)
