---
title: Spring BeanFactory
date: 2020-05-05 17:00:00
tags: 'Spring'
categories:
  - ['开发', ' 框架']
permalink: spring-bean-bean-factory
---

## bean 的加载

对于加载 bean 的功能, 在 Spring 中的调用方式为 `MyTestBean bean=(MyTestBean) bf.getBean("myTestBean");`

步骤大致如下

- 转换对应 beanName: 传入的参数可能是别名, 也可能是 FactoryBean, 需要进行一系列的解析
  - 去除 FactoryBean 的修饰符, 也就是如果 name="&aa", 那么会去除 & 使得 name="aa"
  - 取指定 alias 所表示的最终 beanName, 例如别名 A 指向名称为 B 的 bean 则返回 B, 若别名 A 指向别名 B, 别名 B 又指向名称为 C 的 bean 则返回 C
- 尝试从缓存中加载单例: 首先尝试从缓存中加载, 如果加载不成功则再次尝试从 singletonFactories 中加载, 因为在创建单例 bean 的时候会存在依赖注入的情况, 而在创建依赖的时候为了避免循环依赖, 在 Spring 中创建 bean 的原则是不等 bean 创建完成就会将创建 bean 的 ObjectFactory 提早曝光加入到缓存中, 一旦下一个 bean 创建时候需要依赖上一个 bean 则直接使用 ObjectFactory
- 实例化 bean: 如果从缓存中得到了 bean 的原始状态, 通过 getObjectForBeanInstance 对 bean 进行实例化
- 原型模式的依赖检查: 只有在单例情况下才会尝试解决循环依赖, 如果存在 A 中有 B 的属性, B 中有 A 的属性, 那么当依赖注入的时候, 就会产生当 A 还未创建完的时候因为对于 B 的创建再次返回创建 A, 造成循环依赖
- 检测 parentBeanFactory: 当前加载的 XML 配置文件中不包含 beanName 所对应的配置, 就到 parentBeanFactory 去递归调用 getBean 方法
- 将存储 XML 配置文件的 GernericBeanDefinition 转换为 RootBeanDefinition
- 寻找依赖: 因为 bean 的初始化过程中很可能会用到某些属性, 而某些属性很可能是动态配置的, 并且配置成依赖于其他的 bean, 那么这个时候就有必要先加载依赖的 bean
- 针对不同的 scope 进行 bean 的创建
- 类型转换: requiredType 不为空的情况下将返回的 bean 转换为 requiredType 所指定的类型

<!-- more -->

这里的 getBean 实际上调用了以下方法

