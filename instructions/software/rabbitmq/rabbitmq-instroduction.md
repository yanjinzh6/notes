---
title: RabbitMQ 简介
date: 2020-08-10 10:00:00
tags: 'RabbitMQ'
categories:
  - ['使用说明', '软件']
permalink: rabbitmq-instroduction
photo:
---

## 简介

RabbitMQ 是一个开源的 AMQP 实现, RabbitMQ 在队列的概念多了交换器的概念, 这样发消息者和队列就没有直接联系, 转而变成发消息者把消息给交换器, 交换器根据调度策略再把消息再给队列

概念解析

- 虚拟主机 (vhost): 用于权限控制
  - 一个虚拟主机持有一组交换机, 队列和相应的绑定
  - 用户可以在虚拟主机的维度进行权限控制, 这样可以细腻的控制用户可以使用的交换机和队列
  - 每一个 RabbitMQ 服务器都有一个默认的虚拟主机
- 交换机 (exchange): 用于转发消息, 不会存储
  - 没有队列绑定到交换机的话, 会直接丢弃生产者发送过来的消息
  - 路由键 (routing key): 消息到交换机的时候, 会根据路由键转发到相应的队列, 默认为空
  - 绑定键 (binding key): 交换机与队列的多对多关系, 默认为空
- 信道 (channel): 生产者与消费者和代理服务器之间 TCP 连接内的虚拟连接, 解决 TCP 连接数量限制及降低 TCP 连接代价, 每个信道有一个 ID, 与 "频分多路复用" 类似
- 队列 (queue): 消息最终到达队列

<!-- more -->

## 交换机

### Direct Exchange

Direct Exchange 是 RabbitMQ 默认的交换机模式, 也是最简单的模式, 根据 Binding key 全文匹配去寻找队列

交换机和队列间通过 Binding key 绑定, 当消息中的路由键与绑定的 Binding key 对应上的时候, 就将该消息发送到对应的队列

交换机和队列可以同时绑定多个 Binding key

### Topic Exchange

Topic Exchange 采用通配符转发消息, Binding key 通配符包括以下方式

- 句点号 `.` 标识分隔的单词
- 星号 `*` 模糊匹配一个单词
- 井号 `#` 模糊匹配零个或多个单词

### Fanout Exchange

Fanout Exchange 交换器会简单的把所有发送到该交换器的消息路由到与该交换器绑定的消息队列中

类似于子网广播, 子网内的每台主机都获得了一份消息的拷贝

Fanout Exchange 效率最高

### Headers Exchange

Headers Exchange 根据消息头的 Headers 属性来进行转发, 已经不建议使用
