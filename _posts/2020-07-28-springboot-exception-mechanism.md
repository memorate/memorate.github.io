---
layout: post
title: SpringBoot - 默认异常处理机制探索
tags:
- Java
- Tips
categories: Java
description: Exception处理机制
---  
**SpringBoot默认异常处理机制探索**

<!-- more -->
## 前言
使用 Postman 和浏览器访问 SpringBoot 的接口异常时会返回下面的信息，本文着重于探索异常抛出到返回信息的整个过程。  
![]({{ "/assets/img/20200728/20200728001.jpg"}})![]({{ "/assets/img/20200728/20200728002.jpg"}})  
## DispatcherServlet
**DispatcherServlet** 是 `org.springframework.web.servlet` 包下的一个 Java 类。
```text
官方注释：
Central dispatcher for HTTP request handlers/controllers, e.g. for web UI controllers or HTTP-based remote service exporters. 
Dispatches to registered handlers for processing a web request, providing convenient mapping and exception handling facilities.

个人翻译：
DispatcherServlet 是 SpringBoot 中 HTTP 请求的中央调度器，为 Web UI 或基于 HTTP 的远程发射器发送的请求提供服务。
其主要作用是向注册的可以处理 web 请求的 handlers 分发请求，并提供方便的映射、异常处理设施。

个人理解：
DispatcherServlet 类是 SpringBoot 的入口，它负责组织和协调不同组件完成请求并返回响应结果。
DispatcherServlet 的主要任务是：将请求发送至对应的 Controller、异常处理。
```
