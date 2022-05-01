---
title: Spring Boot是如何实现自动装配的
top: false
cover: false
toc: true
mathjax: false
categories:
  - 框架
tags:
  - Spring Boot
  - 自动装配
abbrlink: 1513
date: 2021-11-21 19:02:32
author:
img:
coverImg:
password:
summary:
---

## 什么是自动装配？

自动装配就是通过注解或者一些简单的配置就能在 Spring Boot 的帮助下实现某块功能。

Spring Boot 定义了一套接口规范，这套规范规定：SpringBoot 在启动时会扫描外部引用 jar 包中的META-INF/spring.factories文件，将文件中配置的类型信息加载到 Spring 容器，并执行类中定义的各种操作。

对于外部 jar 来说，只需要按照 Spring Boot 定义的标准，就能将自己的功能装置进 Spring Boot。

在Spring Boot 中，如果我们需要引入第三方依赖，我们直接引入一个 starter 即可。比如想要在项目中使用 redis 的话，直接在项目中引入对应的 starter 即可。

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
引入 starter 之后，通过少量注解和一些简单的配置就能使用第三方组件提供的功能了。

## 如何实现自动装配
 SpringBoot 的核心注解是 SpringBootApplication。
@SpringBootApplication注解包含
- @Configuration：允许在上下文中注册额外的 bean 或导入其他配置类
- @EnableAutoConfiguration：启用 SpringBoot 的自动配置机制
- @ComponentScan：扫描被@Component (@Service,@Controller)注解的 bean，注解默认会扫描启动类所在的包下所有的类 ，可以自定义不扫描某些 bean

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@ComponentScan
@EnableAutoConfiguration
public @interface SpringBootApplication {

}

@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Configuration 
public @interface SpringBootConfiguration {
}
```


从源码中可以知道，最关键的注解是
- @Import(EnableAutoConfigurationImportSelector.class)，借助EnableAutoConfigurationImportSelector，@EnableAutoConfiguration可以帮助Spring Boot应用将所有符合条件的@Configuration配置都加载到当前SpringBoot创建并使用的IoC容器。
- 同时借助于Spring框架原有的一个工具类：SpringFactoriesLoader，@EnableAutoConfiguration就可以实现智能的自动配置。

- SpringFactoriesLoader中加载配置,SpringFactoriesLoader属于Spring框架私有的一种扩展方案，其主要功能就是从指定的配置文件META-INF/spring.factories加载配置,
- 即根据@EnableAutoConfiguration的完整类名org.springframework.boot.autoconfigure.EnableAutoConfiguration作为查找的Key,获取对应的一组@Configuration类


@EnableAutoConfiguration作用就是从classpath中搜寻所有的META-INF/spring.factories配置文件，并将其中org.springframework.boot.autoconfigure.EnableutoConfiguration对应的配置项通过反射（Java Refletion）实例化为对应的标注了@Configuration的JavaConfig形式的IoC容器配置类，然后汇总为一个并加载到IoC容器。

这些功能配置类要生效的话，会去classpath中找是否有该类的依赖类（也就是pom.xml必须有对应功能的jar包才行）并且配置类里面注入了默认属性值类，功能类可以引用并赋默认值。

生成功能类的原则是自定义优先，没有自定义时才会使用自动装配类。

生效需要的条件：
- spring.factories里面有这个类的配置类（一个配置类可以创建多个围绕该功能的依赖类）
- pom.xml里面需要有对应的jar包


## 案例 - Redis
1、从spring-boot-autoconfigure.jar/META-INF/spring.factories中获取redis的相关配置类全限定名（有120多个的配置类）RedisAutoConfiguration，一般一个功能配置类围绕该功能，负责管理创建多个相关的功能类，比如RedisAutoConfiguration负责：JedisConnectionFactory、RedisTemplate、StringRedisTemplate这3个功能类的创建

![在这里插入图片描述](https://img-blog.csdnimg.cn/38b0ee1058104232a1aaa885ec5bd9f3.png)

2、RedisAutoConfiguration配置类生效的一个条件是在classpath路径下有RedisOperations类存在，因此springboot的自动装配机制会会去classpath下去查找对应的class文件。

```java
@Configuration
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {

}
```

3、pom.xml中有对应的jar包,就能匹配到对应依赖class，

```java
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
```
4、匹配成功，这个功能配置类才会生效，同时会注入默认的属性配置类@EnableConfigurationProperties(RedisProperties.class)
![在这里插入图片描述](https://img-blog.csdnimg.cn/4c06c2d7589f4efea3bb2d4e6fb3e2d0.png)

5、Redis功能配置里面会根据条件生成最终的
JedisConnectionFactory、RedisTemplate,并提供了默认的配置形式
@ConditionalOnMissingBean(name = "redisTemplate")

```java
@Configuration
@ConditionalOnClass({RedisOperations.class})
@EnableConfigurationProperties({RedisProperties.class})
@Import({LettuceConnectionConfiguration.class, JedisConnectionConfiguration.class})
public class RedisAutoConfiguration {
    public RedisAutoConfiguration() {
    }

    @Bean
    @ConditionalOnMissingBean(
        name = {"redisTemplate"}
    )
    public RedisTemplate<Object, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        RedisTemplate<Object, Object> template = new RedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }

    @Bean
    @ConditionalOnMissingBean
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) throws UnknownHostException {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}

```
6、最终创建好的默认装配类，会通过功能配置类里面的 @Bean注解，注入到IOC当中


7、用户使用，当用户在配置文件中自定义时候就会覆盖默认的配置@ConditionalOnMissingBean(name = "redisTemplate")



