---
layout: post
title: Java-Spring Boot集成Mybatis
tags:
- Java 
- Spring Boot
- Study
categories: Java
description: Spring Boot集成Mybatis操作简介
---  
**Spring Boot集成Mybatis实操步骤**

<!-- more -->
# 一、简介  
　　**以下摘自官网：**  
　　MyBatis 是一款优秀的持久层框架，它支持定制化 SQL、存储过程以及高级映射。MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数
以及获取结果集。MyBatis 可以使用简单的 XML 或注解来配置和映射原生类型、接口和 Java 的 POJO为数据库中的记录。
([官方文档](https://mybatis.org/mybatis-3/zh/index.html))  
　　**以下摘自Anchor：**  
　　一款Java程序猿用于操作关系型数据库的框架，遵循JPA标准，准确的说是JPA的一种具体实现。  
　　当今两大巨头（Hibernate、Mybatis）之一，可以当做JDBC的升级版（Plus、Max、Pro）。    
　　很牛逼、很多人用（俺们团队也在用），至于好不好用请自行判断。  
　　[优缺点](https://blog.csdn.net/u014788653/article/details/68489301)。
# 二、使用  
## 1.导入包
在pom.xml文件中引入下述必要的依赖包（版本可自主选择）。
```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>8.0.18</version>
</dependency>
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>2.0.0</version>
</dependency>
```
## 2.<span id="application">基本配置</span>
在**resources**文件夹下创建`application.yml`文件并进行下述配置。  
`注`：application.yml与application.properties作用相同，但前者更加简洁、层次分明、更便于阅读，因此使用application.yml。  
```yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/dataBaseName?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
    username: root
    password: anchor#123
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis:
  type-aliases-package: anchor.mybatis.entity
  mapper-locations: classpath:mapping/*Mapper.xml

logging:
  level:
    anchor.mybatis.mapper: debug
```
`其中`：  
　1）"**url: jdbc:mysql://`localhost:3306`/`dataBaseName`?serverTimezone=Shanghai&useUnicode=true&characterEncoding=utf-8&useSSL=true**"     
　　 "localhost:3306"为数据库IP及端口（端口一般为3306），"dataBaseName"为要连接的数据库名称，其余不做修改；  
　2）"**username**"和"**password**"为数据库账号和密码（若忘记可[重置](https://blog.csdn.net/hero_hope/article/details/82868046)）；    
　3）"**driver-class-name**"：[说明](https://blog.csdn.net/superdangbo/article/details/78732700)；  
　4）"**type-aliases-package**"：Java实体类所在包的位置；  
　5）"**mapper-locations**"：Mapper.xml所在的位置。"classpath:"这里可以理解为是"resources"目录；  
　6）"**logging.level**"：用于打印mybatis真实执行的sql语句。(包名: 日志级别) 例如 (anchor.mybatis.mapper: debug)；   
## 3.项目结构
　　![]({{ "/assets/img/mybatis1.jpg"}})  
`五类约定文件`：  
　1)**entity**：约定存放Java实体类，这些类可与数据库中表对应也可不对应（由Mapper.xml中的映射关系决定）；  
　2)**<span id="mapper">mapper</span>**：约定存放命名以"Mapper"结尾的Java接口，这些接口是连通Java应用于数据库的桥梁，是Java应用操作数据库的入口；  
　3)**service**：业务操作处理层，对mapper接口查到的数据根据业务需要进行处理；  
　4)**controller**：与外界（请求）交互的门户，接收外界请求并调用service层的方法处理该请求；  
　5)**mapping**：mapping是一个Directory，约定放在resources文件夹下，用来存放命名以"Mapper"为结尾的xml文件，
在此类xml文件中编写SQL语句来实现[mapper接口](#mapper)；  
　**创建文件时请遵循上述约定将对应的文件放在对应的Package或Directory下！**  
# 三、细解
## 1.Mapper.xml  
　　Mybatis根据在[application.yml](#application)中mybatis.mapper-locations的值来定位Mapper.xml文件。  
　　根据约定本文中此值设置为“classpath:mapping/*Mapper.xml”（相当于resources/mapping/*Mapper.xml）。  
　　Mybatis的核心就在于Mapper.xml文件，Mybatis根据此类文件中的内容对数据库进行CURD操作，并将操作结果映射到指定的Java类。  
#### 1）xml的框架
```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="anchor.mybatis.mapper.UserMapper">
  <!-- 具体实现... -->
</mapper>
```
前两行必须有但不必修改，标签mapper中的namespace用来绑定Java接口，需写对应Mapper接口的全路径。  
## 2.Entity
## 3.Mapper
## 4.Service
## 5.Controller