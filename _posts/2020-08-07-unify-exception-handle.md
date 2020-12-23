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
2.同时，SpringBoot默认的全局异常处理机制返回数据中所携带的有用信息较少，接口调用者无法准确判断异常原因。  
3.因此，可配置**自定义**的全局异常处理机制，定制自己理想的异常返回信息，从而告诉接口调用者异常原因。  
4.本文中介绍**两种**统一Exception处理方式，二选一使用即可。  
## 一、@ExceptionHandler
使用**@RestControllerAdvice**和**@ExceptionHandler**注解来全局处理异常，使用方式如下：  
```java
@Slf4j
@RestControllerAdvice                            //该注解 = @ControllerAdvice + @ResponseBody
public class GlobalExceptionHandler {
                                                 //BaseResponse是之前文章中设计的统一Response类
    @ExceptionHandler(Exception.class)           //指定要处理的异常类，可以是一个数组
    public BaseResponse<String> ExceptionHandler(Exception e) {
        log.error(e.getMessage(), e);            //手动打印堆栈信息
        BaseResponse<String> response = BaseResponse.with(DefaultStatus.INTERNAL_ERROR);
        if (!StringUtils.isEmpty(e.getMessage())) {
            response.setMessage(e.getMessage());
        }
        response.setData("朋友，我们好像碰到了点麻烦！");
        return response;
    }

    @ExceptionHandler(DefaultException.class)     //DefaultException是之前文章中设计的统一Exception类
    public BaseResponse<String> DefaultExceptionHandler(DefaultException e) {
        log.error(e.getMessage(), e);
        BaseResponse<String> response = BaseResponse.with(e.getCode());
        response.setData(e.getMessage());
        return response;
    }
}
```
**问题：**  
1.@RestControllerAdvice注解中的**属性**可以指定需要接收通知的包(或类)，换句话说就是可以**指定要处理哪些包(或类)**抛出来的异常;  
2.@ExceptionHandler会根据**指定的**`xxxException.class`来匹配执行哪个方法，若匹配不到，会走默认异常处理机制那套(BasicErrorController);  
3.若代码抛出一个**DefaultException异常**，会匹配到第二个方法；其他任何异常，都会匹配到第一个方法;  
4.若@RestControllerAdvice标注的类和项目启动类**不在同一包下**，需要指定包所在的位置，否则异常处理不会生效;  
```java
@SpringBootApplication(scanBasePackages = {"anchor.mybatis.*","anchor.common.*"})       //GlobalExceptionHandler在common包下
public class SpringBootMybatisApp {
    public static void main(String[] args) {
            // 省略
        }
}
```
或者可以使用@Bean直接注入(**推荐使用**);
```java
@Configuration
public class GlobalConfig {
    @Bean
    public GlobalExceptionHandler GlobalExceptionHandler(){
        return new GlobalExceptionHandler();
    }
}
```
5.@ExceptionHandler处理异常后不会在日志中打印堆栈信息，因此需要手动打印，便于排查问题;  
```java
log.error(e.getMessage(), e);
```
6.@RestControllerAdvice注解**[原理](https://zhuanlan.zhihu.com/p/73087879#:~:text=%40ControllerAdvice%E6%98%AF%E5%9C%A8%E7%B1%BB%E4%B8%8A,%E5%BC%82%E5%B8%B8%E5%85%A8%E5%B1%80%E5%A4%84%E7%90%86%E7%9A%84%E7%9B%AE%E7%9A%84%EF%BC%9B&text=%40ModelAttribute%E6%B3%A8%E8%A7%A3%E6%A0%87%E6%B3%A8%E7%9A%84%E6%96%B9%E6%B3%95,%E7%9B%AE%E6%A0%87Controller%E6%96%B9%E6%B3%95%E4%B9%8B%E5%89%8D%E6%89%A7%E8%A1%8C%E3%80%82)**;
## 二、ErrorController
A.实现**ErrorController**接口来覆盖SpringBoot默认的异常处理类**BasicErrorController**。  
B.由于下文的GlobalExceptionController类与应用启动类不在同一个包下，因此采用上文中GlobalConfig的方式注入此类。  
C.有两种获取异常信息的方法（处理HttpServletRequest的方法）：  
**1.ServletWebRequest**  
```java
@RestController
public class GlobalExceptionController implements ErrorController {

    private final static String ERROR_PATH = "/error";

    @RequestMapping(path = ERROR_PATH)
    public BaseResponse<String> error(HttpServletRequest request, HttpServletResponse response) {
        ServletWebRequest webRequest = new ServletWebRequest(request);
        //处理webRequest，获取异常相关的信息
        Exception exception = (Exception) request.getAttribute(RequestDispatcher.ERROR_EXCEPTION);
        Integer code = (Integer) request.getAttribute(RequestDispatcher.ERROR_STATUS_CODE);
        String message = (String) request.getAttribute(RequestDispatcher.ERROR_MESSAGE);
        String type = (String) request.getAttribute(RequestDispatcher.ERROR_EXCEPTION_TYPE);
        BaseResponse<String> baseResponse = new BaseResponse<>();     //未处理获取到的code、message等信息
        return baseResponse;
    }

    @Override
    public String getErrorPath() {
        return ERROR_PATH;
    }
}
```
```java
@Configuration
public class GlobalConfig {
    @Bean
    public GlobalExceptionController globalExceptionController() {
        return new GlobalExceptionController();
    }
}
```
**2.DefaultErrorAttributes**  
```java
@RestController
public class GlobalExceptionController implements ErrorController {

    private final static String ERROR_PATH = "/error";

    private final DefaultErrorAttributes attributes;

    //通过构造方法传入DefaultErrorAttributes
    public GlobalExceptionController(DefaultErrorAttributes attributes) {
        this.attributes = attributes;
    }

    @RequestMapping(path = ERROR_PATH)
    public BaseResponse<String> error(HttpServletRequest request, HttpServletResponse response) {
        ServletWebRequest webRequest = new ServletWebRequest(request);
        //Throwable error = attributes.getError(webRequest);      可以这样获取到异常对象，来自己处理
        //或者这样获取到timestamp、status、error、message、path这五个属性
        Map<String, Object> map = attributes.getErrorAttributes(webRequest, ErrorAttributeOptions.defaults());
        BaseResponse<String> baseResponse = new BaseResponse<>();     //未处理获取到的code、message等信息
        return baseResponse;
    }

    @Override
    public String getErrorPath() {
        return ERROR_PATH;
    }
}
```
```java
@Configuration
public class GlobalConfig {
    @Bean
    public GlobalExceptionController globalExceptionController(DefaultErrorAttributes attributes) {
        return new GlobalExceptionController(attributes);
    }
}
```
## 三、总结
1.**建议使用第一种方式**;  
2.经测试，应用启动后产生的**所有**的异常，两种方式**都能**接收并处理;  
3.**对比:**  
　A.第一种处理方式更简便、目的性更强(指定异常进入指定方法)、自由度更高，可以对不同的异常(比如空指针、越界等)进行定制化处理;  
　B.第二种方式相较于第一种方式，需要多一层处理才能够获取到异常，且获取到的自定义异常和非自定义异常信息是不同的;  
4.若自定义处理不生效，大概率是Bean未成功注入，类与启动类不在同一包下;  