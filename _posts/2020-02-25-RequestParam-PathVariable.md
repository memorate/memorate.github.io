---
layout: post
title: SpringBoot - @RequestParam与@PathVariable
tags:
- Java 
- SpringBoot
- Study
categories: Java
description: 注解@RequestParam与@PathVariable
---  
**@RequestParam与@PathVariable的使用介绍**

<!-- more -->
## 一、简介
**①@RequestParam与@PathVariable一般用于GET请求中（也可用于POST请求中），用来识别URL中传入的参数。**  
**②二者不同之处在于在URL中的传参方式不同。**  
**③个人建议使用@RequestParam。**  
## 二、@RequestParam
### 1、所在包
```text
package org.springframework.web.bind.annotation;
```
### 2、作用
```java
/**
 * 原文：
 * Annotation which indicates that a method parameter should be bound to a web request parameter.
 *
 * 翻译：
 * 该注解表示将Web请求参数与Java方法的参数绑定。
 */
```
Web请求参数：localhost:8080/request?name=Anchor&gender=male，其中两个等号前的name、gender为web请求参数。参数间用'&'连接。
### 3、属性
@RequestParam注解有四个属性：**value**、**name**、**required**、**defaultValue**。  
1）value：根据此属性的值来绑定URI中的变量；  
2）name：作用与value相同；  
3）required：可选true或false，默认值为true。用于标明此变量是否必须，必须时调用该接口时必须传值；  
4）defaultValue：设置所注解变量的默认值，当未传入值时，使用此默认值；
### 4、用法
1）用@GetMapping(或@PostMapping)注解Controller中的方法；  
2）在被注解方法的入参处使用@RequestParam来**申明**一个Web请求参数；  
3）**使用**该参数；  
　　在请求地址后接'?'与参数：localhost:8080/request?name=Anchor&gender=male。  
　　①localhost:8080/request；  
　　②'?'后代表请求参数，name、gender为参数名，Anchor、male为参数值，'&'用来连接多个参数；  
### 5、示例
1）**【单一参数】**  
```java
@GetMapping("/Request}")
public String RequestParam(@RequestParam String name) {
    if (name.isEmpty()) return "404";
    return name + ".This method uses @ParamVariable.";
}
```
调用结果：  
![]({{ "/assets/img/20200225/p1.jpg"}})  
2）**【多个参数】**  
```java
@GetMapping("/multiRequest")
public String MultiRequestParam(@RequestParam String name, @RequestParam String gender) {
    if (name.isEmpty() || gender.isEmpty()) return "404";
    if ("male".equals(gender.toLowerCase())) {
        return "Mr " + name + ".This method uses @RequestParam.";
    } 
    if ("female".equals(gender.toLowerCase())) {
        return "Mrs " + name + ".This method uses @RequestParam.";
    }
    return "Parameter \"gender\" is not illegal";
}
```
调用结果：  
![]({{ "/assets/img/20200225/p2.jpg"}})  
## 三、@PathVariable
### 1、所在包
```text
package org.springframework.web.bind.annotation;
```
### 2、作用
```java
/**
 * 原文：
 * Annotation which indicates that a method parameter should be bound to a URI template
 * variable. Supported for {@link RequestMapping} annotated handler methods.
 *
 * 翻译：
 * 该注解表示将URI模板中的变量与Java方法的参数绑定。支持@RequestMapping所注解的方法。
 */
```
通俗易懂版：**该注解通过URI模板变量的方式来识别URL中的参数。**  
URI模板样例：localhost:8080/variable/{name}/{gender}，其中**{name}、{gender}**表示参数，参数名分别为name、gender。
### 3、属性
@PathVariable注解有三个属性：**value**、**name**、**required**。  
1）value：根据此属性的值来绑定URI中的变量；  
2）name：作用与value相同；  
3）required：可选true或false，默认值为true。用于标明此变量是否必须，必须时调用该接口时必须传值；
### 4、用法
1）用@GetMapping(或)注解Controller中的方法；
2）在@GetMapping注解的value属性中使用大括号及参数名"{paramName}"来**申明**一个参数；  
3）在被注解方法的入参处使用@PathVariable将方法的参数与第1步中申明的变量进行**绑定**；  
4）**使用**该参数；  
`小技巧：`  
I、当@GetMapping(value = "/variable/{name}")注解只使用了value一个属性时，可省略value简写为@GetMapping("/variable/{name}")。  
II、@PathVariable注解有两种绑定参数的方式
```java
/** 使用@PathVariable的value或name属性来绑定
 *  只使用value或name属性时，@PathVariable(value = "name")可简写为@PathVariable("name")
 * （此时Java参数(String param)可任意取名）
 */
@GetMapping(value = "/variable/{name}")
public String ParamVariable(@PathVariable("name") String param) {...}
```  
```java
/** 通过@PathVariable所注解的变量的名称来绑定
 * （此时Java参数(String name)名称必须与@GetMapping中的变量({name})名一致）
 */
@GetMapping("/variable/{name}")
public String ParamVariable(@PathVariable String name) {...}
```
### 5、示例
1）**【单一参数】**
```java
@GetMapping("/variable/{name}")
public String ParamVariable(@PathVariable String name) {
    if (name.isEmpty()) return "404";
    return name + ".This method uses @ParamVariable.";
}
```
调用结果：  
![]({{ "/assets/img/20200225/p3.jpg"}})
2）**【多个参数】**
```java
@GetMapping("/multiVariable/{name}/{gender}")
public String MultiParamVariable(@PathVariable String name, @PathVariable String gender) {
    if (name.isEmpty() || gender.isEmpty()) return "404";
    if ("male".equals(gender.toLowerCase())) {
        return "Mr " + name + ".This method uses @RequestParam.";
    } 
    if ("female".equals(gender.toLowerCase())) {
        return "Mrs " + name + ".This method uses @RequestParam.";
    }
    return "Parameter \"gender\" is not illegal";
}
```
调用结果：  
![]({{ "/assets/img/20200225/p4.jpg"}})