---
layout: post
title: SpringBoot - Mybatis自动分页之PageHelper
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
#### 1.使用方式
**1）ServiceImpl层代码**  
①必须有**PageHelper.startPage()**，且其必须在Mybatis的mapper查询操作之前。  
②PageHelper会为PageHelper.startPage()之后的**第一个**Mybatis**查询**进行分页。  
```java
import com.github.pagehelper.PageHelper;
import com.github.pagehelper.PageInfo;

@Override
public PageInfo<User> pageQuery(int pageNumber, int pageSize) {
    PageHelper.startPage(pageNumber, pageSize);
    List<User> users = userMapper.findAll(UserColumn.NAME);    //startPage()后第一个Mybatis查询操作
    return new PageInfo<>(users);
}
```
**2）mapper.xml**  
mapper.xml中为普通的查询语句，不需要"limit"关键字和其他特殊配置。
```xml
<select id="findAll" resultType="anchor.mybatis.entity.User">
    select * from users order by ${_parameter} asc
</select>
```
#### 2.格式转换
有时从数据库查出的entity需要做类型转换。
```java
@Override
public PageInfo<UserDTO> pageQuery(int pageNumber, int pageSize) {
    PageHelper.startPage(pageNumber, pageSize);
    List<User> users = userMapper.findAll(UserColumn.NAME);
    PageInfo pageInfo = new PageInfo<>(users);      //先用查出的源数据初始化pageInfo，注意不要指定PageInfo<T>中T
    List<UserDTO> collect = pageInfo.getList().stream().map(UserDTO::new).collect(Collectors.toList());    //数据格式转换
    pageInfo.setList(collect);    //用转换后的数据覆盖原有的List
    return pageInfo;
}
```
## 三、问题
**1.关于多个Mybatis查询操作。**  
①如下代码，PageHelper只对users进行了分页操作，未对details进行分页，因此最终的pageInfo并未实现分页效果。  
②PageHelper只对**PageHelper.startPage()后第一个**Mybatis查询操作进行分页。  
```java
PageHelper.startPage(1, 2);
List<User> users = userMapper.findAll(UserColumn.NAME);
List<UserDetail> details = detailMapper.findAll(UserDetailColumn.USER_ID);
PageInfo pageInfo = new PageInfo<>(details);
```
**2.PageHelper最终会帮我们执行count()和limit操作。**  
![]({{ "/assets/img/20200503/20200503001.jpg"}})
**3.PageHelper不支持嵌套结果查询。**  
由于嵌套结果方式会导致结果集被折叠，因此分页查询的结果在折叠后总数会减少，所以无法保证分页结果数量正确。  
**4.分页插件不支持带有for update语句的分页**  
对于带有for update的sql，会抛出运行时异常，对于这样的sql建议手动分页，毕竟这样的sql需要重视。
