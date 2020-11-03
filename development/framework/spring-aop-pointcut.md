---
title: Spring AOP pointcut 表达式
date: 2020-04-25 22:00:00
tags: 'Spring'
categories:
  - ['开发', ' 框架']
permalink: spring-aop-pointcut
---

## 简介

主要记录 spring aop 表达式写法

## execute

语法 `execution(方法类型 返回类型 方法签名)`

拦截任意公共方法

```java
execution(public * *(..))
```

<!-- more -->

拦截以 set 开头的任意方法

```java
execution(* set*(..))
```

拦截 `setData(String str, int i)` 方法

```java
execution(* setData(String,int))
```

参数类型解析

- `String,int`: 指定参数签名
- `String,*`: `*` 号代表的第 2 个参数可以是任意类型
- `String,..`: 限制第 1 个参数, 后续参数个数不限, 类型不限
- `Object+`: 1 个参数, 并且参数是 `Object` 或其子类

拦截类或者接口中的方法

```java
// 拦截 AccountService(类、接口) 中定义的所有方法
execution(* com.xyz.service.AccountService.*(..))
```

拦截包中定义的方法, 不包含子包中的方法

```java
// 拦截 com.xyz.service 包中所有类中任意方法, 不包含子包中的类
execution(* com.xyz.service.*.*(..))
```

拦截包或者子包中定义的方法

```java
// 拦截com.xyz.service包或者子包中定义的所有方法
execution(* com.xyz.service..*.*(..))
```

## within

语法

```java
包名.* 或者 包名..*
```

- 拦截包中任意方法, 不包含子包中的方法 `within(com.xyz.service.*)`: 拦截 service 包中任意类的任意方法
- 拦截包或者子包中定义的方法 `within(com.xyz.service..*)`: 拦截 service 包及子包中任意类的任意方法

## this

代理对象为指定的类型会被拦截

目标对象使用 aop 之后生成的代理对象必须是指定的类型才会被拦截, 注意是目标对象被代理之后生成的代理对象和指定的类型匹配才会被拦截

`this(com.xyz.service.AccountService)`

## target

目标对象为指定的类型被拦截 `target(com.xyz.service.AccountService)`: 目标对象为 AccountService 类型的会被代理

this 和 target 的不同点

- this 作用于代理对象, target 作用于目标对象
- this 表示目标对象被代理之后生成的代理对象和指定的类型匹配会被拦截, 匹配的是代理对象
- target 表示目标对象和指定的类型匹配会被拦截, 匹配的是目标对象

## args

匹配方法中的参数 `@Pointcut("args(String)")`: 匹配只有一个参数, 且类型为 `String`
匹配多个参数 `args(type1,type2,typeN)`
匹配任意多个参数 `@Pointcut("args(String,..)")`: 匹配第一个参数类型为String的所有方法, `..` 表示任意个参数

## @target

匹配的目标对象的类有一个指定的注解

`@target(com.xyz.annotation.MyAnnotation)`

目标对象中包含 `com.xyz.annotation.MyAnnotation` 注解, 调用该目标对象的任意方法都会被拦截

## @within

指定匹配必须包含某个注解的类里的所有连接点

`@within(com.xyz.annotation.MyAnnotation)`

声明有 `com.xyz.annotation.MyAnnotation` 注解的类中的所有方法都会被拦截

`@target` 和 `@within` 的不同点

- `@target(注解A)`: 判断被调用的目标对象中是否声明了注解A, 如果有, 会被拦截
- `@within(注解A)`: 判断被调用的方法所属的类中是否声明了注解A, 如果有, 会被拦截
- `@target` 关注的是被调用的对象, `@within` 关注的是调用的方法所在的类

## @annotation

匹配有指定注解的方法（注解作用在方法上面）

`@annotation(com.xyz.annotation.MyAnnotation)`

被调用的方法包含指定的注解

## @args

方法参数所属的类型上有指定的注解, 被匹配

注意: `是方法参数所属的类型上有指定的注解, 不是方法参数中有注解`

- 匹配1个参数, 且第1个参数所属的类中有 `com.xyz.annotation.MyAnnotation` 注解 `@args(com.xyz.annotation.MyAnnotation)`
- 匹配多个参数, 且多个参数所属的类型上都有指定的注解 `@args(com.xyz.annotation.MyAnnotation, com.xyz.annotation.MyAnnotation2)`
- 匹配多个参数, 且第一个参数所属的类中有 `com.xyz.annotation.MyAnnotation` 注解 `@args(com.xyz.annotation.MyAnnotation, ..)`

## 参考

- [spring aop 中 pointcut 表达式完整版](https://zhuanlan.zhihu.com/p/63001123)
