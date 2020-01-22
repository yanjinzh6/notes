---
title: SkyWalking 简介
date: 2020-1-16 16:00:00
tags: 'SkyWalking'
categories:
  - ['使用说明', '软件']
permalink: skywalking-introduction
photo:
---

# 简介

> 文档仅对官方文档进行筛选和整合, 需要后续整理好逻辑

## 什么是 SkyWalking

SkyWalking：一个开源的观测平台, 用于从服务和云原生基础设施收集, 分析, 聚合以及可视化数据.

## 为什么使用 SkyWalking

观察和监控分布式系统

- 对 Java, C#, NodeJS 提供自动打点代理
- 对 Go, C++提供手动打点 SDK (尚未支持)
- 越来越多的编程语言支持
- 使用服务网格基础探针来收集数据

为服务 (service), 服务实例 (service instance), 以及端点 (endpoint) 提供了观测能力.

- 服务 (Service). 表示对请求提供相同行为的一系列或一组工作负载.
- 服务实例 (Service Instance). 服务中的每一个工作负载称为一个实例. 就像 Kubernetes 中的 pods 一样, 服务实例未必就是操作系统上的一个进程. 但当你在使用打点代理的时候, 一个服务实例实际就是操作系统上的一个真实进程.
- 端点 (Endpoint). 对于特定服务所接收的请求路径, 如 HTTP 的 URI 路径和 gRPC 服务的类名 + 方法签名.

使用 SkyWalking 时, 你可以看到服务与端点之间的拓扑结构, 每个服务 / 服务实例 / 端点的性能指标, 还可以设置报警规则.

除此之外, 你还可以通过 SkyWalking 原生代理, SDK 以及 Zipkin, Jaeger 和 OpenCensus 来进行分布式追踪.

<!-- more -->

## 架构

SkyWalking 逻辑上分为四部分: 探针, 平台后端, 存储和用户界面.

![SkyWalking 架构](./skywalking-introduction/frame.jpeg)

- 探针: 基于不同的来源可能是不一样的, 但作用都是收集数据, 将数据格式化为 SkyWalking 适用的格式.
- 平台后端: 一个支持集群模式运行的后台, 用于数据聚合, 数据分析以及驱动数据流从探针到用户界面的流程. 平台后端还提供了各种可插拔的能力, 如不同来源数据 (如来自 Zipkin) 格式化, 不同存储系统以及集群管理. 你甚至还可以使用观测分析语言来进行自定义聚合分析.
- 存储是开放式的. 你可以选择一个既有的存储系统, 如 ElasticSearch, H2 或 MySQL 集群 (Sharding-Sphere 管理), 也可以选择自己实现一个存储系统.
- 用户界面对于 SkyWalking 的最终用户来说非常炫酷且强大. 同样它也是可定制以匹配你已存在的后端的.

## 设计目标

- 保持可观测性. SkyWalking 提供了数种运行时探针.
- 拓扑结构, 性能指标和追踪一体化.
  - 理解分布式系统的第一步是通过观察其拓扑结构图. 拓扑图可以将复杂的 系统在一张简单的图里面进行可视化展现. 运维支撑系统相关人员需要更多关于服务 / 实例 / 端点 / 调用的性能指标.
  - 链路追踪 (trace) 作为详细的日志, 对于此种性能指标来说很有意义, 如你想知道什么时候端点延时变得很长, 想了解最慢的链路并找出原因. 因此你可以看到, 这些需求都是从大局到细节的, 都缺一不可.
- 轻量级.
  - 探针, 我们通常依赖于网络传输框架, 如 gRPC. 在这种情况下, 探针就应该尽可能小, 防止依赖库冲突以及虚拟机的负载压力 (主要是 JVM 永久代内存占用压力).
  - 作为一个 观测平台, 在你的整个项目环境中只是次要系统, 因此我们使用自己的轻量级框架来构建后端核心服务. 所以你不需要自己部署并维护大数据相关的平台, SkyWalking 在技术栈方面应该足够简单.
- 可插拔. 提供了大量的特性来支持可插拔功能.
- 可移植.
  - 传统的注册中心, 如 Eureka.
  - 带服务自动发现功能的 RPC 框架, 如 Spring Cloud, Apache Dubbo.
  - 现代基础设施中的服务网格.
  - 云服务.
  - 跨云部署.
