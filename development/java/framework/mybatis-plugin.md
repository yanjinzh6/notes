---
title: MyBatis 插件简介
date: 2020-04-25 18:00:00
tags: 'MyBatis'
categories:
  - ['开发', 'Java', ' 框架']
permalink: mybatis-plugin
---

## 简介

Mybatis 中提供了插件的功能, 通过拦截器 ( Interceptor ) 实现

## Interceptor

MyBatis 允许用户使用自定义拦截器对 SQL 语句执行过程中的某一点进行拦截, 可拦截方法如下

- Executor 中的 `update()` 方法, `query()` 方法, `flushStatements()` 方法, `commit()` 方法, `rollback()` 方法, `getTransaction()` 方法, `close()` 方法, `isClosed()` 方法,
- ParameterHandler 中的 `getParameterObject()` 方法, `setParameters()` 方法,
- ResultSetHandler 中的 `handleResultSets()` 方法, `handleOutputParameters()` 方法,
- StatementHandler 中的 `prepare()` 方法, `parameterize()` 方法, `batch()` 方法, `update()` 方法, `query()` 方法

<!-- more -->

Interceptor 接口是 MyBatis 插件模块的核心, 其定义如下

```java
public interface Interceptor {
  // 执行拦截器逻辑方法
  Object intercept(Invocation invocation) throws Throwable;

  // 决定是否触发 intercept 方法
  Object plugin(Object target);

  // 根据配置初始化 Interceptor 对象
  void setProperties(Properties properties);

}
```

`@Intercepts` 注解中指定了一个 `@Signature` 注解列表, 每个 `@Signature` 注解中都标识了该插件需要拦截的方法信息, 其中 `@Signature` 注解的 type 属性指定需要拦截的类型, method 属性指定需要拦截的方法, args 属性指定了被拦截方法的参数列表, 通过这三个属性值, `@Signature` 注解就可以表示一个方法签名, 唯一确定一个方法

```java
@Intercepts(
    @Signature(
        method = "query",
        type = Executor.class,
        args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}
    )
)
public class MyPlugin implements Interceptor {
}
```

定义完成一个自定义拦截器之后, 需要在 `mybatis-config.xml` 配置文件中对该拦截器进行配置

```xml
<plugins>
  <plugin interceptor="com.test.MyPlugin"></plugin>
</plugins>
```

MyBatis 初始化时, 会通过 `XMLConfigBuilder.pluginElement()` 方法解析 `mybatis-config.xml` 配置文件中定义的 `＜plugin＞` 节点, 得到相应的 Interceptor 对象以及配置的相应属性, 之后会调用 `Interceptor.setProperties( properties )` 方法完成对 Interceptor 对象的初始化配置, 最后将 Interceptor 对象添加到 `Configuration.interceptorChain` 字段中保存

MyBatis 中使用的 `Executor`, `ParameterHandler` , `ResultSetHandler` , `StatementHandler`, 都是通过 `Configuration.new*()` 系列方法创建的, 如果配置了用户自定义拦截器, 则会在该系列方法中, 通过 `InterceptorChain.pluginAll()` 方法为目标对象创建代理对象, 所以通过 `Configuration.new*()` 系列方法得到的对象实际是一个代理对象

```java
// org.apache.ibatis.session.Configuration#newExecutor(org.apache.ibatis.transaction.Transaction, org.apache.ibatis.session.ExecutorType)
public Executor newExecutor(Transaction transaction, ExecutorType executorType) {
  executorType = executorType == null ? defaultExecutorType : executorType;
  executorType = executorType == null ? ExecutorType.SIMPLE : executorType;
  Executor executor;
  // 根据参数, 选择合适的 Executor 实现
  if (ExecutorType.BATCH == executorType) {
    executor = new BatchExecutor(this, transaction);
  } else if (ExecutorType.REUSE == executorType) {
    executor = new ReuseExecutor(this, transaction);
  } else {
    executor = new SimpleExecutor(this, transaction);
  }
  // 根据配置决定是否开启二级缓存
  if (cacheEnabled) {
    executor = new CachingExecutor(executor);
  }
  // 通过 interceptorChain.pluginAll 创建 Executor 对象
  executor = (Executor) interceptorChain.pluginAll(executor);
  return executor;
}
```

