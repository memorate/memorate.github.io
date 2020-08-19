---
layout: post
title: Java - 统一Response设计
tags:
- Java
- Tips
categories: Java
description: 统一Response设计
---  
**Java接口的Response设计**

<!-- more -->
## 前言
1.Response用于告诉接口调用者本次调用的结果。  
2.Response的三要素：**响应码**、**返回信息**、**返回数据**。  
3.对Response类的要求：  
　a.通用性，可满足所有Controller的返回需求;  
　b.可扩展性，不同的业务所需要的返回码不同，因此要易扩展、可分类;  
　c.易用性，简洁、便于使用;  
## 一、StatusCode
1.StatusCode接口定义了状态相关的规范，包含两个方法，分别用于返回响应码和返回信息。  
2.后续所有的响应码类实现StatusCode即可。  
```java
public interface StatusCode extends Serializable {
    @JsonValue       //序列化时只显示code，message以BaseResponse中的为准
    int code();

    String message();
}
```
## 二、DefaultStatus
1.DefaultStatus是一个默认的响应码枚举类，仅仅定义了四种简单的响应码。  
2.后续其他业务的状态码类模仿该类实现即可。
```java
public enum DefaultStatus implements StatusCode {

    SUCCESS(200, "Operation Success"),
    PARAM_ERROR(400, "Invalid parameters"),
    NOT_FOUND(404, "Resource not found"),
    FAILURE(500, "Application internal error");

    private final int code;
    private final String message;

    DefaultStatus(int code, String message) {
        this.code = code;
        this.message = message;
    }

    @Override
    public int code() {
        return code;
    }

    @Override
    public String message() {
        return message;
    }

    @Override
    public String toString() {
        return String.valueOf(this.code);
    }
}
```
## 三、BaseResponse
**1**.BaseResponse是用于返回调用结果的类，不可被继承，所有Controller中方法的返回值统一使用此类。  
**2**.BaseResponse有三个属性：**code**(响应码)、**message**(返回信息)、**data**(返回数据)。  
**3**.为什么StatusCode中已经有message()，BaseResponse还要设置message属性？  
　例：DefaultStatus中400携带的信息是"Invalid parameters"，然而有些业务场景中400需要描述为"参数不合法，请检查！"。
这时在BaseResponse中复写message即可，便于灵活交互。  
**4**.提供4个静态with方法，用于灵活实例化BaseResponse。  
```java
public final class BaseResponse<T> implements Serializable {

    private static final long serialVersionUID = 3886133510113334083L;

    private StatusCode code;
    private String message;
    private T data;

    //无参构造方法中将响应码置为DefaultStatus中的SUCCESS
    public BaseResponse() {
        this.setCode(SUCCESS);
        this.message = SUCCESS.message();
    }

    public BaseResponse(T data) {
        this();
        this.data = data;
    }

    public StatusCode getCode() {
        return code;
    }

    public BaseResponse<T> setCode(StatusCode code) {
        this.code = code;
        this.message = code.message();
        return this;
    }

    public String getMessage() {
        return message;
    }

    public BaseResponse<T> setMessage(String message) {
        this.message = message;
        return this;
    }

    public T getData() {
        return data;
    }

    public BaseResponse<T> setData(T data) {
        this.data = data;
        return this;
    }

    public static <T> BaseResponse<T> with(StatusCode code) {
        BaseResponse<T> response = new BaseResponse<>();
        response.code = code;
        response.message = code.message();
        return response;
    }

    public static <T> BaseResponse<T> with(StatusCode code, String message) {
        BaseResponse<T> response = new BaseResponse<>();
        response.code = code;
        response.message = message;
        return response;
    }

    public static <T> BaseResponse<T> with(StatusCode code, T data) {
        BaseResponse<T> response = new BaseResponse<>();
        response.code = code;
        response.message = code.message();
        response.data = data;
        return response;
    }

    public static <T> BaseResponse<T> with(StatusCode code, String message, T data) {
        BaseResponse<T> response = new BaseResponse<>();
        response.code = code;
        response.message = message;
        response.data = data;
        return response;
    }

    @Override
    public String toString() {
        return "BaseResponse{" +
                "code=" + code.code() +
                ", message='" + message + '\'' +
                ", data=" + data +
                '}';
    }
}
```
## 四、使用
```java
@GetMapping("/api1")
public BaseResponse api1(@RequestParam String something) throws Exception {
    //code
    return new BaseResponse();
}
```
```java
@GetMapping("/api2")
public BaseResponse<String> api2(@RequestParam String something) throws Exception {
    //code
    return new BaseResponse(something);
}
```
```java
@GetMapping("/api3")
public BaseResponse<String> api3(@RequestParam String something) throws Exception {
    //code
    return BaseResponse.with(DefaultStatus.SUCCESS, "操作成功", something);
}
```
## 五、问题
**1.为什么不把所有的响应码都放在DefaultStatus类中？**  
　目前我所在的项目中有400+个响应码，都放在同一个类中过于冗长，不便管理，且响应码分类不够明确。  
**2.为什么不使用Spring框架自带的HttpStatus类？**  
　该类是Http的请求状态码，并非业务所需的响应码，且无法对此类进行扩充。  