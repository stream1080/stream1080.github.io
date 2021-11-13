---
title: 基于RedLock的分布式锁
cover: false
top: false
date: 2021-10-12 23:35:21
summary:
categories:
  - 缓存
tags:
  - 分布式锁
  - Redis
---


## 概述
在单个主节点的架构上实现分布式锁，是无法保证高可用的，在生产环境上，我们的Redis都是以集群部署的；

那么如果Redis实现分布式锁的是一个主从集群，可能会发生什么情况呢？
![在这里插入图片描述](https://img-blog.csdnimg.cn/df7cdf3ffafb4a0aab070ad0ab176867.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
- 如果进程A在主节点上加锁成功，然后这个主节点宕机了，则从节点将会晋升为主节点。
- 若此时进程B在新的主节点上加锁成功，之后原主节点重启，成为了从节点，系统中将同时出现两把锁，这是违背锁的唯一性原则的。

## RedLock实现
如果要保证分布式锁的高可用，则需要采用多个节点的实现方案。

Redis的官方给出的建议是采用RedLock算法的实现方案。该算法基于多个Redis节点
#### 基本流程
- 集群中的节点相互独立，不存在主从复制或者集群协调机制；
- 加锁：以相同的KEY向N个实例加锁，只要超过一半节点成功，则认定加锁成功；
- 解锁：向所有的实例发送DEL命令，进行解锁；
![在这里插入图片描述](https://img-blog.csdnimg.cn/6f1b548a5a5549dd8bc90cfff64a715b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_16,color_FFFFFF,t_70,g_se,x_16)
#### 算法流程
1. 获取当前时间戳

2. client尝试按照顺序使用相同的key,value获取所有redis服务的锁，在获取锁的过程中的获取时间比锁过期时间短很多，这是为了不要过长时间等待已经关闭的redis服务。并且试着获取下一个redis实例。比如：TTL为5s,设置获取锁最多用1s，所以如果一秒内无法获取锁，就放弃获取这个锁，从而尝试获取下个锁

3. client通过获取所有能获取的锁后的时间减去第一步的时间，这个时间差要小于TTL时间并且至少有超过一半的redis实例成功获取锁，才算真正的获取锁成功

4. 如果成功获取锁，则锁的真正有效时间是 TTL减去第三步的时间差 的时间；比如：TTL 是5s,获取所有锁用了2s,则真正锁有效时间为3s(其实应该再减去时钟漂移);

6. 如果客户端由于某些原因获取锁失败，便会开始解锁所有redis实例；因为可能已经获取了小于一半的redis实例的锁，必须释放，否则影响其他client获取锁