```java
// org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
/**
  * Return an instance, which may be shared or independent, of the specified bean.
  * @param name the name of the bean to retrieve
  * @param requiredType the required type of the bean to retrieve
  * @param args arguments to use when creating a bean instance using explicit arguments
  * (only applied when creating a new instance as opposed to retrieving an existing one)
  * @param typeCheckOnly whether the instance is obtained for a type check,
  * not for actual use
  * @return an instance of the bean
  * @throws BeansException if the bean could not be created
  */
@SuppressWarnings("unchecked")
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
    @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
  // 提取对应的 beanName
  final String beanName = transformedBeanName(name);
  Object bean;

  // Eagerly check singleton cache for manually registered singletons.
  /*
  * 检查缓存中或者实例工厂中是否有对应的实例
  * 在创建单例 bean 的时候会存在依赖注入的情况, 而在创建依赖的时候为了避免循环依赖,
  * Spring 创建 bean 的原则是不等 bean 创建完成就会将创建 bean 的 ObjectFactory 提早曝光
  * 也就是将 ObjectFactory 加入到缓存中, 一旦下个 bean 创建时候需要依赖上个 bean 则直接使用 ObjectFactory
  */
  //直接尝试从缓存获取或者 singletonFactories 中的 ObjectFactory 中获取
  Object sharedInstance = getSingleton(beanName);
  if (sharedInstance != null && args == null) {
    if (logger.isTraceEnabled()) {
      if (isSingletonCurrentlyInCreation(beanName)) {
        logger.trace("Returning eagerly cached instance of singleton bean '" + beanName +
            "' that is not fully initialized yet - a consequence of a circular reference");
      }
      else {
        logger.trace("Returning cached instance of singleton bean '" + beanName + "'");
      }
    }
    //返回对应的实例, 有时候存在诸如 BeanFactory 的情况并不是直接返回实例本身而是返回指定方法返回的实例
    bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
  }

  else {
    // Fail if we're already creating this bean instance:
    // We're assumably within a circular reference.
    /*
    * 只有在单例情况才会尝试解决循环依赖, 原型模式情况下, 如果存在
    * A 中有 B 的属性, B 中有 A 的属性, 那么当依赖注入的时候, 就会产生当 A 还未创建完的时候因为
    * 对于 B 的创建再次返回创建 A, 造成循环依赖, 也就是下面的情况
    * isPrototypeCurrentlyInCreation (beanName) 为 true
    */
    if (isPrototypeCurrentlyInCreation(beanName)) {
      throw new BeanCurrentlyInCreationException(beanName);
    }

    // Check if bean definition exists in this factory.
    BeanFactory parentBeanFactory = getParentBeanFactory();
    // 如果 beanDefinitionMap 中也就是在所有已经加载的类中不包括 beanName 则尝试从 parentBeanFactory 中检测
    if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
      // Not found -> check parent.
      String nameToLookup = originalBeanName(name);
      if (parentBeanFactory instanceof AbstractBeanFactory) {
        return ((AbstractBeanFactory) parentBeanFactory).doGetBean(
            nameToLookup, requiredType, args, typeCheckOnly);
      }
      else if (args != null) {
        // Delegation to parent with explicit args.
        // 递归调用 parentBeanFactory.getBean
        return (T) parentBeanFactory.getBean(nameToLookup, args);
      }
      else if (requiredType != null) {
        // No args -> delegate to standard getBean method.
        return parentBeanFactory.getBean(nameToLookup, requiredType);
      }
      else {
        return (T) parentBeanFactory.getBean(nameToLookup);
      }
    }
    // 如果不是仅仅做类型检查则是创建 bean 进行记录
    if (!typeCheckOnly) {
      markBeanAsCreated(beanName);
    }

    try {
      // 将存储 XML 配置文件的 GernericBeanDefinition 转换为 RootBeanDefinition, 如果指定 BeanName 是子 Bean 的话同时会合并父类的相关属性
      final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
      checkMergedBeanDefinition(mbd, beanName, args);

      // Guarantee initialization of beans that the current bean depends on.
      String[] dependsOn = mbd.getDependsOn();
      // 若存在依赖则需要递归实例化依赖的 bean
      if (dependsOn != null) {
        for (String dep : dependsOn) {
          if (isDependent(beanName, dep)) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                "Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
          }
          // 缓存依赖调用
          registerDependentBean(dep, beanName);
          try {
            getBean(dep);
          }
          catch (NoSuchBeanDefinitionException ex) {
            throw new BeanCreationException(mbd.getResourceDescription(), beanName,
                "'" + beanName + "' depends on missing bean '" + dep + "'", ex);
          }
        }
      }

      // Create bean instance.
      // 实例化依赖的 bean 后便可以实例化 mbd 本身了
      // singleton 模式的创建
      if (mbd.isSingleton()) {
        sharedInstance = getSingleton(beanName, () -> {
          try {
            return createBean(beanName, mbd, args);
          }
          catch (BeansException ex) {
            // Explicitly remove instance from singleton cache: It might have been put there
            // eagerly by the creation process, to allow for circular reference resolution.
            // Also remove any beans that received a temporary reference to the bean.
            destroySingleton(beanName);
            throw ex;
          }
        });
        bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
      }
      // prototype 模式的创建 new 新实例
      else if (mbd.isPrototype()) {
        // It's a prototype -> create a new instance.
        Object prototypeInstance = null;
        try {
          beforePrototypeCreation(beanName);
          prototypeInstance = createBean(beanName, mbd, args);
        }
        finally {
          afterPrototypeCreation(beanName);
        }
        bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
      }
      // 指定的 scope 上实例化 bean
      else {
        String scopeName = mbd.getScope();
        final Scope scope = this.scopes.get(scopeName);
        if (scope == null) {
          throw new IllegalStateException("No Scope registered for scope name '" + scopeName + "'");
        }
        try {
          Object scopedInstance = scope.get(beanName, () -> {
            beforePrototypeCreation(beanName);
            try {
              return createBean(beanName, mbd, args);
            }
            finally {
              afterPrototypeCreation(beanName);
            }
          });
          bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
        }
        catch (IllegalStateException ex) {
          throw new BeanCreationException(beanName,
              "Scope '" + scopeName + "' is not active for the current thread; consider " +
              "defining a scoped proxy for this bean if you intend to refer to it from a singleton",
              ex);
        }
      }
    }
    catch (BeansException ex) {
      cleanupAfterBeanCreationFailure(beanName);
      throw ex;
    }
  }

  // Check if required type matches the type of the actual bean instance.
  // 检查需要的类型是否符合 bean 的实际类型
  if (requiredType != null && !requiredType.isInstance(bean)) {
    try {
      T convertedBean = getTypeConverter().convertIfNecessary(bean, requiredType);
      if (convertedBean == null) {
        throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
      }
      return convertedBean;
    }
    catch (TypeMismatchException ex) {
      if (logger.isTraceEnabled()) {
        logger.trace("Failed to convert bean '" + name + "' to required type '" +
            ClassUtils.getQualifiedName(requiredType) + "'", ex);
      }
      throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
    }
  }
  return (T) bean;
}
```

