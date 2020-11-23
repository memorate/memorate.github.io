---
layout: post
title: SpringBoot - 默认异常处理机制探索
tags:
- Java
- SpringBoot
categories: SpringBoot
description: Exception处理机制
---  
**SpringBoot默认异常处理机制探索**

<!-- more -->
## 前言
使用 Postman 和浏览器访问 SpringBoot 的接口**异常时默认**会返回如下信息，本文着重于**探索异常抛出到返回下图中信息的整个过程**。  
![]({{ "/assets/img/20200728/20200728001.jpg"}})![]({{ "/assets/img/20200728/20200728002.jpg"}})  
## 流程
![]({{ "/assets/img/20200728/20200728003.png"}})
## 自动配置
自动配置类为**ErrorMvcAutoConfiguration**，其中配置了一些组装错误信息页所需要的组件(Bean)。  
```java
package org.springframework.boot.autoconfigure.web.servlet.error;
```
**1.DefaultErrorAttributes**  
此类用于接收并传递异常相关的关键属性，如时间戳、请求状态、异常类名、异常信息、异常请求地址等。  
**2.BasicErrorController**  
此类用于提取请求中的关键信息，并将其封装成错误信息页返回给请求调用者。(详细解析在[这里](#here))  
**3.ErrorPageCustomizer**  
此类用于向Spring注册一个ErrorPage
## 一、StandardHostValve
## 二、DispatcherServlet
1、**DispatcherServlet** 是 `org.springframework.web.servlet` 包下的一个 Java 类。  
```text
官方注释：
Central dispatcher for HTTP request handlers/controllers, e.g. for web UI controllers or HTTP-based remote service exporters. 
Dispatches to registered handlers for processing a web request, providing convenient mapping and exception handling facilities.

个人翻译：
DispatcherServlet 是 SpringBoot 中 HTTP 请求的中央调度器，为 Web UI 或基于 HTTP 的远程发射器发送的请求提供服务。
其主要作用是向注册的可以处理 web 请求的 handlers 分发请求，并提供方便的映射、异常处理设施。

个人理解：
DispatcherServlet 类是 SpringBoot 的调度器，它负责组织和协调不同组件完成请求并返回响应结果。
DispatcherServlet 的主要任务是：①将请求发送至对应的 Controller/Handler、②请求结果处理及返回（正常及异常处理结果）。(本文只聚焦于异常处理部分)
```
## 三、<span id="here">BasicErrorController</span>  

