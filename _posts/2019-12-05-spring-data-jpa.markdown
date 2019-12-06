---
layout: post
title: Java-Spring Boot集成Spring Data JPA
tags:
- Java 
- Study
categories: Java
description: Spring Data JPA使用简介
---  
**Spring Data JPA使用实操步骤**

<!-- more -->
### 一、简介  
　　Spring Data JPA是 Spring 基于 ORM 框架、JPA 规范封装的一套 JPA 应用框架，
可使开发者用极简的代码即可实现对数据库的访问和操作。它提供了包括增删改查等在内的常用功能，且易于扩展！
（[官方文档](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#reference)、[GitHub地址](https://github.com/spring-projects/spring-data-jpa)）  
### 二、使用  
#### 1.导入包
在pom.xml文件中引入下述依赖。
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <version>2.2.0.RELEASE</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.15</version>
</dependency>
```
#### 2.基本配置
在**resources**文件夹下创建`application.yml`文件并进行下述配置。  
`注`：application.yml与application.properties作用相同，但前者更加简洁、层次分明、更便于阅读，因此使用application.yml。  
```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/springboot-demo?serverTimezone=Shanghai&useUnicode=true&characterEncoding=utf-8&useSSL=true
    username: root
    password: anchor#123
    driver-class-name: com.mysql.cj.jdbc.Driver
  jpa:
    database-platform: org.hibernate.dialect.MySQL5InnoDBDialect
    hibernate:
      ddl-auto: update
    show-sql: true
```
`其中`：  
　1）"**url: jdbc:mysql://`localhost:3306`/`dataBaseName`?serverTimezone=Shanghai&useUnicode=true&characterEncoding=utf-8&useSSL=true**"     
　　 "localhost:3306"为数据库IP及端口（端口一般为3306），"dataBaseName"为要连接的数据库名称，其余不做修改；  
　2）"**username**"和"**password**"为数据库账号和密码（若忘记可[重置](https://blog.csdn.net/hero_hope/article/details/82868046)）；    
　3）"**driver-class-name**"：[说明](https://blog.csdn.net/superdangbo/article/details/78732700)；  
　4）"**database-platform**"：数据库引擎（[介绍](https://blog.csdn.net/ls5718/article/details/52248040)）；  
　5）"**ddl-auto**"：[属性介绍](https://blog.csdn.net/fengyuhan123/article/details/80264795)；  
　6）"**show-sql**"：是否在日志中打印真实操作的sql语句；   
#### 3.项目结构
　![]({{ "/assets/img/springJPA1.png"}})  
`四类约定文件`：  
　1)**model**：存放数据库表对应的Java类；  
　2)**repository**：每个Repository对应一张表的CRUD，java类与数据库交互的媒介；  
　3)**service**：每个service对应一张表的相关业务操作，将repository查到的数据进一步进行处理；  
　4)**controller**：与外界（请求）交互的门户，接收外界请求并调用service层的方法处理该请求；  
　创建类时请遵循上述约定将对应的类放在对应的package下！  
### 三.细解
　　　　　　　![]({{ "/assets/img/springJPA2.png"}})![]({{ "/assets/img/springJPA3.png"}})  
　　　　　　　　　　　　　　(users表)　　　　　　　　　　　　　　　　　　　　　(detail表)
#### 1.Model
创建实体类时注意**类名**要与数据库中表名相同，类中各**属性名**与表中字段名相同。（符合驼峰也可匹配）  
```java
@Entity                 /*用于标注该类对应数据库的表*/                            @Entity 
public class Users {                                                             public class Detail {
    @Id                 /*用于标注id为Users表中的主键*/                                @Id
    @GeneratedValue     /*用于标注主键的生成策略*/                                     @GeneratedValue
    private long id;                                                                  private long id;
    private String name;                                                              private String position;
    private int age;                                                                  private String phoneNumber;   //驼峰
    private String description;                                                       private String education;
}     /*省略getter、setter方法*/                                                  }       //省略getter、setter方法        
```
**@GeneratedValue**：  
　



