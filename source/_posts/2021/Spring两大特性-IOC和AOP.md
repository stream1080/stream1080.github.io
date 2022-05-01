---
title: 'Spring两大特性:IOC和AOP'
top: false
cover: false
toc: true
mathjax: false
categories:
  - 框架
tags:
  - Spring
  - IOC
  - AOP
abbrlink: 9268
date: 2021-07-20 20:41:26
author:
img:
coverImg:
password:
summary:
---

# IOC（Inverse of Control）控制反转
说起控制反转，首先要了解一下软件设计的一个重要思想：依赖倒置
## 依赖倒置原则
假如现在要造一台手机，先设计处理器，然后根据处理器的规格设计主板，接着根据主板设计机身，最后根据机身设计好整个手机。
“依赖”关系：手机依赖机身，机身依赖主板，主板依赖处理器。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210720150809684.png)
这种设计方案下，一旦有某个零部件需要调整规格，
比如处理器需要升级，那么主板是按照处理器设计的，就要修改主板规格，修改主板规格后，机身是按照主板设计的，所以机身也需要修改，那么整个设计方案都需要改动，非常麻烦；
如果将依赖倒置，就会是这样的：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210720151324299.png)
我们先设计手机的轮廓，然后根据手机的轮廓来设计机身，根据机身来设计主板，最后根据主板来设计处理器。
依赖关系就反转过来了：处理器依赖主板， 主板依赖机身， 机身依赖手机。

## 依赖注入
上层决定需要做什么，下层去实现这样的需求，但是上层并不用管下层是怎么实现的。这样就不会出现前面的“牵一发动全身”的情况。依赖注入（Dependency Injection），就是来实现控制反转的。所谓依赖注入，就是把底层类作为参数传入上层类，实现上层类对下层类的“控制”。

把有依赖关系的类放到容器中，解析出这些类的实例，就是依赖注入。目的是实现类的解耦。

## IOC容器
- IOC容器可以自动对你的类进行初始化，你只需要维护一个Configuration，而不用每次初始化一辆车都要亲手去写那一大段初始化的代码。
- 我们在创建实例的时候不需要了解其中的细节。

- IoC容器实际上就是个Map (key, value) ， Map中存放的是各种对象。
- 将对象之间的相互依赖关系交给IoC 容器来管理，并由IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。
- IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。
- 在实际的开发过程中一个Service类可能有几百甚至上千个类作为它的底层，假如我们需要实例化这个Service,你可能要每次都要搞清这个Service所有底层类的构造函数，这可能会把人逼疯。如果利用IoC的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。
- 我们需要的控制反转（IoC），就是上层控制下层，而不是下层控制着上层

- IoC是一种设计思想， 就是将原本在程序中手动创建对象的控制权，交由Spring框架中的IOC容器来管理 

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021072015195877.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
IOC的思想最核心的地方在于，资源不由使用资源的双方管理，而由不使用资源的第三方管理，这可以带来很多好处。
- 资源集中管理，实现资源的可配置和易管理
- 降低了使用资源双方耦合度
## IOC初始化过程
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210720153933500.png)

# AOP(Aspect-Or iented Programming) 面向切面编程

AOP能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任(例如事务处理、日志管理、权限控制等)封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

Spring AOP就是基于动态代理的，如果要代理的对象，实现了某个接口，那么Spring AOP会使用JDK Proxy,去创建代理对象，而对于没有实现接口的对象，就无法使用JDK Proxy 去进行代理了，这时候Spring AOP会 使用Cglib，这时候Spring AOP会使用Cglib 生成一个被代理 对象的子类来作为代理，

使用AOP之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样大大简化了代码量。我们需要增加新功能时也方便，这样也提高了系统扩展性。日志功能、事务管理等等场景都用到了AOP

# Spring中Bean的作用域
- singleton :唯一bean实例，Spring 中的bean 默认都是单例的
- prototype :每次请求都会创建个新的 bean 实例
- request :每次HTTP请求都会产生一个 新的bean,该bean仅在 当前HTTP request内有效
- session :每一次HTTP请求都会产生一个新的bean, 该bean仅在当前HTTP session内有效
- global -session:全 局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了
- Portlet是 能够生成语义代码(例如: HTML) 片段的小型Java Web插件。它们基于
- portlet容器，可以像servlet- 样处理HTTP请求。但是，与servlet 不同，每个portlet都有
不同的会话
# Spring中Bean的线程安全
单例bean存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程
## 解决方法：
- 在Bean对象中尽量避免定义可变的成员变量
- 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中

