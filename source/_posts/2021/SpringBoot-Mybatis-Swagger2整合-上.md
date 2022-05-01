---
title: SpringBoot+Mybatis+Swagger2整合(上)
top: false
cover: false
toc: true
mathjax: false
abbrlink: 14393
date: 2021-02-01 22:34:03
author:
img:
coverImg:
password:
summary:
categories:
  - 项目
tags:
  - Spring Boot
  - MyBatis
---

## 摘要
* Spring家族框架已经统治Java后端开发很多年了，很多大型项目都是用SSM(Spring+SpringMVC+Mybatis)作为后端框架。而且在可预知的未来，Spring没有被替代的可能。
* 但是SSM项目配置起来非常繁琐，需要写大量的配置文件，为了简化开发的步骤，防止把时间浪费在没用必要的配置上。SpringBoot应运而生。
* Spring Boot 为简化 Spring 应用开发而生，Spring Boot 中的 Boot 一词，即为快速启动的意思。Spring Boot 可以在零配置情况下一键启动，简洁而优雅。

#### SpringBoot
为了让 Spring 开发者痛快到底，Spring 团队做了以下设计：

* 简化依赖，提供整合的依赖项，告别逐一添加依赖项的烦恼；
* 简化配置，提供约定俗成的默认配置，告别编写各种配置的繁琐；
* 简化部署，内置 servlet 容器，开发时一键即运行。可打包为 jar 文件，部署时一行命令即启动；
* 简化监控，提供简单方便的运行监控方式。

基于以上设计目的，Spring 团队推出了 Spring Boot 。

#### Mybatis
* MyBatis 是企业级应用数据持久层框架，他与 Hibernate 和 JPA都是
 ORM 对象 - 关系映射框架，使用 Hibernate ，开发者可以不考虑 SQL 语句的编写与执行，直接操作对象即可。
* 但是MyBatis 是需要手工编写 SQL 语句的。应用越复杂对SQL的性能要求就跟高，能够手工编写SQL语句的Mybatis就展现出灵活性。MyBatis 是更加简单，更容易上手的框架，但是功能也是相对简陋点。而且在国内，Mybatis用的比较多。

#### Swagger2
* 我们做开发的时候，经常需要对一些功能进行测试，一个一个的测试非常麻烦。

* Swagger2 可以识别控制器中的方法，然后自动生成可视化的测试界面。后端开发人员编写完 Spring Boot 后端接口后，直接可视化测试就行了。不用编写测试类和测试方法，不用联系前端开发确认接口是否正常。

* 现在非常流行前后端分离开发，因此前后端开发人员的交流就成了问题，接口文档应运而生

* 如果给控制器方法添加注解，还能自动生成在线 API 文档，方便前端开发人员使用和交流。

#### 整合springboot+mybatis+swagger2
虽然springboot简化了spring的配置，但是整合这些组件仍然需要不少的时间。这篇教程就演示一下如何整合这三个组件。
## 整合SpringBoot+Mybatis
#### 创建数据库和表
创建一个名为demo的数据库，并创建一个名为“tb_user”的表用于测试
```javascript
CREATE TABLE `tb_user` (
  `user_id` int(2) NOT NULL AUTO_INCREMENT,
  `user_name` varchar(200) DEFAULT NULL,
  `age` int(2) DEFAULT NULL,
  PRIMARY KEY (`user_id`) USING BTREE,
  UNIQUE KEY `area_name_UNIQUE` (`user_name`)
) ENGINE=InnoDB AUTO_INCREMENT=8 DEFAULT CHARSET=utf8;
```
#### 创建项目
 * 创建一个springboot项目，版本选择2.4.2。添加web需要的依赖
 
```javascript
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
```


* 在pom.xml下添加MySQL和mybatis的依赖
 
```javascript
	 <!--mysql数据库-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>

        <!--mybatis相关的依赖-->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>1.3.1</version>
        </dependency>
```

#### 配置application.properties文件
* 这个文件是springboot项目的配置文件，我们在里面添加一些mysql的连接信息，和mybatis配置文件的位置

```javascript
# 配置数据库驱动 ，新版本需要加上cj
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 配置数据库url
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/demo?useUnicode=true&characterEncoding=utf-8&serverTimezone=UTC
# 配置数据库用户名
spring.datasource.username=root
# 配置数据库密码
spring.datasource.password=root


# 指定MyBatis配置文件位置
mybatis.mapper-locations=classpath:mapper/*.xml
# 开启驼峰命名转换 user_name --> userName
mybatis.configuration.map-underscore-to-camel-case=true
```
#### 编写entity文件
* 在项目目录下建立一个名为entity的package，新建一个User的class文件,这是一个实体类，用于存放数据库访问对象

```javascript
package com.example.demo.entity;

public class User {

    //主键，唯一识别Id
    private Integer userId;

    //姓名
    private String userName;

    //年龄
    private Integer age;

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    public String getUserName() {
        return userName;
    }

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}

```

