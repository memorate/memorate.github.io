---
layout: post
title: Java - 统一Exception设计
tags:
- Java
- Tips
categories: Java
description: 统一Exception设计
---  
**Java自定义Exception设计**

<!-- more -->
## 前言
**1.Throwable、Exception、RuntimeException**  
继承关系：
## 小知识
**问：Java类中的Fields和Properties有什么区别？**  
**答：Fields** —— 成员变量，可直接访问(意味着访问控制符必须为public)。pi是成员变量。    
　　**Properties** —— 属性，需通过get/set访问。name、AGE、gender是属性。  
`注：`属性的名字是由get/set后的字符串决定的。例：属性名为AGE而不是age。  
　　![]({{ "/assets/img/20200716/20200716001.png"}})![]({{ "/assets/img/20200716/20200716002.png"}})