### 缓存单例 bean

变量描述:

- singletonObjects: 用于保存 BeanName 和创建 bean 实例之间的关系, bean name --> bean instance
- singletonFactories: 用于保存 BeanName 和创建 bean 的工厂之间的关系, bean name --> ObjectFactory
- earlySingletonObjects: 也是保存 BeanName 和创建 bean 实例之间的关系, 与 singletonObjects 的不同之处在于, 当一个单例 bean 被放到这里面后, 那么当 bean 还在创建过程中, 就可以通过 getBean 方法获取到了, 其目的是用来检测循环引用
- registeredSingletons: 用来保存当前所有已注册的 bean

```java
/**
  * Return the (raw) singleton object registered under the given name.
  * <p>Checks already instantiated singletons and also allows for an early
  * reference to a currently created singleton (resolving a circular reference).
  * @param beanName the name of the bean to look for
  * @param allowEarlyReference whether early references should be created or not
  * @return the registered singleton object, or {@code null} if none found
  */
// allowEarlyReference: 是否允许早期依赖
@Nullable
protected Object getSingleton(String beanName, boolean allowEarlyReference) {
  // 检查缓存中是否存在实例
  Object singletonObject = this.singletonObjects.get(beanName);
  if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
    // 如果为空, 则锁定全局变量并进行处理
    synchronized (this.singletonObjects) {
      // 再从 earlySingletonObjects 中获取 bean
      singletonObject = this.earlySingletonObjects.get(beanName);
      if (singletonObject == null && allowEarlyReference) {
        // 当某些方法需要提前初始化的时候则会调用 addSingletonFactory 方法将对应的
        // ObjectFactory 初始化策略存储在 singletonFactories
        // 如果还是没有并且允许早期依赖, 则从 singletonFactories 里面获取对应的 ObjectFactory
        ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
        if (singletonFactory != null) {
          // 调用预先设定的 getObject 方法
          singletonObject = singletonFactory.getObject();
          // 记录在缓存中, earlySingletonObjects 和 singletonFactories 互斥
          this.earlySingletonObjects.put(beanName, singletonObject);
          // 从 singletonFacotories 里面 remove 掉这个 ObjectFactory
          this.singletonFactories.remove(beanName);
        }
      }
    }
  }
  return singletonObject;
}
```

### 从 bean 的实例中获取对象

getObjectForBeanInstance 用于检测当前 bean 是否是 FactoryBean 类型的 bean

- 对 FactoryBean 正确性的验证
- 对非 FactoryBean 不做任何处理
- 对 bean 进行转换
- 将从 Factory 中解析 bean 的工作委托给 getObjectFromFactoryBean

