---
layout: post
title: SpringBoot - 如何发送Http请求
tags:
- Java 
- SpringBoot 
- Tips
categories: Java
description: 使用RestTemplate发送Http请求
---  
**SpringBoot发送Http请求 — RestTemplate的使用**

<!-- more -->
## 一、引入
本文中代码用于请求[手机号归属地](https://github.com/MZCretin/RollToolsApi#%E6%89%8B%E6%9C%BA%E5%8F%B7%E7%A0%81%E5%BD%92%E5%B1%9E%E5%9C%B0%E6%9F%A5%E8%AF%A2)
和[生成带logo二维码](https://github.com/MZCretin/RollToolsApi#%E5%85%AB%E7%94%9F%E6%88%90%E4%BA%8C%E7%BB%B4%E7%A0%81)（实操皆已成功）。
```text
import org.springframework.web.client.RestTemplate;
```
## 二、介绍
**①RestTemplate**类是Spring3.0版本后自带的发送Http请求的客户端，使用简单、便捷。    
**②**RestTemplate有常用的**Get**、**Post**方法，也有**Put**、**Delete**方法，还有通用的方法**execute**和**exchange**（可以发送所有类型Http请求）。  
**③getForObject()**与**getForEntity()**的区别：  
　**-** getForObject()将Get请求的response序列化为一个**Object**（类型可以指定）并返回；  
　**-** getForEntity()将response序列化为**ResponseEntity\<T>**类并返回（T是getForObject()返回的Object）；  
　**-** ***getForEntity()实际上是将getForObject()的返回值进行二次加工，包装为ResponseEntity类***；  
**④postForObject()**与**postForEntity()**的区别：  
　**-** postForObject()将Post请求的response序列化为一个**Object**（类型可以指定）并返回；  
　**-** postForEntity()将response序列化为**ResponseEntity\<T>**类并返回（T是getForObject()返回的Object）；  
　**-** ***postForEntity()实际上是将postForObject()的返回值进行二次加工，包装为ResponseEntity类***；  
**⑤execute()**与**exchange()**的区别：  
　**-** execute()将Http请求的response序列化为一个**Object**（类型可以指定）并返回；  
　**-** exchange()将response序列化为**ResponseEntity\<T>**类并返回（execute()返回的Object）；  
　**-** ***exchange()实际上是将execute()的返回值进行二次加工，包装为ResponseEntity类***；  
## 三、Get
#### 1.普通Get
必须在url中使用占位符"{}"来声明请求参数。
```java
<T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException;
```
1）**将Get请求的Response序列化为String**
```java
public String getForString(String mobile) {
    String url = MXN_URL + "/mobile_location/aim_mobile?mobile={mobile}&app_id={app_id}&app_secret={app_secret}";
    Map<String, String> params = new HashMap<>();
    params.put("mobile", mobile);
    params.put("app_id", MXN_APP_ID);
    params.put("app_secret", MXN_APP_SECRET);
    return restTemplate.getForObject(url, String.class, params);    //将请求的response序列化为String类
}
```
2）**将Get请求的Response序列化为指定类**
```java
public MobileResponse getForResponse(String mobile) {
    String url = MXN_URL + "/mobile_location/aim_mobile?mobile={mobile}&app_id={app_id}&app_secret={app_secret}";
    Map<String, String> params = new HashMap<>();
    params.put("mobile", mobile);
    params.put("app_id", MXN_APP_ID);
    params.put("app_secret", MXN_APP_SECRET);
    return restTemplate.getForObject(url, MobileResponse.class, params);    //将请求的response序列化为MobileResponse类
}
```
#### 2.带Header的Get
## 四、Post
#### 1.普通Post
#### 2.带Header的Post