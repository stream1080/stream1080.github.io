---
title: volatile 关键字
cover: false
top: false
categories:
  - 多线程
tags:
  - volatile
abbrlink: 38250
date: 2021-09-19 21:03:41
summary:
---

## volatile 关键字
- volatile 关键字解决内存可见性问题，是一种弱形式的同步。
- 该关键字可以确保当一个线程更新共享变量时，更新操作对其他线程马上可见。
- 当一个变量被声明为 volatile 时，线程在写入变量时不会把值缓存在寄存器或者其他地方，而是会把值刷新回主内存。
- 当其他线程读取该共享变量时，会从主内存重新获取最新值，而不是使用当前线程的工作内存中的值。

## 原理
Java 语言提供了一种弱同步机制，即 volatile 变量，用来确保将变量的更新操作通知到其他线程。

当把变量声明为 volatile 类型后，编译器与运行时都会注意到这个变量是共享的，volatile 变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取 volatile 类型的变量时总会返回最新写入的值。

在访问 volatile 变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此 volatile 变量是一种比 sychronized 关键字更轻量级的同步机制。

![](https://img-blog.csdnimg.cn/9e8d11a885064a3ab64812f35796106b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_19,color_FFFFFF,t_70,g_se,x_16)

- 当对非 volatile 变量进行读写的时候，每个线程先从内存拷贝变量到 CPU 缓存中。如果计算机有多个 CPU，每个线程可能在不同的 CPU 上被处理，这意味着每个线程可以拷贝到不同的 CPU cache 中。

- 而声明变量是 volatile 的，JVM 保证了每次读变量都从内存中读，跳过 CPU cache。

## 作用
- 保证变量的可见性
- 禁止指令重排

#### 保证变量的可见性
当一条线程修改了变量值，新值对于其他线程来说是立即可以得知的。volatile 变量在各个线程的
工作内存中不存在-致性问题，但Java的运算操作符并非原子操作，导致volatile变量运算在并
发下仍不安全。
#### 禁止指令重排
使用volatile变量进行写操作，汇编指令带有lock前缀，相当于一个内存屏障，后面的指令不能
重排到内存屏障之前。
#### 使用lock前缀引发两件事:
1. 将当前处理器缓存行的数据写回系统内存。
2. 使其他处理器的缓存无效。

相当于对缓存变量做了- -次store和write操作，让volatile变量的修改对其他处理器立即可见。


## volatile 与 synchronized 的区别
#### 相似处：
volatile 的内存语义和 synchronized 有相似之处，具体来说就是，当线程写入了 volatile 变量值时就等价于线程退出 synchronized 同步块（把写入工作内存的变量值同步到主内存），读取 volatile 变量值时就相当于进入 synchronized 同步块（ 先清空本地内存变量值，再从主内存获取最新值）。

#### 区别：
使用锁的方式可以解决共享变量内存可见性问题，但是使用锁太笨重，因为它会带来线程上下文的切换开销。具体区别如下：
- volatile 本质是在告诉 jvm 当前变量在寄存器（工作内存）中的值是不确定的，需要从主存中读取；
- synchronized 则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住；
- volatile 仅能使用在变量级别；synchronized 则可以使用在变量、方法、和类级别的；
- volatile 仅能实现变量的修改可见性，不能保证原子性；而 synchronized 则可以保证变量的修改可见性和原子性；
- volatile 不会造成线程的阻塞；synchronized 可能会造成线程的阻塞；
- volatile 标记的变量不会被编译器优化；synchronized 标记的变量可以被编译器优化
