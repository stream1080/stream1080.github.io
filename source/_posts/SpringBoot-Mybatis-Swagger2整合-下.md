---
title: SpringBoot+Mybatis+Swagger2整合(下)
top: false
cover: false
toc: true
mathjax: false
abbrlink: 43064
date: 2021-02-02 22:34:11
author:
img:
coverImg:
password:
summary:
categories:
  - 项目
tags:
  - Spring Boot
  - Mybatis
---
## 前言
#### 书接上回
上一篇文章我们讲了 SpringBoot整合Mybatis，没看的小伙伴可以点击链接看一下
- [SpringBoot+Mybatis+Swagger2整合(上)](https://blog.csdn.net/upstream480/article/details/113528828)

这一篇我们接着讲整合Swagger2实现自动化测试和API接口文档

#### Swagger2介绍
- 我们做开发的时候，经常需要对一些功能进行测试，一个一个的测试非常麻烦。

- Swagger2 可以识别控制器中的方法，然后自动生成可视化的测试界面。后端开-发人员编写完 Spring Boot 后端接口后，直接可视化测试就行了。不需要编写测试类和测试方法，也不需要与前端开发确认接口是否正常。

- 现在非常流行前后端分离开发，因此前后端开发人员的交流就成了问题，接口文档应运而生

- 如果给控制器方法添加注解，还能自动生成在线 API 文档，方便前端开发人员使用和交流。
## 代码配置
#### 在pom.xml中引入Swagger2的依赖
- springfox-swagger2 引入 swagger2相关的依赖包
- springfox-swagger-ui 引入 swagger-ui相关的依赖包

```xml
        <!-- 引入swagger2相关依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>

        <!-- 引入添加swagger-ui相关依赖 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
```
#### 配置Swagger相关的功能
- 在项目目录下新建一个package，命名为config
- 在config中新建一个命名为Swagger2Config的class文件
- 添加注解@Configuration  告诉Spring容器，这个类是一个配置类
- 添加@EnableSwagger2  启用Swagger2的各项功能
- 通过 @Bean 标注的方法将对 Swagger2 功能的设置放入容器。
```java
package com.example.demo.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import springfox.documentation.builders.ApiInfoBuilder;
import springfox.documentation.builders.PathSelectors;
import springfox.documentation.builders.RequestHandlerSelectors;
import springfox.documentation.service.ApiInfo;
import springfox.documentation.spi.DocumentationType;
import springfox.documentation.spring.web.plugins.Docket;
import springfox.documentation.swagger2.annotations.EnableSwagger2;

@Configuration // 告诉Spring容器，这个类是一个配置类
@EnableSwagger2 // 启用Swagger2功能
public class Swagger2Config {

    /**
     * 配置Swagger2相关的bean
     */
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com"))// com包下所有API都交给Swagger2管理
                .paths(PathSelectors.any()).build();
    }

    /**
     * API文档地址：http://127.0.0.1:8080/swagger-ui.html#/
     *
     * 此处主要是API文档页面显示信息
     */
    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("项目API") // 标题
                .description("整个项目的各个API") // 描述
                .termsOfServiceUrl("http://www.baidu.com") // 服务网址，一般写公司地址
                .version("1.0") // 版本
                .build();
    }
}
```

#### 生成在线API文档
- 在各个controller中添加@Api注解
- @Api(tags = "用户管理API") 类文档显示的接口名称

```java
@RestController
@Api(tags = "用户管理API") // 类文档显示内容
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
```

#### 启动项目，测试swagger2的相关功能
- 启动项目
- 在浏览器上访问  http://127.0.0.1:8080/swagger-ui.html#/
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202224223153.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)

- API中使用的参数模型
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202224343716.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
- 至此，我们的在线API文档就完成了，前端开发人员就可以根据这个文档使用各个API接口进行开发了，接下来我们进入测试环节
## 测试
#### 测试添加用户API
- 点击第一个添加用户API
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202224703156.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
- 点击Try it out进行测试
- 输入测试参数，点击Execute执行测试方法
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202224851388.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
- 返回结果，success：true 说明添加用户成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202225022490.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
#### 测试根据Id查询用户信息API
- 点击第二个方法，根据用户Id获取用户信息
- 输入用户Id，也就是刚刚我们添加的哪一个，点击Execute执行
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202225219789.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
- 返回结果为刚刚我们添加的测试用户的信息，测试成功
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210202225450138.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3Vwc3RyZWFtNDgw,size_16,color_FFFFFF,t_70)
- 依次测试其他的各个API
## 总结
- 我们用两篇文章讲解了SpringBoot+Mybatis+Swagger2整合，实现springboot与数据库的连接和自动生成API文档，并且进行了测试各个API的功能。
- 这个项目可以作为一个模板，以后进行其他的项目的时候，就可以直接用这个模板进行修改或者扩充，省去配置mybatis和swagger2的时间。大家下载下来直接在这个基础上进行修改就可以了，我还在上面添加了Junit的测试模块，方便大家使用。
- [Github项目源码](https://github.com/stream1080/springboot-mybatis-swagger2)
- [Gitee项目源码](https://gitee.com/stream1080/springboot-mybatis-swagger2)

