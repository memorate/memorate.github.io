---
layout: post
title: Java-时间类型大全(二)
tags:
- Java 
- Study
categories: Java
description: 各Java时间类介绍
---  
**Java中各类时间介绍与基本使用(二)**

<!-- more -->
## 九、Calendar
#### 1.引用
```text
import java.util.Calendar;
```
#### 2.作用
①Calendar类提供了一些方法用于转换特定的时刻和Calendar字段(如YEAR、MONTH、DAY_OF_MONTH等)。  
②Calendar类还提供了一些操作Calendar字段的方法，例如"getFirstDayOfWeek()"。  
③简而言之，**Calendar是专门用来操作年月日时分秒的类**。  
④Calendar始于JDK1.1（[原文](https://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html)）。
#### 3.初始化
**1）当前时间**  
Calendar中的两个构造方法均为protected，因此只能使用静态方法**getInstance()**来创建Calendar对象。  
```java
Calendar calendar = Calendar.getInstance();    //初始化一个表示当前时刻的Calendar对象
```
getInstance()共有四个重载方法，用来初始化不同时区、不同地区的Calendar对象。
```java
public static Calendar getInstance();                                //默认时区、默认地区
public static Calendar getInstance(TimeZone zone);                   //指定时区
public static Calendar getInstance(Locale aLocale);                  //指定地区
public static Calendar getInstance(TimeZone zone, Locale aLocale);   //指定时区和地区
```
`注意：`getInstance()方法最终会将时间设为System.currentTimeMillis()。  
**2）指定时间**  
在调用getInstance()创建Calendar对象之后，再调用下述set方法来指定特定的时间。
```java
public void setTimeInMillis(long millis);        //设定时间戳
public void set(int field, int value);           //设定某个Calendar字段的值，field为Calendar中定义的各个常量
public final void set(int year, int month, int date, int hourOfDay, int minute, int second);    //直接设定年月日时分秒
```  
#### 4.转换

## 十、LocalDateTime