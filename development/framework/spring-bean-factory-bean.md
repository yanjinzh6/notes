---
title: Spring FactoryBean
date: 2020-05-05 18:00:00
tags: 'Spring'
categories:
  - ['开发', ' 框架']
permalink: spring-bean-factory-bean
---

## 简介

Spring 提供了一个 org.Springframework.bean.factory.FactoryBean 的工厂类接口采用编码的方式来实例化 bean, 解决某些 bean 比较复杂导致配置繁琐

Spring 自身提供了 70 多个 FactoryBean 的实现, 它们隐藏了实例化一些复杂 bean 的细节, 给上层应用带来了便利, 从 Spring 3.0 开始, FactoryBean 开始支持泛型, 即接口声明改为 FactoryBean<T> 的形式

<!-- more -->

```java
// org.springframework.beans.factory.FactoryBean
public interface FactoryBean<T> {

  /**
   * Return an instance (possibly shared or independent) of the object
   * managed by this factory.
   * <p>As with a {@link BeanFactory}, this allows support for both the
   * Singleton and Prototype design pattern.
   * <p>If this FactoryBean is not fully initialized yet at the time of
   * the call (for example because it is involved in a circular reference),
   * throw a corresponding {@link FactoryBeanNotInitializedException}.
   * <p>As of Spring 2.0, FactoryBeans are allowed to return {@code null}
   * objects. The factory will consider this as normal value to be used; it
   * will not throw a FactoryBeanNotInitializedException in this case anymore.
   * FactoryBean implementations are encouraged to throw
   * FactoryBeanNotInitializedException themselves now, as appropriate.
   * @return an instance of the bean (can be {@code null})
   * @throws Exception in case of creation errors
   * @see FactoryBeanNotInitializedException
   */
  // 返回由 FactoryBean 创建的 bean 实例, 如果 isSingleton()返回 true, 则该实例会放到 Spring 容器中单实例缓存池中
  @Nullable
  T getObject() throws Exception;

  /**
   * Return the type of object that this FactoryBean creates,
   * or {@code null} if not known in advance.
   * <p>This allows one to check for specific types of beans without
   * instantiating objects, for example on autowiring.
   * <p>In the case of implementations that are creating a singleton object,
   * this method should try to avoid singleton creation as far as possible;
   * it should rather estimate the type in advance.
   * For prototypes, returning a meaningful type here is advisable too.
   * <p>This method can be called <i>before</i> this FactoryBean has
   * been fully initialized. It must not rely on state created during
   * initialization; of course, it can still use such state if available.
   * <p><b>NOTE:</b> Autowiring will simply ignore FactoryBeans that return
   * {@code null} here. Therefore it is highly recommended to implement
   * this method properly, using the current state of the FactoryBean.
   * @return the type of object that this FactoryBean creates,
   * or {@code null} if not known at the time of the call
   * @see ListableBeanFactory#getBeansOfType
   */
  // 返回 FactoryBean 创建的 bean 类型
  @Nullable
  Class<?> getObjectType();

  /**
   * Is the object managed by this factory a singleton? That is,
   * will {@link #getObject()} always return the same object
   * (a reference that can be cached)?
   * <p><b>NOTE:</b> If a FactoryBean indicates to hold a singleton object,
   * the object returned from {@code getObject()} might get cached
   * by the owning BeanFactory. Hence, do not return {@code true}
   * unless the FactoryBean always exposes the same reference.
   * <p>The singleton status of the FactoryBean itself will generally
   * be provided by the owning BeanFactory; usually, it has to be
   * defined as singleton there.
   * <p><b>NOTE:</b> This method returning {@code false} does not
   * necessarily indicate that returned objects are independent instances.
   * An implementation of the extended {@link SmartFactoryBean} interface
   * may explicitly indicate independent instances through its
   * {@link SmartFactoryBean#isPrototype()} method. Plain {@link FactoryBean}
   * implementations which do not implement this extended interface are
   * simply assumed to always return independent instances if the
   * {@code isSingleton()} implementation returns {@code false}.
   * <p>The default implementation returns {@code true}, since a
   * {@code FactoryBean} typically manages a singleton instance.
   * @return whether the exposed object is a singleton
   * @see #getObject()
   * @see SmartFactoryBean#isPrototype()
   */
  // 返回由 FactoryBean 创建的 bean 实例的作用域是 singleton 还是 prototype, 默认为 true
  default boolean isSingleton() {
    return true;
  }

}
```

Spring 通过反射机制发现 XXXFactoryBean 实现了 FactoryBean 的接口, 这时 Spring 容器就调用接口方法 XXXFactoryBean#getObject() 方法返回

要获取 XXXFactoryBean 的实例, 则需要在使用 getBean(beanName) 方法时在 beanName 前显示的加上 "&" 前缀, 例如 getBean("&xxx")
