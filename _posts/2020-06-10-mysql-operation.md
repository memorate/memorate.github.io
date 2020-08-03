---
layout: post
title: MySQL - 日常使用sql语句收集
tags:
- MySQL
- Tips
categories: MySQL
description: 高频使用sql语句收集
---  
**MySQL日常使用的sql语句积累**

<!-- more -->
## 注
1.编写sql语句时一律使用**小写**字母(保留字可大写，仅仅是为了便于人类可以迅速识别sql的含义)。Win下MySQL大小写不敏感，Linux下MySQL大小写敏感。  
2.
## 一、增
**1.创建数据库**  
```sql
create database collection;
```
```sql
create database if not exists collection;
```
**2.创建表**  
```sql
create table student(
    stu_id int auto_increment primary key,
    stu_name varchar(16) not null comment '学生姓名',
    stu_age smallint not null default 18 comment '学生年龄',
    stu_gender char(8) not null default 'male' comment '学生性别',
    stu_class int not null default 0 comment '学生所在班级，1—N',
    stu_grade char(10) not null default 'first' comment '学生所在年级,'
)comment '学生信息表';
```
**3.创建记录**  
```sql
insert into student (stu_id, stu_name, stu_age, stu_gender, stu_class, stu_grade)
    value (default, 'Anchor', 17, 'male', 13, 'sixth')
```
## 二、删
## 三、查
## 四、修