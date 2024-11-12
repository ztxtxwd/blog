---
title: RabbitMQ消息去重实践：使用rabbitmq-message-deduplication插件
published: 2024-11-13
description: ''
image: ''
tags: ['RabbitMQ', '消息队列', '去重']
category: '开发'
draft: false 
lang: ''
---

## 背景介绍

在分布式系统中，消息重复是一个常见的问题。传统解决方案通常需要实现一个专门的去重服务，这增加了系统的复杂性和维护成本。通过使用 RabbitMQ 的 message-deduplication 插件，我们可以在消息队列层面直接实现去重，大大简化了系统架构。

## 插件介绍

rabbitmq-message-deduplication 是一个开源的 RabbitMQ 插件，它提供了在消息队列层面进行消息去重的功能。该插件的主要特点包括：

- 支持基于消息属性的去重
- 可配置的去重时间窗口
- 简单易用，无需修改现有代码
- 性能开销小

## 安装配置

### 1. 安装插件

```bash
# 下载插件
wget https://github.com/noxdafox/rabbitmq-message-deduplication/releases/download/0.6.4/rabbitmq_message_deduplication-0.6.4.ez
wget https://github.com/noxdafox/rabbitmq-message-deduplication/releases/download/0.6.4/elixir-1.16.3.ez

# 复制到 RabbitMQ 插件目录
cp rabbitmq_message_deduplication-0.6.4.ez /usr/lib/rabbitmq/plugins/
cp elixir-1.16.3.ez /usr/lib/rabbitmq/plugins/

# 启用插件
rabbitmq-plugins enable rabbitmq_message_deduplication
```

## 使用示例

### Java 代码示例

```java

// 声明一个带去重功能的交换机
rabbitMQClient.exchangeDeclare(EXCHANGE_NAME, "x-message-deduplication", true, false, JsonObject.of("x-cache-size",90000,"x-cache-ttl",10000));

// 发送消息时设置消息ID
HashMap<String,Object> hashMap = new HashMap<>();
hashMap.put("x-deduplication-header", msgId);
BasicProperties properties = new BasicProperties().builder().headers(hashMap).build();
rabbitMQClient.basicPublish(EXCHANGE_NAME, routingKey, properties, Buffer.buffer(payload.toByteArray()));
```

## 工作原理

1. 插件通过监控消息的 x-deduplication-header 来实现去重
2. 当收到新消息时，插件会检查该 x-deduplication-header 是否在去重缓存中
3. 如果存在，则丢弃该消息；如果不存在，则将消息投递并记录 x-deduplication-header
4. x-deduplication-header 在缓存中的保存时间由配置的 x-cache-ttl 参数决定

## 优势分析

1. **简化架构**
   - 无需额外的去重服务
   - 减少系统组件数量

2. **性能优势**
   - 在消息入队时直接去重
   - 避免了消息的重复处理

3. **维护成本低**
   - 插件级别的功能，无需应用层代码修改
   - 配置简单，易于管理

## 注意事项

1. 确保每条消息都有唯一的 x-deduplication-header
2. 合理配置去重时间窗口，避免内存占用过大
3. 在集群环境下需要所有节点都安装该插件

## 总结

通过使用 rabbitmq-message-deduplication 插件，我们可以优雅地解决消息重复问题，无需开发专门的去重服务。这不仅简化了系统架构，还提高了系统的可维护性。对于需要消息去重功能的项目来说，这是一个非常推荐的解决方案。