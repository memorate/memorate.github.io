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
使用**@ControllerAdvice**和**@ExceptionHandler**注解来全局处理异常，使用方式如下：  
```java
@ControllerAdvice
public class BasicExceptionHandler {

    @ResponseBody
    @ExceptionHandler(value = Exception.class)          //指定要处理的异常类，value可以接收一个数组
    public BaseResponse<String> ExceptionHandler(Exception e) {
        BaseResponse<String> response = new BaseResponse<>();
        response.setCode(ErrorStatus.INTERNAL_ERROR);
        response.setMessage(e.getMessage());
        response.setData(e.getClass().getName());
        return response;
    }

    @ResponseBody
    @ExceptionHandler(value = DefaultException.class)    //DefaultException为之前设计的统一异常类
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
1.@ControllerAdvice注解中的属性可以指定需要接收通知的包(或类)，换句话说就是可以指定要处理哪些包(或类)抛出来的异常;  
2.@ExceptionHandler会根据value属性指定的类来匹配执行哪个方法，若匹配不到，会走默认异常处理机制那套;  
3.若自定义的BasicExceptionHandler类和项目启动类(main入口)不在同一包下，需要指定包所在的位置，否则异常处理不会生效;  
```java
@SpringBootApplication(scanBasePackages = {"anchor.mybatis.*","anchor.common.*"})       //BasicExceptionHandler在common包下
public class SpringBootMybatisApp {
    public static void main(String[] args) {
            SpringApplication.run(SpringBootMybatisApp.class, args);
            log.info("Anchor-Mybatis Researcher 启动成功...");
        }
}
```
或者可以使用@Bean直接注入;
```java
@Configuration
public class MybatisConfig {
    @Bean
    public BasicExceptionHandler basicExceptionHandler(){
        return new BasicExceptionHandler();
    }
}
```
## 二、ErrorController