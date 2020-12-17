---
layout: post
title: SpringBoot - 统一Exception处理
tags:
- Java
- SpringBoot
categories: SpringBoot
description: 统一Exception处理
---  
**SpringBoot统一异常处理实践**

<!-- more -->
## 前言
1.当代码中手动或者被动抛出异常时，需要用try-catch语句来处理，大部分异常处理过程是相同的，代码冗余量较高。  
2.同时，SpringBoot默认的全局异常处理机制返回数据中所携带的有用信息较少。  
3.因此，可配置自定义的全局异常处理机制，定制自己需要的异常返回信息。  
4.本文中介绍两种处理方式，二选一使用即可。  
## 一、@ExceptionHandler
使用**@RestControllerAdvice**和**@ExceptionHandler**注解来全局处理异常，使用方式如下：  
```java
@RestControllerAdvice                               //该注解 = @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {
                                                    //BaseResponse是之前设计的统一Response类
    @ExceptionHandler(Exception.class)              //指定要处理的异常类，可以是一个数组
    public BaseResponse<String> ExceptionHandler(Exception e) {
        BaseResponse<String> response = new BaseResponse<>();
        response.setCode(ErrorStatus.INTERNAL_ERROR);
        response.setMessage(e.getMessage());
        response.setData(e.getClass().getName());
        return response;
    }

    @ExceptionHandler(DefaultException.class)       //DefaultException是之前设计的统一Exception类
    public BaseResponse<String> DefaultExceptionHandler(DefaultException e) {
        BaseResponse<String> response = new BaseResponse<>();
        response.setCode(e.getCode());
        response.setMessage(e.getCode().message());
        response.setData(e.getMessage());
        return response;
    }
}
```
**问题：**  
1.@RestControllerAdvice注解中的**属性**可以指定需要接收通知的包(或类)，换句话说就是可以**指定要处理哪些包(或类)**抛出来的异常;  
2.@ExceptionHandler会根据**指定的类**来匹配执行哪个方法，若匹配不到，会走默认异常处理机制那套;  
3.若代码抛出一个DefaultException异常，会匹配到第二个方法；其他任何异常，都会匹配到第一个方法;  
4.若@RestControllerAdvice标注的类和项目启动类**不在同一包下**，需要指定包所在的位置，否则异常处理不会生效;  
```java
@SpringBootApplication(scanBasePackages = {"anchor.mybatis.*","anchor.common.*"})       //GlobalExceptionHandler在common包下
public class SpringBootMybatisApp {
    public static void main(String[] args) {
            // 省略
        }
}
```
或者可以使用@Bean直接注入;
```java
@Configuration
public class GlobalConfig {
    @Bean
    public GlobalExceptionHandler GlobalExceptionHandler(){
        return new GlobalExceptionHandler();
    }
}
```
5.@RestControllerAdvice注解**[原理](https://zhuanlan.zhihu.com/p/73087879#:~:text=%40ControllerAdvice%E6%98%AF%E5%9C%A8%E7%B1%BB%E4%B8%8A,%E5%BC%82%E5%B8%B8%E5%85%A8%E5%B1%80%E5%A4%84%E7%90%86%E7%9A%84%E7%9B%AE%E7%9A%84%EF%BC%9B&text=%40ModelAttribute%E6%B3%A8%E8%A7%A3%E6%A0%87%E6%B3%A8%E7%9A%84%E6%96%B9%E6%B3%95,%E7%9B%AE%E6%A0%87Controller%E6%96%B9%E6%B3%95%E4%B9%8B%E5%89%8D%E6%89%A7%E8%A1%8C%E3%80%82)**;
## 二、ErrorController
