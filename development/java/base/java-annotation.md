---
title: Java 内部注解
date: 2020-04-27 20:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '基础']
permalink: java-annotation
---

## 简介

注解 (Annotation) 是 Java 提供的设置程序中元素的关联信息和元数据 (MetaData) 的方法, 它是一个接口, 程序可以通过反射获取指定程序中元素的注解对象, 然后通过该注解对象获取注解中的元数据信息

<!-- more -->

## @Target

`@Target` 说明了注解所修饰的对象范围, 注解可被用于 packages, types (类, 接口, 枚举, 注解类型) , 类型成员 (方法, 构造方法, 成员变量, 枚举值) , 方法参数和本地变量 (循环变量, catch 参数等) , 在注解类型的声明中使用 target 明确其修饰的目标

### 语法

```java
// 可以多个目标
@Target({ElementType.METHOD})
```

| 名称 | 修饰目标 |
| -- | -- |
| `TYPE` | 用于描述类, 接口 (包括注解类型) 或 `enum` 声明 |
| `FIELD` | 用于描述域 |
| `METHOD` | 用于描述方法 |
| `PARAMETER` | 用于描述参数 |
| `CONSTRUCTOR` | 用于描述构造器 |
| `LOCAL_VARIABLE` | 用于描述局部变量 |
| `ANNOTATION_TYPE` | 用于声明一个注解 |
| `PACKAGE` | 用于描述包 |
| `TYPE_PARAMETER` | 对普通变量的声明 |
| `TYPE_USE` | 能标注任何类型的名称 |

## @Retention

`@Retention` 定义了该注解被保留的级别, 即被描述的注解在什么级别有效, 有以下 `3` 种类型

- SOURCE: 在源文件中有效, 即在源文件中被保留
- CLASS: 在 Class 文件中有效, 即在 Class 文件中被保留
- RUNTIME: 在运行时有效, 即在运行时被保留

## @Documented

`@Documented` 表明这个注解应该被 javadoc 工具记录, 因此可以被 javadoc 类的工具文档化

## @Inherited

`@Inherited` 是一个标记注解, 表明某个被标注的类型是被继承的, 如果有一个使用了 `@Inherited` 修饰的 Annotation 被用于一个 Class, 则这个注解将被用于该 Class 的子类
