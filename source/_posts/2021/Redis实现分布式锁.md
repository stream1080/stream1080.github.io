---
title: Redis实现分布式锁
cover: false
top: false
categories:
  - 缓存
tags:
  - 分布式锁
  - Redis
abbrlink: 2566
date: 2021-10-10 23:35:46
summary:
---

## 什么是分布式锁
分布式锁其实可以理解为:控制分布式系统有序的去对共享资源进行操作,通过互斥来保持一致性
## 为什么要分布式锁
当多个线程需要并发修改一个数据时，为了避免竞争，在单机的情况下，加synchronized或者Lock即可实现互斥

但在分布式的环境下，当多个server并发修改同一个资源时，为了避免竞争就需要使用分布式锁。

那为什么不能使用Java自带的锁（synchronized或者Lock）呢？
- 因为Java中的锁是面向多线程设计的，它只局限于当前的实例。
- 而多个server实际上是多进程，是不同的实例，所以Java自带的锁机制在这个场景下是无效的。

## 如何实现分布式锁
采用Redis实现分布式锁，就是在Redis里存一份代表锁的数据，通常用随机字符串即可。

### 加锁：
#### 第一种方式

```java
setnx key value
```
这种方式的缺点是容易产生死锁，因为客户端有可能忘记解锁，或者解锁失败。

#### 第二种方式
```java
setnx key value
expire key seconds
```
给锁增加了过期时间，避免出现死锁。但这两个命令不是原子的，第二步可能会失败，依然无法避免死锁问题。
#### 第三种方式
```java
set key value nx ex seconds 
```

通过“set...nx...”命令，将加锁、过期命令编排到一起，它们是原子操作了，可以避免死锁。

### 解锁：
```java
del key
```
解锁就是删除代表锁的那份数据，直接删除redis上面的数据。



### 问题

上述方法看起来没有问题，但实际是有隐患的
![在这里插入图片描述](https://img-blog.csdnimg.cn/4678bef1efc04094b02729431c2069d8.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)

1. 进程A在任务没有执行完毕时，锁已经到期被释放了。
2. 等进程A的任务执行结束后，A会尝试释放锁，但是，它的锁已经过期不存在了，它此时释放的可能是其他线程的锁，比如B进程；
3. 红框时间内，两个进程同时操作数据，极有可能出现线程安全的问题；

### 如何解决
在加锁时就要给锁设置一个标识，加锁进程要记住这个标识。
- 当进程解锁的时候，进行判断，是自己持有的锁才能释放
- 否则无法释放。可以为key设置一个随机字符串，来充当进程的标识。

但是解锁的时候，判断、释放，这两步需要保证原子性，否则第二步失败的话，就会出现死锁，但判断和删除命令不是原子的。

在Redis中可以使用Lua脚本，通过Lua脚本将两个命令编排在一起，而整个Lua脚本的执行是原子的。

```java
# 加锁
set key random-value nx ex seconds 

# 解锁
if redis.call("get",KEYS[1]) == ARGV[1] then
    return redis.call("del",KEYS[1])
else
    return 0
end
```

