---
layout: post
title: Java-@RequestParam与@PathVariable
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
@RequestParam与@PathVariable一般用于GET请求中，用来识别前端在URL中传入的参数。
## 二、@RequestParam
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
 * 该注解指明应将URI模板中的变量与Java方法的参数相绑定。支持@RequestMapping所注解的方法。
 */
```
URI模板样例：localhost:8080/variable/{name}
### 3、用法

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
 * 该注解表示将URI模板中的变量与Java方法的参数相绑定。支持@RequestMapping所注解的方法。
 */
```
通俗易懂版：**该注解通过URI模板变量的方式来识别URL中的参数。**  
URI模板样例：localhost:8080/variable/{name}，其中**{name}**表示一个参数，参数名为name。
### 3、用法
第1步：在@GetMapping注解的value属性中使用大括号及参数名"{paramName}"来**申明**一个参数；  
第2步：在被注解方法的入参处使用@PathVariable将方法的参数与第1步中申明的变量进行**绑定**；  
第3步：**使用**该参数；  
PS：  
I、当@GetMapping(value = "/variable/{name}")注解只使用了value一个属性时，可省略value简写为@GetMapping("/variable/{name}")。  
II、@PathVariable注解有两种绑定参数的方式
```java
/** 使用@PathVariable的默认属性来绑定
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
##### ①、用法示例一【单一参数】
```java
@GetMapping("/variable/{name}")
public String ParamVariable(@PathVariable String name) {
    if (name.isEmpty()) return "404";
    return name + ".This method uses @ParamVariable.";
}
```
调用结果：  
![]({{ "/assets/img/20200225/p3.jpg"}})
##### ②、用法示例二【多个参数】
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