#### 编写dao文件
* 在项目目录下新建一个package，命名为dao。在dao目录下新建一个UserDao的class文件

```javascript
package com.example.demo.dao;

import com.example.demo.entity.User;
import org.apache.ibatis.annotations.Mapper;
import org.springframework.stereotype.Repository;

import java.util.List;

@Mapper       //标识这个类是一个数据访问层的bean，由spring容器管理
@Repository   //将这个mapper的bean注册到spring容器，不加也行
public interface UserDao {

    /**
     * 查询所有用户
     * @return
     */
    List<User> queryUser();

    /**
     * 根据用户Id查询用户
     * @param userId
     * @return
     */
    User queryUserById(int userId);

    /**
     * 增加用户
     * @param user
     * @return
     */
    int insertUser(User user);

    /**
     * 修改用户信息
     * @param user
     * @return
     */
    int updateUser(User user);

    /**
     * 删除用户
     * @param userId
     * @return
     */
    int deleteUser(int userId);

}

```

* @Mapper       //标识这个类是一个数据访问层的bean，由spring容器管理
* @Repository   //将这个mapper的bean注册到spring容器，不加也行。不加的话，编译器会报错，但不影响程序的运行。强迫症的同学可以在Idea修改一下代码审查的级别
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210201210209291.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)

#### 编写service文件
* UserService的接口interface文件
```javascript
package com.example.demo.service;

import com.example.demo.entity.User;

import java.util.List;


public interface UserService {

    /**
     *
     * 获取用户列表
     * @return
     */
    List<User> getUserList();

    /**
     * 根据用户Id获取用户信息
     * @param userId
     * @return
     */
    User getUserById(int userId);

    /**
     * 增加用户信息
     * @param user
     * @return
     */
    boolean addUser(User user);

    /**
     * 修改用户信息
     * @param user
     * @return
     */
    boolean modifyUser(User user);

    /**
     * 删除用户信息
     * @param userId
     * @return
     */
    boolean deleteUser(int userId);

}

```

* UserServiceImpl的实现类

```javascript
package com.example.demo.service.Impl;

import com.example.demo.dao.UserDao;
import com.example.demo.entity.User;
import com.example.demo.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service  //标识这个bean是service层的，并交给spring容器管理
public class UserServiceImpl implements UserService {

    @Autowired
    private UserDao userDao;


    @Override
    public List<User> getUserList() {
        return userDao.queryUser();
    }

    @Override
    public User getUserById(int userId) {
        return userDao.queryUserById(userId);
    }

    @Transactional
    @Override
    public boolean addUser(User user) {
        if (user.getUserId() != null && !"".equals(user.getUserId())) {
            try {
                int effectedNum = userDao.insertUser(user);
                if (effectedNum > 0) {
                    return true;
                } else {
                    throw new RuntimeException("添加用户失败");
                }
            } catch (Exception e) {
                throw new RuntimeException("添加用户失败" + e.getMessage());
            }
        } else {
            throw new RuntimeException(("用户信息不能为空"));
        }

    }


    @Override
    public boolean modifyUser(User user) {
        if (user.getUserId() != null && (user.getUserId()) > 0) {
            try {
                int effectedNum = userDao.updateUser(user);
                if (effectedNum > 0) {
                    return true;
                } else {
                    throw new RuntimeException("更新用户信息失败");
                }
            } catch (Exception e) {
                throw new RuntimeException("更新用户信息失败" + e.getMessage());
            }
        } else {
            throw new RuntimeException(("用户信息不能为空"));
        }
    }

    @Override
    public boolean deleteUser(int userId) {
        if (userId > 0) {
            try {
                int effectedNum = userDao.deleteUser(userId);
                if (effectedNum > 0) {
                    return true;
                } else {
                    throw new RuntimeException("删除用户信息失败");
                }
            } catch (Exception e) {
                throw new RuntimeException("删除用户信息失败" + e.getMessage());
            }
        } else {
            throw new RuntimeException(("用户Id不能为空"));
        }
    }
}

```
* @Service  //标识这个bean是service层的，并交给spring容器管理

#### 编写mapper文件
* 在resources目录下新建一个目录名为mapper，在mapper目录下新建UserDao.xml文件

```javascript
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.demo.dao.UserDao">

    <select id="queryUser" resultType="com.example.demo.entity.User">
        SELECT user_id, user_name, age
        FROM tb_user
    </select>

    <select id="queryUserById" resultType="com.example.demo.entity.User">
        SELECT user_id, user_name, age
        FROM tb_user
        WHERE
            user_id = #{userId}
    </select>

    <insert id="insertUser" useGeneratedKeys="true" keyProperty="userId"
            keyColumn="userId" parameterType="com.example.demo.entity.User">
        INSERT INTO
            tb_user(user_name, age)
        VALUES
            (#{userName},#{age})
    </insert>

    <update id="updateUser" parameterType="com.example.demo.entity.User">
        update tb_user
        <set>
            <if test="userName != null">user_name=#{userName},</if>
            <if test="age != null">age=#{age}</if>
        </set>
        where
            user_id = #{userId}
    </update>

    <delete id="deleteUser">
        DELETE FROM
            tb_user
        WHERE
            user_id = #{userId}
    </delete>

</mapper>

```
    <mapper namespace="com.example.demo.dao.UserDao"> 
    //添加UserDao的命名空间，引入UserDao
