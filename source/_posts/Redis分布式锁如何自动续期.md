---
title: Redis分布式锁如何自动续期
top: false
cover: false
toc: true
mathjax: false
categories:
  - 缓存
tags:
  - 分布式锁
  - 自动续期
abbrlink: 52824
date: 2021-11-27 17:46:38
author:
img:
coverImg:
password:
summary:
---

## Redis 实现分布式锁
- 指定一个 key 作为锁标记，存入 Redis 中，指定一个 唯一的用户标识作为 value。
- 当 key 不存在时才能设置值，确保同一时间只有一个客户端进程获得锁，满足互斥性特性。
- 设置一个过期时间，防止因系统异常导致没能删除这个 key，满足防死锁特性。
- 当处理完业务之后需要清除这个 key 来释放锁，清除 key 时需要校验 value 值，需要满足只有加锁的人才能释放锁 。

## 问题
如果这个锁的过期时间是30秒，但是业务运行超过了30秒，比如40秒，当业务运行到30秒的时候，锁过期了，其他客户端拿到了这个锁，怎么办

我们可以设置一个合理的过期时间，让业务能够在这个时间内完成业务逻辑，但LockTime的设置原本就很不容易。
- LockTime设置过小，锁自动超时的概率就会增加，锁异常失效的概率也就会增加；
- LockTime设置过大，万一服务出现异常无法正常释放锁，那么出现这种异常锁的时间也就越长。

我们只能通过经验去配置，一个可以接受的值，基本上是这个服务历史上的平均耗时再增加一定的buff。总体来说，设置一个合理的过期时间并不容易

我们也可以不设置过期时间，让业务运行结束后解锁，但是如果客户端出现了异常结束了或宕机了，那么这个锁就无法解锁，变成死锁；

## 自动续期
我们可以先给锁设置一个LockTime，然后启动一个守护线程，让守护线程在一段时间后，重新去设置这个锁的LockTime。

看起来很简单，但实现起来并不容易

1. 和释放锁的情况一样，我们需要先判断持有锁客户端是否有变化。否则会造成无论谁持有锁，守护线程都会去重新设置锁的LockTime。
2. 守护线程要在合理的时间再去重新设置锁的LockTime，否则会造成资源的浪费。不能动不动就去续。
3. 如果持有锁的线程已经处理完业务了，那么守护线程也应该被销毁。不能业务运行结束了，守护者还在那里继续运行，浪费资源。