```java
// org.springframework.beans.factory.support.AbstractBeanFactory#getObjectForBeanInstance
/**
  * Get the object for the given bean instance, either the bean
  * instance itself or its created object in case of a FactoryBean.
  * @param beanInstance the shared bean instance
  * @param name name that may include factory dereference prefix
  * @param beanName the canonical bean name
  * @param mbd the merged bean definition
  * @return the object to expose for the bean
  */
protected Object getObjectForBeanInstance(
    Object beanInstance, String name, String beanName, @Nullable RootBeanDefinition mbd) {

  // Don't let calling code try to dereference the factory if the bean isn't a factory.
  // 如果指定的 name 是工厂相关 (以 & 为前缀) 且 beanInstance 又不是 FactoryBean 类型则验证不通过
  if (BeanFactoryUtils.isFactoryDereference(name)) {
    if (beanInstance instanceof NullBean) {
      return beanInstance;
    }
    if (!(beanInstance instanceof FactoryBean)) {
      throw new BeanIsNotAFactoryException(beanName, beanInstance.getClass());
    }
  }

  // Now we have the bean instance, which may be a normal bean or a FactoryBean.
  // If it's a FactoryBean, we use it to create a bean instance, unless the
  // caller actually wants a reference to the factory.
  // 现在我们有了个 bean 的实例, 这个实例可能会是正常的 bean 或者是 FactoryBean
  // 如果是 FactoryBean 我们使用它创建实例, 如果用户想要直接获取工厂实例而不是工厂的
  // getObject 方法对应的实例那么传入的 name 应该加入前缀 &
  if (!(beanInstance instanceof FactoryBean) || BeanFactoryUtils.isFactoryDereference(name)) {
    return beanInstance;
  }
  // 加载 FactoryBean
  Object object = null;
  if (mbd == null) {
    // 尝试从缓存中加载 bean
    object = getCachedObjectForFactoryBean(beanName);
  }
  if (object == null) {
    // Return bean instance from factory.
    // FactoryBean 类型的 beanInstance
    FactoryBean<?> factory = (FactoryBean<?>) beanInstance;
    // Caches object obtained from FactoryBean if it is a singleton.
    // containsBeanDefinition 检测 beanDefinitionMap 中也就是在所有已经加载的类中检测是否定义 beanName
    if (mbd == null && containsBeanDefinition(beanName)) {
      // 将存储 XML 配置文件的 GernericBeanDefinition 转换为 RootBeanDefinition, 如果指定 BeanName 是子 Bean 的话同时会合并父类的相关属性
      mbd = getMergedLocalBeanDefinition(beanName);
    }
    // 是否是用户定义的而不是应用程序本身定义的
    boolean synthetic = (mbd != null && mbd.isSynthetic());
    object = getObjectFromFactoryBean(factory, beanName, !synthetic);
  }
  return object;
}
```

```java
// org.springframework.beans.factory.support.FactoryBeanRegistrySupport#getObjectFromFactoryBean
/**
  * Obtain an object to expose from the given FactoryBean.
  * @param factory the FactoryBean instance
  * @param beanName the name of the bean
  * @param shouldPostProcess whether the bean is subject to post-processing
  * @return the object obtained from the FactoryBean
  * @throws BeanCreationException if FactoryBean object creation failed
  * @see org.springframework.beans.factory.FactoryBean#getObject()
  */
protected Object getObjectFromFactoryBean(FactoryBean<?> factory, String beanName, boolean shouldPostProcess) {
  if (factory.isSingleton() && containsSingleton(beanName)) {
    // 单例模式, 锁住 singletonObjects 对象
    synchronized (getSingletonMutex()) {
      // 从缓存里获取对象
      Object object = this.factoryBeanObjectCache.get(beanName);
      if (object == null) {
        // 直接获取获取对象
        object = doGetObjectFromFactoryBean(factory, beanName);
        // Only post-process and store if not put there already during getObject() call above
        // (e.g. because of circular reference processing triggered by custom getBean calls)
        Object alreadyThere = this.factoryBeanObjectCache.get(beanName);
        if (alreadyThere != null) {
          object = alreadyThere;
        }
        else {
          // 如果是应用程序本身定义的 RootBeanDefinition
          if (shouldPostProcess) {
            if (isSingletonCurrentlyInCreation(beanName)) {
              // Temporarily return non-post-processed object, not storing it yet..
              return object;
            }
            beforeSingletonCreation(beanName);
            try {
              // 调用 ObjectFactory 的后处理器
              object = postProcessObjectFromFactoryBean(object, beanName);
            }
            catch (Throwable ex) {
              throw new BeanCreationException(beanName,
                  "Post-processing of FactoryBean's singleton object failed", ex);
            }
            finally {
              afterSingletonCreation(beanName);
            }
          }
          if (containsSingleton(beanName)) {
            this.factoryBeanObjectCache.put(beanName, object);
          }
        }
      }
      return object;
    }
  }
  else {
    Object object = doGetObjectFromFactoryBean(factory, beanName);
    // 如果是应用程序本身定义的 RootBeanDefinition
    if (shouldPostProcess) {
      try {
        // 调用 ObjectFactory 的后处理器
        object = postProcessObjectFromFactoryBean(object, beanName);
      }
      catch (Throwable ex) {
        throw new BeanCreationException(beanName, "Post-processing of FactoryBean's object failed", ex);
      }
    }
    return object;
  }
}
```

