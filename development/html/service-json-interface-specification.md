---
title: 服务暴露 JSON 接口规范
date: 2020-10-04 10:00:00
tags: 'HTML'
categories:
  - ['开发', 'HTML']
permalink: service-json-interface-specification
---

## 简介

为了去掉服务的业务耦合和优化性能, 常见的数据完整性校验可以由服务暴露出来让客户端进行校验

这里使用较常见的 json 格式进行描述

```json
{
  "id": {
    "description": "主键",
    "read": 1,
    "write": 0,
    "query": 1,
    "require": 0
  },
  "name": {
    "description": "名称",
    "read": 1,
    "write": 1,
    "query": 1,
    "require": 0
  },
  "orderNo": {
    "description": "排序",
    "read": 1,
    "write": 1,
    "query": 0,
    "require": 0
  },
  "sysTag": {
    "description": "系统标识",
    "read": 0,
    "write": 0,
    "query": 0,
    "require": 0
  }
}
```

- read: 可读
- write: 可写
- query: 可查询
- require: 写数据库时必要字段

## JSON API 网关

网关通过读取到服务的配置, 根据配置进行数据校验并转换为相应的请求

如下请求, 将会出现 2 个错误

- sysTag 是不可以查询字段
- orderNo 是不可以作为查询条件的字段

```json
{
  "query": {
    "demo": {
      "id": 1,
      "name": 1,
      "orderNo": 1,
      "sysTag": 1
    }
  },
  "params": {
    "demo": {
      "orderNo": 111
    }
  }
}
```

## 创建服务

随着 NoSQL 的发展, 定义数据库模式已经到了很容易的时候了, 除了少数的限制条件, 例如唯一值等需要依赖数据库的实现, 其他数据模型可以通过保存对象进行处理, 相应的序列化反序列化则通过上面定义的 JSON 规范进行处理

这样可以方便的通过一定的 JSON 配置来提供一个特定的服务

## 处理查询条件

受到 JSON 格式的限制, 如果使用键值对的方法传递参数, 则只能满足 eq 这一特定的查询条件

## 处理限制条件

限制条件包括两种

- 单个限制
  - 类型
  - 长度大小
  - 必须存在
  - 正则匹配
- 全局限制
  - 全局唯一

单个限制可以在客户端处理, 全局限制只能使用数据库提供的功能处理才能有效提供性能, 客户端处理将会导致多次的交互降低速度

单个限制通过定义结构进行标识, 这样会导致 JSON 文件过度臃肿

```json
{
  "name": {
    "type": "number",
    "min": 1,
    "max": 9,
    "required": 1,
    "regex": "\w"
  }
}
```

## 处理嵌套对象