## 看门狗
Redisson的看门狗机制就是这种机制实现自动续期的
![在这里插入图片描述](https://img-blog.csdnimg.cn/4d78502535604ab18ec0d26bdbb0bfb6.png)
##  Redissson tryLock

```java
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        long threadId = Thread.currentThread().getId();
        // 1.尝试获取锁
        Long ttl = tryAcquire(leaseTime, unit, threadId);
        // lock acquired
        if (ttl == null) {
            return true;
        }

        // 申请锁的耗时如果大于等于最大等待时间，则申请锁失败.
        time -= System.currentTimeMillis() - current;
        if (time <= 0) {
            acquireFailed(threadId);
            return false;
        }

        current = System.currentTimeMillis();

        /**
         * 2.订阅锁释放事件，并通过 await 方法阻塞等待锁释放，有效的解决了无效的锁申请浪费资源的问题：
         * 基于信息量，当锁被其它资源占用时，当前线程通过 Redis 的 channel 订阅锁的释放事件，一旦锁释放会发消息通知待等待的线程进行竞争.
         *
         * 当 this.await 返回 false，说明等待时间已经超出获取锁最大等待时间，取消订阅并返回获取锁失败.
         * 当 this.await 返回 true，进入循环尝试获取锁.
         */
        RFuture<RedissonLockEntry> subscribeFuture = subscribe(threadId);
        // await 方法内部是用 CountDownLatch 来实现阻塞，获取 subscribe 异步执行的结果（应用了 Netty 的 Future）
        if (!subscribeFuture.await(time, TimeUnit.MILLISECONDS)) {
            if (!subscribeFuture.cancel(false)) {
                subscribeFuture.onComplete((res, e) -> {
                    if (e == null) {
                        unsubscribe(subscribeFuture, threadId);
                    }
                });
            }
            acquireFailed(threadId);
            return false;
        }

        try {
            // 计算获取锁的总耗时，如果大于等于最大等待时间，则获取锁失败.
            time -= System.currentTimeMillis() - current;
            if (time <= 0) {
                acquireFailed(threadId);
                return false;

              }

            /**
             * 3.收到锁释放的信号后，在最大等待时间之内，循环一次接着一次的尝试获取锁
             * 获取锁成功，则立马返回 true，
             * 若在最大等待时间之内还没获取到锁，则认为获取锁失败，返回 false 结束循环
             */
            while (true) {
                long currentTime = System.currentTimeMillis();

                // 再次尝试获取锁
                ttl = tryAcquire(leaseTime, unit, threadId);
                // lock acquired
                if (ttl == null) {
                    return true;
                }
                // 超过最大等待时间则返回 false 结束循环，获取锁失败
                time -= System.currentTimeMillis() - currentTime;
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }

                /**
                 * 6.阻塞等待锁（通过信号量(共享锁)阻塞,等待解锁消息）：
                 */
                currentTime = System.currentTimeMillis();
                if (ttl >= 0 && ttl < time) {
                    //如果剩余时间(ttl)小于wait time ,就在 ttl 时间内，从Entry的信号量获取一个许可(除非被中断或者一直没有可用的许可)。
                    getEntry(threadId).getLatch().tryAcquire(ttl, TimeUnit.MILLISECONDS);
                } else {
                    //则就在wait time 时间范围内等待可以通过信号量
                    getEntry(threadId).getLatch().tryAcquire(time, TimeUnit.MILLISECONDS);
                }

                // 更新剩余的等待时间(最大等待时间-已经消耗的阻塞时间)
                time -= System.currentTimeMillis() - currentTime;
                if (time <= 0) {
                    acquireFailed(threadId);
                    return false;
                }
            }
        } finally {
            // 7.无论是否获得锁,都要取消订阅解锁消息
            unsubscribe(subscribeFuture, threadId);
        }
        return get(tryLockAsync(waitTime, leaseTime, unit));
    }
```
1. 尝试获取锁，返回 null 则说明加锁成功，返回一个数值，则说明已经存在该锁，ttl 为锁的剩余存活时间。
2. 如果此时客户端 2 进程获取锁失败，那么使用客户端 2 的线程 id（其实本质上就是进程 id）通过 Redis 的 channel 订阅锁释放的事件。如果等待的过程中一直未等到锁的释放事件通知，当超过最大等待时间则获取锁失败，返回 false，也就是第 39 行代码。如果等到了锁的释放事件的通知，则开始进入一个不断重试获取锁的循环。
3. 循环中每次都先试着获取锁，并得到已存在的锁的剩余存活时间。如果在重试中拿到了锁，则直接返回。如果锁当前还是被占用的，那么等待释放锁的消息，具体实现使用了信号量 Semaphore 来阻塞线程，当锁释放并发布释放锁的消息后，信号量的 release() 方法会被调用，此时被信号量阻塞的等待队列中的一个线程就可以继续尝试获取锁了。
4. 当锁正在被占用时，等待获取锁的进程并不是通过一个 while(true) 死循环去获取锁，而是利用了 Redis 的发布订阅机制,通过 await 方法阻塞等待锁的进程，有效的解决了无效的锁申请浪费资源的问题。

## 看门狗如何自动续期
Redisson看门狗机制， 只要客户端加锁成功，就会启动一个 Watch Dog。
```java
private <T> RFuture<Long> tryAcquireAsync(long leaseTime, TimeUnit unit, long threadId) {
    if (leaseTime != -1) {
        return tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
    }
    RFuture<Long> ttlRemainingFuture = tryLockInnerAsync(commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
    ttlRemainingFuture.onComplete((ttlRemaining, e) -> {
        if (e != null) {
            return;
        }

        // lock acquired
        if (ttlRemaining == null) {
            scheduleExpirationRenewal(threadId);
        }
    });
    return ttlRemainingFuture;
}
```

1. leaseTime 必须是 -1 才会开启 Watch Dog 机制，如果需要开启 Watch Dog 机制就必须使用默认的加锁时间为 30s。
2. 如果你自己自定义时间，超过这个时间，锁就会自定释放，并不会自动续期。

## 续期原理
续期原理其实就是用lua脚本，将锁的时间重置为30s

```java
private void scheduleExpirationRenewal(long threadId) {
    ExpirationEntry entry = new ExpirationEntry();
    ExpirationEntry oldEntry = EXPIRATION_RENEWAL_MAP.putIfAbsent(getEntryName(), entry);
    if (oldEntry != null) {
        oldEntry.addThreadId(threadId);
    } else {
        entry.addThreadId(threadId);
        renewExpiration();
    }
}

protected RFuture<Boolean> renewExpirationAsync(long threadId) {
    return commandExecutor.evalWriteAsync(getName(), LongCodec.INSTANCE, RedisCommands.EVAL_BOOLEAN,
            "if (redis.call('hexists', KEYS[1], ARGV[2]) == 1) then " +
                "redis.call('pexpire', KEYS[1], ARGV[1]); " +
                "return 1; " +
            "end; " +
            "return 0;",
        Collections.<Object>singletonList(getName()),
        internalLockLeaseTime, getLockName(threadId));
}
```
1. Watch Dog 机制其实就是一个后台定时任务线程，获取锁成功之后，会将持有锁的线程放入到一个 RedissonLock.EXPIRATION_RENEWAL_MAP里面，然后每隔 10 秒 （internalLockLeaseTime / 3） 检查一下，如果客户端 还持有锁 key（判断客户端是否还持有 key，其实就是遍历 EXPIRATION_RENEWAL_MAP 里面线程 id 然后根据线程 id 去 Redis 中查，如果存在就会延长 key 的时间），那么就会不断的延长锁 key 的生存时间。

2. 如果服务宕机了，Watch Dog 机制线程也就没有了，此时就不会延长 key 的过期时间，到了 30s 之后就会自动过期了，其他线程就可以获取到锁。