- 可互操作. 可观测性是一个庞大的领域, 即使有强大的社区, SkyWalking 不可能支持所有方方面面, 因此 SkyWalking 支持与其他运维支撑系统进行互操作,
  - 主要是探针, 如 Zipkin, Jaeger, OpenTracing 和 OpenCensus. SkyWalking 接收并理解他们的数据格式, 这对于终端用户来说是非常有用的, 因为不需要他们更换已有的库.

## 探针

### 简介

探针表示集成到目标系统中的代理或 SDK 库, 它负责收集遥测数据, 包括链路追踪和性能指标. 收集并格式化数据, 并发送到后端.

从高层次上来讲, SkyWalking 探针可分为以下三组.

- 基于语言的原生代理.
  - 这种类型的代理运行在目标服务的用户空间中, 就像用户代码的一部分一样, 如 SkyWalking Java 代理, 使用 `-javaagent` 命令行参数在运行期间对代码进行操作, 操作一词表示修改并注入用户代码.
  - 另一种代理是使用目标库提供的钩子函数或拦截机制. 这些探针是基于特定的语言和库的.
- 服务网格探针. 服务网格探针从服务网格的 Sidecar 和控制面板收集数据. 在以前, 代理只用作整个集群的入口, 但是有了服务网格和 Sidecar 之后, 我们可以基于此进行观测了.
- 第三方打点类库. SkyWalking 也能够接收其他流行的打点库产生的数据格式, SkyWalking 通过分析数据, 将数据格式化成自身的数据格式. 该功能最初只能接收 Zipkin 的跨度数据, 参考其他追踪系统的接收器了解更多.

因为基于语言的原生代理和服务网格探针的功能都是收集指标数据, 所以不能同时使用两者

有如下几种推荐的方式来使用探针:

- 只使用基于语言的原生代理.
- 只使用第三方打点库, 如 Zipkin 打点系统.
- 只使用服务网格探针.
- 使用服务网格探针, 配合语言原生代理或第三方打点库, 来追踪状态. (高级用法) 让我们举例说明什么是追踪状态

默认情况下, 基于语言的原生代理和第三方打点库都会发送分布式追踪数据到后台, 后者分析 / 聚合这些追踪数据. 追踪状态意味着, 后端把这些追踪数据看作是日志, 仅仅将他们保存下来, 并且在追踪和指标之间建立联系, 比如 "这个追踪数据属于哪个入口哪个服务?".

### 服务自动打点代理

服务自动打点代理是基于语言的原生代理的一部分, 这种代理需要依靠某些语言特定的特性, 通常是一种基于虚拟机的语言.

#### 什么是自动打点

自动打点代理利用了虚拟机提供的用于修改代码的接口来动态加入打点的代码, 如通过 `javaagent premain` 来修改 Java 类. 此外, 大部分自动打点代理是基于虚拟机的, 但实际上你也可以在编译期构建这样的工具.

#### 限制

- 进程内传播在大多数情况下成为可能. 许多高级编程语言 (如 Java, .NET) 都是用于构建业务系统. 大部分业务逻辑代码对于每一个请求来说都运行在同一个线程内, 这使得传播是基于线程 ID 的, 以确保上下文是安全的.
- 仅仅对某些框架和库奏效. 因为是代理来在运行时修改代码的, 这也意味着代理插件开发者事先就要知道所要修改的代码是怎么样的.
- 跨线程可能并非总是奏效. 如上所述, 每个请求的代码大都运行在一个线程之内, 对于业务代码来说尤其如此. 但是在其他一些场景下, 它们也会在不同线程下工作, 比如指派任务到其他线程, 任务池, 以及批处理. 对于一些语言, 可能还提供了协程或类似的概念如 `Goroutine`, 使得开发者可以低开销地来执行异步操作, 在这些场景下, 自动打点可能会遇到一些问题.

所以说自动打点没有什么神秘的, 总而言之就是, 自动打点代理开发者写了一个激活程序, 使得打点的代码 自动运行, 仅此而已.

### 手动打点 SDK

介绍了手动打点 SDK 在 SkyWalking 生态中所扮演的角色.

### 服务网格 (Service Mesh) 探针

介绍了 SkyWalking 为何需要以及如何从服务网格和代理探针接收遥测数据的.

## 后端

### 观测分析平台 (Observability Analysis Platform, OAP)

OAP 从多种数据源接收数据, 这些数据分为两大类, 链路追踪和度量指标.

- 链路追踪. 包括 SkyWalking 原生数据格式, Zipkin V1 和 V2 数据格式, 以及 Jaeger 数据格式.
- 度量指标. SkyWalking 集成了服务网格平台, 如 Istio, Envoy 和 Linkerd, 并在数据面板和控制面板进行观测. 此外, SkyWalking 原生代理还可以运行在度量模式, 这极大提升了性能.

