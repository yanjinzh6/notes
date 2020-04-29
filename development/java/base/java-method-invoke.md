---
title: Java Method 类 invoke 方法
date: 2020-04-28 21:30:00
tags: 'Java'
categories:
  - ['开发', 'Java', '基础']
permalink: java-method-invoke
---

## 简介

通过 invoke 方法可以实现动态调用, 比如可以动态传入参数及将方法参数化

- 获取 Method 对象: 通过调用 Class 对象的 `getMethod(String name, Class<?>... parameterTypes)` 返回一个 Method 对象, 它描述了此 Class 对象所表示的类或接口指定的公共成员方法。name 参数是 `String` 类型, 用于指定所需方法的名称。parameterTypes 参数是按声明顺序标识该方法的形参类型的 Class 对象的一个数组, 如果 parameterTypes 为 `null`, 则按空数组处理
- 调用 `invoke` 方法: 指通过调用 Method 对象的 `invoke` 方法来动态执行函数

<!-- more -->

## 声明

```java
public Object invoke(Object obj, Object... args) throws IllegalAccessException, IllegalArgumentException,InvocationTargetException
```

- obj: 调用底层方法的对象
- args: 方法调用的参数

返回对象执行的方法结果

- IllegalAccessException: 此方法对象正在强制执行 Java 语言访问控制, 并且底层方法无法访问
- IllegalArgumentException:
  - 方法是一个实例方法, 并且指定的对象参数不是声明底层方法 (或其子类或实现者) 的类或接口的实例
  - 实际和正式参数的数量不同
  - 原始参数的解包转换失败
  - 在可能的展开之后, 通过方法调用转换, 参数值不能转换为相应的形式参数类型
- InvocationTargetException: 底层方法抛出异常
- NullPointerException: 指定的对象为空, 并且该方法是实例方法
- ExceptionInInitializerError: 由此方法引发的初始化失败

## 示例

```java
Method[] methods = SampleClass.class.getMethods();
SampleClass sampleObject = new SampleClass();
methods[1].invoke(sampleObject, "data");
System.out.println(methods[0].invoke(sampleObject));
class SampleClass {
   private String sampleField;

   public String getSampleField() {
      return sampleField;
   }

   public void setSampleField(String sampleField) {
      this.sampleField = sampleField;
   }
}
```
