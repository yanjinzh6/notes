---
title: Spring 自定义配置
date: 2020-07-19 12:00:00
tags: 'Spring'
categories:
  - ['开发', 'Java', ' 框架']
permalink: spring-custom-configuration
---

## 简介

程序中自定义的组件运行过程中需要到各种配置, 比较好的开发模式是将配置信息通过外置自定义文件进行保存, 在运行时传入组件即可

- 使用 Spring 的 [PropertyPlaceholderConfigurer](http://bit.ly/2riQBKw) 类可以使用配置文件中解析的值来替换 XML 配置的 bean 中的字符串
- [Environment](http://bit.ly/2s3GiHf) 接口提供了运行中应用程序及其运行时环境之间的隔离
  - @PropertySource 和 @Value 组合使用，可以将自定义属性文件中的属性变量值注入到当前类的使用@Value注解的成员变量中
  - @PropertySource 和 @ConfigurationProperties 组合使用，可以将属性文件与一个Java类绑定，将属性文件中的变量值注入到该Java类的成员变量中
- Environment 接口还引入了 [Profile](http://bit.ly/2s3RjbM) 的概念。它可以通过一些标签（Profile）对 bean 进行分类
