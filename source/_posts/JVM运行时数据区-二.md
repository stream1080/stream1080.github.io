---
title: JVM运行时数据区(二)
top: false
cover: false
toc: true
mathjax: false
categories:
  - JVM
tags:
  - 内存区域
abbrlink: 40122
date: 2021-08-15 23:12:04
author:
img:
coverImg:
password:
summary:
---

# JVM运行时数据区(二)
 [上篇文章](https://blog.csdn.net/upstream480/article/details/119703305)写了JVM运行时数据区中的程序计数器，Java虚拟机栈和本地方法栈。这篇文章我们接着班Java运行时数据区中的堆和方法区说一下
 ![在这里插入图片描述](https://img-blog.csdnimg.cn/f3594045c19443f6ade8f38be3b57652.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)

## 方法区(Method Area)
方法区，也称非堆（Non-Heap），是一个被线程共享的内存区域。其中主要存储加载的类字节码、class/method/field 等元数据对象、static-final 常量、static 变量、JIT 编译器编译后的代码等数据。
### 方法区也叫永久代

- 方法区和永久代的关系很像Java中接口和类的关系，类实现了接口，
- 而永久代就是HotSpot虚拟机对虚拟机规范中方法区的一种实现方式。
- 永久代是HotSpot的概念，方法区是Java虚拟机规范中的定义，是一种规范， 
- 永久代是一种实现，一个是标准一个是实现，其他的虚拟机实现没有永久。

### 方法区存放的数据
- 类型全限定名：全限定名为 package 路径与类名称组合起来的路径；
- 类型的直接超类的全限定名：父类或超类的全限定名；
- 类的类型：判定当前类是 Class 还是接口 Interface；
- 类型的访问修饰符：判断修饰符，如 pulic，private 等；
- 类型的常量池：常量池
- 字段信息：类中字段的信息；
- 方法信息：类中方法的信息；
- 静态变量：类中的静态变量信息；
- 一个到类 ClassLoader 的引用：对 ClassLoader 的引用，这个引用指向对内存；
- 一个到 Class 类的引用：对对象实例的引用，这个引用指向对内存。

另外，方法区包含了一个特殊的区域 “运行时常量池”。
### 运行时常量池

- 运行时常量池是方法区的一部分。
- Class 文件中有类的版本、字段、方法、接口等描述信息，还
- 有常量池信息(用于存放编译期生成的各种字面量和符号引用)；
- 当常量池无法再申请到内存时会抛出OutOfMemoryError 异常。

常量池主要用于存放两大类常量：字面量（Literal）和符号引用量（Symbolic References）
![在这里插入图片描述](https://img-blog.csdnimg.cn/bc4eb4934f784c65bcf5459eeba04c87.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
### 常量池好处
常量池是为了避免频繁的创建和销毁对象而影响系统性能，其实现了对象的共享。

- 节省内存空间：常量池中所有相同的字符串常量被合并，只占用一个空间。
- 节省运行时间：比较字符串时，== 比 equals () 快。对于两个引用变量，只用 == 判断引用是否相等，就可以判断实际值是否相等。

### 变化
Java8后，JVM移除了方法区并使用 metaspace（元数据空间）作为替代实现。metaspace 占用系统内存，只要不碰触到系统内存上限，方法区会有足够的内存空间。

## 堆区(Heap)

堆内存是运行时数据区中非常重要的结构，实例对象会存放于堆内存中。绝大多数的垃圾回收都发生在堆内存中，因此对于 JVM 来说，堆内存占据着十分重要的且不可替代的位置。
![在这里插入图片描述](https://img-blog.csdnimg.cn/b1b6e103091a43cf8f71e50ca8d813b9.png)
### 内存划分
- 堆内存从结构上来说分为年轻代（YoungGen）和老年代（OldGen）两部分；
- 年轻代（YoungGen）又可以分为生成区（Eden）和幸存者区（Survivor）两部分；
- 幸存者区（Survivor）又可细分为 S0区（from space）和 S1区 (to space)两部分。
- 年轻代中容量划分默认比例是 8:1:1；
### 为什么要分代
将堆内存从概念层面进行模块划分，总体分为两大部分，年轻代和老年代。从物理层面将堆内存进行内存容量划分，一部分分给年轻代，一部分分给老年代。这就是我们所说的分代。
### 分代的意义：
分代主要是为了易于堆内存分类管理，易于垃圾回收。

- 易于管理：对于堆空间的分代也是如此，比如新创建的对象会进入年轻代（YoungGen）的生成区（Eden），生命周期未结束的且可达的对象，在经历多次垃圾回收之后，会存放入老年代（OldGen），这就是分类管理；

- 易于垃圾回收：将对象根据存活概率进行分类，对存活时间长的对象，放到固定区，从而减少扫描垃圾时间及 GC 频率。针对分类进行不同的垃圾回收算法，对算法扬长避短。
### 创建对象
![在这里插入图片描述](https://img-blog.csdnimg.cn/cf38fb89883543f0bca582403125828c.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)

- 新生成的对象首先放到年轻代 Eden 区，当 Eden 空间满了，触发 Minor GC，存活下来的对象移动到Survivor0 区，Survivor0 区满后触发执行 Minor GC，Survivor0 区存活对象移动到 Suvivor1 区，这样保证了一段时间内总有一个 survivor 区为空。经过多次 Minor GC 仍然存活的对象移动到老年代；

- 在一次新生代垃圾回收后，如果对象还存活，则会进入S0或者S1,并且对象的
年龄还会加1，当它的年龄增加到一定程度(默认为15岁)，就会被晋升到老年代中。

- 老年代存储长期存活的对象，GC 期间会停止所有线程等待 GC 完成，所以对响应要求高的应用尽量减少发生 Major GC，避免响应超时。

