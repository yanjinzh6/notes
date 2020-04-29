---
title: Java 反射 API
date: 2020-04-27 22:00:00
tags: 'Java'
categories:
  - ['开发', 'Java', '基础']
permalink: java-refect-api
---

## 简介

Java 的反射 API 主要用于在运行过程中动态生成类, 接口或对象等信息, 其常用 API 如下

- `Class` 类: 用于获取类的属性, 方法等信息
- `Field` 类: 表示类的成员变量, 用于获取和设置类中的属性值
- `Method` 类: 表示类的方法, 用于获取方法的描述信息或者执行某个方法
- `Constructor` 类: 表示类的构造方法

优点: 可以实现动态创建对象和编译, 体现出很大的灵活性
缺点: 影响性能

<!-- more -->

## 作用

- 在运行时判断任意一个对象所属的类
- 在运行时判断任意一个类所具有的成员变量和方法
- 在运行时任意调用一个对象的方法
- 在运行时构造任意一个类的对象

## 步骤

- 获取想要操作的类的 Class 对象, 该 Class 对象是反射的核心, 通过它可以调用类的任意方法
- 调用 Class 对象所对应的类中定义的方法, 这是反射的使用阶段
- 使用反射 API 来获取并调用类的属性和方法等信息

## 获取类

```java
// 调用某个对象的 getClass 方法以获取该类对应的 Class 对象
Object o = new Object();
Class clazz = o.getClass();

// 调用某个类的 class 属性以获取该类对应的 Class 对象
Class clazz = Object.class

// 调用 Class 类中的 forName 静态方法以获取该类对应的 Class 对象, 这是最安全, 性能也最好的方法
// 通过类的完整路径获取类
Class clazz = Class.forName("fullClassPath");
```

## 获取构造方法

```java
// 返回指定参数类型 public 的构造器
Constructor<T> getConstructor(类<?>... parameterTypes)

// 返回指定参数类型的构造器 (public, protected, private)
Constructor<T> getDeclaredConstructor(类<?>... parameterTypes)

// 返回所有的 public 类型的构造器
Constructor<?>[] getConstructors()

// 返回所有的构造器
Constructor<?>[] getDeclaredConstructors()
```

## 获取变量

```java
// 根据变量名获得对应的变量, 访问权限不限
Field getDeclaredField(String name)

// 获得类中所有属性变量
Field[] getDeclaredFields()

// 根据变量名获取对应 public 类型的属性变量
Field getField(String name)

// 获取类中所有 public 类型的属性变量
Field[] getFields()
```

## 获取方法

```java
// 获取 "名称是 name, 参数是 arameterTypes" 的 public 的函数 (包括从基类继承的, 从接口实现的所有 public 函数)
public Method  getMethod(String name, Class[] parameterTypes)

// 获取全部的 public 的函数 (包括从基类继承的, 从接口实现的所有 public 函数)
public Method[]  getMethods()

// 获取 "名称是 name, 参数是parameterTypes", 并且是类自身声明的函数, 包含 public, protected 和 private 方法
public Method  getDeclaredMethod(String name, Class[] parameterTypes)

// 获取全部的类自身声明的函数, 包含 public, protected 和 private 方法
public Method[]  getDeclaredMethods()

// 如果这个类是 "其它类中某个方法的内部类", 调用 getEnclosingMethod() 就是这个类所在的方法, 若不存在, 返回 null
public Method  getEnclosingMethod()
```
