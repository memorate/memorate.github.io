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
①GMT（Greenwish Mean Time），格林威治标准时间，24小时制。  
②根据观测天体运动而产生的前世界统一标准时间，12点为太阳横穿本初子午线时的时间，0点、24点分别向前、向后推12小时。  
## 二、UTC
①UTC（Coordinated Universal Time），世界协调时间，24小时制。  
②以GMT为准，结合地球自转时间与原子钟的高精度度量所综合精算而成的现世界统一标准时间（说是时间，实际是一个标准）。  
③由于地球分为24个时区（经度每15°为1个时区，时间为1小时），不同地区UTC的表示方式也不同，本初子午线为0时区。  
④计算公式：UTC + 时区差 = 本地时间。时区差0时区向东为正，0时区向西为负。  
　UTC + (+0800) = 北京时间，UTC标准时间10:22，北京时间为18:22；  
　UTC + (-0500) = 纽约时间，UTC标准时间10:22，纽约时间为05:22；  
## 三、Date
## 四、Calendar
## 五、LocalDateTime