InterceptorChain 中使用 `interceptors` 字段保存了 `mybatis-config.xml` 文件中配置的拦截器, 在 `InterceptorChain.pluginAll()` 方法中会遍历该 `interceptors` 集合, 并调用其中每个元素的 `plugin()` 方法创建代理对象

```java
// org.apache.ibatis.plugin.InterceptorChain
public class InterceptorChain {

  private final List<Interceptor> interceptors = new ArrayList<Interceptor>();

  public Object pluginAll(Object target) {
    for (Interceptor interceptor : interceptors) {
      target = interceptor.plugin(target);
    }
    return target;
  }

  public void addInterceptor(Interceptor interceptor) {
    interceptors.add(interceptor);
  }

  public List<Interceptor> getInterceptors() {
    return Collections.unmodifiableList(interceptors);
  }

}
```

用户自定义拦截器的 `plugin()` 方法, 可以考虑使用 MyBatis 提供的 `Plugin` 工具类实现, 它实现了 `InvocationHandler` 接口, 并提供了一个 `wrap()` 静态方法用于创建代理对象

```java
// org.apache.ibatis.plugin.Plugin#wrap
public static Object wrap(Object target, Interceptor interceptor) {
  // 获取自定义 Interceptor 的 @Signature 注解信息
  // getSignatureMap 处理注解内容
  Map<Class<?>, Set<Method>> signatureMap = getSignatureMap(interceptor);
  Class<?> type = target.getClass();
  // 获取目标类实现的接口
  // 拦截器可以拦截的 4 类对象都实现了相应的接口
  // 这也是使用 JDK 动态代理方式创建对象的基础
  Class<?>[] interfaces = getAllInterfaces(type, signatureMap);
  if (interfaces.length > 0) {
    // 使用 JDK 动态代理方式创建对象
    return Proxy.newProxyInstance(
        type.getClassLoader(),
        interfaces,
        // 使用 interceptor 对象的是 Plugin
        new Plugin(target, interceptor, signatureMap));
  }
  return target;
}
```

在 `Plugin.invoke()` 方法中, 会将当前调用的方法与 `signatureMap` 集合中记录的方法信息进行比较, 如果当前调用的方法是需要被拦截的方法, 则调用其 `intercept()` 方法进行处理, 如果不能被拦截则直接调用 target 的相应方法

```java
// org.apache.ibatis.plugin.Plugin#invoke
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
  try {
    // 获取当前方法所在类或接口中, 可被当前 Interceptor 拦截的方法
    Set<Method> methods = signatureMap.get(method.getDeclaringClass());
    // 如果当前调用的方法需要被拦截, 就调用 interceptor.intercept
    if (methods != null && methods.contains(method)) {
      return interceptor.intercept(new Invocation(target, method, args));
    }
    // 不需要则调用原本的方法
    return method.invoke(target, args);
  } catch (Exception e) {
    throw ExceptionUtil.unwrapThrowable(e);
  }
}
```

`Interceptor.intercept()` 方法的参数是 Invocation 对象, 其中封装了目标对象, 目标方法以及调用目标方法的参数, 并提供了 `proceed()` 方法调用目标方法, 在 `Interceptor.intercept()` 方法中执行完拦截处理之后, 如果需要调用目标方法, 则通过 `Invocation.proceed()` 方法实现

```java
// org.apache.ibatis.plugin.Invocation#proceed
public Object proceed() throws InvocationTargetException, IllegalAccessException {
  return method.invoke(target, args);
}
```

## 简单应用