```java
// org.springframework.beans.factory.support.FactoryBeanRegistrySupport#doGetObjectFromFactoryBean
/**
  * Obtain an object to expose from the given FactoryBean.
  * @param factory the FactoryBean instance
  * @param beanName the name of the bean
  * @return the object obtained from the FactoryBean
  * @throws BeanCreationException if FactoryBean object creation failed
  * @see org.springframework.beans.factory.FactoryBean#getObject()
  */
private Object doGetObjectFromFactoryBean(final FactoryBean<?> factory, final String beanName)
    throws BeanCreationException {

  Object object;
  try {
    // 需要权限验证
    if (System.getSecurityManager() != null) {
      AccessControlContext acc = getAccessControlContext();
      try {
        object = AccessController.doPrivileged((PrivilegedExceptionAction<Object>) factory::getObject, acc);
      }
      catch (PrivilegedActionException pae) {
        throw pae.getException();
      }
    }
    else {
      // 直接调用 getObject 方法
      object = factory.getObject();
    }
  }
  catch (FactoryBeanNotInitializedException ex) {
    throw new BeanCurrentlyInCreationException(beanName, ex.toString());
  }
  catch (Throwable ex) {
    throw new BeanCreationException(beanName, "FactoryBean threw exception on object creation", ex);
  }

  // Do not accept a null value for a FactoryBean that's not fully
  // initialized yet: Many FactoryBeans just return null then.
  if (object == null) {
    if (isSingletonCurrentlyInCreation(beanName)) {
      throw new BeanCurrentlyInCreationException(
          beanName, "FactoryBean which is currently in creation returned null from getObject");
    }
    object = new NullBean();
  }
  return object;
}
```

在 bean 初始化后都会调用注册的 BeanPostProcessor 的 postProcessAfterInitialization 方法进行处理

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#postProcessObjectFromFactoryBean
/**
  * Applies the {@code postProcessAfterInitialization} callback of all
  * registered BeanPostProcessors, giving them a chance to post-process the
  * object obtained from FactoryBeans (for example, to auto-proxy them).
  * @see #applyBeanPostProcessorsAfterInitialization
  */
@Override
protected Object postProcessObjectFromFactoryBean(Object object, String beanName) {
  return applyBeanPostProcessorsAfterInitialization(object, beanName);
}

// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {

  Object result = existingBean;
  for (BeanPostProcessor processor : getBeanPostProcessors()) {
    Object current = processor.postProcessAfterInitialization(result, beanName);
    if (current == null) {
      return result;
    }
    result = current;
  }
  return result;
}
```

### 获取单例

```java
/**
  * Return the (raw) singleton object registered under the given name,
  * creating and registering a new one if none registered yet.
  * @param beanName the name of the bean
  * @param singletonFactory the ObjectFactory to lazily create the singleton
  * with, if necessary
  * @return the registered singleton object
  */
