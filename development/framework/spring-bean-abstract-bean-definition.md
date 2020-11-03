---
title: Spring AbstractBeanDefinition
date: 2020-05-05 16:00:00
tags: 'Spring'
categories:
  - ['开发', ' 框架']
permalink: spring-bean-abstract-bean-definition
---

## 属性

<!-- more -->

```java
public abstract class AbstractBeanDefinition extends BeanMetadataAttributeAccessor
         implements BeanDefinition,  Cloneable {

  // 此处省略静态变量以及 final 常量

  /**
  * bean 的作用范围, 对应 bean 属性 scope
  */
  private String scope = SCOPE_DEFAULT;

  /**
  * 是否是单例, 来自 bean 属性 scope
  */
  private boolean singleton = true;

  /**
  * 是否是原型, 来自 bean 属性 scope
  */
  private boolean prototype = false;

  /**
  * 是否是抽象, 对应 bean 属性 abstract
  */
  private boolean abstractFlag = false;

  /**
  * 是否延迟加载, 对应 bean 属性 lazy-init
  */
  private boolean lazyInit = false;

  /**
  * 自动注入模式, 对应 bean 属性 autowire
  */
  private int autowireMode = AUTOWIRE_NO;

  /**
  * 依赖检查, Spring 3.0 后弃用这个属性
  */
  private int dependencyCheck = DEPENDENCY_CHECK_NONE;
  /**
  * 用来表示一个 bean 的实例化依靠另一个 bean 先实例化, 对应 bean 属性 depend-on
  */
  private String[] dependsOn;

  /**
  * autowire-candidate 属性设置为 false, 这样容器在查找自动装配对象时,
  * 将不考虑该 bean, 即它不会被考虑作为其他 bean 自动装配的候选者, 但是该 bean 本身还是可以使用自动装配来注入其他 bean 的,
  *  对应 bean 属性 autowire-candidate
  */
  private boolean autowireCandidate = true;

  /**
  * 自动装配时当出现多个 bean 候选者时, 将作为首选者, 对应 bean 属性 primary
  */
  private boolean primary = false;

  /**
  * 用于记录 Qualifier, 对应子元素 qualifier
  */
  private final Map<String,  AutowireCandidateQualifier> qualifiers =
          new LinkedHashMap<String,  AutowireCandidateQualifier>(0);

  /**
  * 允许访问非公开的构造器和方法, 程序设置
  */
  private boolean nonPublicAccessAllowed = true;

  /**
  * 是否以一种宽松的模式解析构造函数, 默认为 true,
  * 如果为 false, 则在如下情况
  * interface ITest{}
  * class  ITestImpl implements ITest{};
  * class Main{
  *     Main（ITest i）{}
  *     Main（ITestImpl i）{}
  * }
  * 抛出异常, 因为 Spring 无法准确定位哪个构造函数
  * 程序设置
  */
  private boolean lenientConstructorResolution = true;

  /**
  * 记录构造函数注入属性, 对应 bean 属性 constructor-arg
  */
  private ConstructorArgumentValues constructorArgumentValues;

  /**
  * 普通属性集合
  */
  private MutablePropertyValues propertyValues;

  /**
  * 方法重写的持有者 , 记录 lookup-method、replaced-method 元素
  */
  private MethodOverrides methodOverrides = new MethodOverrides();

  /**
  * 对应 bean 属性 factory-bean, 用法:
  * <bean id="instanceFactoryBean" class="example.chapter3.InstanceFactoryBean"/>
  * <bean id="currentTime" factory-bean="instanceFactoryBean" factory-method="createTime"/>

  */
  private String factoryBeanName;

  /**
  * 对应 bean 属性 factory-method
  */
  private String factoryMethodName;

  /**
  * 初始化方法, 对应 bean 属性 init-method
  */
  private String initMethodName;

  /**
  * 销毁方法, 对应 bean 属性 destory-method
  */
  private String destroyMethodName;

  /**
  * 是否执行 init-method, 程序设置
  */
  private boolean enforceInitMethod = true;

  /**
  * 是否执行 destory-method, 程序设置
  */
  private boolean enforceDestroyMethod = true;

  /**
  * 是否是用户定义的而不是应用程序本身定义的, 创建 AOP 时候为 true, 程序设置
  */
  private boolean synthetic = false;

  /**
  * 定义这个 bean 的应用 , APPLICATION: 用户, INFRASTRUCTURE: 完全内部使用, 与用户无关, SUPPORT: 某些复杂配置的一部分
  * 程序设置
  */
  private int role = BeanDefinition.ROLE_APPLICATION;

  /**
  * bean 的描述信息
  */
  private String description;
  /**
  * 这个 bean 定义的资源
  */
  private Resource resource;

  // 此处省略 set/get 方法
}
```
