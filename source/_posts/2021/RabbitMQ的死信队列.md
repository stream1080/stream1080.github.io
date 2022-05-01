---
title: RabbitMQ的死信队列
cover: false
top: false
categories:
  - 消息队列
tags:
  - RabbitMQ
  - 死信队列
abbrlink: 37640
date: 2021-11-13 15:00:33
summary:
---

## 什么是死信
在 RabbitMQ 中充当主角的就是消息，在不同场景下，消息会有不同地表现。
死信就是消息在特定场景下的一种表现形式，这些场景包括：
- 消息被拒绝访问，即 RabbitMQ返回 nack 的信号时
- 消息的 TTL 过期时
- 消息队列达到最大长度
- 消息不能入队时。

上述场景经常产生死信，即消息在这些场景中时，被称为死信。

## 什么是死信队列
死信队列就是用于储存死信的消息队列，在死信队列中，有且只有死信构成，不会存在其余类型的消息。

死信队列在 RabbitMQ 中并不会单独存在，往往死信队列都会绑定这一个普通的消息队列，当所绑定的消息队列中，有消息变成死信了，那么这个消息就会重新被交换机路由到指定的死信队列中去，我们可以通过对这个死信队列进行监听，从而手动的去对这一消息进行补偿。

那么，我们到底如何来使用死信队列呢？

## 死信队列基本使用
在 RabbitMQ 中，死信队列的标识为 x-dead-letter-exchange ，通过观察死信队列的标识，我们不难发现，其标识最后为 exchange ，即 RabbitMQ 中的交换机，RabbitMQ 中的死信队列就是由死信交换机而得出的。

要想使用死信队列，我们需要首先声明一个普通的消息队列，并将死信队列的标识绑定到这个普通的消息队列上， 这个过程需要我们在生产端进行配置，代码如下所示：

```java
// 使用 ConnectionFactory 创建了一个客户端连接 RabbitMQ Server 的连接。
ConnectionFactory connectionFactory = new ConnectionFactory();
connectionFactory.setHost("xx");
connectionFactory.setPort("5672");
connectionFactory.setVirtualHost("/");
Connection connection = connectionFactory.newConnection();

// 使用建立好的连接，来创建了一个频道 channel 。
Channel channel = connection.createChanel();

// 声明了一个普通队列的额外参数的 Map ，
// 这个 Map 的 key 就是死信队列的标识，value 就是我们后续声明的真正的死信交换机的名称。
Map<String, Object> argumentsMap = new HashMap();
argumentsMap.put("x-dead-letter-exchange", "dlx_exchange");

/*
使用 channel 的 exchangeDeclare 方法和 queueDeclare 方法，
分别声明了一个名为 dlx_common_exchange 的交换机和名为 dlx_common_queue 的普通消息队列，
之所以名称中有 common ，是因为要对这个交换机和队列做一个标识，表示该交换机和队列是绑定了死信队列的。
*/
channel.exchangeDeclare("dlx_common_exchange", "direct", true, false, null);
channel.queueDeclare("dlx_common_queue", true, false, false, argumentsMap);

/*
使用 channel 的 queueBind 方法来讲声明的普通交换机和消息队列进行绑定，并且制定了 routingKey ，
这样消息就可以经 dlx_common_exchange 根据 routingKey 来路由到 dlx_common_queue 中。
*/
channel.queueBind("dlx_common_queue", "dlx_common_exchange", routingKey);

```
声明了要绑定死信队列的普通队列之后，最后我们需要声明真正的死信队列
```java
// 使用 chanel 的 exchangeDeclare 方法来声明了一个名为 dlx_exchange 的交换机。
channel.exchangeDeclare("dlx_exchange", "direct", true, false, null);
// 使用 channel 的 queueDeclare 方法来声明了一个名为 dlx_queue 的队列。
channel.queueDeclare("dlx_queue", true, false, false, null);
// 使用 channel 的 queueBind 方法，来将 dlx_exchange 的交换机与 dlx_queue 队列进行了绑定。
channel.queueBind("dlx_queue", "dlx_exchange", routingKey);
```

## MQ处理消息失败了怎么办
在生产环境中，使用MQ的时候设计两个队列：
- 一个是业务队列，专门用来处理消息；
- 一个是死信队列。用来处理异常情况。

比如说消费消息的时候，数据库故障了，此时无法将数据落盘，那么消费者每次消费一条消息，尝试落盘持久化的时候，都会遇到数据库报错。此时消费者就可以把这条消息拒绝访问，或者标志位处理失败！

- 一旦标志这条消息处理失败了之后，MQ就会把这条消息转入提前设置好的一个死信队列中。
- 在数据库故障期间，所有消息全部处理失败，全部会转入死信队列。
- 然后消费者会专门有一个后台线程，监控数据库是否正常，能否请求的，不停的监视。
- 一旦发现对方恢复正常，这个后台线程就从死信队列消费出来处理失败的订单，重新处理逻辑
- 死信队列的使用，其实就是MQ在生产实践中非常重要的一环，也就是架构设计必须要考虑的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/8a20d6e5492041dd9a6e0b02a6aef248.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)

