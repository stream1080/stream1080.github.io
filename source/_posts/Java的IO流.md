---
title: Java的IO流
cover: false
top: false
categories:
  - 多线程
tags:
  - IO流
abbrlink: 33363
date: 2021-10-03 20:47:21
summary:
---

## 基本概念
### 同步和异步
同步和异步是通信机制
- **同步**：同步IO是用户线程发起IO请求后需要等待或轮询内核IO操作完成后才能继续执行。
- **异步**：异步IO是用户线程发起IO请求后可以继续执行，当内核IO操作完成后会通知用户线程，或调用用户线程注册的回调函数。


## 阻塞和非阻塞
阻塞和非阻塞是调用状态

- **阻塞**：阻塞IO是IO操作需要彻底完成后才能返回用户空间。
- **非阻塞**：非阻塞IO是IO操作调用后立即返回一个状态值，无需等IO操作彻底完成。

## BIO
BIO是同步阻塞式IO，是JDK1.4 之前的IO模型。
- 服务器实现模式为一个连接请求对应一个线程，服务器需要为每一个客户端请求创建一个线程，如果这个连接不做任何事会造成不必要的线程开销。
- 可以通过线程池改善，这种IO称为伪异步IO。适用连接数目少且服务器资源多的场景。

## NIO
NIO是同步非阻塞IO，JDK1.4引入。服务器实现模式为多个连接请求对应一个线程，客户端连接请求会注册到一个多路复用器Selector，Selector 轮询到连接有IO请求时才启动一个线程处理。适用连接数目多且连接时间短的场景。

- 同步是指线程还是要不断接收客户端连接并处理数据
- 非阻塞是指如果一个管道没有数据，不需要等待，可以轮询下一个管道。

核心组件
![在这里插入图片描述](https://img-blog.csdnimg.cn/998e97c315244aa8a478105aac7f055b.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_20,color_FFFFFF,t_70,g_se,x_16)
#### Selector
多路复用器，轮询检查多个Channel的状态，判断注册事件是否发生，即判断
Channel是否处于可读或可写状态。使用前需要将Channel注册到Selector，注册后会得到一个SelectionKey，通过SelectionKey获取Channel和Selector相关信息。
#### Channel
双向通道，替换了BIO 中的Stream流，不能直接访问数据，要通过Buffer来读写数据，也可以和其他Channel交互。
![在这里插入图片描述](https://img-blog.csdnimg.cn/edee367f257d4e99bed4bd022bd46dd6.png?x-oss-process=image/watermark,type_ZHJvaWRzYW5zZmFsbGJhY2s,shadow_50,text_Q1NETiBA5LiA5rGf5rqq5rC0,size_17,color_FFFFFF,t_70,g_se,x_16)

#### Buffer
![在这里插入图片描述](https://img-blog.csdnimg.cn/f0d063ff0fac40a88dfc149007f61bf0.png)

缓冲区，本质是一块可读写数据的内存，用来简化数据读写。
Buffer三个重要属性：
- position下次读写数据的位置
- limit 本次读写的极限位置
- capacity 最大容量
    - flip将写转为读，底层实现原理把position置0，并把limit设为当前的position值
    - clear将读转为写模式(用于读完全部数据的情况，把position置0，limit设为capacity)
    - compact将读转为写模式(用于存在未读数据的情况，让position指向未读数据的下一个)
   - 通道方向和Buffer方向相反，读数据相当于向Buffer写，写数据相当于从Buffer读
   
使用步骤
- 向Buffer写数据，调用flip方法转为读模式；
- 从Buffer中读数据，调用clear或compact方法清空缓冲区；

## AIO
异步非阻塞IO，JDK1.7引入，服务器实现模式为一个有效请求对应一个线程，客户端的IO请求都是由操作系统先完成IO操作后再通知服务器应用来直接使用准备好的数据。适用连接数目多且连接时间长的场景。

- 异步是指服务端线程接收到客户端管道后就交给底层处理IO通信，自己可以做其他事情
- 非阻塞是指客户端有数据才会处理，处理好再通知服务器。

#### 实现方式
- 通过Future的get方法进行阻塞式调用
- 实现CompletionHandler接口，重写请求成功的回调方法completed 和请求失败回调方法failed 

