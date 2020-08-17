---
layout: post
title: Java - 时间类型大全(二)
tags:
- Java 
- Study
categories: Java
description: 各Java时间类介绍
---  
**Calendar、LocalDateTime介绍与基本使用**

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
#### 3.实例化
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
```java
Calendar calendar = Calendar.getInstance();
int year = calendar.get(Calendar.YEAR);
int month = calendar.get(Calendar.MONTH);               //month = 2，并不是指2月，而是Calendar类中3月对应的int是2
int day = calendar.get(Calendar.DATE);
int dayOfWeek = calendar.get(Calendar.DAY_OF_WEEK);     //dayOfWeek = 3，并不是指周三，而是周二
```
```text
上述结果为：year = 2020，month = 2，day = 10，dayOfWeek = 3
```
**2）getDisplayName()**  
获取指定字段的值，**返回值为String**。
```java
public String getDisplayName(int field, int style, Locale locale);      //根据style和locale展示某个field的值
```
```java
Calendar calendar = Calendar.getInstance();
calendar.set(Calendar.MONTH, 11);               //11 也可用 Calendar.DECEMBER 来代替，更加直观
String shortMon = calendar.getDisplayName(Calendar.MONTH, Calendar.SHORT, Locale.US);
String narrowMon = calendar.getDisplayName(Calendar.MONTH, Calendar.NARROW_FORMAT, Locale.US);
String longMon = calendar.getDisplayName(Calendar.MONTH, Calendar.LONG, Locale.US);
```
```text
上述结果为：shortMon = Dec，narrowMon = D，longMon = December
```
`注意：`  
①一般只使用**Calendar.SHORT**、**Calendar.LONG**两种style。**Calendar.NARROW_FORMAT**只显示首字母，存在重复无法识别的情况。  
②get()获取到的是数字值，getDisplayName()获取到的是String值。  
③Year、Day这类数据只有int一种表示方法，因此使用getDisplayName()会返回null。  
**3）getDisplayNames()**  
获取某个字段的`所有展示值`，**返回值为Map<String, Integer>**。  
```java
public String getDisplayNames(int field, int style, Locale locale);     //根据style和locale展示某个字段的所有展示
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
**4）add()**  
```java
abstract public void add(int field, int amount);      //增、减某个字段的值，amount正为增，负为减
```
```java
Calendar calendar = Calendar.getInstance();
calendar.add(Calendar.MONTH, -7);
calendar.add(Calendar.DATE, 10);
```
`注意：`若当前为3月，amount为-5，add()之后月份为9月；日期类似，也可倒退到1之前。  
**5）比较**  
```java
public boolean before(Object when);                  //是否早于
```  
```java
public boolean after(Object when);                   //是否迟于
```  
```java
public int compareTo(Calendar anotherCalendar);      //比较。0 = 相等；小于0 = 早于；大于0 = 迟于
```  
## 十、LocalDateTime
#### 1.为什么使用
1）LocalDateTime是Java8推出的Date类的升级版，若条件允许，推荐使用LocalDateTime。  
2）Date体系混乱且复杂，存在两个***java.util.Date***、***java.sql.Date***类，还需配套的Calendar、TimeZone、Locale类等。  
3）Date若不格式化，打印出的日期可读性差，而SimpleDateFormat格式化Date的操作是***线程不安全***的。  
4）LocalDateTime***简单***，学习简单、使用简单。  
#### 2.引用
```text
import java.time.LocalDateTime;
```
#### 3.实例化
构造方法访问权限为private，不可用。使用静态方法***now()***来实例化。
```java
LocalDateTime now = LocalDateTime.now();        //实例化一个表示当前时间的LocalDateTime类
```
或者使用静态方法***of()***直接指定年月日时分秒来实例化。
```java
LocalDateTime time1 = LocalDateTime.of(2020, 3, 10, 18, 22, 34, 633);    //指定 年月日、时分秒、纳秒
LocalDateTime time2 = LocalDateTime.of(2020, 3, 10, 18, 22, 34);         //指定 年月日、时分秒
LocalDateTime time3 = LocalDateTime.of(2020, 3, 10, 18, 22);             //指定 年月日、时分
```
`注意：`  
　1）月的范围是1到12，也可用枚举类***java.time.Month***来代替。  
　2）日的范围是1到31，时的范围是0到23，分、秒的范围是0到59。  
　3）若使用***of()***时超出上述范围，编译会报错。  
#### 4.转换
**1）LocalDateTime转String**  
利用***toString()***方法进行转换,其默认格式为：`yyyy-MM-ddTHH:mm:ss.SSS`。
```java
LocalDateTime now = LocalDateTime.now();
System.out.println(now);        //输出 2020-03-10T15:34:48.926
```
或者使用LocalDateTime的***format()***方法转换为自定义格式的String。通过DateTimeFormatter来指定需要的String格式。
```java
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
String time = now.format(formatter);
```
DateTimeFormatter有15种内置格式（以下为部分）：
```text
DateTimeFormatter.BASIC_ISO_DATE               20200310
DateTimeFormatter.ISO_LOCAL_DATE_TIME          2020-03-10T18:30:34
DateTimeFormatter.ISO_LOCAL_DATE               2020-03-10
DateTimeFormatter.ISO_DATE_TIME                2020-03-10T18:30:34
DateTimeFormatter.ISO_ORDINAL_DATE             2020-070
```
**2）LocalDateTime转指定时区的String**  
```java
LocalDateTime now = LocalDateTime.now(ZoneId.of("UTC"));        //实例化时指定时区
String time = now.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```
或者
```java
LocalDateTime now = LocalDateTime.now();
LocalDateTime utc = now.atZone(ZoneId.systemDefault()).withZoneSameInstant(ZoneOffset.UTC).toLocalDateTime();
String time = utc.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));
```
**3）String转LocalDateTime**  
使用***LocalDateTime.parse()***进行解析，String中的时间格式必须与DateTimeFormatter的pattern匹配。
```java
String time = "2020-03-10 11:11:11";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
LocalDateTime dateTime = LocalDateTime.parse(time, formatter);
```  
**4）Date转LocalDateTime**  
时间戳是所有系统通用的表示时间的方式，因此通过时间戳来转换Date和LocalDateTime。  
具体使用***LocalDateTime.ofInstant()***进行转换（Java中将时间戳抽象为***Instant***类）。  
```java
Date date = new Date();
LocalDateTime time = LocalDateTime.ofInstant(date.toInstant(), ZoneId.of("Asia/Shanghai"));
```
`注`：因为时间戳不带有任何地区信息，所以***ofInstant()***方法需要借助***ZoneId***类。  
**5）LocalDateTime转Date**  
借助***Instant***类使用***Date.from()***方法来转换。
```java
LocalDateTime now = LocalDateTime.now();
Date date = Date.from(now.toInstant(ZoneOffset.of("+8")));
```
`注`：***ZoneOffset.of("+8")***中**+8**代表中国时区，其他时区可**+/- N**。  
**6）LocalDateTime转Unix**  
```java
long utc = LocalDateTime.now().toEpochSecond(ZoneOffset.UTC);          //转表示UTC的unix
long cst = LocalDateTime.now().toEpochSecond(ZoneOffset.of("+8"));     //转表示CST的unix
```
#### 5.使用
LocalDateTime常用的几类方法。  
**1）获取某个字段值**
```java
LocalDateTime now = LocalDateTime.now();
int year = now.getYear();                       //年
Month month = now.getMonth();                   //月
int monthValue = now.getMonthValue();           //int类型的月
int day = now.getDayOfMonth();                  //日
DayOfWeek dayOfWeek = now.getDayOfWeek();       //星期几，dayOfWeek.getValue()获取int值，周一int值为1，周天为7
int hour = now.getHour();                       //时
int minute = now.getMinute();                   //分
```
**2）加减**  
a.加减操作有三类方法，分别以**plus**、**minus**、**with**开头；  
b.三类方法入参都可正可负，意味着每类方法都可以单独实现加、减两个功能，但是根据约定plus用来加，minus用来减；  
c.minus实际调用了plus方法，而plus方法最终又调用了with方法；    
```java
LocalDateTime now = LocalDateTime.now();
LocalDateTime plusDays = now.plusDays(3);
LocalDateTime minusDays = now.minusDays(3);
LocalDateTime withDays = now.withDayOfMonth(3);
```
`注意`：三类方法并不改变自己身的值，而是返回一个修改过值得副本。
**3）比较**  
```java
LocalDateTime now = LocalDateTime.now();
LocalDateTime future = now.plusDays(3);
boolean after = now.isAfter(future);            //是否迟于
boolean before = now.isBefore(future);          //是否早于
boolean equal = now.isEqual(future);            //是否相等
int i = now.compareTo(future);                  //负为早于，正为迟于，0为相等
```  
**4）计算**  
利用***java.time.Duration***类来计算两个LocalDateTime之间的时间差。
```java
LocalDateTime now = LocalDateTime.now();
LocalDateTime plusDays = now.plusDays(3);
Duration between = Duration.between(now, plusDays);
long days = between.toDays();         //正为plusDays迟于now几天，负为早于
long hours = between.toHours();
long minutes = between.toMinutes();
```