---
title: js-custom-elements
date: 2021-02-01 15:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: js-custom-elements
photo:
---

https://blog.csdn.net/u013423085/article/details/82872533

### bean 的作用域,@Scope 注解与 proxyMode 属性

*   *   [前言](#_1)
    *   [bean 的作用域](#bean_4)
    *   [@Scope 注解](#Scope_12)
    *   [作用域代理——proxyMode 属性](#proxyMode_43)

## 前言

对于 Spring 而言, 在默认情况下其所有的 bean 都是以单例的形式创建的. 即无论给定的一个 bean 被注入到其他 bean 多少次, 每次所注入的都是同一个实例.

## bean 的作用域

首先, 我们需要了解下 Spring 定义了多种作用域:
1.**单例 (Singleton)**: 在整个应用中, 只创建 bean 的一个实例.
2.**原型 (Prototype)**: 每次注入或者通过 spring 应用上下文获取的时候, 都会创建一个新的 bean 实例.
3.**会话 (Session)**: 在 web 应用中, 为每个会话创建一个 bean 实例.
4.**请求 (Request)**: 在 Web 应用中, 为每个请求创建一个 bean 实例.

## @Scope 注解

单例是默认的作用域, 但有些时候并不符合我们的实际运用场景, 因此我们可以使用@Scope 注解来选择其他的作用域. 该注解可以配合@Component 和@Bean 一起使用

例如, 如果你使用组件扫描来发现和声明 bean, 那么你可以在 bean 的类上使用@Scope 配合@Component, 将其声明为原型 bean:

```
 @Component
  @Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
  public class Notepad {
  	//todo: dosomething
  }
```

这里使用的是 `ConfigurableBeanFactory` 类的 `SCOPE_PROTOTYPE` 常量设置了原型作用域. 当然你也可以使用 `@Scope(value = "prototype")`, 相对而言笔者更喜欢使用 `SCOPE_PROTOTYPE` 常量, 因为这样使用不易出现拼写错误以及便于代码的维护.

如果你想在 Java 配置中将 Notepad 声明为原型 bean, 那么可以组合使用@Scope 和@Bean 来指定所需的作用域:

```
 @Bean
    @Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public Notepad notepad() {
        return new Notepad();
    }
```

如果你使用 xml 来配置 bean 的话, 可以使用 `<bean>` 元素的 scope 属性来设置作用域:

```
 <bean id="notepad" class="com.lixiang.Notepad" scope="prototype"/>
```

## 作用域代理——proxyMode 属性

**对于 bean 的作用域, 有一个典型的电子商务应用: 需要有一个 bean 代表用户的购物车.**

如果购物车是**单例**, 那么将会导致所有的用户都往一个购物车中添加商品.
如果购物车是**原型**作用域的, 那么在应用中某个地方往购物车中添加商品, 然后到应用中的另外一个地方可能就没法使用了, 因为在这里被注入了另外一个原型作用域的的购物车.
就购物车 bean 而言,**会话**作用域是最合适的, 因为他与给定用户的关联性最大.

```
 @Component
    @Scope(value = WebApplicationContext.SCOPE_SESSION,
    	proxyMode = ScopedProxyMode.INTERFACES)
    public class ShippingCart {
    	//todo: dosomething
    }
```

这里我们将 value 设置成了 `WebApplicationContext.SCOPE_SESSION` 常量. 这会告诉 Spring 为 Web 应用的每个会话创建一个 `ShippingCart`. 这会创建多个 `ShippingCart bean` 的实例.**但是对于给定的会话只会创建一个实例, 在当前会话各种操作中, 这个 bean 实际上相当于单例的**.

要注意的是,@Scope 中使用了 `proxyMode` 属性, 被设置成了 `ScopedProxyMode.INTERFACES`. 这个属性是用于解决将**会话**或**请求**作用域的 bean 注入到**单例**bean 中所遇到的问题.

假设我们将 `ShippingCart bean` 注入到**单例**`StoreService bean` 的 setter 方法中:

```
 @Component
    public class StoreService {

        private ShippingCart shippingCart;

        public void setShoppingCart(ShippingCart shoppingCart) {
            this.shippingCart = shoppingCart;
        }
        //todo: dosomething
    }
```

因为 `StoreService` 是个**单例**bean, 会在 Spring 应用上下文加载的时候创建. 当它创建的时候, Spring 会试图将 `ShippingCart bean` 注入到 `setShoppingCart()` 方法中. 但是 `ShippingCart bean` 是会话作用域, 此时并不存在. 直到用户进入系统创建会话后才会出现 `ShippingCart` 实例.

另外, 系统中会有多个 `ShippingCart` 实例, 每个用户一个. 我们并不希望注入固定的 `ShippingCart` 实例, 而是希望当 `StoreService` 处理购物车时, 它所使用的是当前会话的 `ShippingCart` 实例.

Spring 并不会将实际的 `ShippingCart bean` 注入到 `StoreService`, Spring 会注入一个 `ShippingCart bean` 的代理. 这个代理会暴露与 `ShippingCart` 相同的方法, 所以 `StoreService` 会认为它就是一个购物车. 但是, 当 `StoreService` 调用 `ShippingCart` 的方法时, 代理会对其进行懒解析并将调用委任给会话作用域内真正的 `ShippingCart bean`.

在上面的配置中,`proxyMode` 属性, 被设置成了 `ScopedProxyMode.INTERFACES`, 这表明这个代理要实现 `ShippingCart` 接口, 并将调用委托给实现 bean.
但如果 `ShippingCart` 是一个具体的类而不是接口的话, Spring 就没法创建基于接口的代理了. 此时, 它必须使用 CGLib 来生成基于类的代理. 所以, 如果 bean 类型是具体类的话我们必须要将 `proxyMode` 属性, 设置成 `ScopedProxyMode.TARGET_CLASS`, 以此来表明要以生成目标类扩展的方式创建代理.
**请求作用域的 bean 应该也以作用域代理的方式进行注入.**

如果你需要使用**xml**来声明会话或请求作用域的 bean, 那么就需要使用 `<aop: scoped-proxy />` 元素来指定代理模式.

```
<? xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns: xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns: aop="http://www.springframework.org/schema/aop"
       xsi: schemaLocation="http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-3.2.xsd
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

  <bean id="cart" class="com.lixiang.bean.ShoppingCart" scope="session"/>
  <aop: scoped-proxy />

</beans>
```

`<aop: scoped-proxy />` 是与@Scope 注解的 `proxyMode` 属性相同的 xml 元素. 它会告诉 Spring 为 bean 创建一个作用域代理. 默认情况下, 它会使用**CGLib**创建目标类的代理, 如果要生成基于接口的代理可以将 `proxy-target-class` 属性设置成 false, 如下:

```
<bean id="cart" class="com.lixiang.bean.ShoppingCart" scope="session"/>
<aop: scoped-proxy proxy-target-class="false"/>
```