public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
  Assert.notNull(beanName, "Bean name must not be null");
  // 锁定全局变量
  synchronized (this.singletonObjects) {
    //因为 singleton 模式其实就是复用以创建的 bean, 检查对应的 bean 是否已经加载过
    Object singletonObject = this.singletonObjects.get(beanName);
    if (singletonObject == null) {
      // 为空进行 singleto 的 bean 的初始化
      if (this.singletonsCurrentlyInDestruction) {
        throw new BeanCreationNotAllowedException(beanName,
            "Singleton bean creation not allowed while singletons of this factory are in destruction " +
            "(Do not request a bean from a BeanFactory in a destroy method implementation!)");
      }
      if (logger.isDebugEnabled()) {
        logger.debug("Creating shared instance of singleton bean '" + beanName + "'");
      }
      // 通过 this.singletonsCurrentlyInCreation.add(beanName) 记录当前正创建的 bean 加载状态, 可以对循环依赖进行检测
      beforeSingletonCreation(beanName);
      boolean newSingleton = false;
      boolean recordSuppressedExceptions = (this.suppressedExceptions == null);
      if (recordSuppressedExceptions) {
        this.suppressedExceptions = new LinkedHashSet<>();
      }
      try {
        // 初始化 bean
        singletonObject = singletonFactory.getObject();
        newSingleton = true;
      }
      catch (IllegalStateException ex) {
        // Has the singleton object implicitly appeared in the meantime ->
        // if yes, proceed with it since the exception indicates that state.
        singletonObject = this.singletonObjects.get(beanName);
        if (singletonObject == null) {
          throw ex;
        }
      }
      catch (BeanCreationException ex) {
        if (recordSuppressedExceptions) {
          for (Exception suppressedException : this.suppressedExceptions) {
            ex.addRelatedCause(suppressedException);
          }
        }
        throw ex;
      }
      finally {
        if (recordSuppressedExceptions) {
          this.suppressedExceptions = null;
        }
        // 加载单例后的处理方法调用, 通过 this.singletonsCurrentlyInCreation.remove(beanName) 移除 bean 创建状态
        afterSingletonCreation(beanName);
      }
      if (newSingleton) {
        // 结果记录至缓存并删除加载 bean 过程中所记录的各种辅助状态
        addSingleton(beanName, singletonObject);
      }
    }
    // 返回处理结果
    return singletonObject;
  }
}
// org.springframework.beans.factory.support.DefaultSingletonBeanRegistry#addSingleton
/**
  * Add the given singleton object to the singleton cache of this factory.
  * <p>To be called for eager registration of singletons.
  * @param beanName the name of the bean
  * @param singletonObject the singleton object
  */
protected void addSingleton(String beanName, Object singletonObject) {
  synchronized (this.singletonObjects) {
    this.singletonObjects.put(beanName, singletonObject);
    this.singletonFactories.remove(beanName);
    this.earlySingletonObjects.remove(beanName);
    this.registeredSingletons.add(beanName);
  }
}
```

### 创建 bean 准备工作

可以发现调用 getSingleton 方法传入了一个 ObjectFactory 的匿名实例, 里面返回了创建 bean 的逻辑

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#createBean(java.lang.String, org.springframework.beans.factory.support.RootBeanDefinition, java.lang.Object[])
/**
  * Central method of this class: creates a bean instance,
  * populates the bean instance, applies post-processors, etc.
  * @see #doCreateBean
  */
@Override
protected Object createBean(String beanName, RootBeanDefinition mbd, @Nullable Object[] args)
    throws BeanCreationException {

  if (logger.isTraceEnabled()) {
    logger.trace("Creating instance of bean '" + beanName + "'");
  }
  RootBeanDefinition mbdToUse = mbd;

  // Make sure bean class is actually resolved at this point, and
  // clone the bean definition in case of a dynamically resolved Class
  // which cannot be stored in the shared merged bean definition.
  // 锁定 class, 根据设置的 class 属性或者根据 className 来解析 Class
  Class<?> resolvedClass = resolveBeanClass(mbd, beanName);
  if (resolvedClass != null && !mbd.hasBeanClass() && mbd.getBeanClassName() != null) {
    mbdToUse = new RootBeanDefinition(mbd);
    mbdToUse.setBeanClass(resolvedClass);
  }

  // Prepare method overrides.
  try {
    // 验证及准备覆盖的方法
    // Spring 配置中 BeanDefinition 的 methodOverrides 属性对应的就是 xml 中的 lookup-method 和 replace-method
    // 这个函数的操作其实也就是针对于这两个配置
    mbdToUse.prepareMethodOverrides();
  }
  catch (BeanDefinitionValidationException ex) {
    throw new BeanDefinitionStoreException(mbdToUse.getResourceDescription(),
        beanName, "Validation of method overrides failed", ex);
  }

  try {
    // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
    // BeanPostProcessors 返回代理来替代真正的实例
    Object bean = resolveBeforeInstantiation(beanName, mbdToUse);
    // 应用初始化前的后处理器, 解析指定 bean 是否存在初始化前的短路操作
    if (bean != null) {
      return bean;
    }
  }
  catch (Throwable ex) {
    throw new BeanCreationException(mbdToUse.getResourceDescription(), beanName,
        "BeanPostProcessor before instantiation of bean failed", ex);
  }

  try {
    Object beanInstance = doCreateBean(beanName, mbdToUse, args);
    if (logger.isTraceEnabled()) {
      logger.trace("Finished creating instance of bean '" + beanName + "'");
    }
    return beanInstance;
  }
  catch (BeanCreationException | ImplicitlyAppearedSingletonException ex) {
    // A previously detected exception with proper bean creation context already,
    // or illegal singleton state to be communicated up to DefaultSingletonBeanRegistry.
    throw ex;
  }
  catch (Throwable ex) {
    throw new BeanCreationException(
        mbdToUse.getResourceDescription(), beanName, "Unexpected exception during bean creation", ex);
  }
}
```

