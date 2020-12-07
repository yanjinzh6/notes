---
title: Spring 4.x 整合 MyBatis 多数据类型配置
date: 2020-07-05 8:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: spring-mybatis-multi-db
---

## 简介

MyBatis 支持多数据源多数据库类型的配置, 但是对于各种复杂的自定义 SQL, 每种数据库有各自的实现和优化的方法, 这样就需要使用 MyBatis 的多数据库类型的特性了

<!-- more -->

## MyBatis 配置

修改 `mybatis-config.xml` 配置文件

```xml
<databaseIdProvider type="DB_VENDOR">
    <property name="MySQL" value="mysql"/>
    <property name="Oracle" value="oracle"/>
</databaseIdProvider>
```

然后在映射文件中就可以使用 `databaseId` 属性, 在 OGNL 表达式中可以使用 `_databaseId` 参数来判断是属于哪种数据库, 例如有个批量插入的实例

```xml
<insert id="insertBatch" parameterType="java.util.List" keyProperty="UUID" databaseId="mysql">
    INSERT INTO t_user
    (
      UUID,
      NAME
    )
    VALUES
    <foreach collection="list" item="item" index="index" open="(" close=")" separator="),(">
      #{item.uuid},
      #{item.name},
    </foreach>
</insert>

<insert id="insertBatch" parameterType="java.util.List" keyProperty="UUID" databaseId="oracle">
    INSERT INTO t_user
    (
      UUID,
      NAME
    )
    <foreach item="item" index="index" collection="list" separator="union all">
      (
      SELECT
        #{item.uuid},
        #{item.name}
      FROM DUAL
      )
    </foreach>
</insert>
```

也可以使用 if 语句

```xml
<insert id="insertBatch" parameterType="java.util.List" keyProperty="UUID">
    INSERT INTO t_user
    (
      UUID,
      NAME
    )
    <if test=" _databaseId == 'mysql'">
      VALUES
      <foreach collection="list" item="item" index="index" open="(" close=")" separator="),(">
        #{item.uuid},
        #{item.name},
      </foreach>
    </if>
    <if test=" _databaseId == 'oracle'">
      <foreach item="item" index="index" collection="list" separator="union all">
        (
        SELECT
          #{item.uuid},
          #{item.name}
        FROM DUAL
        )
      </foreach>
    </if>
</insert>
```

使用的是 `mybatis-3.4.0`

可以看到源码中需要配置环境才会进行设置

```java
// org.apache.ibatis.builder.xml.XMLConfigBuilder#databaseIdProviderElement
private void databaseIdProviderElement(XNode context) throws Exception {
  DatabaseIdProvider databaseIdProvider = null;
  if (context != null) {
    String type = context.getStringAttribute("type");
    // awful patch to keep backward compatibility
    if ("VENDOR".equals(type)) {
        type = "DB_VENDOR";
    }
    Properties properties = context.getChildrenAsProperties();
    databaseIdProvider = (DatabaseIdProvider) resolveClass(type).newInstance();
    databaseIdProvider.setProperties(properties);
  }
  Environment environment = configuration.getEnvironment();
  if (environment != null && databaseIdProvider != null) {
    String databaseId = databaseIdProvider.getDatabaseId(environment.getDataSource());
    configuration.setDatabaseId(databaseId);
  }
}
```

所以还需要在配置文件中添加环境相应的配置

```xml
<environments default="dev">
    <environment id="dev">
        <transactionManager type="JDBC" />
        <dataSource type="POOLED">
            <property name="driver" value="${driver}" />
            <property name="url" value="${url}" />
            <property name="username" value="${username}" />
            <property name="password" value="${password}" />
        </dataSource>
    </environment>
</environments>
```

## Spring 配置

与 Spring 集成时, 上面的配置会导致出现报错 `org.apache.ibatis.binding.BindingException: Invalid bound statement (not found)`, 使用的是 `mybatis-spring-1.3.0` 版本, 可以看到源码

```java
// mybatis-spring-1.3.0-sources.jar!/org/mybatis/spring/SqlSessionFactoryBean.java:479
if (this.databaseIdProvider != null) {//fix #64 set databaseId before parse mapper xmls
  try {
    configuration.setDatabaseId(this.databaseIdProvider.getDatabaseId(this.dataSource));
  } catch (SQLException e) {
    throw new NestedIOException("Failed getting a databaseId", e);
  }
}
```

可以看到 `SqlSessionFactoryBean` 中需要注入 `databaseIdProvider` 属性

所以在 Spring 的配置文件中添加

```xml
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <property name="dataSource" ref="dataSources"/>
  <property name="typeAliasesPackage" value="com.xxx"/>
  <property name="typeAliasesSuperType" value="com.xxx.common.bean.BaseEntity"/>
  <property name="mapperLocations" value="classpath*:/mappings/**/*.xml"/>
  <property name="configLocation" value="classpath:/spring/mybatis-config.xml"></property>
  <property name="databaseIdProvider" ref="databaseIdProvider"/>
</bean>

<!-- spring 支持 MyBatis 多数据库操作 -->
<bean id="vendorProperties" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
  <property name="properties">
    <props>
      <prop key="Oracle">oracle</prop>
      <prop key="MySQL">mysql</prop>
    </props>
  </property>
</bean>

<bean id="databaseIdProvider" class="org.apache.ibatis.mapping.VendorDatabaseIdProvider">
  <property name="properties" ref="vendorProperties"/>
</bean>
```
