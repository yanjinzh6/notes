---
title: JSON API 网关分析
date: 2020-10-05 12:00:00
tags: 'HTML'
categories:
  - ['开发', 'HTML']
permalink: json-api-gateway
---

## 简介

```mermaid
grap TB
  user[用户] -->|访问| page(页面)
  page -->|鉴权| auth([权限网关])
  auth ---|无权限| 401page(401 页面)
  auth -->|通过| gateway([API 网关])
  gateway -->|获取定义好的 API| json([JSON API 网关])
  json -->|解析 API 并请求服务| service>服务]
```

<!-- more -->

## 查询

通过 GET 请求进行查询, 携带下面的参数, 主要封装了所有查询实体

### 条件和参数

```json
{
  "query": {
    "user": {
      "id": true,
      "name": true,
      "profile": {
        "image": true
      },
      "relation": [{
        "id": true,
        "friendId": true
      }]
    },
    "hot": [{
      "id": true,
      "title": true
    }]
  },
  "params": {
    "user": {
      "id": "xxx",
      "profile": "embedded",
      "relation": [{
        "first": 10,
        "after": "yyy"
      }]
    },
    "hot": [{
      "first": 20,
      "after": "zzz"
    }]
  }
}
```

### 处理

通过解析 query 将所有实体拆分并形成服务请求, embedded 标记使得 profile 作为 user 的一个参数而不是单独的实体

```sh
user?field=id,name&id=xxx
relations?field=id,friendId&userId=xxx&first=10&after=yyy
hots?field=id,title&first=20&after=zzz
```

这里处理后端指数级请求需要实现

- 限制 query 层数
- 使用批量接口处理
  - 例如一次性获取多个符合条件的 user 后, 在查询 relation 的时候传递的参数为 userIds 列表
  - relation 服务将通过聚合查询返回所有符合条件的数据并进行分组处理, 通过额外的 forUserId 字段区分当前数据是属于上一层 user 列表中的哪个

### 结果

```json
{
  "data": {
    "user": {
      "id": "xxx",
      "name": "xxx",
      "profile": {
        "image": "xyy"
      },
      "relation": [{
        "id": "yyy",
        "friendId": "yya"
      }, {
        "id": "yy2",
        "friendId": "yyb"
      }]
    },
    "hot": [{
      "id": "zzz",
      "title": "ttt"
    }, {
      "id": "zzz2",
      "title": "ttt2"
    }{
      "id": "zzz3",
      "title": "ttt3"
    }]
  },
  "status": {
    "user": {
      "code": 400,
      "error": "xxx",
      "profile": {
        "code": 400,
        "error": "xxx",
      },
      "relation": {
        "code": 400,
        "error": "xxx",
        "hasNext": true,
        "nextIndex": "xxy"
      }
    },
    "hot": {
      "code": 400,
      "error": "xxx",
      "hasNext": true,
      "nextIndex": "zzy"
    }
  }
}
```

返回结果分为数据和状态, 数据和请求结构保持一致, 可以保存为 data 字段或置于最顶层, 状态保存着每个实体的错误和相应的参数, 例如分页数据

这里采用结构保存 http 请求状态并不会影响后端服务的健壮性, 在网关请求后端服务的过程中是记录正规的 http 请求状态

## 修改

### 请求参数

```json
{
  "query": {
    "user": {
      "id": true,
      "name": true,
      "profile": {
        "image": true
      },
      "relation": [{
        "id": true,
        "friendId": true
      }]
    }
  },
  "body": {
    "user": {
      "id": "xxx",
      "name": "xxxn",
      "password": "xxxp"
    }
  }
}
```
