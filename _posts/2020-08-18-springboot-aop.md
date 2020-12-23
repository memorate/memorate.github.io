---
layout: post
title: SpringBoot - AOP使用手册
tags:
- Java
- SpringBoot
categories: SpringBoot
description: SpringBoot-AOP使用
---  
**SpringBoot中AOP使用教程**

<!-- more -->
## AOP
1.AOP(Aspect Oriented Programming，**面向切面编程**)，是对OOP(Object Oriented Programming，面向对象编程)的一种**补充**；  
2.OOP侧重于对象的封装、继承，AOP则侧重于对OOP封装对象的某一公共行为的提取；  
3.AOP可以在不修改OOP代码逻辑的前提下，在切入点(Pointcut)向现有代码中添加一些自定义行为(Advice)；  
4.SpringBoot中AOP的底层原理是动态代理技术，主要应用场景是日志记录、用户权限校验等；  
## 术语
