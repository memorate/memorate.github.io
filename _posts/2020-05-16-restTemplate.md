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
**①**本文中代码用于测试**Get请求([手机号归属地](https://github.com/MZCretin/RollToolsApi#%E6%89%8B%E6%9C%BA%E5%8F%B7%E7%A0%81%E5%BD%92%E5%B1%9E%E5%9C%B0%E6%9F%A5%E8%AF%A2))**
和**Post请求([生成带logo二维码](https://github.com/MZCretin/RollToolsApi#%E5%85%AB%E7%94%9F%E6%88%90%E4%BA%8C%E7%BB%B4%E7%A0%81))**，
且实操皆已成功([源码](https://github.com/memorate/SpringBootResearch/blob/master/SpringBootMybatis/src/main/java/anchor/mybatis/service/CommonService.java))。  
**②**本文采用IOC方式使用RestTemplate类，当然使用时new一个RestTemplate类也可正常工作。
```java
import org.springframework.web.client.RestTemplate;

@Bean
public RestTemplate restTemplate(){     //在SpringBoot启动类中注册RestTemplate
    return new RestTemplate();
}
```
```java
@Resource
private RestTemplate restTemplate;      //使用时注入RestTemplate类
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
**⑥**RestTemplate中各类发送请求的方法**最终都是执行execute()**。它们只不过是对execute()的各类包装。
## 三、Get请求
注意，Get请求的请求参数必须先在URL中用占位符"{}"声明，调用方法时才能识别并传入。
#### 1.getForObject
```java
<T> T getForObject(String url, Class<T> responseType, Map<String, ?> uriVariables) throws RestClientException;
```
1）**将Get请求的Response序列化为String**
```java
public String getForString(String mobile) {
    String url = MXN_HOST + "/mobile_location/aim_mobile?mobile={mobile}&app_id={app_id}&app_secret={app_secret}";    //声明请求参数
    Map<String, String> params = new HashMap<>();
    params.put("mobile", mobile);
    params.put("app_id", MXN_APP_ID);
    params.put("app_secret", MXN_APP_SECRET);
    return restTemplate.getForObject(url, String.class, params);    //将请求的response序列化为String类
}
```
2）**将Get请求的Response序列化为指定类**
```java
@Data
public class MXNResponse {
    private int code;
    private String msg;
}
```
```java
@Data
public class MobileResponse extends MXNResponse{
    private Data data;

    @lombok.Data
    static class Data {
        private String mobile;
        private String province;
        private String carrier;
    }
}
```
```java
public MobileResponse getForResponse(String mobile) {
    String url = MXN_HOST + "/mobile_location/aim_mobile?mobile={mobile}&app_id={app_id}&app_secret={app_secret}";
    Map<String, String> params = new HashMap<>();
    params.put("mobile", mobile);
    params.put("app_id", MXN_APP_ID);
    params.put("app_secret", MXN_APP_SECRET);
    return restTemplate.getForObject(url, MobileResponse.class, params);    //将请求的response序列化为MobileResponse类
}
```
#### 2.getForEntity
```java
public ResponseEntity<MobileResponse> getForEntity(String mobile) {
    String url = MXN_HOST + "/mobile_location/aim_mobile?mobile={mobile}&app_id={app_id}&app_secret={app_secret}";
    Map<String, String> params = new HashMap<>();
    params.put("mobile", mobile);
    params.put("app_id", MXN_APP_ID);
    params.put("app_secret", MXN_APP_SECRET);
    return restTemplate.getForEntity(url, MobileResponse.class, params);
}
```
#### 3.带Header的Get
HttpEntity类有两个属性：headers和body，body是Post发送的body，因此只能将Get的请求参数写入URL中。
```java
public ResponseEntity<MobileResponse> getForEntityWithHeader(String mobile) {
    String url = MXN_HOST + "/mobile_location/aim_mobile?mobile=" + mobile;      //请求参数写入URL
    HttpHeaders headers = new HttpHeaders();
    headers.add("app_id", MXN_APP_ID);
    headers.add("app_secret", MXN_APP_SECRET);
    HttpEntity httpEntity = new HttpEntity(headers);
    return restTemplate.exchange(url, HttpMethod.GET, httpEntity, MobileResponse.class);
}
```
## 四、Post请求
RestTemplate中Post用法与Get一致，简单的用法不再赘述，只写一个终极Post。
#### 终极Post
此Post请求需携带Header，且需要Post一张图片至服务器，最终将Response序列化为ResponseEntity\<T>。
```java
@Data
public class QRCodeResponse extends MXNResponse{
    private Data data;

    @lombok.Data
    static class Data {
        private String qrCodeUrl;
        private String content;
        private int type;
        private String qrCodeBase64;
    }
}
```
```java
public ResponseEntity<QRCodeResponse> postForEntityWithHeader(String content) {
    String url = MXN_HOST + "qrcode/create/logo";
    FileSystemResource resource = new FileSystemResource(new File("D:/文件/壁纸/cat.png"));
    MultiValueMap<String, Object> params = new LinkedMultiValueMap<>();      //只能是MultiValueMap，不能是Map
    params.add("content", content);
    params.add("size", 400);
    params.add("logo_size", 120);
    params.add("type", 0);
    params.add("logo_img", resource);
    HttpHeaders headers = new HttpHeaders();
    headers.add("app_id", MXN_APP_ID);
    headers.add("app_secret", MXN_APP_SECRET);
    HttpEntity httpEntity = new HttpEntity(params, headers);      //将Post的body和header塞进HttpEntity
    return restTemplate.postForEntity(url, httpEntity, QRCodeResponse.class);
}
```
## 五、问题
1.将请求的Response序列化为指定类时，若因指定类中属性不符合驼峰命名而导致序列化失败，可在getter方法上使用@JsonProperty注解来映射。  