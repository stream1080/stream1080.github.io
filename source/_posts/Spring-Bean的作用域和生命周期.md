---
title: Spring Bean的作用域和生命周期
cover: false
top: false
date: 2021-10-02 20:51:21
summary:
categories:
  - 框架
tags:
  - Spring
---

## Spring Bean的作用域
Spring 为Bean定义了5种作用域，分别为：
- singleton (单例)
- prototype (原型)
- request
- session 
- global session

### singleton-单例模式（多线程下不安全）
- Spring loC容器中只会存在一 个共享的Bean实例，无论有多少个Bean引用它，始终指向同一个对象。
- 该模式在多线程下是不安全的。Singleton 作用域是Spring中的缺省作用域，也可以显示的将Bean定义为singleton模式

```xml
<bean id="userDao" class= "com.ioc.UserDaolmpl" scope= "singleton"/>
```
### prototype-原型模式（每次使用时创建）
-  每次通过Spring容器获取prototype定义的bean时，容器都将创建一个新的Bean实例，每个Bean实例都有自己的属性和状态，而singleton全局只有一个对象。
- 一般来说,对有状态的bean使用prototype作用域，而对无状态的bean使用singleton作用域。

### Request （一次 request 创建一个实例）
- 在一次Http请求中，容器会返回该Bean的同一实例。 而对不同的Http请求则会产生新的Bean,而且该bean仅在当前Http Request 内有效
- 当前Http请求结束，该bean实例也将会被销毁。

```xml
<bean id= "loginAction" class="com.userLogin" scope= "request'"/>
```

### session
- 在一次Http Session中，容器会返回该Bean的同一实例。
- 而对不同的Session请求则会创建新的实例，该bean实例仅在当前Session 内有效。
- 同Http请求相同，每一次session请求创建新的实例，而不同的实例之间不共享属性，且实例仅在自己的session请求内有效，请求结束，则实例将被销毁。

```xml
<bean id=" userPreference" class=" com.ioc.UserPreference" scope= "session"/>
```
### global Session
在一个全局的Http Session 中，容器会返回该Bean的同一个实例，仅在使用portlet context时有效。

## Spring Bean的生命周期
1. **实例化**一个Bean,也就是我们常说的new。

2. **IOC依赖注入**，按照 Spring上下文对实例化的Bean进行配置，也就是IOC依赖注入。

3. **setBeanName实现**，如果这个Bean已经实现了BeanNameAware接口，会调用它实现的setBeanName(String)方法，此处传递的就是Spring配置文件中Bean的id值

4. **BeanFactoryAware实现**，如果这个Bean已经实现了BeanFactoryAware 接口，会调用它实现的setBeanFactory，setBeanFactory(BeanFactory)传递的是Spring工厂自身(可以用这个方式来获取其它Bean，只需在Spring配置文件中配置一个普通的Bean就可以)。


5. **ApplicationContextAware实现**，如果这个 Bean已经实现了ApplicationContextAware接口，会调用
setApplicationContext(ApplicationContext)方法,传入Spring.上下文(同样这个方式也可以实现步骤4的内容，但比4更好,因为ApplicationContext是BeanFactory的子接口，有更多的实现方法)

6. **postProcessBeforelnitialization接口实现初始化预处理**，如果这个Bean关联了BeanPostProcessor接口，将会调用postProcessBeforelnitialization(Object obj, String s)方法，BeanPostProcessor 经常被用作是Bean内容的更改，并且由于这个是在Bean初始化结束时调用那个的方法，也可以被应用于内存或缓存技术。

7. **init-method**，如果 Bean在Spring配置文件中配置了init-method属性会自动调用其配置的初始化方法。

8. **postProcessAfterlnitialization**，如果这个 Bean关联了BeanPostProcessor 接口，将会调用
postProcessAfterlnitialization(Object obj, String s)方法。
注:以上工作完成以后就可以应用这个Bean了,那这个Bean是一个Singleton的，所以一
般情况下我们调用同一个id的Bean会是在内容地址相同的实例，当然在Spring配置文件中
也可以配置非Singleton。

9. **Destroy过期自动清理阶段**，当Bean不再需要时，会经过清理阶段,如果Bean实现了DisposableBean这个接口，会调用那个其实现的destroy()方法;

10. **destroy-method自配置清理**，最后，如果这个Bean的Spring配置中配置了destroy-method属性,会自动调用其配置的销毁方法。

