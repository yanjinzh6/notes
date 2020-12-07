---
title: Spring 外置参数配置
date: 2020-07-19 10:00:00
tags: 'Spring'
categories:
  - ['开发', '框架']
permalink: spring-external-parameter-configuration
---

## 简介

Spring boot 可以通过获取外部参数配置来覆盖自动配置, 可以通过各种方式来微调内置的各种属性配置, 事实上, Spring Boot 自动配置的 Bean 提供了 300 多个用于微调的属性, 只要在环境变量、Java 系统属性、JNDI（Java Naming and Directory Interface）、命令行参数或者属性文件里进行指定就可以了, 大大方便了定制开发

## 配置方式

1. 命令行参数
1. java:comp/env 里的 JNDI 属性
1. JVM 系统属性
1. 操作系统环境变量
1. 随机生成的带 random.*前缀的属性（在设置其他属性时, 可以引用它们, 比如${random.long}）
1. 应用程序以外的 application.properties 或者 appliaction.yml 文件
1. 打包在应用程序内的 application.properties 或者 appliaction.yml 文件
1. 通过@PropertySource 标注的属性源
1. 默认属性

Spring boot 获取配置是按照列表优先级的顺序, 任何在高优先级属性源里设置的属性都会覆盖掉低优先级的相同属性

application.properties 和 application.yml 文件能放在以下四个位置

1. 外置, 在相对于应用程序运行目录的 `/config` 子目录里
1. 外置, 在应用程序运行的目录里
1. 内置, 在 config 包内
1. 内置, 在 Classpath 根目录

获取配置文件的优先级也是按照列表的顺序, 也就是外置优先, 同已配置下的 `/config` 目录下的配置优先于 Classpath 的相同配置. 同一优先级位置中的 `application.yml` 优先于 `application.properties`

<!-- more -->

## 示例

通过命令行的方式设置 `spring.main.show-banner=false`

```sh
java -jar demo-0.0.1-SNAPSHOT.jar --spring.main.show-banner=false
```

通过环境变量的方式设置 `spring.main.show-banner=false`, 注意: **这里需要遵守环境变量命名要求, 使用下划线替换点和横杠**

```sh
export spring_main_show_banner=false
```

通过 YAML 配置文件的方式设置 `spring.main.show-banner=false`

```yaml
spring:
  main:
    show-banner: false
```

通过配置文件的方式设置 `spring.main.show-banner=false`

```prop
spring.main.show-banner=false
```

## 部分实用属性

### 禁用模板缓存

开发过程中的各种缓存就是噩梦般的存在, 所以可以在开发环境的配置文件中添加如下配置

```yaml
spring:
  thymeleaf:
    cache: false
```

或者每次执行时使用参数

```sh
java -jar demo-0.0.1-SNAPSHOT.jar --spring.thymeleaf.cache=false
```

当然在开发机器上添加环境变量也是非常简单的

```sh
export spring_thymeleaf_cache=false
```

各种模板的缓存参数如下, 默认都会开启, 只需要设置为 false 即可以关闭

- Freemarker: spring.freemarker.cache
- Groovy 模板: spring.groovy.template.cache
- Velocity: spring.velocity.cache

### 配置嵌入式服务器

```yaml
server:
  port: 8443
  ssl:
    key-store: file:///path/to/mykeys.jks
    key-store-password: letmein
    key-password: letmein
```

此处的 server.port 设置为 8443, 开发环境的 HTTPS 服务器大多会选这个端口, server.ssl.key-store 属性指向密钥存储文件的存放路径, 这里用了一个 file://开头的 URL, 从文件系统里加载该文件, 你也可以把它打包在应用程序的 JAR 文件里, 用 classpath: URL 来引用它, server.ssl.key-store-password 和 server.ssl.key-password 设置为创建该文件时给定的密码

### 配置日志

默认情况下, Spring Boot 会用 [Logback](http://logback.qos.ch) 来记录日志, 并用 INFO 级别输出到控制台

要完全使用自定义日志配置, 可以在 Classpath 的根目录（src/main/resources）里创建 logback.xml 文件, 简单示例如下

```xml
<configuration>
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
    <encoder>
      <pattern>
        %d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n
      </pattern>
    </encoder>
  </appender>

  <logger name="root" level="INFO"/>

  <root level="INFO">
    <appender-ref ref="STDOUT" />
  </root>
</configuration>
```

可以通过 logging.config 属性指定自定义日志配置文件名称

```yaml
logging:
  config:
    classpath:logging-config.xml
```

如果仅需要控制日志级别或者指定日志输出, 可以使用 Spring Boot 的配置属性

设置日志级别需要创建 logging.level 开头的属性, 后面是需要输出日志的包名和输出日志级别, 如下配置定义根日志级别要设置为 WARN, 但 Spring Security 的日志要用 DEBUG 级别

```yaml
logging:
  level:
    root: WARN
    # org.springframework.security: DEBUG
    org:
      springframework:
        security: DEBUG
```

使用 logging.path 和 loggin.file 属性可以将日志写到相应的目录和文件中

```yaml
logging:
  path: /var/logs/
  file: demo.log
  level:
    root: WARN
    org:
      springframework:
        security: DEBUG
```

#### 用其他日志实现替换 Logback

使用 Log4j 或者 Log4j2，只需要修改依赖，引入对应该日志实现的起步依赖，同时排除掉 Logback

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter</artifactId>
  <exclusions>
    <exclusion>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-logging</artifactId>
    </exclusion>
  </exclusions>
</dependency>

<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-log4j</artifactId>
</dependency>
```

```gradle
configurations {
  all*.exclude group:'org.springframework.boot',
               module:'spring-boot-starter-logging'
}

compile("org.springframework.boot:spring-boot-starter-log4j")
```

### 配置数据源

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost/demo
    username: dbuser
    password: dbpass
```

spring.datasource.driver-class-name 属性可以指定数据库驱动名称

在自动配置 DataSource Bean 的时候, Spring Boot 会使用这里的连接数据, DataSource Bean 是一个连接池, 如果 Classpath 里有 Tomcat 的连接池 DataSource, 那么就会使用这个连接池, 否则, Spring Boot 会在 Classpath 里查找以下连接池

- HikariCP
- Commons DBCP
- Commons DBCP 2

这里列出的只是自动配置支持的连接池, 还可以配置 DataSource Bean, 自定义各种连接池