```java
@Override
public Object intercept(Invocation invocation) throws Throwable {
    // 从 Invocation 中获取被拦截方法的参数列表
    // Executor query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler)
    final Object[] args = invocation.getArgs();
    // 获取 MappedStatement
    MappedStatement ms = (MappedStatement) args[0];
    // 这里可以根据 ms.id 判断方法是否需要处理
    // 根据 ms.SqlCommandType 可以判断当前方法类型
    // 获取用户传入参数
    Object params = args[1];
    // 获取 Rowbounds, 主要是记录 offset 和 limit
    RowBounds rb = (RowBounds) args[1];
    // 获取包含 ? 占位符的 SQL
    // 这里返回了 SQL 和相应的参数列表
    BoundSql boundSql = ms.getBoundSql(params);
    // 获取 sql 语句
    StringBuffer bufferSql = new StringBuffer(boundSql.getSql().trim());
    // 格式化 SQL, 很多时候自定义 SQL 里面有很多多余空格和换行等
    // 解析 SQL
    Statement statement = CCJSqlParserUtil.parse((bufferSql.toString().trim()));
    if (statement instanceof Select) {
        // 解析为 Select 语句
        Select select = (Select) statement;
        // 解析列
        // 解析表
        // 解析 where
        SelectBody selectBody = select.getSelectBody();
        PlainSelect plainSelect = (PlainSelect) selectBody;
        // where
        Expression where = plainSelect.getWhere();
        // from 后面的项和别名, 注意, 这里有可能是子查询
        FromItem fromItem = plainSelect.getFromItem();
        // 新增加相应的 Where 条件
        EqualsTo equalsTo = new EqualsTo();
        equalsTo.setLeftExpression(new Column(fromItem.getAlias() + ".CREATOR"));
        equalsTo.setRightExpression(new StringValue(user.getCreator()));

        if (where == null) {
            plainSelect.setWhere(equalsTo);
        } else {
            AndExpression and = new AndExpression(where, equalsTo);
            plainSelect.setWhere(and);
        }
        InExpression inExpression = new InExpression();
        inExpression.setLeftExpression(new Column(fromItem.getAlias() + ".DEPT_ID"));
        List<Expression> list = new ArrayList<Expression>();
        for (int i = 0; i < user.getDeptList().size(); i ++) {
            // StringValue 会直接去掉首尾 '
            list.add(new StringValue("'" + user.getDeptList().get(i) + "'"));
        }
        inExpression.setRightItemsList(new ExpressionList(list));

        if (where == null) {
            plainSelect.setWhere(inExpression);
        } else {
            AndExpression and = new AndExpression(where, inExpression);
            plainSelect.setWhere(and);
        }
        // plainSelect.toString() 保存着完整的 SQL
        // 下面采用注入的方式是无效的
        // 直接注入 boundSql
//            Field field = boundSql.getClass().getDeclaredField("sql");
//            field.setAccessible(true);
//            field.set(boundSql, plainSelect.toString());
        // 通过新的 SQL 返回新的 ms 对象, 并设置到 invocation.args 中
        // 这里的 SQL 只是简单拼接上, 需要使用占位符处理 ParameterMapping
        args[0] = updateMs(ms, params, plainSelect.toString());
    }
    // 这里如果不需要进行其他处理就直接执行原先的方法并返回
    return invocation.proceed();
}
/**
  * @param sql
  * @throws SQLException
  * @return
  */
private MappedStatement updateMs(MappedStatement statement, Object params, String sql) throws SQLException {
//        Object parameterObject = args[1];
    BoundSql boundSql = statement.getBoundSql(params);
    // 拼装新的参数列表
//        List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();
    MappedStatement newStatement = newMappedStatement(statement, new StaticSqlSource(statement.getConfiguration(), sql, boundSql.getParameterMappings()));
//        MetaObject msObject =  MetaObject.forObject(newStatement, new DefaultObjectFactory(), new DefaultObjectWrapperFactory(), new DefaultReflectorFactory());
//        msObject.setValue("sqlSource.boundSql.sql", sql);
    return newStatement;
}

private MappedStatement newMappedStatement(MappedStatement ms, SqlSource newSqlSource) {
    MappedStatement.Builder builder =
            new MappedStatement.Builder(ms.getConfiguration(), ms.getId(), newSqlSource, ms.getSqlCommandType());
    builder.resource(ms.getResource());
    builder.fetchSize(ms.getFetchSize());
    builder.statementType(ms.getStatementType());
    builder.keyGenerator(ms.getKeyGenerator());
    if (ms.getKeyProperties() != null && ms.getKeyProperties().length != 0) {
        StringBuilder keyProperties = new StringBuilder();
        for (String keyProperty : ms.getKeyProperties()) {
            keyProperties.append(keyProperty).append(",");
        }
        keyProperties.delete(keyProperties.length() - 1, keyProperties.length());
        builder.keyProperty(keyProperties.toString());
    }
    builder.timeout(ms.getTimeout());
    builder.parameterMap(ms.getParameterMap());
    builder.resultMaps(ms.getResultMaps());
    builder.resultSetType(ms.getResultSetType());
    builder.cache(ms.getCache());
    builder.flushCacheRequired(ms.isFlushCacheRequired());
    builder.useCache(ms.isUseCache());

    return builder.build();
}
```

## 参考

- [GitHub](https://github.com/mybatis/mybatis-3)
