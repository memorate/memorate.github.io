---
layout: post
title: Java-@RequestParam与@PathVariable的故事
tags:
- Java 
- Spring Boot
- Study
categories: Java
description: 注解@RequestParam与@PathVariable
---  
**@RequestParam与@PathVariable的使用介绍**

<!-- more -->
## 一、@RequestParam
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

## 二、@PathVariable
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
URI模板样例：localhost:8080/variable/{name}，其中{name}表示一个参数，参数名为name
### 3、用法
第1步：在@GetMapping注解的value属性中使用大括号及参数名"{paramName}"来申明一个参数。  
第2步：在被注解方法的入参处使用@PathVariable将方法的参数与第1步中申明的变量进行绑定  
第3步：使用该参数  
PS：  
1、当@GetMapping(value = "/variable/{name}")注解只使用了value一个属性时，可简写为@GetMapping("/variable/{name}")  
2、@PathVariable注解有两种使用方式
```java
//使用@PathVariable的属性来绑定@GetMapping中变量（此时@PathVariable所注解的变量名可任意取）
@GetMapping(value = "/variable/{name}")
public String ParamVariable(@PathVariable("name") String param) {...}
```  
```java
/** 通过@PathVariable所注解的变量的名称来绑定@GetMapping中变量
 * （此时@PathVariable所注解的变量名必须与@GetMapping中的变量名一致）
 */
@GetMapping("/variable/{name}")
public String ParamVariable(@PathVariable String name) {...}
```
#### 单一参数
```java
@GetMapping("/variable/{name}")
public String ParamVariable(@PathVariable String name) {
    return name + ".This method uses @ParamVariable.";
}
```
![]({{ "/assets/img/20200225/p3.jpg"}})
#### 多个参数
```java
@GetMapping("/multiVariable/{name}/{gender}")
public String MultiParamVariable(@PathVariable String name, @PathVariable String gender) {
    if (gender.isEmpty()){
        return "404";
    }
    if ("male".equals(gender.toLowerCase())) {
        return "Mr " + name + ".This method uses @RequestParam.";
    } else if ("female".equals(gender.toLowerCase())) {
        return "Mrs " + name + ".This method uses @RequestParam.";
    }else {
        return "Parameter \"gender\" is not illegal";
    }
}
```
![]({{ "/assets/img/20200225/p4.jpg"}})