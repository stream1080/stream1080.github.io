---
title: IO多路复用
date: 2021-11-06 16:03:21
cover: true
categories:
  - 操作系统
tags: 
  - IO流
top: false
---

## 概述
IO多路复用是一种同步IO模型，实现一个线程可以监视多个文件句柄
- 一旦某个文件句柄就绪，就能够通知应用程序进行相应的读写操作；
- 没有文件句柄就绪时会阻塞应用程序，交出cpu；
- 多路是指网络连接，复用指的是同一个线程。

## 三种实现方式
### select
- 时间复杂度O(n)，它仅仅知道了，有I/O事件发生了，却并不知道是哪那几个流（可能有一个，多个，甚至全部），
- 只能无差别轮询所有流，找出能读出数据，或者写入数据的流，对他们进行操作。
- 所以select具有O(n)的无差别轮询复杂度，同时处理的流越多，无差别轮询时间就越长。
#### 实现
```java
 int select (int n, fd_set *readfds, fd_set *writefds, 
             fd_set *exceptfds, struct timeval *timeout);
```

select 函数监视的文件描述符分3类，分别是writefds、readfds、和exceptfds。
调用后select函数会阻塞，直到有：
- 描述符就绪（有数据可读、可写、或者有except）
- 超时（timeout指定等待时间，如果立即返回设为null即可）
- 函数返回。

当select函数返回后，可以通过遍历fdset，来找到就绪的描述符。
#### 优缺点
- 良好的跨平台支持，select目前几乎在所有的平台上支持。
- 单个进程能够监视的文件描述符的数量存在最大限制，在Linux上一般为1024可以修改限制，但是这样也会造成效率的降低。

### poll
时间复杂度O(n)，poll本质上和select没有区别，它将用户传入的数组拷贝到内核空间，然后查询每个fd对应的设备状态， 
- 没有最大连接数的限制，原因是它是基于链表来存储的。

```java
int poll (struct pollfd *fds, unsigned int nfds, int timeout);
```
#### 实现
poll使用一个 pollfd的指针实现。

```java
struct pollfd {
    int fd;            /* file descriptor */
    short events;      /* requested events to watch */
    short revents;     /* returned events witnessed */
};
```
#### 优缺点
pollfd结构包含了要监视的event和发生的event，不再使用select“参数-值”传递的方式。
- pollfd并没有最大数量限制（但是数量过大后性能也是会下降）
- 和select函数一样，poll返回后，需要轮询pollfd来获取就绪的描述符。

### epoll
时间复杂度O(1)，epoll可以理解为event poll，不同于忙轮询和无差别轮询，epoll会把哪个流发生了怎样的I/O事件通知我们。
- epoll实际上是事件驱动（每个事件关联上fd）的，此时对这些流的操作都是有意义的。

#### 实现
epoll操作过程需要三个接口，分别如下：

```java
int epoll_create(int size);
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
int epoll_wait(int epfd, struct epoll_event * events, 
               int maxevents, int timeout);
```


```java
int epoll_create(int size)
```

- 创建一个epoll的句柄，size用来告诉内核这个监听的数目一共有多大，这个参数不同于select()中的第一个参数，给出最大监听的fd+1的值
- 参数size并不是限制了epoll所能监听的描述符最大个数，只是对内核初始分配内部数据结构的一个建议。
- 当创建好epoll句柄后，它就会占用一个fd值，在使用完epoll后，必须调用close()关闭，否则可能导致fd被耗尽。

```java
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event)
```

函数是对指定描述符fd执行op操作。
- epfd：是epoll_create()的返回值。
- op：表示op操作，用三个宏来表示，分别添加、删除和修改对fd的监听事件。
  - 添加EPOLL_CTL_ADD，
  - 删除EPOLL_CTL_DEL，
  - 修改EPOLL_CTL_MOD。
- fd：是需要监听的fd（文件描述符）
- epoll_event：是告诉内核需要监听什么事，

```java
int epoll_wait(int epfd, struct epoll_event * events, 
               int maxevents, int timeout)
```

等待epfd上的io事件，最多返回maxevents个事件。
- 参数events用来从内核得到事件的集合
- maxevents告之内核这个events有多大，这个maxevents的值不能大于创建epoll_create()时的size，
- 参数timeout是超时时间（毫秒，0会立即返回，-1将不确定，也有说法说是永久阻塞）。该函数返回需要处理的事件数目，如返回0表示已超时。