#### 处理 override 属性

在 bean 实例化的时候如果检测到存在 methodOverrides 属性, 会动态地为当前 bean 生成代理并使用对应的拦截器为 bean 做增强处理

- 如果一个类中存在若干个重载方法, 那么, 在函数调用及增强的时候还需要根据参数类型进行匹配, 来最终确认当前调用的到底是哪个函数
- 如果当前类中的方法只有一个, 那么就设置重载该方法没有被重载, 这样在后续调用的时候便可以直接使用找到的方法, 而不需要进行方法的参数匹配验证了, 而且还可以提前对方法存在性进行验证

```java
// org.springframework.beans.factory.support.AbstractBeanDefinition#prepareMethodOverrides
/**
  * Validate and prepare the method overrides defined for this bean.
  * Checks for existence of a method with the specified name.
  * @throws BeanDefinitionValidationException in case of validation failure
  */
public void prepareMethodOverrides() throws BeanDefinitionValidationException {
  // Check that lookup methods exist and determine their overloaded status.
  if (hasMethodOverrides()) {
    // 循环处理
    getMethodOverrides().getOverrides().forEach(this::prepareMethodOverride);
  }
}
// org.springframework.beans.factory.support.AbstractBeanDefinition#prepareMethodOverride
/**
  * Validate and prepare the given method override.
  * Checks for existence of a method with the specified name,
  * marking it as not overloaded if none found.
  * @param mo the MethodOverride object to validate
  * @throws BeanDefinitionValidationException in case of validation failure
  */
protected void prepareMethodOverride(MethodOverride mo) throws BeanDefinitionValidationException {
  // 获取对应类中对应方法名的个数
  int count = ClassUtils.getMethodCountForName(getBeanClass(), mo.getMethodName());
  if (count == 0) {
    throw new BeanDefinitionValidationException(
        "Invalid method override: no method with name '" + mo.getMethodName() +
        "' on class [" + getBeanClassName() + "]");
  }
  else if (count == 1) {
    // Mark override as not overloaded, to avoid the overhead of arg type checking.
    // 标记 MethodOverride 暂未被覆盖, 避免参数类型检查的开销
    mo.setOverloaded(false);
  }
}
```

#### 实例化的前置处理

通过 `resolveBeforeInstantiation(beanName, mbdToUse)` 方法对 BeanDefinigiton 中的属性做些前置处理, 接着提供了一个短路判断, 当经过前置处理后返回的结果如果不为空, 那么会直接略过后续的 bean 的创建而直接返回结果, 这一点很重要, AOP 功能就是基于这里的判断的

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#resolveBeforeInstantiation
/**
  * Apply before-instantiation post-processors, resolving whether there is a
  * before-instantiation shortcut for the specified bean.
  * @param beanName the name of the bean
  * @param mbd the bean definition for the bean
  * @return the shortcut-determined bean instance, or {@code null} if none
  */
@Nullable
protected Object resolveBeforeInstantiation(String beanName, RootBeanDefinition mbd) {
  Object bean = null;
  // 如果还没被解析
  if (!Boolean.FALSE.equals(mbd.beforeInstantiationResolved)) {
    // Make sure bean class is actually resolved at this point.
    if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
      Class<?> targetType = determineTargetType(beanName, mbd);
      if (targetType != null) {
        // 调用 后处理器中的所有 InstantiationAwareBeanPostProcessor 类型的后处理器进行 postProcessBeforeInstantiation 方法
        bean = applyBeanPostProcessorsBeforeInstantiation(targetType, beanName);
        if (bean != null) {
          // 调用 BeanPostProcessor 的 postProcessAfterInitialization 方法
          bean = applyBeanPostProcessorsAfterInitialization(bean, beanName);
        }
      }
    }
    mbd.beforeInstantiationResolved = (bean != null);
  }
  return bean;
}
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsBeforeInstantiation
/**
  * Apply InstantiationAwareBeanPostProcessors to the specified bean definition
  * (by class and name), invoking their {@code postProcessBeforeInstantiation} methods.
  * <p>Any returned object will be used as the bean instead of actually instantiating
  * the target bean. A {@code null} return value from the post-processor will
  * result in the target bean being instantiated.
  * @param beanClass the class of the bean to be instantiated
  * @param beanName the name of the bean
  * @return the bean object to use instead of a default instance of the target bean, or {@code null}
  * @see InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation
  */
