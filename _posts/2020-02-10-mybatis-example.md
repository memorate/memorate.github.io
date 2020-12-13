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
**本文中所有代码皆已实操成功。**  
**本文中使用了 Mybatis-Plus 来简化开发。**  
**本文中每个示例将会展示两部分代码：Mapper.java中方法的`声明`，Mapper.xml中方法的`实现`。**   
#### 1.表结构
```hql
create table customer(
    id          bigint unsigned auto_increment primary key comment '顾客id',
    name        varchar(18)       not null comment '顾客真实姓名',
    age         int unsigned comment '年龄',
    gender      smallint unsigned not null comment '性别，1-男、2-女、3-保密',
    mail        varchar(256) comment '邮箱地址',
    province    varchar(32)       not null comment '顾客所在省份',
    create_time timestamp default current_timestamp comment '信息创建时间',
    update_time timestamp default current_timestamp on update current_timestamp comment '信息更新时间'
) comment '顾客信息';
```
#### 2.实体类  
```java
@Data
@Accessors(chain = true)
@NoArgsConstructor
public class Customer {
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

public interface CustomerMapper extends BaseMapper<Customer> {
}
```
## 一、增
#### 1.新增一条
**BaseMapper**中自带，无需造轮子。  
```java
int insert(T entity);
```
`注意：`  
1）插入时若未配置，主键生成策略默认使用**IdType.ASSIGN_ID**(雪花算法实现)，此时数据库中配置的**auto_increment失效**;  
2）默认配置在com.baomidou.mybatisplus.core.config.**GlobalConfig.DbConfig**类中;  
3）可在entity中使用注解来更改主键生成策略(或者在yml中配置全局策略 - 自行搜索);  
```java
import com.baomidou.mybatisplus.annotation.IdType;
import com.baomidou.mybatisplus.annotation.TableId;

public class Customer {
    @TableId(type = IdType.AUTO)
    Long id;
}
```
#### 2.新增多条
```java
int batchInsert(List<Customer> userList);
```
```xml
<insert id="batchInsert" parameterType="java.util.List" useGeneratedKeys="true" keyProperty="id">
    insert into customer
    (name, age, gender, mail, province)
    value 
    <foreach collection="list" item="item" separator=",">
        (#{item.name}, #{item.age}, #{item.gender}, #{item.mail}, #{item.province})
    </foreach>
</insert>
```
`注意：`  
1）**useGeneratedKeys：**是否使数据库设置的主键auto_increment生效;  
2）**keyProperty：**设置主键值返回;  
3）这两个属性组合使用时可以在代码中获取到插入数据库中主键的值;  
## 二、删
**BaseMapper**中自带，无需造轮子。  
```java
int delete(@Param(Constants.WRAPPER) Wrapper<T> wrapper);
```
```java
int num = mapper.delete(Wrappers.lambdaQuery(Customer.class)
        .eq(Customer::getGender, 1)         //等于
        .ge(Customer::getAge, 20)           //大于等于
        .le(Customer::getAge, 30)           //小于等于
        .in(Customer::getProvince, Arrays.asList("Hubei", "Chongqing", "Beijing"))     //in
);
```
## 三、查
```xml
<sql id="Base_Column_List">
    id, name, age, gender, mail, province, create_time, update_time
</sql>
```
#### 1.单一参数
```java
List<Customer> findByName(String name);
```
```xml
<select id="findByName" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List"/>
    from customer
    <if test="name != null and name !=''">
        where name like concat ('%', #{name} ,'%')
    </if>
</select>
```
#### 2.多个参数  
```java
import org.apache.ibatis.annotations.Param;

List<Customer> search(@Param("customerAge") int age, 
                      @Param("customerGender") int age, 
                      @Param("data") List<String> provinces);
```
```xml
<select id="search" resultMap="BaseResultMap">
    select 
    <include refid="Base_Column_List"/>
    from customer
    where age = #{customerAge} and gender = #{customerGender}
      and province in
      <foreach collection="data" item="item" open="(" close=")" separator=",">
          #{item}
      </foreach>
    order by name
</select>
```
#### 3.类作参数
```java
public class CustomerQuery {
    private int startAge;
    private int endAge;
    private int customerGender;
    private List<String> provinces;
}
```
```java
List<Customer> listByQuery(CustomerQuery query);
```
```xml
<select id="listByQuery" resultType="anchor.mybatis.entity.Customer">
    select
    <include refid="Base_Column_List"/>
    from customer
    where age &gt;= #{startAge} and age &lt;= #{endAge} and gender = #{customerGender}
    and province in
    <foreach collection="provinces" item="item" open="(" close=")" separator=",">
        #{item}
    </foreach>
    order by name
</select>
```
## 四、改
```java
int update();
```
```xml
<update id="update">
    update customer set name = 
    case
        when name = 'Jhonny' then 'aa'
        when name = 'Andy' then 'bb'
        else 'cc'
        end
    where gender = 1;
</update>
```
## 五、小知识
1.Mysql中 **order by** 的**默认顺序**是**升序(asc)**;  
2.当 name = Anchor 时，**#{name}** 最终替换为 `'Anchor'`，**${name}** 替换为 `Anchor`;  
3.XML中的大于号和小于号:  
```text
&gt;  表示 >         &lt;  表示 <
&gt;= 表示 >=        &lt;= 表示 <=
```