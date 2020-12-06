---
layout: post
title: SpringBoot - Mybatis常用语句收集
tags:
- Java 
- SpringBoot
- Mybatis
categories: Java
description: Mybatis常用语句示例
---  
**Mybatis常用语句示例**

<!-- more -->
## 注
**本文中每个示例将会展示两部分代码：Mapper.java中方法的`声明`，Mapper.xml中方法的`实现`。**   
#### 1.表结构
```hql
create table user(
    id          int unsigned auto_increment primary key comment '用户id',
    name        varchar(18)       not null comment '用户真实姓名',
    age         int unsigned comment '年龄',
    gender      smallint unsigned not null comment '性别，1-男、2-女、3-保密',
    mail        varchar(256) comment '邮箱地址',
    province    varchar(32)       not null comment '用户所在省份',
    create_time timestamp default current_timestamp comment '信息创建时间',
    update_time timestamp default current_timestamp on update current_timestamp comment '信息更新时间'
) comment '用户信息 ';
```
#### 2.实体类  
```java
@Data
@Accessors(chain = true)
@NoArgsConstructor
public class User {
    Long id;
    String name;
    Integer age;
    Integer gender;
    String mail;
    String province;
    LocalDateTime createTime;
    LocalDateTime updateTime;
}
```
#### 3.Mapper定义 
```java
import com.baomidou.mybatisplus.core.mapper.BaseMapper;

public interface UserMapper extends BaseMapper<User> {
}
```
## 一、增
#### 1.新增一条
**BaseMapper**中自带，无需造轮子。  
```java
int insert(T entity);
```
#### 2.新增多条
```java
int batchInsert(List<User> userList);
```
```xml
<insert id="batchInsert" parameterType="java.util.List" useGeneratedKeys="true" keyProperty="id">
    insert into user
    (name, age, gender, mail, province)
    value 
    <foreach collection="list" item="item" index="index" separator=",">
        (#{item.name}, #{item.age}, #{item.gender}, #{item.mail}, #{item.province})
    </foreach>
</insert>
```
## 二、删
#### 1.删除一条  
**BaseMapper**中自带，无需造轮子。  
```java
int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
```
```java
int num = userMapper.delete(new QueryWrapper<User>()
              .lambda()
              .eq(User::getName, "Anchor")
);
```
#### 2.删除多条
这里主要记录关键字`in`的用法，用上面BaseMapper的delete()方法基本可以满足所有删除的需求。  
```java
int deleteByName(List<String> nameList);
```
```xml
<insert id="deleteByName" >
    delete from user
    where name in (
    <foreach collection="list" item="item" separator=",">
        #{item}
    </foreach>)
</insert>
```
## 三、查
#### 1.单一参数
```java
List<User> findLikeName(String name);
```
```xml
<select id="findLikeName" resultMap="BaseResultMap">
    select id, name, age, gender, mail, province, create_time, update_time
    from user
    where name like concat ('%', #{name} ,'%')
    order by name
</select>
```
#### 2.多参数  
```java
import org.apache.ibatis.annotations.Param;

List<User> search(@Param("age") int age, 
                  @Param("gender") int gender, 
                  @Param("data") List<String> provinces);
```
```xml
<select id="findLikeName" resultMap="BaseResultMap">
    select id, name, age, gender, mail, province, create_time, update_time
    from user
    where age = #{age} 
      and gender = #{gender}
      and province in(
      <foreach collection="data" item="item" separator=",">
          #{item}
      </foreach>)
    order by name
</select>
```
## 四、改
## 五、小知识
1.Mysql中 **order by** 的**默认顺序**是**升序(asc)**;  
2.当 name = Anchor 时，**#{name}** 最终替换为 `'Anchor'`，**${name}** 替换为 `Anchor`;