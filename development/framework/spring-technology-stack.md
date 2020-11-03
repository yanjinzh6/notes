---
title: Spring 技术栈
date: 2020-07-19 14:00:00
tags: 'Spring'
categories:
  - ['开发', 'Java', '框架']
permalink: spring-technology-stack
---

## Spring 核心框架

Spring 核心框架提供了核心容器和依赖注入框架, 还提供了一些对数据持久化的基础支持
Spring MVC 也就是 Spring 的 Web 框架
Spring WebFlux 新反应式 Web 框架

## Spring Boot

提供了 starter 依赖和自动配置, 基本满足了开箱即用的开发流程, 还包括其他有用的特性:

- Actuator 能够洞察应用运行时的内部工作状况, 包括指标, 线程 dump 信息, 应用的健康状况以及应用可用的环境属性
- 灵活的环境属性规范
- 在核心框架的测试辅助功能之上提供了对测试的额外支持
- 基于 Groovy 脚本的编程模型的 Spring Boot 命令行接口 (Command-Line Interface, CLI) , 可以将整个应用程序编写为 Groovy 脚本的集合, 并通过命令行运行

## Spring Data

将应用程序的数据 repository 定义为简单的 Java 接口, 使用一种命名约定定义驱动存储和检索数据的方法, Spring Data 能够处理多种不同类型的数据库, 包括关系型数据库 (JPA) , 文档数据库 (Mongo) , 图数据库 (Neo4j) 等

## Spring Security

Spring Security 解决了应用程序通用的安全性需求, 包括身份验证, 授权和 API 安全性

## Spring Integration 和 Spring Batch

Spring Integration 负责实时集成问题, 数据在可用时马上就会得到处理
Spring Batch 负责批处理集成的问题, 在此过程中, 数据可以收集一段时间, 直到某个触发器 (可能是一个时间触发器) 发出信号, 表示该处理批量数据了才会对数据进行批处理

## Spring Cloud

应用程序开发领域正在进入一个新的时代, 使用由微服务组成的多个独立部署单元来组合形成应用程序, Spring Cloud 是使用 Spring 开发云原生应用程序的一组项目
