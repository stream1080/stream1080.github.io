---
title: AQS原理
cover: false
top: false
date: 2021-09-22 20:56:37
summary:
categories:
  - 多线程
tags:
  - 队列
---

## 概念
AQS是AbstarctQueuedSynchronizer 的简称 ，是一个用于构建锁和同步容器的框架。

juc并发 包内许多类都是基于 AQS 构建的
- ReentrantLock
- ReentrantReadWriteLock
- FutureTask

AQS 解决了在实现同步容器时大量的细节问题。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d49c82af4c5a4687806b853b3dd7065e.png)


- AQS 使用一个 FIFO 队列表示排队等待锁的线程，队列头结点称作 “哨兵节点” ，它不与任何线程关联。
- 其他的节点与等待线程关联，每个阶段维护一个等待状态 waitStatus。

## 功能
- 独占锁：每次只能有一个线程持有锁， ReentrantLock 就是以独占方式实现的互斥锁；
- 共享锁：允许多个线程同时获取锁，并发访问共享资源，比如 ReentrantReadWriteLock。

## 内部原理
AQS 的实现依赖内部的同步队列，也就是 FIFO 的双向队列，如果当前线程竞争锁失败，那么 AQS 会把当前线程以及等待状态信息构造成一个 Node 加入到同步队列中，同时再阻塞该线程。当获取锁的线程释放锁以后，会从队列中唤醒一个阻塞的节点 (线程)。
![在这里插入图片描述](https://img-blog.csdnimg.cn/8695a8335e9a4807877d1af86fe1b3d2.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
一个节点表示一个线程，它保存着线程的引用（thread）、状态（waitStatus）、前驱节点（prev）、后继节点（next），其实就是个双端双向链表

- AQS 队列内部维护的是一个 FIFO 的双向链表，这种结构的特点是每个数据结构都有两个指针，分别指向直接的后继节点和直接前驱节点。
- 双向链表可以从任意一个节点开始，很方便的访问前驱和后继。
- 每个 Node 其实是由线程封装，当线程争抢锁失败后会封装成 Node 加入到 ASQ 队列中去。

## 工作流程
#### 添加线程
当出现锁竞争以及释放锁的时候，AQS 同步队列中的节点会发生变化
![在这里插入图片描述](https://img-blog.csdnimg.cn/33188515e0eb44faa67106d48c0558ea.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
- 队列操作的变化：新的线程封装成 Node 节点追加到同步队列中，设置 prev 节点以及修改当前节点的前置节点的 next 节点指向自己；
- tail 指向变化：通过同步器将 tail 重新指向新的尾部节点。
#### 销毁线程
第一个 head 节点表示获取锁成功的节点，当头结点在释放同步状态时，会唤醒后继节点，如果后继节点获得锁成功，会把自己设置为头结点
![在这里插入图片描述](https://img-blog.csdnimg.cn/58333e8c7310454e819205749b8e93e9.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
- head 节点指向：修改 head 节点指向下一个获得锁的节点；
- 新的获得锁的节点：第二个节点被 head 指向了，此时将 prev 的指针指向 null，因为它自己本身就是第一个首节点，所以 pre 指向 null。

## AQS 与 ReentrantLock 的联系
ReentrantLock 是根据 AQS 实现的独占锁，提供了两个构造方法

```java
 public ReentrantLock() {
        sync = new NonfairSync();
    }
    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

```

ReentrantLock 有三个内部类：Sync，NonfairSync，FairSync
![在这里插入图片描述](https://img-blog.csdnimg.cn/52499b5bb0d5442c8c604837e047566c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_13,color_FFFFFF,t_70,g_se,x_16)

这三个内部类都是基于 AQS 进行的实现，所以ReentrantLock 是基于 AQS 进行的实现。

ReentrantLock 提供两种类型的锁：
- 公平锁-FairSync
- 非公平锁-NonfairSync
- 默认实现是 NonFairSync。