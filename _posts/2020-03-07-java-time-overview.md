---
layout: post
title: Java-时间类型大全
tags:
- Java 
- Study
categories: Java
description: 各Java时间类介绍
---  
**Java中各类时间介绍与基本使用**

<!-- more -->
## 一、GMT
**①GMT（Greenwish Mean Time）**，**格林威治标准时间**，24小时制。  
**②**根据观测天体运动而产生的`前世界统一标准时间`，12点为太阳横穿本初子午线时的时间，0点、24点分别向前、向后调整12小时。  
## 二、UTC
**①UTC（Coordinated Universal Time）**，***世界协调时间***，24小时制。  
**②**以GMT为准，由地球自转时间与原子钟的高精度度量所综合精算而成的`现世界统一标准时间`（说是时间，实际是一个标准）。  
**③**由于地球分为24个时区（经度每15°为1个时区，时间为1小时），不同地区UTC的表示方式也不同，本初子午线为0时区。  
**④**计算公式：`UTC + 时区差 = 本地时间`。时区差0时区**向东为正**，0时区**向西为负**。  
　UTC + (+0800) = 北京时间，UTC标准时间10:22，北京本地时间为18:22；  
　UTC + (-0500) = 纽约时间，UTC标准时间10:22，纽约本地时间为05:22；  
## 三、CST
**①CST（China Standard Time），中国标准时间、北京时间**，为 UTC + 8。  
**②**CST还可以表示其他3种时间：  
```text
Central Standard Time (USA)         美国标准时间          UTC - 06:00 
Central Standard Time (Australia)   澳大利亚标准时间      UTC + 09:30
Cuba Standard Time                  古巴标准时间          UTC - 04:00
```
## 四、ISO
**①**国际标准化组织发布的日期和时间的**表示方法**，目前最新为第三版ISO8601:2004。  
**②**ISO 8601的**标准格式**是：yyyy-MM-ddTHH:mm:ssZ（`严格区分大小写`），分别表示：  
```text
YYYY：   年，0000 — 9999　　　　　　　　　　
MM：     月， 01  —  12  
DD：     日， 01  —  31　　　　　　　　　　　　　　
HH：     时， 00  —  24  
mm：     分， 00  —  59　　　　　　　　　　　　　
ss：     秒， 00  —  59                T：分隔符，用来分隔日期和时间，T之前为日期，T之后为时间
.sss：  毫秒，000 —  999               Z：零时区，可以是：Z、+HH:mm、-HH:mm
```
更多格式——>[这里](https://baijiahao.baidu.com/s?id=1660164866833260888&wfr=spider&for=pc)
## 五、Unix
**①Unix时间戳**，也称为时间戳，是指从1970年1月1日00:00开始所经过的**秒数**，不考虑闰秒。  
**②为什么是从1970-01-01 00:00开始？**  
　由于32位机器能表示的最大时间为68年，若取1900年，会在1968年发生时间回归现象。  
**③为什么使用Unix时间戳？**  
　不同数据库对时间的格式定义不同，导致跨平台时间无法转换，而使用Unix时间戳可以跨平台通用。  
**④为什么Unix时间戳只有10位？**  
　10位Unix时间戳耗尽的准确的时间是：2286年11月21日1时46分39秒，所以暂时**没必要**使用11位。  
**⑤13位Unix时间戳是怎么回事？**  
　10位Unix时间戳精确到**秒**，13位Unix时间戳精确到**毫秒**。  
```text
1583547742     ——  2020/3/7 10:22:22  
1583547742622  ——  2020/3/7 10:22:22.622  
```
## 六、TimeZone
#### 1.所在包
```text
import java.util.TimeZone;
```
#### 2.作用
TimeZone表示一个时区（相对UTC）的偏移量，并且可以推算出[夏令时](https://baike.baidu.com/item/%E5%A4%8F%E4%BB%A4%E6%97%B6/1809579?fr=aladdin)，始于JDK1.1。
简而言之，用来表示各个时区。   
```text
原文：TimeZone represents a time zone offset, and also figures out daylight savings.
``` 
#### 3.常用时区
```text
Asia/Shanghai      Asia/Urumqi             Hongkong
Europe/London      America/Los_Angeles     Japan
```
可使用`TimeZone.getAvailableIDs()`来获取所有可用时区Id。
## 七、Date
#### 1.所在包
```text
import java.util.Date;
```
#### 2.作用
拥有毫秒级精确度的、用来表示一个特定时刻的Java类，始于JDK1.0。 
```text
原文：The class Date represents a specific instant in time, with millisecond precision.
``` 
#### 3.初始化
Date类目前只有两个推荐使用的构造方法，其余已被弃用。  
1）使用**无参构造函数**new一个Date类，表示当前时刻。内部实现：new Date(System.currentTimeMillis())（非源码，仅用于表意）；
```java
Date date = new Date();   
```
2）使用**Unix时间戳**new一个Date类，10位、13位时间戳皆可；
```java
Date date = new Date(1583547742);
```
#### 4.转换
使用SimpleDateFormat类可将Date转化为指定ISO格式字符串、或解析符合ISO格式的字符串为Date类。  
**1）Date转String**
```java
SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");    //new SimpleDateFormat()时指定ISO格式
Date date = new Date();
String dateToString = format.format(date);
```
**2）Date转**`指定时区`**String**  
`TimeZone.getTimeZone("Japan")`中参数也可直接填写"UTC+XX00"和"UTC-XX00"。
```java
SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");    //new SimpleDateFormat()时指定ISO格式
format.setTimeZone(TimeZone.getTimeZone("Japan"));
Date date = new Date();
String dateToString = format.format(date);
```
**3）String转Date**，注意此时字符串中日期格式要与SimpleDateFormat指定的格式相符；
```java
SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");    //new SimpleDateFormat()时指定ISO格式
String stringDate = "2020-03-07 10:22";
Date date = format.parse(stringDate);
```
**4）Date转时间戳**，注意此时的timeStamp为13位；
```java
Date date = new Date();
long timeStamp = date.getTime();
```
## 八、Calendar
#### 1.所在包
```text
import java.util.Calendar;
```
#### 2.作用
抽象类，它提供了一些方法用于在特定时间与日历字段(如YEAR、MONTH、DAY_OF_MONTH等)之间进行转换，始于JDK1.1。  
简而言之，**专门用于操作年月日时分秒的类**（[原文](https://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html)）。  
#### 3.初始化
## 九、LocalDateTime