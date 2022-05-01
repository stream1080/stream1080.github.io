---
title: RabbitMQ队列模型
top: false
cover: false
toc: true
mathjax: false
abbrlink: 54078
date: 2021-07-28 23:54:11
author:
img:
coverImg:
password:
summary:
categories:
  - 消息队列
tags:
  - 队列模型
  - 发布/订阅
---

## 模型定义
- 生产者（Producer）：发送消息到队列的模块，可以理解为消息的提供者
- 队列（Queue）：存储消息的一段空间，作为消息的缓存模块；
- 消费者（Consumer）：从队列中接受消息的模块，可以理解为消息的处理者
- 交换机（Exchange）：消息不直接发到队列，首先发到 Exchange 模块，再根据路由规则转发到定制化的队列。
## 队列模型
- 简单队列
- 工作队列
- 发布/订阅队列
- 路由队列
- 主题队列

## 简单队列

在简单队列模型中，只有一个生产者、一个消息队列、一个消费者。
![在这里插入图片描述](https://img-blog.csdnimg.cn/5ec9510d3e9e4bbba1bdfe6e6956c967.png)
- 优点是不需要配置复杂的路由规则
- 缺点是只支持一对一通信
- 在大多数场景中一般不会使用这种队列
## 工作队列（Work Queue）
在工作队列模型中，一个生产者，拥有向多个消息消费者发送消息的能力，但是一条消息只能被一个消费者消费。
如何保证详细不被重复消费，在使用消息队列时也是需要考虑的问题

![在这里插入图片描述](https://img-blog.csdnimg.cn/daf04b39bb484ab989e8104067a620e3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
- 应用场景：需要将流量打散到多个消费者模块的场景，也就是削峰
- 例如美团点餐中，某一时段点餐高峰，电商秒杀等，瞬时流量很大的应用
## 发布/订阅队列（Publish/Subscribe）
![在这里插入图片描述](https://img-blog.csdnimg.cn/cb5ce4cf7f8c4ae3b6b66b2ac5704ced.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
- 在发布/订阅队列模型中，生产者只能向交换机发送消息，交换机绑定了多个队列，
- 所有绑定该交换机的队列都会收到交换机中的所有消息。

交换机存在四种路由方式：
- direct
- topic
- headers
- fanout-广播

发布/订阅模型是非常常见的队列模型
例如上篇文章中的注册流程中，一个用户的注册请求需要往短信服务和邮箱服务发送消息，可以使用该模型。

## 路由队列（Routing）
路由队列也是在发布/订阅队列模型中的一种，只是交换机的路由方式不一样，交换机的路由方式为direct模式
![在这里插入图片描述](https://img-blog.csdnimg.cn/cb5ce4cf7f8c4ae3b6b66b2ac5704ced.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
- 在路由队列模型中，生产者将消息发送到 direct 模式的交换机，交换机和队列绑定的时候限制了路由 Key。
- 当生产者发送一条消息的时候，会指定一条路由 Key，这条消息只会发送到对应的队列中
- 每个队列都会绑定一个key，只有发送消息的key和绑定key相同的队列才能收到消息

```java
@Configuration
public class RabbitmqDirectConfig {

    /**
     * 队列 queue
     */
    private static final String QUEUE_DIRECT_01 = "queue_direct01";
    private static final String QUEUE_DIRECT_02 = "queue_direct02";

    /**
     * 交换机 exchange
     */
    private static final String DIRECT_EXCHANGE = "directExchange";

    /**
     * 路由键 routingkey #表示0或多个单词，*表示一个单词
     */
    private static final String QUEUE_RED = "queue.red";
    private static final String QUEUE_GREEN = "queue.green";

    @Bean
    public Queue queueDirect01() {
        return new Queue(QUEUE_DIRECT_01);
    }

    @Bean
    public Queue queueDirect02() {
        return new Queue(QUEUE_DIRECT_02);
    }

    @Bean
    public DirectExchange directExchange() {
        return new DirectExchange(DIRECT_EXCHANGE);
    }

    @Bean
    public Binding bindingDirectRed() {
        return BindingBuilder.bind(queueDirect01()).to(directExchange()).with(QUEUE_RED);
    }

    @Bean
    public Binding bindingDirectGreen() {
        return BindingBuilder.bind(queueDirect02()).to(directExchange()).with(QUEUE_GREEN);
    }

}

```

路由队列的优势是能够定制化发送消息，消费者选择性收听消息，适合灵活变通的应用场景。


## 主题队列（Topics）
主题队列也是在发布/订阅队列模型中的一种，只是交换机的路由方式不一样，交换机的路由方式为topic模式
![在这里插入图片描述](https://img-blog.csdnimg.cn/cb5ce4cf7f8c4ae3b6b66b2ac5704ced.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
- 在主题队列模型中，实际上是基于路由队列的定制化配置，支持了key的通配符匹配
- 简单理解就是正则表达式+路由key。
- 例如*.queue只能匹配到a.queue类似的key，但是queue.#可以匹配到queue.a或者queue.a.b类似的key。
- 应用场景与路由队列相同。
```java
/**
 * Title: RabbitmqTopicConfig
 * Description: Topic模式常用
 */
@Configuration
public class RabbitmqTopicConfig {

    /**
     * 队列 queue
     */
    private static final String QUEUE_TOPIC_01 = "queue_topic01";
    private static final String QUEUE_TOPIC_02 = "queue_topic02";

    /**
     * 交换机 exchange
     */
    private static final String TOPIC_EXCHANGE = "topicExchange";

    /**
     * 路由键 routingkey
     * #表示匹配0或多个单词
     * *表示匹配一个单词
     */
    private static final String ROUTINGKEY_01 = "#.queue.#";
    private static final String ROUTINGKEY_02 = "*.queue.#";

    @Bean
    public Queue queueTopic01() {
        return new Queue(QUEUE_TOPIC_01);
    }

    @Bean
    public Queue queueTopic02() {
        return new Queue(QUEUE_TOPIC_02);
    }

    @Bean
    public TopicExchange topicExchange() {
        return new TopicExchange(TOPIC_EXCHANGE);
    }

    @Bean
    public Binding bindingTopic01() {
        return BindingBuilder.bind(queueTopic01()).to(topicExchange()).with(ROUTINGKEY_01);
    }

    @Bean
    public Binding bindingTopic02() {
        return BindingBuilder.bind(queueTopic02()).to(topicExchange()).with(ROUTINGKEY_02);
    }

}

```




