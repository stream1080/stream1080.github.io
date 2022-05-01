---
title: ThreadLocal的使用
cover: false
top: false
categories:
  - 多线程
tags:
  - ThreadLocal
  - 共享变量
abbrlink: 22538
date: 2021-09-15 21:11:04
summary:
---

## 概述
ThreadLocal 为解决多线程程序的并发问题提供了一种新的思路，使用这个工具类可以很简洁地编写出优美的多线程程序。

- ThreadLocal 很容易让人望文生义，想当然地认为是一个 “本地线程”。其实，ThreadLocal 并不是一个 Thread，而是 Thread 的局部变量，也许把它命名为 ThreadLocalVariable 更容易让人理解一些。

- 当使用 ThreadLocal 维护变量时，ThreadLocal 为每个使用该变量的线程提供独立的变量副本，所以每一个线程都可以独立地改变自己的副本，而不会影响其它线程所对应的副本。

从线程的角度看，目标变量就象是线程的本地变量，这也是类名中 “Local” 所要表达的意思。


## 作用
ThreadLocal提供了线程本地变量，如果你创建了一个 ThreadLocal 变量，那么访问这个变量的每个线程都会有这个变量的一个本地副本。当多个线程操作这个变量时，实际操作的是自己本地内存里面的变量，从而避免了线程安全问题。
![在这里插入图片描述](https://img-blog.csdnimg.cn/da73098b996a4ca582c20451c3665a3c.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)

ThreadLocal 是线程本地存储，在每个线程中都创建了一个 ThreadLocalMap 对象，每个线程可以访问自己内部 ThreadLocalMap 对象内的 value。通过这种方式，避免资源在多线程间共享。

## set()和get()方法
set 方法是为了设置 ThreadLocal 变量，设置成功后，该变量只能够被当前线程访问，其他线程不可直接访问操作改变量。参数是泛型
![在这里插入图片描述](https://img-blog.csdnimg.cn/add11667446f4362b06ddd7dd0650846.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)

get 方法是为了获取 ThreadLocal 变量的值，get 方法没有任何入参，直接调用即可获取。
![在这里插入图片描述](https://img-blog.csdnimg.cn/402c399592cb48ac9707f2629e92f9ca.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
## remove() 方法
remove 方法是为了清除 ThreadLocal 变量，清除成功后，该 ThreadLocal 中没有变量值。
![在这里插入图片描述](https://img-blog.csdnimg.cn/144cb735fb68441f9b4db94121c50f34.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
## 单线程使用案例
![在这里插入图片描述](https://img-blog.csdnimg.cn/d06fb2b2ea2e4d96a728e4cc6a93099f.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
## 多线程下的 ThreadLocal使用案例
看个简单案例
- 线程1设置了ThreadLocal（thread-1 local value）
- 线程2设置了ThreadLocal（thread-2 local value）
- 线程2remove变量ThreadLocal
- 看看线程1是否还能打印之前设置的ThreadLocal

```java
package com.stream.juc;

/**
 * @author stream
 * @since 2021/9/15 16:11
 */
public class ThreadLocalTest {

    public static ThreadLocal<String> local = new ThreadLocal<>();

    public static void main(String[] args) {
        Thread threadOne = new Thread(new Runnable() {
            @Override
            public void run() {
                local.set("thread-1 local value");
                try {
                    // 等待5000毫秒，确保threadTwo 执行remove完成
                    Thread.sleep(5000); 
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(local.get());
            }
        });
        Thread threadTwo = new Thread(new Runnable() {
            @Override
            public void run() {
                local.set("thread-2 local value");
                System.out.println(local.get());
                local.remove();
                System.out.println("thread-2 remove 变量local 操作完毕。");
            }
        });
        
        threadTwo. start();
        threadOne. start();
    }
}

```
执行结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/d9aa7d2f2fd4417babdb562ef719b084.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
- 从结果来看，在 threadTwo 执行完 remove 方法后，threadOne 仍然能够成功打印
- 从而证明了 ThreadLocal 的专属特性，线程独有，其他线程不可修改

## 小结
- ThreadLocal 是解决线程安全问题一个很好的思路，
- 它通过为每个线程提供一个独立的变量副本解决了变量并发访问的冲突问题。
- ThreadLocal 比直接使用 synchronized 同步机制解决线程安全问题更简单，更方便，且程序拥有更高的并发性。
