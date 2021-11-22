---
title: Java集合之HashMap(三)
top: false
cover: false
toc: true
mathjax: false
abbrlink: 60692
date: 2021-07-23 20:40:44
author:
img:
coverImg:
password:
summary:
categories:
  - Java基础
tags:
  - HashMap
---

## 线程安全
在多线程，高并发的场景下，HashMap存在线程安全问题
- 主要原因在于并发下的rehash会造成元素之间会形成一个循环链表。
- jdk 1.8 后解决了这个问题，但是还是不应该在多线程下使用HashMap ，因为多线程下使用HashMap 还是会存在其他问题比如数据丢失。
- 并发环境下推荐使用Concur rentHashMap

## ConcurrentHashMap和HashMap的区别
Concur rentHashMap和Hashtable 的区别主要体现在实现线程安全的方式上不同。
### 数据结构
- Java8以前的ConcurrentHashMap采用分段的数组+链表实现
- Java8以后采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树。
- HashMap 和ConcurrentHashMap的底层数据结构类似都是采用数组+链表的形式
- 数组是HashMap 的主体，链表则是主要为了解决哈希冲突而存在的;
#### java8前
![在这里插入图片描述](https://img-blog.csdnimg.cn/813a80c60f4d4ec4b22098c8f98c5e1c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
#### Java8后
![在这里插入图片描述](https://img-blog.csdnimg.cn/eafa20c43aa14a999185c67c3b5bd3bd.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)


### 实现方式:
- 在Java8以前，ConcurrentHashMap 对整个数组进行了分段(Segment),
- 每一把锁只锁数组中一部分数据，多线程访问数组里不同数据段的数据，就不会存在锁竞争，提高并发访问率
- Java8以后已经放弃了段的概念，直接用数组+链表+红黑树的数据结构来实现
- 并发控制使用synchronized和CAS(比较交换)来操作
- Java8中虽然还能看到分段的数据结构，但属性都已经大大简化了，只是为了兼容旧版本
![在这里插入图片描述](https://img-blog.csdnimg.cn/1f692597e280492b97fe11f1658bb428.png)

### Hashtable实现线程安全的方式
- Hashtable(同一把锁) :使用synchronized 来保证线程安全，效率非常低下
- 当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态
- 如使用put添加元素，另一个线程不能使用put添加元素，也不能使用get, 竞争会越来越激烈
效率越低。

### JDK1.8与JDK1.7的性能对比
- HashMap中，如果key经过hash算法得出的数组索引位置全部不相同，即Hash算法非常好，那样的话，getKey方法的时间复杂度就是O(1)，
- 如果Hash算法技术的结果碰撞非常多，假如Hash算极其差，所有的Hash算法结果得出的索引位置一样，那样所有的键值对都集中到一个数组中，或者在一个链表中，或者在一个红黑树中，时间复杂度分别为O(n)和O(lgn)

### ConcurrentHashMap线程安全的具体实现方式/底层具体实现
#### JDK1.7 
- 首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当个线程占用锁访问其中一个段数据
时，其他段的数据也能被其他线程访问。
- ConcurrentHashMap是由Segment 数组结构和HashEntry 数组结构组成。
- Segment实现了Reentr antLock ，所以Segment 是一种可重入锁，扮演锁的角色。HashEntry 用于存储
键值对数据。
- 一个 ConcurrentHashMap 里包含一个Segment数组。Segment 的结构和HashMap类似，是一种数组和
链表结构，一个Segment 包含一个HashEntry 数组，每个HashEntry 是个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry 数组的数据进行修改时，必须首先获得
对应的Segment的锁 。
#### JDK1.8
- ConcurrentHashMap取消了分段锁，采用CAS和synchronized来保证并发安全。
- 数据结构跟HashMap1.8的结构类似，数组+链表/红黑二叉树。
- Java 8后在链表长度超过0时将链表转换为红黑树
- synchronized只锁定当前链表或红黑树的首节点，这样只要hash不冲突，就不会产生并发，提升效率


