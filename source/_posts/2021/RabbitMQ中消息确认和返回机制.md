---
title: RabbitMQ中消息确认和返回机制
top: false
cover: false
toc: true
mathjax: false
categories:
  - 消息队列
tags:
  - 确认机制
  - 返回机制
abbrlink: 25789
date: 2021-08-02 23:47:09
author:
img:
coverImg:
password:
summary:
---

# RabbitMQ中消息确认和返回机制
- 为了保证 RabbitMQ 中消息的可靠性投递，以及消息在发生特定异常时的补偿策略
- RabbitMQ诞生了消息确认和返回机制
- 这两种机制是 RabbitMQ 自带的补偿机制，可以直接使用
## 消息确认
### 消息确认机制
- 消息确认机制，是保障消息与 RabbitMQ消息之间可靠传输消息一种保障机制
- 其主要内容就是用来监听RabbitMQ消息队列收到消息之后返回给生产端的ack确认消息
- 消息确认机制描述了一种消息是否已经被发送到 RabbitMQ消息队列中以及 RabbitMQ消息队列是否以及接收到生产端发送的消息。
![在这里插入图片描述](https://img-blog.csdnimg.cn/99aeca6ce8204c8594533573a094443a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
### 消息确认机制的作用
- 监听生产者的消息是否已经发送到了 RabbitMQ消息队列中；
- 如果消息没有被发送到 RabbitMQ消息队列中，则消息确认机制不会给生产端返回任何确认应答，也就是没有发送成功
- 相反，如果消息被成功发送到了 RabbitMQ消息队列中，则消息确认机制会给生产端返回一个确认应答，
- 以通知生产者，消息已经发送到了 RabbitMQ消息队列

## 返回机制
### 消息返回机制
- 描述不可达的消息与生产者之间的一种保障策略
- 其主要是用来监听，RabbitMQ消息队列中是否存在不可达的消息，并根据监听结果返回给生产端的一种监听机制
- 消息返回机制描述了一种 RabbitMQ消息队列中的不可达消息与生产端的关系
![在这里插入图片描述](https://img-blog.csdnimg.cn/cf4a5ba4eba842edb8351b4d6d9f9edc.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
### 什么是不可达的消息
- 消息在被成功发送到RabbitMQ消息队列中之后，如果消息在经过当前配置的 exchangeName 或 routingKey 没有找到指定的交换机，或没有匹配到对应的消息队列，
- 那么这个消息就被称为不可达的消息，如果此时配置了消息返回机制，那么此时RabbitMQ消息队列会返回给生产者一个信号，信号中包括消息不可达的原因，以及消息本身的内容。

### 消息确认机制的作用
- 监听生产端发动到RabbitMQ消息队列中的消息是否可达
- 如果消息不可达，则返回一个信号通知生产端，相反，如果消息可达，则不会返回任何信号


