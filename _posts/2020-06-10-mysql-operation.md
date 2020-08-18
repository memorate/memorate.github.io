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
2.char和varchar的区别？  
3.int(10)、char(10)、varchar(10)分别能存多少个字符？  
## 一、增
**1.创建数据库**  
```sql
create database collection;
```
```sql
create database if not exists collection;
```
**2.创建表**  
1）枚举类型数据（stu_gender）使用int来保存，在注释中写清对应的含义即可。  
2）创建时间和更新时间一般必有，使用**current_timestamp**函数来自动生成。  
```sql
drop table if exists student;
create table student
(
    id          int unsigned auto_increment primary key comment '学生id',
    stu_name    varchar(16) not null comment '学生姓名',
    stu_age     smallint    not null default 18 comment '学生年龄',
    stu_gender  int(2)      not null default 0 comment '0-男生，1女生',
    stu_class   int(2)      not null default 0 comment '学生所在班级，1—N',
    stu_grade   char(10)    not null default 'first' comment '学生所在年级',
    stu_boarder boolean     not null default false comment '是否住校，true-是，false-否',
    create_time timestamp   not null default current_timestamp comment '学生记录创建时间',
    update_time timestamp   not null default current_timestamp on update current_timestamp comment '学生记录更新时间',
    index index_stu_name (stu_name),
    index index_stu_grade (stu_grade)
) comment '学生信息表' character set 'utf8mb4';
```
**3.新增字段**  
```sql
alter table student add column stu_boarder boolean not null default false comment '是否住校，true-是，false-否';
```
**4.新增索引**
```sql
alter table student add index index_stu_grade (stu_grade);
```  
**5.新增记录**  
1）新增记录时**自增的主键**可使用**default**或**不填**。  
2）新增记录时某字段若使用**默认值**可用**default**或**不填**。  
3）关键字value和values**都可以**用来插入单条/多条记录，`value插入多条记录时较快，values插入单条记录时较快`。（It confused）  
```sql
insert into student (id, stu_name, stu_age, stu_gender, stu_class, stu_grade) 
    values (default, 'Anchor', 17, 'male', 13, 'sixth');          <!-- 插入自增主键id时用default -->
```
```sql
insert into student (stu_name, stu_age, stu_gender, stu_class, stu_grade, stu_address, stu_boarder)
value ('Anchor', 17, 'male', 13, 'sixth', 'Nanjing', default),     <!-- stu_boarder使用默认值时用default -->
      ('Michel', 25, 'male', 6, 'ninth', 'Beijing', true);
```
## 二、删
## 三、查
## 四、改
**1.修改字段**  
```sql

```