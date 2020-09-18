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
　　　　　　　　　　　　![]({{ "/assets/img/20200716/20200716003.png"}})  
**1）Throwable：**所有Error和Exception的父类。只有Throwable与其子类才能被JVM或throw关键字抛出、被catch语句捕捉。
Throwable主要包含了该类被创建时当前线程的执行信息。  
**2）Error：**程序运行时出现的系统错误或资源错误，代码无法捕获、处理，如栈溢出(StackOverFlowError)、内存溢出(OutOfMemoryError)等。  
**3）Exception：**代码编写引起的异常，代码可以捕获并处理。Exception分为**受检异常**和**非受检异常**。  
　**a.Checked**：除RuntimeException(及其子类)外的所有Exception，此类异常编译器**会强制**要求处理。如IOException、SQLException。  
　**b.Unchecked**：RuntimeException及其子类，此类异常是程序员代码问题造成的的，编译器**不强制**要求处理，JVM会处理。如NullPointerException。  
`综上，自定义异常应是Unchecked异常，也就是说需要继承RuntimeException`  
**2.应在方法签名处抛出异常还是在方法中捕获处理异常？**  
　原则是：捕捉并处理知道如何处理的异常，抛出不知如何处理的异常。  
## 设计
1.异常类只需提供**异常码**和**异常信息**两类数据即可，异常码可以用来定位发生异常代码所在的位置，异常信息用来提示用户。  
2.异常码数量会逐渐增多，因此定义StatusCode接口，所有异常码类实现此接口即可。  
```java
class Exception extends RuntimeException{           //伪代码
    StatusCode code;         //异常码
    String message;          //异常信息
}
```
**Exception与上篇中的Response设计基本相同，二者的响应码(异常码)部分可以共用**  
## 一、StatusCode
1.StatusCode接口定义了异常码相关的规范，包含两个方法，分别用于返回异常码和异常信息。  
2.后续所有的异常码类实现StatusCode即可。  
（此接口与统一Response中的相同）  
```java
public interface StatusCode extends Serializable {
    @JsonValue       //序列化时只显示code，message以DefaultException中的为准
    int code();

    String message();
}
```
## 小知识
**问：Java类中的Fields和Properties有什么区别？**  
**答：Fields** —— 成员变量，可直接访问(意味着访问控制符必须为public)。例，下图中pi是成员变量。    
　　**Properties** —— 属性，需通过get/set访问。例，下图中name、AGE、gender是属性。  
`注：`属性的名字是由get/set后的字符串决定的。例：属性名为AGE而不是age。  
　　![]({{ "/assets/img/20200716/20200716001.png"}})![]({{ "/assets/img/20200716/20200716002.png"}})
**2.**