* 添加增删改查等5个方法的SQL语句


#### 编写controller文件
在项目目录下新建一个名为controller的package，在包内新建一个UserController的class文件，用于web的控制器

```javascript
package com.example.demo.controller;

import com.example.demo.entity.User;
import com.example.demo.service.UserService;
import com.fasterxml.jackson.core.JsonParseException;
import com.fasterxml.jackson.databind.JsonMappingException;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

import java.io.IOException;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

@RestController
@Api(tags = "商品API") // 类文档显示内容
@RequestMapping("/admin")   //地址映射的注解
public class UserController {

    @Autowired
    private UserService userService;

    /**
     * 获取用户列表
     * @return
     */
    @ApiOperation(value = "获取用户列表信息") // 接口文档显示内容
    @RequestMapping(value = "listuser",method = RequestMethod.GET)
    private Map<String,Object> listUser(){
        Map<String,Object> modelMap = new HashMap<String,Object>();
        List<User> list = userService.getUserList();
        modelMap.put("userList",list);
        return modelMap;
    }

    /**
     * 通过用户Id获取用户信息
     *
     * @return
     */
    @ApiOperation(value = "通过用户Id获取用户信息") // 接口文档显示内容
    @RequestMapping(value = "/getuserbyid", method = RequestMethod.GET)
    private Map<String, Object> getUserById(Integer userId) {
        Map<String, Object> modelMap = new HashMap<String, Object>();
        // 获取用户信息
        User user = userService.getUserById(userId);
        modelMap.put("user", user);
        return modelMap;
    }


    /**
     * 添加用户信息
     *
     * @param userStr
     * @param request
     * @return
     * @throws IOException
     * @throws JsonMappingException
     * @throws JsonParseException
     */
    @ApiOperation(value = "添加用户信息") // 接口文档显示内容
    @RequestMapping(value = "/adduser", method = RequestMethod.POST)
    private Map<String,Object> addUser(@RequestBody User user)
            throws JsonParseException, JsonMappingException, IOException {
        Map<String, Object> modelMap = new HashMap<String, Object>();
        modelMap.put("success", userService.addUser(user));
        return modelMap;
    }


    /**
     * 修改用户信息
     *
     * @param userStr
     * @param request
     * @return
     * @throws IOException
     * @throws JsonMappingException
     * @throws JsonParseException
     */
    @ApiOperation(value = "修改用户信息") // 接口文档显示内容
    @RequestMapping(value = "/modifyuser", method = RequestMethod.POST)
    private Map<String, Object> modifyUser(@RequestBody User user)
            throws JsonParseException, JsonMappingException, IOException {
        Map<String, Object> modelMap = new HashMap<String, Object>();
        modelMap.put("success", userService.modifyUser(user));
        return modelMap;
    }


    /**
     * 删除用户信息
     * @param userId
     * @return
     */
    @ApiOperation(value = "删除用户信息") // 接口文档显示内容
    @RequestMapping(value = "/removeuser", method = RequestMethod.GET)
    private Map<String, Object> removeUser(Integer userId) {
        Map<String, Object> modelMap = new HashMap<String, Object>();
        modelMap.put("success", userService.deleteUser(userId));
        return modelMap;
    }
}

```

#### 编写handler文件，用于统一异常处理
* 在项目目录下新建一个名为handler的package，在包内新建一个GlobalExceptionHandler的class文件，用于统一异常处理

```javascript
package com.example.demo.handler;

import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import java.util.HashMap;
import java.util.Map;

@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(value = Exception.class)
    @ResponseBody
    public Map<String, Object> exceptionHandler(HttpServletRequest req, Exception e) throws Exception {
        Map<String, Object> modelMap = new HashMap<String, Object>();
        modelMap.put("success", false);
        modelMap.put("errMsg", e.getMessage());
        return modelMap;
    }
}

```
#### 总结
* 项目到这里基本就完成了springboot+muybatis的整合，实现对数据库的增删改查等接口的功能，下一篇将为大家描述整合Swagger2组件进行自动化测试和生成API文档。
* [SpringBoot+Mybatis+Swagger2整合(下)](https://blog.csdn.net/upstream480/article/details/113575440)
- [Github项目源码](https://github.com/stream1080/springboot-mybatis-swagger2)
- [Gitee项目源码](https://gitee.com/stream1080/springboot-mybatis-swagger2)