使用集成方案的同时, SkyWalking 还提供了可视化集成来对追踪和日志进行绑定, 这是通过使用 trace id 和 span id 实现的.

所有的服务都是通过 gRPC 和 HTTP 协议实现, 使得未来集成那些尚未支持的生态系统更加容易.

### 观测分析语言 (Observability Analysis Language, OAL)

在流模式 (Streaming mode) 下, SkyWalking 提供了 OAL 来分析流入的数据.

OAL 聚焦于服务, 服务实例以及端点的度量指标, 因此 OAL 非常易于学习和使用.

考虑到性能, 可读性以及可调试性, OAL 被定义成一种编译语言. OAL 脚本会在打包阶段被编译为正常的 Java 代码.

#### 语法

OAL 脚本文件应该以 `.oal` 为后缀.

```
METRIC_NAME = from(SCOPE.(* | [FIELD][,FIELD ...]))
[.filter(FIELD OP [INT | STRING])]
.FUNCTION([PARAM][, PARAM ...])
```

#### 域( Scope)

主要的**域 (Scope)**包括 `All`, `Service`, `ServiceInstance`, `Endpoint`, `ServiceRelation`, `ServiceInstanceRelation`,`EndpointRelation`. 当然还有一些二级域, 他们都属于以上某个一级域.

#### 过滤器(Filter)

使用在使用过滤器的时候, 通过指定字段名或表达式来构建字段值的过滤条件.

表达式可以使用 `and`, `or` 和 `()` 进行组合. 操作符包含 `=`, `!=`, `>`, `<`, `in (v1, v2, ...)`, `like "%..."`, 他们可以基于字段类型进行类型检测, 如果类型不兼容会在编译/代码生成期间报错.

#### 聚合函数 (Aggregation Function)

默认的聚合函数有 SkyWalking OAP 核心实现, 并可自由扩展更多函数.

提供的函数

**`longAvg`**. 某个域实体所有输入的平均值. 输入字段必须是 long 类型.
`instance_jvm_memory_max = from(ServiceInstanceJVMMemory.max).longAvg();`

在这个例子中, 输入是 `ServiceInstanceJVMMemory` 域的每个请求, 平均值是基于字段 `max` 进行求值的.

**`doubleAvg`**. 某个域实体的所有输入的平均值. 输入的字段必须是 `double` 类型.
`instance_jvm_cpu = from(ServiceInstanceJVMCPU.usePercent).doubleAvg();`

在这个例子中, 输入是 `ServiceInstanceJVMCPU` 域的每个请求, 平均值是基于 `usePercent` 字段进行求值的.

**`percent`**. 对于输入中匹配指定条件的百分比数.
`endpoint_percent = from(Endpoint.*).percent(status == true);`

在这个例子中, 输入是每个端点的请求, 条件是 `endpoint.status == true`.

**`sum`**. 某个域实体的调用总数.
`Service_Calls_Sum = from(Service.*).sum();`

本例统计每个服务的调用数.

