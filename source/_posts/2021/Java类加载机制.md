---
title: Java类加载机制
top: false
cover: false
toc: true
mathjax: false
categories:
  - JVM
tags:
  - 双亲委派
  - 类加载
abbrlink: 48192
date: 2021-08-06 23:15:28
author:
img:
coverImg:
password:
summary:
---

## 类加载过程
Java类加载过程为：加载-链接-初始
链接的过程包括验证，准备，解析
![在这里插入图片描述](https://img-blog.csdnimg.cn/14b229396a0d48d686a01c6f18e1d702.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
#### 加载
把编译后的class字节码文件通过类加载器装载入内存中，并将这些数据转换成方法区中的运行时数据（静态变量、静态代码块、常量池等），在堆中生成一个Class类对象代表这个类（反射原理），作为方法区类数据的访问入口。
- 通过全类名获取定义此类的二进制字节流
- 将字节流所代表的静态存储结构转换为方法区的运行时数据结构
- 在内存中生成一个代表该类的Class 对象，作为方法区这些数据的访问入口

#### 验证
保证加载进来的字节流符合虚拟机规范，不会造成安全错误
- 对文件格式的验证，比如常量中是否有不被支持的常量？文件中是否有不规范的或者附加的其他信息？

- 对元数据的验证，比如该类是否继承了被final修饰的类？类中的字段，方法是否与父类冲突？是否出现了不合理的重载？

- 对字节码的验证，保证程序语义的合理性，比如要保证类型转换的合理性。

- 对符号引用的验证，比如校验符号引用中通过全限定名是否能够找到对应的类？校验符号引用中的访问性（private，public等）是否可被当前类访问？
#### 准备
- 为类变量（不是实例变量）分配内存，并且赋予初值。
- 初值并不是代码中写的初始化的值，而是Java虚拟机根据不同变量类型的默认初始值。
- 比如8种基本类型的初值，默认为0；引用类型的初值则为null；
- 常量的初值即为代码中设置的值，final static num= 456， 那么该阶段num的初值就是456
#### 解析
将常量池内的符号引用替换为直接引用的过程；
在解析阶段，虚拟机会把所有的类名，方法名，字段名这些符号引用替换为具体的内存地址或偏移量，也就是直接引用。

- 符号引用：一个字符串给出了一些能够唯一性识别一个方法，一个变量，一个类的相关信息。
- 直接引用：一个内存地址或者偏移量。比如类方法，类变量的直接引用是指向方法区的指针；
- 而实例方法，实例变量的直接引用则是从实例的头指针开始算起到这个实例变量位置的偏移量


#### 初始化
- 对类变量初始化，是执行类构造器的过程。
- 只对static修饰的变量或语句进行初始化。
- 如果初始化一个类的时候，其父类尚未初始化，则优先初始化其父类。
- 如果同时包含多个静态变量和静态代码块，则按照自上而下的顺序依次执行。
- 虚拟机会保证一个类的<clinit>()方法在多线程环境中被正确加锁和同步。

## 类加载器
![在这里插入图片描述](https://img-blog.csdnimg.cn/c111bacbdc15478db391e81c6be73d5f.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)

#### 启动类加载器（bootstrap class loader）
- 用来加载 Java 的核心库(JAVA_HOME/jre/lib/rt.jar,sun.boot.class.path路径下的内容)，是用原生代码C++来实现的，并不继承自 java.lang.ClassLoader。
- 加载扩展类和应用程序类加载器。并指定他们的父类加载器。


#### 扩展类加载器（extensions class loader）
- 用来加载 Java 的扩展库(JAVA_HOME/jre/ext/*.jar，或java.ext.dirs路径下的内容) 。Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java类。
- 由sun.misc.Launcher$ExtClassLoader实现。

#### 应用程序类加载器（application class loader）
- 它根据 Java 应用的类路径（classpath，java.class.path 路径下的内容）来加载 Java 类
- 一般来说，Java 应用的类都是由它来完成加载的
- 由sun.misc.Launcher$AppClassLoader实现

#### 自定义类加载器
- 开发人员可以通过继承 java.lang.ClassLoader类的方式实现自己的类加载器，满足一些特殊的需求。
## 双亲委派模型
其实这里的双亲很容易让人误解，大家都觉得双亲表示父母的意思，其实这里的双亲是父类的意思，
每一个类都有它自己的类加载器。系统中的ClassLoder 在协同工作的时候会默认使用双亲委派
模型。
#### 原理
- 向上委托：如果一个类加载器收到了类加载请求，它并不会自己先去加载，而是把这个请求委托给父类的加载器去执行，如果父类加载器还存在其父类加载器，则进一步向上委托，依次递归，请求最终将到达顶层的启动类加载器。如果父类加载器可以完成类加载任务，就成功返回；
- 向下委派：倘若父类加载器无法完成此加载任务，子加载器才会尝试自己去加载，一直向下委派加载，这就是双亲委派模式。如果都不能加载，则会报错。
![在这里插入图片描述](https://img-blog.csdnimg.cn/5f3e0d23e0d44ed6aede34fd3c482d2a.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)

- 在类加载的时候，系统会首先判断当前类是否被加载过。
- 已经被加载的类会直接返回，否则才会尝试加载。
- 加载的时候，首先会把该请求委派该父类加载器来处理， 因此所有的请求最终都应该传送到顶层的启动类加载器BootstrapClassLoader 中。
- 当父类加载器无法处理时，才由自己来处理。当父类加载器为null时，会使用启动类加载器BootstrapClassLoader 作为父类加载器。



## 为什么要打破双亲委派

Tomcat作为一个Web容器，会包含各种Web应用程序，为了使各个应用程序不互相干扰，至少需要达到以下要求：
- 部署在同一个Web容器上的两个Web应用程序所使用的Java类库可以实现相互隔离
- 部署在同一个Web容器上的两个Web应用程序所使用的Java类库可以相互共享
- Web容器需要保证自身的安全不受Web应用程序所影响

tomcat 为了实现隔离性，没有遵守双亲委派模型，所以在Tomcat中，类的加载不能使用简单的ClassLoader来加载，而是需要自定义分级的ClassLoader。

#### Tomcat类加载过程
1. 先在本地缓存中查看是否已经加载过该类，如果已经加载过就返回
2. 否则就让系统类加载器(AppClassLoader)尝试加载该类
3. 这一步主要是为了防止一些基础类会被web中的类覆盖，如果加载到即返回
4. 前两步均没加载到目标类，那么web应用的类加载器将自行加载，如果加载到则返回，否则继续下一步。
5. 最后还是加载不到的话，则委托父类加载器去加载。


