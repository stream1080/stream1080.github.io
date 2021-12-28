---
title: 悲观锁VS乐观锁
top: false
cover: false
toc: true
mathjax: false
categories:
  - 多线程
tags:
  - 锁
abbrlink: 32478
date: 2021-12-27 23:59:39
author:
img:
coverImg:
password:
summary:
---

### 前言
Java中有很多锁，每种锁因其特性的不同，在适当的场景下的效率也有很大的差别。今天我们对比一下乐观锁和悲观锁，看看他们有什么不同和相同。

乐观锁与悲观锁是一种广义上的概念，体现了看待线程同步的不同角度。在Java和数据库中都有比较广泛的应用。
### 悲观锁

对于同一个数据的并发操作，悲观锁认为自己在使用数据的时候一定有别的线程来修改数据，因此在获取数据的时候会先加锁，确保数据不会被别的线程修改。Java中，synchronized关键字和Lock的实现类都是悲观锁。
![在这里插入图片描述](https://img-blog.csdnimg.cn/4a396949d1e8496bb91413d287f2bbdb.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/d9d86aa4f8eb4e9d96ed9b3db6ac5088.png)
### 悲观锁调用方式
分别是 synchronized 和 ReentrantLock 
```java
// synchronized
public synchronized void testMethod() {
	// 操作同步资源
}

// ReentrantLock
private ReentrantLock lock = new ReentrantLock(); 
// 需要保证多个线程使用的是同一个锁
public void modifyPublicResources() {
	lock.lock();
	// 操作同步资源
	lock.unlock();
}

```
### 乐观锁
乐观锁认为自己在使用数据时不会有别的线程修改数据，所以不会添加锁，只是在更新数据的时候去判断之前有没有别的线程更新了这个数据。如果这个数据没有被更新，当前线程将自己修改的数据成功写入。如果数据已经被其他线程更新，则根据不同的实现方式执行不同的操作（报错或者重试）

![在这里插入图片描述](https://img-blog.csdnimg.cn/0f7de6532433403eb66df416a5399138.png)
### 乐观锁调用方式
乐观锁在Java中是通过使用无锁编程来实现
- 最常采用的是CAS算法；
- Java原子类中的递增操作就通过CAS自旋实现的。
```java
private AtomicInteger atomicInteger = new AtomicInteger();  
// 需要保证多个线程使用的是同一个AtomicInteger
atomicInteger.incrementAndGet(); //执行自增
```

### 适用场景
- 悲观锁适合写操作多的场景，先加锁可以保证写操作时数据正确。
- 乐观锁适合读操作多的场景，不加锁的特点能够使其读操作的性能大幅提升。

### 实现方式
悲观锁基本都是在显式的锁定之后再操作同步资源，而乐观锁则直接去操作同步资源。为什么乐观锁能够做到不锁定同步资源也可以正确的实现线程同步呢？主要就是CAS的思想了。
### CAS
CAS全称 Compare And Swap（比较与交换），是一种无锁算法。在不使用锁（没有线程被阻塞）的情况下实现多线程之间的变量同步。java.util.concurrent包中的原子类就是通过CAS来实现了乐观锁。

CAS算法涉及到三个操作数：

- 需要读写的内存值 V。
- 进行比较的值 A。
- 要写入的新值 B。
当且仅当 V 的值等于 A 时，CAS通过原子方式用新值B来更新V的值（“比较+更新”整体是一个原子操作），否则不会执行任何操作。一般情况下，“更新”是一个不断重试的操作。

之前提到java.util.concurrent包中的原子类，就是通过CAS来实现了乐观锁，那么我们进入原子类AtomicInteger的源码，看一下AtomicInteger的定义：

```java
public class AtomicInteger extends Number impLements java.io.Serializable {
private static final long serialVersionUID = 6214790243416807050L;

// setup to use Unsafe. compareAndSwapInt for updates
private static final Unsafe unsafe = Unsafe. getUnsafe();
private static final long value0ffset;
static {
	try {
	value0ffset = unsafe.objectField0ffset
		(AtomicInteger.class.getDeclaredField("value"));
	} catch (Exception ex) { throw new Error(ex); }
}

private volatile int vaLue;

```
各属性的含义：

- **unsafe：** 获取并操作内存的数据。
- **valueOffset：** 存储value在AtomicInteger中的偏移量。
- **value：** 存储AtomicInteger的int值，该属性需要借助volatile关键字保证其在线程间是可见的。

查看AtomicInteger的自增函数incrementAndGet()的源码时，发现自增函数底层调用的是unsafe.getAndAddInt()。但这个方法并不能看出自增过程

```java
// AtomicInteger 自增方法
public final int incrementAndGet() {
  return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}

// Unsafe.class
public final int getAndAddInt(Object var1, long var2, int var4) {
  int var5;
  do {
      var5 = this.getIntVolatile(var1, var2);
  } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
  return var5;
}
```
getAndAddInt()循环获取给定对象o中的偏移量处的值v，然后判断内存值是否等于v。如果相等则将内存值设置为 v + delta，否则返回false，继续循环进行重试，直到设置成功才能退出循环，并且将旧值返回。

整个“比较+更新”操作封装在compareAndSwapInt()中，在JNI里是借助于一个CPU指令完成的，属于原子操作，可以保证多个线程都能够看到同一个变量的修改值。

后续JDK通过CPU的cmpxchg指令，去比较寄存器中的 A 和 内存中的值 V。如果相等，就把要写入的新值 B 存入内存中。如果不相等，就将内存值 V 赋值给寄存器中的值 A。然后通过Java代码中的while循环再次调用cmpxchg指令进行重试，直到设置成功为止。

OpenJDK8 中的 Unsafe.java 源码
```java
// Unsafe.java
public final int getAndAddInt(Object o, long offset, int delta) {
   int v;
   do {
       v = getIntVolatile(o, offset);
   } while (!compareAndSwapInt(o, offset, v, v + delta));
   return v;
}
```


### CAS问题
- ABA问题。CAS需要在操作值的时候检查内存值是否发生变化，没有发生变化才会更新内存值。
   - 但是如果内存值原来是A，后来变成了B，然后又变成了A，那么CAS进行检查时会发现值没有发生变化，但是实际上是有变化的。
  - JDK从1.5开始提供了AtomicStampedReference类来解决ABA问题，具体操作封装在compareAndSet()中。compareAndSet()首先检查当前引用和当前标志与预期引用和预期标志是否相等，如果都相等，则以原子方式将引用值和标志的值设置为给定的更新值。
- 循环时间长开销大。CAS操作如果长时间不成功，会导致其一直自旋，给CPU带来非常大的开销。
- 只能保证一个共享变量的原子操作。对一个共享变量执行操作时，CAS能够保证原子操作，但是对多个共享变量操作时，CAS是无法保证操作的原子性的。
  - Java从1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，可以把多个变量放在一个对象里来进行CAS操作。

