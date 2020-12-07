---
title: Spring 创建 Bean
date: 2020-05-05 20:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: spring-do-create-bean
---

## doCreateBean

经过 resolveBeforeInstantiation 方法后, 如果创建了代理或者说重写了 InstantiationAwareBeanPostProcessor 的 postProcessBeforeInstantiation 方法并在方法 postProcessBeforeInstantiation 中改变了 bean, 则直接返回就可以了, 否则需要进行常规 bean 的创建, 常规 bean 的创建就是在 doCreateBean 中完成的

<!-- more -->

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean
/**
  * Actually create the specified bean. Pre-creation processing has already happened
  * at this point, e.g. checking {@code postProcessBeforeInstantiation} callbacks.
  * <p>Differentiates between default bean instantiation, use of a
  * factory method, and autowiring a constructor.
  * @param beanName the name of the bean
  * @param mbd the merged bean definition for the bean
  * @param args explicit arguments to use for constructor or factory method invocation
  * @return a new instance of the bean
  * @throws BeanCreationException if the bean could not be created
  * @see #instantiateBean
  * @see #instantiateUsingFactoryMethod
  * @see #autowireConstructor
  */
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final @Nullable Object[] args)
    throws BeanCreationException {

  // Instantiate the bean.
  BeanWrapper instanceWrapper = null;
  if (mbd.isSingleton()) {
    // 如果是单例则需要首先处理缓存
    instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
  }
  if (instanceWrapper == null) {
    // 根据指定 bean 使用对应的策略创建新的实例, 如: 工厂方法、构造函数自动注入、简单初始化
    // 实例化 bean, 将 BeanDefinition 转换为 BeanWrapper
    instanceWrapper = createBeanInstance(beanName, mbd, args);
  }
  final Object bean = instanceWrapper.getWrappedInstance();
  Class<?> beanType = instanceWrapper.getWrappedClass();
  if (beanType != NullBean.class) {
    mbd.resolvedTargetType = beanType;
  }

  // Allow post-processors to modify the merged bean definition.
  synchronized (mbd.postProcessingLock) {
    if (!mbd.postProcessed) {
      try {
        // 应用 MergedBeanDefinitionPostProcessor
        // bean 合并后的处理, Autowired 注解正是通过此方法实现诸如类型的预解析
        applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
      }
      catch (Throwable ex) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
            "Post-processing of merged bean definition failed", ex);
      }
      mbd.postProcessed = true;
    }
  }

  // Eagerly cache singletons to be able to resolve circular references
  // even when triggered by lifecycle interfaces like BeanFactoryAware.
  // 是否需要提早曝光: 判断条件 (单例 && 允许循环依赖 && 当前 bean 正在创建中), 检测循环依赖
  boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
      isSingletonCurrentlyInCreation(beanName));
  if (earlySingletonExposure) {
    if (logger.isTraceEnabled()) {
      logger.trace("Eagerly caching bean '" + beanName +
          "' to allow for resolving potential circular references");
    }
    // 为避免后期循环依赖, 可以在 bean 初始化完成前将创建实例的 ObjectFactory 加入工厂
    // 对 bean 再一次依赖引用, 主要应用 SmartInstantiationAware BeanPost Processor
    // AOP 就是在这里将 advice 动态织入 bean 中, 若没有则直接返回bean, 不做任何处理
    addSingletonFactory(beanName, () -> getEarlyBeanReference(beanName, mbd, bean));
  }

  // Initialize the bean instance.
  Object exposedObject = bean;
  try {
    // 对 bean 进行填充, 将各个属性值注入, 其中, 可能存在依赖于其他 bean 的属性, 则会递归初始依赖 bean
    populateBean(beanName, mbd, instanceWrapper);
    // 调用初始化方法, 比如 init-method
    exposedObject = initializeBean(beanName, exposedObject, mbd);
  }
  catch (Throwable ex) {
    if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
      throw (BeanCreationException) ex;
    }
    else {
      throw new BeanCreationException(
          mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
    }
  }

  if (earlySingletonExposure) {
    Object earlySingletonReference = getSingleton(beanName, false);
    // earlySingletonReference 只有在检测到有循环依赖的情况下才会不为空
    if (earlySingletonReference != null) {
      // 如果 exposedObject 没有在初始化方法中被改变, 也就是没有被增强
      if (exposedObject == bean) {
        exposedObject = earlySingletonReference;
      }
      else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
        String[] dependentBeans = getDependentBeans(beanName);
        Set<String> actualDependentBeans = new LinkedHashSet<>(dependentBeans.length);
        for (String dependentBean : dependentBeans) {
          // 检测依赖
          if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
            actualDependentBeans.add(dependentBean);
          }
        }
        // Sping 中解决循环依赖只对单例有效, 对于 prototype 的 bean 抛出异常处理, 在这个步骤里面会检测已经加载的 bean 是否已经出现了依赖循环, 并判断是否需要抛出异常
        // 因为 bean 创建后其所依赖的 bean 一定是已经创建的
        // actualDependentBeans 不为空则表示当前 bean 创建后其依赖的 bean 却没有没全部创建完, 就存在循环依赖
        if (!actualDependentBeans.isEmpty()) {
          throw new BeanCurrentlyInCreationException(beanName,
              "Bean with name '" + beanName + "' has been injected into other beans [" +
              StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
              "] in its raw version as part of a circular reference, but has eventually been " +
              "wrapped. This means that said other beans do not use the final version of the " +
              "bean. This is often the result of over-eager type matching - consider using " +
              "'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
        }
      }
    }
  }

  // Register bean as disposable.
  try {
    // 根据 scopse 注册 bean
    // 如果配置了 destroy-method, 这里需要注册以便于在销毁时候调用
    registerDisposableBeanIfNecessary(beanName, bean, mbd);
  }
  catch (BeanDefinitionValidationException ex) {
    throw new BeanCreationException(
        mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
  }

  return exposedObject;
}
```

```java
/**
  * Create a new instance for the specified bean, using an appropriate instantiation strategy:
  * factory method, constructor autowiring, or simple instantiation.
  * @param beanName the name of the bean
  * @param mbd the bean definition for the bean
  * @param args explicit arguments to use for constructor or factory method invocation
  * @return a BeanWrapper for the new instance
  * @see #obtainFromSupplier
  * @see #instantiateUsingFactoryMethod
  * @see #autowireConstructor
  * @see #instantiateBean
  */
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, @Nullable Object[] args) {
  // Make sure bean class is actually resolved at this point.
  // 解析 class
  Class<?> beanClass = resolveBeanClass(mbd, beanName);

  if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
    throw new BeanCreationException(mbd.getResourceDescription(), beanName,
        "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
  }

  Supplier<?> instanceSupplier = mbd.getInstanceSupplier();
  // 实例供应商不为空则使用
  if (instanceSupplier != null) {
    return obtainFromSupplier(instanceSupplier, beanName);
  }
  // 如果工厂方法不为空则使用工厂方法初始化策略
  if (mbd.getFactoryMethodName() != null) {
    return instantiateUsingFactoryMethod(beanName, mbd, args);
  }

  // Shortcut when re-creating the same bean...
  boolean resolved = false;
  boolean autowireNecessary = false;
  if (args == null) {
    synchronized (mbd.constructorArgumentLock) {
      // 一个类有多个构造函数, 每个构造函数都有不同的参数, 所以调用前需要先根据参数锁定构造函数或对应的工厂方法
      if (mbd.resolvedConstructorOrFactoryMethod != null) {
        resolved = true;
        autowireNecessary = mbd.constructorArgumentsResolved;
      }
    }
  }
  // 如果已经解析过则使用解析好的构造函数方法不需要再次锁定
  if (resolved) {
    if (autowireNecessary) {
      // 构造函数自动注入
      return autowireConstructor(beanName, mbd, null, null);
    }
    else {
      // 使用默认构造函数构造
      return instantiateBean(beanName, mbd);
    }
  }

  // Candidate constructors for autowiring?
  // 如果一个类有多个构造函数, 每个构造函数都有不同的参数, 需要根据参数锁定构造函数并进行初始化
  Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
  if (ctors != null || mbd.getResolvedAutowireMode() == AUTOWIRE_CONSTRUCTOR ||
      mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args)) {
    // 构造函数自动注入
    return autowireConstructor(beanName, mbd, ctors, args);
  }

  // Preferred constructors for default construction?
  ctors = mbd.getPreferredConstructors();
  if (ctors != null) {
    // 构造函数自动注入
    return autowireConstructor(beanName, mbd, ctors, null);
  }

  // No special handling: simply use no-arg constructor.
  // 如果既不存在工厂方法也不存在带有参数的构造函数, 则使用默认的构造函数进行 bean 的实例化
  return instantiateBean(beanName, mbd);
}
```

### 带参数构造方法

带有参数的实例化过程复杂, 因为存在着不确定性, 所以在判断对应参数上做了大量工作

```java
/**
  * "autowire constructor" (with constructor arguments by type) behavior.
  * Also applied if explicit constructor argument values are specified,
  * matching all remaining arguments with beans from the bean factory.
  * <p>This corresponds to constructor injection: In this mode, a Spring
  * bean factory is able to host components that expect constructor-based
  * dependency resolution.
  * @param beanName the name of the bean
  * @param mbd the merged bean definition for the bean
  * @param chosenCtors chosen candidate constructors (or {@code null} if none)
  * @param explicitArgs argument values passed in programmatically via the getBean method,
  * or {@code null} if none (-> use constructor argument values from bean definition)
  * @return a BeanWrapper for the new instance
  */
