---
title: RabbitMQ如何保证消息顺序消费
cover: false
top: false
categories:
  - 消息队列
tags:
  - RabbitMQ
  - 顺序消费
abbrlink: 56438
date: 2021-10-29 19:08:55
summary:
---

## 为什么要顺序消费
保证消息的顺序消费是生产业务场景下经常面临的挑战，例如电商的下单逻辑，在用户下单之后，会发送创建订单和扣减库存的消息，我们需要保证扣减库存在创建订单之后执行。

- 处理业务逻辑后，向MQ发送一条消息，再由消费者从 MQ 中获取 消息落盘到MySQL 中。
- 在这个过程中，可能会有增删改的操作，比如执行顺序是增加、修改、删除。
- 消费者可能换了顺序给执行成删除、修改、增加，所以我们要保证消息的顺序消费

## 为什么会不按顺序消费

对于 RabbitMQ 来说，导致上面顺序错乱的原因通常是消费者是集群部署，不同的消费者消费到了同一订单的不同的消息。

- 如消费者1执行了增加，消费者2执行了修改，消费者C执行了删除
- 但是消费者C执行比消费者B快，消费者B又比消费者A快，就会导致消费消息的时候顺序错乱
- 本该顺序是增加、修改、删除，变成了删除、修改、增加.
![在这里插入图片描述](https://img-blog.csdnimg.cn/77a568c090d24eb68f041cd17126220b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)


## 如何解决
RabbitMQ 的问题是由于不同的消息都发送到了同一个 queue 中，多个消费者都消费同一个 queue 的消息。
- 我们可以给 RabbitMQ 创建多个 queue，每个消费者固定消费一个 queue 的消息，
- 生产者发送消息的时候，同一个类型的消息发送到同一个 queue 中
- 由于同一个 queue 的消息是一定会保证有序的，那么同一个订单号的消息就只会被一个消费者顺序消费，从而保证了消息的顺序性。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d8c2d643644646f9be78cf4884a828ff.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
