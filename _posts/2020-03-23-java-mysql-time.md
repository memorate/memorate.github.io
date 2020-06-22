---
layout: post
title: Java-MySQL时间对照表
tags:
- Java 
- Study
categories: Java
description: Java与MySQL时间类型对照表
---  
**实操结果：Java与MySQL时间类型对照表**

<!-- more -->
　　　　　　　　　　　　　　　　![]({{ "/assets/img/20200323/time.png"}})  
## 先上结论
***不论数据库的时间类型是什么，在Java中一律使用 java.time.LocalDateTime即可。***  
## 注意事项  
**1.实操时MySQL版本为8.0.20。**  
**2.上表中右列的Java类型数据均可对应入库为左列的MySQL类型数据。**    
**3.采坑：MySQL时区设置为UTC导致Java、MySQL显示的时间不一致（[解决方案](https://blog.csdn.net/starlemon2016/article/details/90314649?depth_1-utm_source=distribute.pc_relevant.none-task&utm_source=distribute.pc_relevant.none-task)）。**  
## 细节问题  
**1.MySQL中time类型只会存储时间，`不会`存储日期。因此读取到的Java类日期会显示为1970-01-01。**  
**2.MySQL中timestamp类型会存储日期、时间，可以和Java类无缝转换。**  
**3.MySQL中date类型只会存储日期，`不会`存储时间。因此读取到的Java类时间会显示为00:00**（比较奇怪的是Instant类会显示为16:00）。  
**4.MySQL中datetime类型会存储日期、时间，可以和Java类无缝转换。**  