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
Calendar calendar = Calendar.getInstance();        //初始化一个表示当前时刻的Calendar对象
```
getInstance()共有四个重载方法，用来初始化不同时区、不同地区的Calendar对象。
```java
public static Calendar getInstance();                                //默认时区、默认地区
public static Calendar getInstance(TimeZone zone);                   //指定时区、默认地区
public static Calendar getInstance(Locale aLocale);                  //默认时区、指定地区
public static Calendar getInstance(TimeZone zone, Locale aLocale);   //指定时区、指定地区
```
`注意：`getInstance()方法最终会将时间设为System.currentTimeMillis()。  
**2）指定时间**  
在调用getInstance()创建Calendar对象之后，再调用下述set方法可将Calendar置为指定时间。
```java
public final void setTime(Date date);          //根据Date类来设定
```
```java
public void setTimeInMillis(long millis);      //根据时间戳来设定
```  
```java
public void set(int field, int value);         //设置指定Calendar字段的值，field为Calendar中定义的各个常量，例：Calendar.MONTH
//注意：calendar.set(Calendar.MONTH, 11); 其中11并不是11月，而是12月，可使用Calendar.DECEMBER代替
```
```java
public final void set(int year, int month, int date, int hourOfDay, int minute, int second);    //设定年月日时分秒
```
#### 4.转换
**1）Calendar转Date**
```java
Calendar calendar = Calendar.getInstance();
Date date = calendar.getTime();
```
**2）Date转Calendar**
```java
Calendar calendar = Calendar.getInstance();
Date date = new Date();
calendar.setTime(date);     
```
**3）Calendar转String**  
```text
先将Calendar转为Date类，再使用SimpleDateFormat类将Date转String。
```
#### 5.使用  
Calendar常用的几个方法。  
**1）get()**  
获取指定字段的值，**返回值为int**。
```java
public int get(int field);
```
**2）getDisplayName()**  
获取指定字段的值，**返回值为String**。
```java
public String getDisplayName(int field, int style, Locale locale);      //根据style和locale展示某个字段的值
```
```java
Calendar calendar = Calendar.getInstance();
calendar.set(Calendar.MONTH, 11);
String shortMon = calendar.getDisplayName(Calendar.MONTH, Calendar.SHORT, Locale.US);
String narrowMon = calendar.getDisplayName(Calendar.MONTH, Calendar.NARROW_FORMAT, Locale.US);
String longMon = calendar.getDisplayName(Calendar.MONTH, Calendar.LONG, Locale.US);
```
```text
上述结果为：shortMon = Dec，narrowMon = D，longMon = December
```
一般只使用**Calendar.SHORT**、**Calendar.LONG**两种style。**Calendar.NARROW_FORMAT**只显示首字母，存在重复无法识别的情况。  
**3）getDisplayNames()**  
```java
public String getDisplayNames(int field, int style, Locale locale);     //根据style和locale展示某个字段的所有可用值
```
```java
Calendar calendar = Calendar.getInstance();
Map<String, Integer> chinaNames = calendar.getDisplayNames(Calendar.DAY_OF_WEEK, Calendar.LONG, Locale.CHINA);
Map<String, Integer> usNames = calendar.getDisplayNames(Calendar.DAY_OF_WEEK, Calendar.LONG, Locale.US);
```
```text
上述结果为：
chinaNames = {星期二=3, 星期六=7, 星期三=4, 星期四=5, 星期五=6, 星期日=1, 星期一=2}
usNames = {Monday=2, Thursday=5, Friday=6, Sunday=1, Wednesday=4, Tuesday=3, Saturday=7}
```
`注意：`当没有可用的符合style和locale的展示形式时，getDisplayName()会返回null。  
**4）add()**  
```java

```
## 十、LocalDateTime