`p99`, `p95`, `p90`, `p75`, `p50`. 参考 [Wiki 上对 p99 的解释](https://en.wikipedia.org/wiki/Percentile)
`All_p99 = from(All.latency).p99(10);`

本例中统计所有接入请求的 `p99` 值.

`thermodynamic`. 参考 [Wiki 上对 HeatMap 的解释](https://en.wikipedia.org/wiki/Heat_map)
`All_heatmap = from(All.latency).thermodynamic(100, 20);`

本例中统计所有进入请求的热力学热点图.

#### 度量指标名称

存储实现, 报警以及查询模块的度量指标名称. SkyWalking 内核支持自动类型推断.

#### 组(Group)

所有度量指标数据都会使用 `Scope.ID` 和最小时间桶 (min-level time bucket)  进行分组.

- 在端点 (Endpoint) 域中, `Scope.ID = Endpoint` 的 ID (基于服务及其端点的唯一标志).

```
#示例
// 计算 Endpoint1 和 Endpoint2 的 p99 值
Endpoint_p99 = from(Endpoint.latency).filter(name in ("Endpoint1", "Endpoint2")).summary(0.99)

// 计算端点名以 `serv` 开头的端点的 p99 值
serv_Endpoint_p99 = from(Endpoint.latency).filter(name like ("serv%")).summary(0.99)

// 计算每个端点的响应平均时长
Endpoint_avg = from(Endpoint.latency).avg()

// 计算每个端点的延迟柱状图, 每隔 50 毫秒一条柱
// 在用户界面中, 匹配此度量指标的都会显示热力学图
Endpoint_histogram = from(Endpoint.latency).histogram(50)

// 统计每个服务响应状态为 true 的百分比
Endpoint_success = from(Endpoint.*).filter(status = "true").percent()

// 统计每个服务响应码在 [200, 299] 之间的百分比
Endpoint_200 = from(Endpoint.*).filter(responseCode like "2%").percent()

// 统计每个服务响应码在 [500, 599] 之间的百分比
Endpoint_500 = from(Endpoint.*).filter(responseCode like "5%").percent()

// 统计每个服务的调用总量
EndpointCalls = from(Endpoint.*).sum()
```

### 协议

- [探针协议](https://skyapm.github.io/document-cn-translation-of-skywalking/zh/master/protocols/#%E6%8E%A2%E9%92%88%E5%8D%8F%E8%AE%AE). 包括对探针如何发送收集到的度量数据, 跟踪信息以及涉及到的每个实体格式的描述和定义.
- [查询协议](https://skyapm.github.io/document-cn-translation-of-skywalking/zh/master/protocols/#%E6%9F%A5%E8%AF%A2%E5%8D%8F%E8%AE%AE). 服务后端给 SkyWalking 自有的 UI 和任何第三方 UI 提供了查询的能力. 这些查询都是基于 GraphQL 的.

# 安装

**重要: 请确保被监控的服务器上的系统时间和OAP服务器上的系统时间是相同的**

## 下载官方发行版

[Apache SkyWalking 下载页](http://skywalking.apache.org/downloads/)

## 各语言代理

- [Java agent](https://skyapm.github.io/document-cn-translation-of-skywalking/zh/master/setup/service-agent/java-agent/). 介绍了如何将java agent安装到你的服务中, 不需要修改任何代码.

下面的agent和SDK都与SkyWalking的数据格式, 传输协议兼容, 但是是由第三方维护. 你可以到它们的项目仓库中找到对应的发行版, 以及如何使用它们的说明.

- [SkyAPM .NET Core agent](https://github.com/SkyAPM/SkyAPM-dotnet). 可以通过 .NET Core agent的项目文档查看到更详细的信息.
- [SkyAPM Node.js agent](https://github.com/SkyAPM/SkyAPM-nodejs). 可以通过Node.js服务端agent的项目文档查看到更详细的信息.
- [SkyAPM PHP SDK](https://github.com/SkyAPM/SkyAPM-php-sdk). 可以通过PHP agent项目文档查看到更详细的信息.
- [SkyAPM GO2Sky](https://github.com/SkyAPM/go2sky). 参考 GO2Sky 项目文档获得更多信息.

[参考](https://github.com/apache/skywalking/blob/master/docs/en/setup/README.md)

## 部署后端

[参考](https://github.com/apache/skywalking/blob/master/docs/en/setup/backend/backend-ui-setup.md)

### 容器部署

配置文件如下

```yml
# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

version: '3.3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:6.8.1
    container_name: elasticsearch
    restart: always
    ports:
      - 9200:9200
      - 9300:9300
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    # volumes:
    #   - /etc/localtime:/etc/localtime
    ulimits:
      memlock:
        soft: -1
        hard: -1
  oap:
    image: apache/skywalking-oap-server:6.5.0
    container_name: oap
    depends_on:
      - elasticsearch
    links:
      - elasticsearch
    restart: always
    ports:
      - 11800:11800
      - 12800:12800
    environment:
      SW_STORAGE: elasticsearch
      SW_STORAGE_ES_CLUSTER_NODES: elasticsearch:9200
    # volumes:
    #   - /etc/localtime:/etc/localtime
  ui:
    image: apache/skywalking-ui:6.5.0
    container_name: ui
    depends_on:
      - oap
    links:
      - oap
    restart: always
    ports:
      - 8080:8080
    environment:
      SW_OAP_ADDRESS: oap:12800
    # volumes:
    #   - /etc/localtime:/etc/localtime
```

使用命令起点即可

```sh
docker-compose up -d
```

# 插件开发指南

针对自己项目需要开发相应的插件来精确处理

[参考](https://skyapm.github.io/document-cn-translation-of-skywalking/zh/master/guides/Java-Plugin-Development-Guide.html)

# 参考

- [SkyWalking docs](https://github.com/apache/skywalking/tree/master/docs)
- [SkyWalking 文档](https://skyapm.github.io/document-cn-translation-of-skywalking/zh/master/)