// 实例化前的后处理器应用
// 子类一个修改 BeanDefinition 的功能
@Nullable
protected Object applyBeanPostProcessorsBeforeInstantiation(Class<?> beanClass, String beanName) {
  for (BeanPostProcessor bp : getBeanPostProcessors()) {
    if (bp instanceof InstantiationAwareBeanPostProcessor) {
      InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
      Object result = ibp.postProcessBeforeInstantiation(beanClass, beanName);
      if (result != null) {
        return result;
      }
    }
  }
  return null;
}
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#applyBeanPostProcessorsAfterInitialization
// 实例化后的后处理器应用
@Override
public Object applyBeanPostProcessorsAfterInitialization(Object existingBean, String beanName)
    throws BeansException {

  Object result = existingBean;
  for (BeanPostProcessor processor : getBeanPostProcessors()) {
    // Spring 中的规则是在 bean 的初始化后尽可能保证将注册的后处理器的 postProcessAfterInitialization 方法应用到该 bean 中
    Object current = processor.postProcessAfterInitialization(result, beanName);
    if (current == null) {
      return result;
    }
    result = current;
  }
  return result;
}
```

### 循环依赖

Spring 容器循环依赖包括构造器循环依赖和 setter 循环依赖

#### 构造器循环依赖

通过构造器注入构成的循环依赖, 此依赖是无法解决的, 只能抛出 BeanCurrentlyInCreationException 异常表示循环依赖

如在创建 TestA 类时, 构造器需要 TestB 类, 那将去创建 TestB, 在创建 TestB 类时又发现需要 TestC 类, 则又去创建 TestC, 最终在创建 TestC 时发现又需要 TestA, 从而形成一个环, 没办法创建

Spring 容器将每一个正在创建的 bean 标识符放在一个 "当前创建 bean 池" 中, bean 标识符在创建过程中将一直保持在这个池中, 因此如果在创建 bean 过程中发现自己已经在 "当前创建 bean 池" 里时, 将抛出 BeanCurrentlyInCreationException 异常表示循环依赖, 而对于创建完毕的 bean 将从 "当前创建 bean 池" 中清除掉

#### setter 循环依赖

对于 setter 注入造成的依赖是通过 Spring 容器提前暴露刚完成构造器注入但未完成其他步骤 (如 setter 注入) 的 bean 来完成的, 而且只能解决单例作用域的 bean 循环依赖, 通过提前暴露一个单例工厂方法, 从而使其他 bean 能引用到该 bean

```java
addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
```

具体步骤

- Spring 容器创建单例 "testA" bean, 首先根据无参构造器创建 bean, 并暴露一个 "ObjectFactory" 用于返回一个提前暴露一个创建中的 bean, 并将 "testA" 标识符放到 "当前创建 bean 池" , 然后进行 setter 注入 "testB"
- Spring 容器创建单例 "testB" bean, 首先根据无参构造器创建 bean, 并暴露一个 "ObjectFactory" 用于返回一个提前暴露一个创建中的 bean, 并将 "testB" 标识符放到 "当前创建 bean 池" , 然后进行 setter 注入 "circle"
- Spring 容器创建单例 "testC" bean, 首先根据无参构造器创建 bean, 并暴露一个 "ObjectFactory" 用于返回一个提前暴露一个创建中的 bean, 并将 "testC" 标识符放到 "当前创建 bean 池" , 然后进行 setter 注入 "testA" , 进行注入 "testA" 时由于提前暴露了 "ObjectFactory" 工厂, 从而使用它返回提前暴露一个创建中的 bean
- 最后在依赖注入 "testB" 和 "testA" , 完成 setter 注入

#### prototype 范围的依赖处理

对于 "prototype" 作用域 bean, Spring 容器无法完成依赖注入, 因为 Spring 容器不进行缓存 "prototype" 作用域的 bean, 因此无法提前暴露一个创建中的 bean

对于 "singleton" 作用域 bean, 可以通过 "setAllowCircularReferences(false) , " 来禁用循环引用