public BeanWrapper autowireConstructor(String beanName, RootBeanDefinition mbd,
    @Nullable Constructor<?>[] chosenCtors, @Nullable Object[] explicitArgs) {

  BeanWrapperImpl bw = new BeanWrapperImpl();
  this.beanFactory.initBeanWrapper(bw);

  Constructor<?> constructorToUse = null;
  ArgumentsHolder argsHolderToUse = null;
  Object[] argsToUse = null;
  // explicitArgs 通过 getBean 方法传入
  // 如果 getBean 方法调用的时候指定方法参数那么直接使用
  if (explicitArgs != null) {
    argsToUse = explicitArgs;
  }
  else {
    // 如果在 getBean 方法时候没有指定则尝试从配置文件中解析
    Object[] argsToResolve = null;
    // 尝试从缓存中获取
    synchronized (mbd.constructorArgumentLock) {
      constructorToUse = (Constructor<?>) mbd.resolvedConstructorOrFactoryMethod;
      if (constructorToUse != null && mbd.constructorArgumentsResolved) {
        // Found a cached constructor...
        // 从缓存中取
        argsToUse = mbd.resolvedConstructorArguments;
        if (argsToUse == null) {
          // 配置的构造函数参数
          argsToResolve = mbd.preparedConstructorArguments;
        }
      }
    }
    // 如果缓存中存在
    if (argsToResolve != null) {
      // 解析参数类型, 如给定方法的构造函数 A (int,int) 则通过此方法后就会把配置中的
      // ( "1" ,  "1" ) 转换为 (1,1)
      // 缓存中的值可能是原始值也可能是最终值
      argsToUse = resolvePreparedArguments(beanName, mbd, bw, constructorToUse, argsToResolve, true);
    }
  }
  // 没有被缓存
  if (constructorToUse == null || argsToUse == null) {
    // Take specified constructors, if any.
    Constructor<?>[] candidates = chosenCtors;
    if (candidates == null) {
      Class<?> beanClass = mbd.getBeanClass();
      try {
        candidates = (mbd.isNonPublicAccessAllowed() ?
            beanClass.getDeclaredConstructors() : beanClass.getConstructors());
      }
      catch (Throwable ex) {
        throw new BeanCreationException(mbd.getResourceDescription(), beanName,
            "Resolution of declared constructors on bean Class [" + beanClass.getName() +
            "] from ClassLoader [" + beanClass.getClassLoader() + "] failed", ex);
      }
    }

    if (candidates.length == 1 && explicitArgs == null && !mbd.hasConstructorArgumentValues()) {
      Constructor<?> uniqueCandidate = candidates[0];
      if (uniqueCandidate.getParameterCount() == 0) {
        synchronized (mbd.constructorArgumentLock) {
          mbd.resolvedConstructorOrFactoryMethod = uniqueCandidate;
          mbd.constructorArgumentsResolved = true;
          mbd.resolvedConstructorArguments = EMPTY_ARGS;
        }
        bw.setBeanInstance(instantiate(beanName, mbd, uniqueCandidate, EMPTY_ARGS));
        return bw;
      }
    }

    // Need to resolve the constructor.
    boolean autowiring = (chosenCtors != null ||
        mbd.getResolvedAutowireMode() == AutowireCapableBeanFactory.AUTOWIRE_CONSTRUCTOR);
    ConstructorArgumentValues resolvedValues = null;

    int minNrOfArgs;
    if (explicitArgs != null) {
      minNrOfArgs = explicitArgs.length;
    }
    else {
      // 提取配置文件中的配置的构造函数参数
      ConstructorArgumentValues cargs = mbd.getConstructorArgumentValues();
      // 用于承载解析后的构造函数参数的值
      resolvedValues = new ConstructorArgumentValues();
      // 能解析到的参数个数
      minNrOfArgs = resolveConstructorArguments(beanName, mbd, bw, cargs, resolvedValues);
    }
    // 排序给定的构造函数, public 构造函数优先参数数量降序、非 public 构造函数参数数量降序
    AutowireUtils.sortConstructors(candidates);
    int minTypeDiffWeight = Integer.MAX_VALUE;
    Set<Constructor<?>> ambiguousConstructors = null;
    LinkedList<UnsatisfiedDependencyException> causes = null;

    for (Constructor<?> candidate : candidates) {
      Class<?>[] paramTypes = candidate.getParameterTypes();

      if (constructorToUse != null && argsToUse != null && argsToUse.length > paramTypes.length) {
        // Already found greedy constructor that can be satisfied ->
        // do not look any further, there are only less greedy constructors left.
        // 如果已经找到选用的构造函数或者需要的参数个数小于当前的构造函数参数个数则终止, 已经按照参数个数降序排列
        break;
      }
      if (paramTypes.length < minNrOfArgs) {
        // 参数个数不相等
        continue;
      }

      ArgumentsHolder argsHolder;
      if (resolvedValues != null) {
        // 有参数则根据值构造对应参数类型的参数
        try {
          // 注释上获取参数名称
          String[] paramNames = ConstructorPropertiesChecker.evaluate(candidate, paramTypes.length);
          if (paramNames == null) {
            // 获取参数名称探索器
            ParameterNameDiscoverer pnd = this.beanFactory.getParameterNameDiscoverer();
            if (pnd != null) {
              // 获取指定构造函数的参数名称
              paramNames = pnd.getParameterNames(candidate);
            }
          }
          // 根据名称和数据类型创建参数持有者
          argsHolder = createArgumentArray(beanName, mbd, resolvedValues, bw, paramTypes, paramNames,
              getUserDeclaredConstructor(candidate), autowiring, candidates.length == 1);
        }
        catch (UnsatisfiedDependencyException ex) {
          if (logger.isTraceEnabled()) {
            logger.trace("Ignoring constructor [" + candidate + "] of bean '" + beanName + "': " + ex);
          }
          // Swallow and try next constructor.
          if (causes == null) {
            causes = new LinkedList<>();
          }
          causes.add(ex);
          continue;
        }
      }
      else {
        // Explicit arguments given -> arguments length must match exactly.
        if (paramTypes.length != explicitArgs.length) {
          continue;
        }
        // 构造函数没有参数的情况
        argsHolder = new ArgumentsHolder(explicitArgs);
      }
      // 探测是否有不确定性的构造函数存在, 例如不同构造函数的参数为父子关系
      int typeDiffWeight = (mbd.isLenientConstructorResolution() ?
          argsHolder.getTypeDifferenceWeight(paramTypes) : argsHolder.getAssignabilityWeight(paramTypes));
      // Choose this constructor if it represents the closest match.
      // 如果它代表着当前最接近的匹配则选择作为构造函数
      if (typeDiffWeight < minTypeDiffWeight) {
        constructorToUse = candidate;
        argsHolderToUse = argsHolder;
        argsToUse = argsHolder.arguments;
        minTypeDiffWeight = typeDiffWeight;
        ambiguousConstructors = null;
      }
      else if (constructorToUse != null && typeDiffWeight == minTypeDiffWeight) {
        if (ambiguousConstructors == null) {
          ambiguousConstructors = new LinkedHashSet<>();
          ambiguousConstructors.add(constructorToUse);
        }
        ambiguousConstructors.add(candidate);
      }
    }

    if (constructorToUse == null) {
      if (causes != null) {
        UnsatisfiedDependencyException ex = causes.removeLast();
        for (Exception cause : causes) {
          this.beanFactory.onSuppressedException(cause);
        }
        throw ex;
      }
      throw new BeanCreationException(mbd.getResourceDescription(), beanName,
          "Could not resolve matching constructor " +
          "(hint: specify index/type/name arguments for simple parameters to avoid type ambiguities)");
    }
    else if (ambiguousConstructors != null && !mbd.isLenientConstructorResolution()) {
      throw new BeanCreationException(mbd.getResourceDescription(), beanName,
          "Ambiguous constructor matches found in bean '" + beanName + "' " +
          "(hint: specify index/type/name arguments for simple parameters to avoid type ambiguities): " +
          ambiguousConstructors);
    }

    if (explicitArgs == null && argsHolderToUse != null) {
      // 将解析的构造函数加入缓存
      argsHolderToUse.storeCache(mbd, constructorToUse);
    }
  }

  Assert.state(argsToUse != null, "Unresolved constructor arguments");
  // 将构建的实例加入 BeanWrapper 中
  bw.setBeanInstance(instantiate(beanName, mbd, constructorToUse, argsToUse));
  return bw;
}
```

- 构造函数参数的确定,
  - 根据 explicitArgs 参数判断: 如果传入的参数 explicitArgs 不为空, 可以确定参数, 因为 explicitArgs 参数是在调用 Bean 的时候用户指定的, 在 BeanFactory 类中存在这样的方法, `Object getBean(String name, Object... args) throws BeansException;`, 在获取 bean 的时候, 用户不但可以指定 bean 的名称还可以指定 bean 所对应类的构造函数或者工厂方法的方法参数, 主要用于静态工厂方法的调用, 而这里是需要给定完全匹配的参数, 所以可以判断, 如果传入参数 explicitArgs 不为空, 则可以确定构造函数参数就是 explicitArgs,
  - 缓存中获取: 构造函数参数已经记录在缓存中, 可以直接拿来使用, 在缓存中缓存的可能是参数的最终类型也可能是参数的初始类型, 例如: 构造函数参数要求的是 int 类型, 但是原始的参数值可能是 String 类型的 "1" , 那么从缓存中得到了参数, 需要经过类型转换器的过滤以确保参数类型与对应的构造函数参数类型对应,
  - 配置文件获取: 不能根据传入的参数 explicitArgs 确定构造函数的参数也无法在缓存中得到相关信息, 那么只能分析从获取配置文件中配置的构造函数信息开始, 经过之前的分析, Spring 中配置文件中的信息经过转换都会通过 BeanDefinition 实例承载, 也就是参数 mbd 中包含, 那么可以通过调用 mbd.getConstructorArgumentValues()来获取配置的构造函数信息, 有了配置中的信息便可以获取对应的参数值信息了, 获取参数值的信息包括直接指定值, 如: 直接指定构造函数中某个值为原始类型 String 类型, 或者是一个对其他 bean 的引用, 而这一处理委托给 resolveConstructorArguments 方法, 并返回能解析到的参数的个数,
- 构造函数的确定: 确定了构造函数的参数, 接下来的任务就是根据构造函数参数在所有构造函数中锁定对应的构造函数, 而匹配的方法就是根据参数个数匹配, 所以在匹配之前需要先对构造函数按照 public 构造函数优先参数数量降序、非 public 构造函数参数数量降序, 这样可以在遍历的情况下迅速判断排在后面的构造函数参数个数是否符合条件,
  - 由于在配置文件中并不是唯一限制使用参数位置索引的方式去创建, 同样还支持指定参数名称进行设定参数值的情况, 如 <constructor-arg name= "aa" >, 那么这种情况就需要首先确定构造函数中的参数名称, 获取参数名称可以有两种方式, 一种是通过注解的方式直接获取, 另一种就是使用 Spring 中提供的工具类 ParameterNameDiscoverer 来获取, 构造函数、参数名称、参数类型、参数值都确定后就可以锁定构造函数以及转换对应的参数类型了,
- 根据确定的构造函数转换对应的参数类型: 使用 Spring 中提供的类型转换器或者用户提供的自定义类型转换器进行转换,
- 构造函数不确定性的验证: 有时候即使构造函数、参数名称、参数类型、参数值都确定后也不一定会直接锁定构造函数, 不同构造函数的参数为父子关系, 所以 Spring 在最后又做了一次验证,
- 根据实例化策略以及得到的构造函数及构造函数参数实例化 Bean,

### 默认构造方法

```java
// org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#instantiateBean
/**
  * Instantiate the given bean using its default constructor.
  * @param beanName the name of the bean
  * @param mbd the bean definition for the bean
  * @return a BeanWrapper for the new instance
  */
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
  try {
    Object beanInstance;
    final BeanFactory parent = this;
    if (System.getSecurityManager() != null) {
      beanInstance = AccessController.doPrivileged((PrivilegedAction<Object>) () ->
          getInstantiationStrategy().instantiate(mbd, beanName, parent),
          getAccessControlContext());
    }
    else {
      // 直接调用实例化策略
      beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
    }
    BeanWrapper bw = new BeanWrapperImpl(beanInstance);
    initBeanWrapper(bw);
    return bw;
  }
  catch (Throwable ex) {
    throw new BeanCreationException(
        mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
  }
}
```

### 实例化策略

```java
// org.springframework.beans.factory.support.SimpleInstantiationStrategy#instantiate(org.springframework.beans.factory.support.RootBeanDefinition, java.lang.String, org.springframework.beans.factory.BeanFactory)
@Override
public Object instantiate(RootBeanDefinition bd, @Nullable String beanName, BeanFactory owner) {
  // Don't override the class with CGLIB if no overrides.
  // 有需要覆盖或者动态替换的方法则需要使用 cglib 进行动态代理, 因为可以在创建代理的同时将动态方法织入类中
  // 如果没有需要动态改变得方法, 为了方便直接反射就可以了
  if (!bd.hasMethodOverrides()) {
    Constructor<?> constructorToUse;
    synchronized (bd.constructorArgumentLock) {
      constructorToUse = (Constructor<?>) bd.resolvedConstructorOrFactoryMethod;
      if (constructorToUse == null) {
        final Class<?> clazz = bd.getBeanClass();
        if (clazz.isInterface()) {
          throw new BeanInstantiationException(clazz, "Specified class is an interface");
        }
        try {
          if (System.getSecurityManager() != null) {
            constructorToUse = AccessController.doPrivileged(
                (PrivilegedExceptionAction<Constructor<?>>) clazz::getDeclaredConstructor);
          }
          else {
            constructorToUse = clazz.getDeclaredConstructor();
          }
          bd.resolvedConstructorOrFactoryMethod = constructorToUse;
        }
        catch (Throwable ex) {
          throw new BeanInstantiationException(clazz, "No default constructor found", ex);
        }
      }
    }
    return BeanUtils.instantiateClass(constructorToUse);
  }
  else {
    // Must generate CGLIB subclass.
    return instantiateWithMethodInjection(bd, beanName, owner);
  }
}
```

- 没有使用 replace 或者 lookup 的配置方法, 那么直接使用反射的方式
- 使用了这两个特性, 需要将这两个配置提供的功能切入进去, 必须要使用动态代理的方式将包含两个特性所对应的逻辑的拦截增强器设置进去, 这样才可以保证在调用方法的时候会被相应的拦截器增强, 返回值为包含拦截器的代理实例
