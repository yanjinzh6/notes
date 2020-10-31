---
title: Maven 常用命令
date: 2020-10-25 09:00:00
tags: 'Maven'
categories:
  - ['开发', 'Java', '工具']
permalink: maven-common-commands
---

## Maven 生命周期

在 Maven 工程中, 已经默认定义了构建工程的生命周期, 可以通过引用插件来扩展相应的构建阶段

| 阶 段 | 描 述 |
| -- | -- |
| validate | 验证工程的完整性 |
| initialize | 初始化 |
| generate-sources | 生成源码 |
| process-sources | 处理源码 |
| generate-resources | 生成所有源码 |
| process-resources | 处理所有源码 |
| compile | 编译 |
| process-classes | 处理 class 文件 |
| generate-test-sources | 生成测试源码 |
| process-test-sources | 处理测试源码 |
| generate-test-resources | 生成所有测试源码 |
| process-test-resources | 处理所有测试源码 |
| test-compile | 测试编译 |
| process-test-classes | 处理测试 class 文件 |
| test | 测试 |
| prepare-package | 预打包 |
| package | 打包（如 Jar, War） |
| pre-integration-test | 预集成测试 |
| integration-test | 集成测试 |
| post-integration-test | 完成集成测试 |
| verify | 验证 |
| install | 安装到本地仓库 |
| deploy | 提交到远程仓库 |

## 常用 Maven 命令

- `mvn clean` 删除工程的 target 目录下的所有文件
- `mvn package` 将工程打为 Jar 包
- `mvn package -Dmaven.test.skip=ture`, 打包时跳过测试
- `mvn compile` 编译工程代码, 不生成 Jar 包
- `mvn install` 命令包含了 `mvn package` 的所有过程, 并且将生成的 Jar 包安装到本地仓库
- `mvn spring-boot:run` 使用 spring-boot 插件, 启动 Spring Boot 工程, 该命令执行时先检查 Spring Boot 工程源码是否编译, 如果工程源码没有编译, 则先编译；如果编译了, 则启动工程
- `mvn test` 测试
- `mvn idea:idea` 生成 idea 项目
- `mvn jar:jar` 只打 Jar 包
- `mvn validate` 检验资源是否可用

`mvn package` 打包命令不是一个简单的命令, 它是由一系列有序的命令构成的, `mvn package` 命令执行过程包含以下 6 个阶段

- 验证
- 编译代码
- 处理代码
- 生成资源文件
- 生成 Jar 包
- 测试
