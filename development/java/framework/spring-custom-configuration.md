---
title: Spring 自定义配置
date: 2020-07-19 12:00:00
tags: 'Spring'
categories:
  - ['开发', 'Java', '框架']
permalink: spring-custom-configuration
---

## 简介

程序中自定义的组件运行过程中需要到各种配置, 比较好的开发模式是将配置信息通过外置自定义文件进行保存, 在运行时传入组件即可

- 使用 Spring 的 [PropertyPlaceholderConfigurer](http://bit.ly/2riQBKw) 类可以使用配置文件中解析的值来替换 XML 配置的 bean 中的字符串
- [Environment](http://bit.ly/2s3GiHf) 接口提供了运行中应用程序及其运行时环境之间的隔离
  - @PropertySource 和 @Value 组合使用，可以将自定义属性文件中的属性变量值注入到当前类的使用@Value注解的成员变量中
  - @PropertySource 和 @ConfigurationProperties 组合使用，可以将属性文件与一个Java类绑定，将属性文件中的变量值注入到该Java类的成员变量中
- Environment 接口还引入了 [Profile](http://bit.ly/2s3RjbM) 的概念。它可以通过一些标签（Profile）对 bean 进行分类

<!-- more -->

## 加载配置属性

Spring Boot 的属性解析器非常智能，它会自动把驼峰规则的属性和使用连字符或下划线的同名属性关联起来

可以这么理解 `testId` 与 `test_id` 和 `test-id` 是同一个属性

```yaml
demo:
  testId: 'demo test'
```

```java
@Component
@ConfigurationProperties('demo')
@Getter
@Setter
public class AmazonProperties {
  // 注入相应的属性 demo.testId
  private String testId；

}
```

## 使用 Profile 进行配置

当应用程序需要部署到不同的运行环境时，一些配置细节通常会有所不同, Spring Framework 从 Spring 3.1 开始支持基于 Profile 的配置。Profile 是一种条件化配置，基于运行时激活的 Profile，会使用或者忽略不同的 Bean 或配置类, 当相应的 Profile 中没有自定义的参数, 便会使用自动配置的参数

使用 `spring.profiles.active` 属性激活 Profile

也可以通过 `@Profile` 注解来标识该类使用特定的环境配置

### 特定 Profile 配置文件

遵循 application-{profile}.properties 这种命名格式，这样就能提供特定于 Profile 的属性了

例如开发环境的配置文件 `application-development.properties`

```prop
logging.level.root=DEBUG
```

生产环境的配置文件 `application-production.properties`

```prop
logging.path=/var/logs/
logging.file=demo.log
logging.level.root=WARN
```

默认所有环境的缺省配置可以放在 `application.properties`

### 多 Profile YAML 文件

使用 YAML 来配置属性，则可以遵循与配置文件相同的命名规范，即创建 application{profile}.yml 这样的 YAML 文件，并将与 Profile 无关的属性继续放在 application.yml 里

也可以将所有的配置放在 application.yml 文件里, 通过 YAML 特殊的分隔符区分即可

```yaml
logging:
  level:
    root: INFO

---

spring:
  profiles: development

logging:
  level:
    root: DEBUG

---

spring:
  profiles: production

logging:
  path: /tmp/
  file: demo.log
  level:
    root: WARN
```
