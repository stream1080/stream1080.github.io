---
title: synchronized关键字
cover: false
top: false
categories:
  - 多线程
tags:
  - synchronized
  - 锁
abbrlink: 27249
date: 2021-09-17 21:05:57
summary:
---

## 概念
synchronized 同步块是 Java 提供的一种原子性内置锁，Java 中的每个对象都可以把它当作一个同步锁来使用，这些 Java 内置的使用者看不到的锁被称为内部锁，也叫作监视器锁。

#### 内置锁
- 也叫排它锁，也就是当一个线程获取这个锁后，其他线程必须等待该线程释放锁后才能获取该锁。

- 代码在进入 synchronized 代码块前会自动获取内部锁，这时候其他线程访问该同步代码块时会被阻塞挂起。
- 拿到内部锁的线程会在正常退出同步代码块或者抛出异常后或者在同步块内调用了该内置锁资源的 wait 系列方法时释放该内置锁。

#### 上下文切换
由于 Java 中的线程是与操作系统的线程一一对应的，所以当阻塞一个线程时，需要从用户态切换到内核态执行阻塞操作，这是很耗时的操作，这就是上下文切换，而 synchronized 的使用就会导致上下文切换。

## 作用
在并发编程中存在线程安全问题，使用 synchronized 关键字能够有效的避免多线程环境下的线程安全问题。
#### 线程安全问题
- 存在共享数据，共享数据是对多线程可见的，所有的线程都有权限对共享数据进行操作；
- 多线程共同操作共享数据。关键字 synchronized 可以保证在同一时刻，只有一个线程可以执行某个同步方法或者同步代码块，同时 synchronized 关键字可以保证一个线程变化的可见性；
- 多线程共同操作共享数据且涉及增删改操作。如果只是查询操作，是不需要使用 synchronized 关键字的，在涉及到增删改操作时，为了保证数据的准确性，可以选择使用 synchronized 关键字。

## 三种使用方式
Java 中每一个对象都可以作为锁，synchronized 有三种使用方式
- 作用在普通同步方法（实例方法）：锁是当前实例对象 ，进入同步代码前要获得当前实例的锁；
- 作用在静态同步方法：锁是当前类的 class 对象 ，进入同步代码前要获得当前类对象的锁；
- 作用在同步方法块：锁是括号里面的对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁。

## 双重检验锁实现单例模式

```java
public class DoubleCheck{
    private volatile static DoubleCheck instance;
    private DoubleCheck(){};

    public synchronized static DoubleCheck getIntstance(){
        //先判断对象是否实例过，没有则进入加锁代码
        if(instance == null){
            //类对象加锁
            synchronized(DoubleCheck.class){
                if(instance == null){
                    instance = new DoubleCheck();
                }
            }
        }
        return instance;
    }
}
```
- 使用synchronized关键字，内外两次判断对象是否存在
- 必须使用volatile关键字修饰，防止jvm指令重排
