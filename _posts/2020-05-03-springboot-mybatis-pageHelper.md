---
layout: post
title: SpringBoot-Mybatis自动分页之PageHelper
tags:
- Java 
- SpringBoot 
- Tips
categories: Java
description: 自动分页插件PageHelper使用
---  
**SpringBoot项目中Mybatis自动分页插件 — PageHelper的使用**

<!-- more -->
## 一、引入
[官方文档](https://pagehelper.github.io/docs/)、[GitHub地址](https://github.com/pagehelper/Mybatis-PageHelper)
```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper-spring-boot-starter</artifactId>
    <version>1.2.13</version>
</dependency>
```
## 二、使用
#### 1.正常使用
**1.ServiceImpl层代码**
```java
@Override
public PageInfo<User> pageQuery(int pageNumber, int pageSize) {
    PageHelper.startPage(pageNumber, pageSize);
    return new PageInfo<>(userMapper.findAll(UserColumn.COLUMN_NAME));
}
```
**2.mapper.xml**
```xml
<select id="findAll" resultType="anchor.mybatis.entity.User">
    select * from users order by ${_parameter} asc
</select>
```
#### 2.格式转换
## 三、引入
## 四、引入