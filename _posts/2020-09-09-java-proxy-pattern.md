---
layout: post
title: Java - 代理模式
tags: 
- Java
- 设计模式
categories: Java
description: Java Proxy Pattern
---  
**Java 代理模式解析**

<!-- more -->
## 一、代理模式
#### 1、定义
```text
代理模式是一种结构性模式。
代理模式给某个对象(用户)提供一个代理对象，通过代理对象来控制对原对象的访问，并允许在将请求交给对象前后进行一些处理。
```
用户通过中介租房的整个流程就是一个标准的代理模式。
#### 2、三要素
- **服务接口**：是一个**接口**，其中声明了服务。代理类只有实现了此接口才能伪装成服务对象。  
- **服务**：是一个**类**，它实现了服务接口，用于为用户提供服务。  
- **代理**：是一个**类**，它需要实现服务接口，是对服务类的包装。  

在租房流程中：出租房屋这个动作是服务接口；房东可以看作是服务类，房东出租房屋就是服务类实现了服务接口；中介即代理。
#### 3、使用场景
## 二、静态代理
#### 1、普通模式
```java
public interface Rent {         //服务接口
    void rentHouse();
}
```
```java
public class NormalRentService implements Rent {           //服务
    @Override
    public void rentHouse() {
        System.out.println("Rent a normal house to user...");
    }
}
```
```java
public class RentProxy implements Rent{                    //代理

    private Rent rent;

    public RentProxy(Rent rent) {
        this.rent = rent;
    }

    @Override
    public void rentHouse() {
        System.out.println("I'm a rent proxy,i am looking for a suitable house for user...");
        rent.rentHouse();
        System.out.println("Rent successfully,i earned one billion...");
    }
}
```
#### 2、指定代理
## 三、动态代理
#### 1、JDK动态代理
#### 2、CGLIB动态代理
## 四、总结