---
title: Spring @RabbitListener 消费队列
date: 2020-08-10 12:00:00
tags: 'Spring'
categories:
  - ['开发', 'Java', '框架']
permalink: spring-rabbit-listener
---

## 简介

RabbitMQ 版本为 3.8.4

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-amqp</artifactId>
  <version>2.3.3.RELEASE</version>
</dependency>
```

使用 `@RabbitListener` 注解可以简单的标识一个队列的消费者

```java
@RabbitListener(queues = "topic.a")
public void consumerExistsQueue(String data) {
    System.out.println("consumerExistsQueue: " + data);
}
```

这种方式适合已经配置好相应的 exchange 和 queue 的情况, 如果队列不存在或者队列在不使用的情况下会自动删除, 即 `autoDelete` 为 `true` 的情况下, 会出现错误提示队列不存在, `reply-code=404, repl-text=NOT_FOUND`

## @QueueBinding 注解

使用该注解创建了队列, 并且与对应的交换机绑定

```java
/**
 * 队列不存在时, 需要创建一个队列, 并且与exchange绑定
 */
@RabbitListener(bindings = @QueueBinding(
        value = @Queue(value = "topic.n1", durable = "false", autoDelete = "true"),
        exchange = @Exchange(value = "topic.e", type = ExchangeTypes.TOPIC),
        key = "r"))
public void consumerNoQueue(String data) {
    System.out.println("consumerNoQueue: " + data);
}
```

- value: 队列参数, 这里使用 @Queue 注解声明队列
  - value: 队列名称
  - durable: 表示队列是否持久化
  - autoDelete: 表示在没有消费者之后队列是否自动删除
- exchange: 交换机参数, 这里使用 @Exchange 注解声明交换机
  - value: 交换机名称
  - type: 指定消息投递策略, 这里使用 topic 方式
- key: 绑定键, 在 topic 模式下为 routing key

## ACK 配置

ack 是 rabbitmq 通过消息确认保证数据一致性的机制, `@RabbitListener` 中的 ack 模式是针对消费端的, 可以通过配置 ackMode 参数来修改 (noack, auto, manual)

```java
/**
 * 手动ack
 *
 * @param data
 * @param deliveryTag 消息的唯一标识, 标识哪个消息被 ack/nak 了
 * @param channel mq 和 consumer 之间的管道, 通过它来 ack/nak
 * @throws IOException
 */
@RabbitListener(bindings = @QueueBinding(value = @Queue(value = "topic.n3", durable = "false", autoDelete = "true"),
        exchange = @Exchange(value = "topic.e", type = ExchangeTypes.TOPIC), key = "r"), ackMode = "MANUAL")
public void consumerDoAck(String data, @Header(AmqpHeaders.DELIVERY_TAG) long deliveryTag, Channel channel)
        throws IOException {
    System.out.println("consumerDoAck: " + data);

    if (data.contains("success")) {
        // RabbitMQ 的 ack 机制中, 第二个参数返回 true, 表示需要将这条消息投递给其他的消费者重新消费
        channel.basicAck(deliveryTag, false);
    } else {
        // 第三个参数 true, 表示这个消息会重新进入队列
        channel.basicNack(deliveryTag, false, true);
    }
}
```

通过设置 `ackMode=MANUAL` 来手动 ack, 这时需要在逻辑中主动进行 `ack/nak` 操作

注意: `如果没有手动 ack, 这就相当于一致都没有 ack, 在后面的测试中, 可以看出这种不 ack 时, 会发现数据一直在unacked这一栏, 当 Unacked 数量超过限制的时候, 就不会再消费新的数据了`

## 并发消费

可以通过 `concurrency` 参数设置并行消费

```java
@RabbitListener(bindings = @QueueBinding(value = @Queue(value = "topic.n4", durable = "false", autoDelete = "true"),
        exchange = @Exchange(value = "topic.e", type = ExchangeTypes.TOPIC), key = "r"), concurrency = "4")
public void multiConsumer(String data) {
    System.out.println("multiConsumer: " + data);
}
```

实例中 `concurrency = "4"` 设置了固定的 4 个消费者, 还可以设置 `m-n` 格式表示从 m 到 n 一定范围数量的消费者, 参考 `SimpleMessageListenerContainer`

## 参考

- [参考](https://cloud.tencent.com/developer/article/1620186)
