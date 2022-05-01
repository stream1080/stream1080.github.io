---
title: JVM垃圾收集器
top: false
cover: false
toc: true
mathjax: false
categories:
  - JVM
tags:
  - 垃圾收集器
abbrlink: 51819
date: 2021-08-18 23:05:48
author:
img:
coverImg:
password:
summary:
---

垃圾收集算法是内存回收的方法论，那么垃圾收集器就是内存回收的具体实现。
## 垃圾收集器分类
目前，有很多的垃圾收集器，各类垃圾收集器各有优缺点，但目前为止还没有最好的
垃圾收集器出现，更加没有万能的垃圾收集器，我们能做的就是根据具体应用场景选择适合自己的垃圾
收集器
下图是7 种垃圾回收器
![在这里插入图片描述](https://img-blog.csdnimg.cn/89edeadfaf1842daaea33649a198963b.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
## Serial收集器
Serial收集器是最基本、发展历史最久的收集器，这个收集器是采用标记-复制算法的单线程的收集器。

- 单线程一方面意味着他只会使用一个 CPU 或者一条线程去完成垃圾收集工作，
- 另一方面也意味着他进行垃圾收集时必须暂停其他线程的所有工作，直到它收集结束为止。

![在这里插入图片描述](https://img-blog.csdnimg.cn/34c23b3debb74c669df6ed6844bd16c9.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
## Parnew收集器
Parnew 收集器其实就是Serial收集器的多线程版本，除了使用多线程进行垃圾收集外，其余行为和 Serial 收集器完全一样，但是他却是 Server 模式下的虚拟机首选的新生代收集器。

- 除了 Serial 收集器外，目前只有它能与 CMS 收集器配合工作。CMS 收集器第一次实现了让垃圾收集器与用户线程基本上同时工作；CMS收集器我们后面会说
- Parnew 收集器默认开启的收集线程数与 CPU 数量相同，在 CPU 数量非常多的情况下，可以使用 -XX:ParallelGCThreads 参数来限制垃圾收集的线程数。
![在这里插入图片描述](https://img-blog.csdnimg.cn/60910e6d076d4f4f92ff4136c90614e3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
## Parallel Scavenge收集器
Parallel Scavenge 收集器也是一个新生代收集器，也采用了标志-复制算法，也是并行的多线程收集器。Parallel Scavenge 收集器的目标是达到一个可控制的吞吐量。Parallel Scavenge 收集器是虚拟机运行在 Server 模式下的默认垃圾收集器。被称为“吞吐量优先收集器”。
- 采用标记-复制算法
- 多线程收集
- 控制吞吐量的目标。

Parallel Scavenge 收集器运行过程同 Parnew 收集器基本一致
![在这里插入图片描述](https://img-blog.csdnimg.cn/60910e6d076d4f4f92ff4136c90614e3.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
- 控制吞吐量：CMS 等收集器的关注点是尽可能缩短垃圾收集时用户线程的停顿时间，而 Parallel Scavenge 收集器的目标则是达到一个可控制的吞吐量。
- 吞吐量就是 CPU 用于运行用户代码时间与 CPU 总消耗时间的比值，即吞吐量=运行用户代码时间/（运行用户代码时间+垃圾收集时间）。

- 空间吞吐量参数介绍：虚拟机提供了-XX：MaxGCPauseMills 和 -XX：GCTimeRatio 两个参数来精确控制最大垃圾收集停顿时间和吞吐量大小。
- GC 停顿时间的缩短是以牺牲吞吐量和新生代空间换取的。由于与吞吐量关系密切，Parallel Scavenge 收集器也被称为“吞吐量优先收集器”。

## Serial Old收集器
 Serial Old 收集器与Serial收集器是一个单线程收集器，作用于老年代，使用“标记-整理算法”，这个收集器的主要意义也是在于给 Client 模式下的虚拟机使用。
 运行过程与Serial基本一致
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/34c23b3debb74c669df6ed6844bd16c9.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
## Parallel Old收集器
Parallel Old 收集器是 Parallel Scavenge 收集器的老年代版本，使用多线程和“标记-整理算法”进行垃圾回收。
在注重吞吐量以及CPU资源的场合，都可以优先考虑Parallel Scavenge收集器和Parallel old收集器。
![在这里插入图片描述](https://img-blog.csdnimg.cn/d801f35d82964bfd98b10b2d378dca38.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
## CMS收集器
CMS（Conrrurent Mark Sweep，连续标记扫描）收集器是以获取最短回收停顿时间为目标的收集器。使用标记-清除算法。CMS收集器是HotSpot虚拟机第一款真正意义上的并发收集器，它第一次实现了让垃圾收集线程与用户线程(基本上)同时工作。

#### 收集过程
- 初始标记：暂停所有的其他线程，标记 GC Roots 能直接关联到的对象，时间很短；
- 并发标记：同时开启GC和用户线程，进行 GC Roots Tracing（可达性分析）过程，时间很长；
- 重新标记：修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，时间比初始标记长；但远远小于并发标记的时间
- 并发清除：开启用户线程，回收内存空间，时间很长。其中，并发标记与并发清除两个阶段耗时最长，但是可以与用户线程并发执行。
#### 运行过程
![在这里插入图片描述](https://img-blog.csdnimg.cn/fcce9b8b4741450b9a2c54e765ae5a48.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
#### 优点
- 并发收集
- 低停顿
#### 缺点
- 对CPU资源敏感;
- 无法处理浮动垃圾;
- 它使用的回收算法“标记-清除"算法会导致收集结束时会有大量空间碎片产生。


## G1收集器
G1 是目前技术发展的比较最前沿成果之一，HotSpot开发团队赋予它的使命是未来可以替换掉 JDK1.5 中发布的 CMS 收集器。
#### 特点
- 并发和并行：使用多个 CPU 来缩短 Stop The World 停顿时间，与用户线程并发执行；
- 分代收集：独立管理整个堆，但是能够采用不同的方式去处理新创建对象和已经存活了一段时间、熬过多次 GC 的旧对象，以获取更好的收集效果；
- 空间整合：基于标记-整理算法，无内存碎片产生；
- 可预测的停顿：能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在垃圾收集器上的时间不得超过N毫秒。
#### 回收步骤
- 初始标记:标记GC Roots能直接关联到的对象，让下一阶段用户线程并发运行时能正确地在可用
Region中分配新对象。需要STW但耗时很短，在Minor GC时同步完成。
- 并发标记:从GC Roots开始对堆中对象进行可达性分析，递归扫描整个堆的对象图。耗时长但可
与用户线程并发，扫描完成后要重新处理SATB记录的在并发时有变动的对象。
- 最终标记:对用户线程做短暂暂停，处理并发阶段结束后仍遗留下来的少量SATB记录。
- 筛选回收:对各Region的回收价值排序，根据用户期望停顿时间制定回收计划。必须暂停用户线
程，由多条收集线程并行完成。

G1收集器在后台维护了一个个优先列表，每次根据允许的收集时间，优先选择回收价值最大的Region(这
也就是它的名字Garbage-First的由来)。这种使用Region划分内存空间以及有优先级的区域回收方式，
保证了收集器在有限时间内可以尽可能高的收集效率(把内存化整为零)

在G1之前的垃圾收集器，收集的范围都是整个新生代或者老年代，而 G1 不再是这样。使用 G1 收集器时，Java 堆的内存布局与其他收集器有很大差别，它将整个Java堆划分为多个大小相等的独立区域（Region），虽然还保留有新生代和老年代的概念，但新生代和老年代不再是物理隔离的了，它们都是一部分（可以不连续）Region 的集合。



