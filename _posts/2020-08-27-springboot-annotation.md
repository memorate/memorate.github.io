---
layout: post
title: SpringBoot - 自定义注解
tags: 
- Java
- SpringBoot
categories: SpringBoot
description: Java - Annotation
---  
**SpringBoot 如何自定义一个注解**

<!-- more -->
## 一、注解
在Java中注解是一种用来标注类、方法、类属性、参数的元数据(metadata，描述数据的数据)，始自Java1.5版本。  
**1.注释与注解**  
```
注释是给程序员看的，用于给其它程序员传递被注释代码的相关信息，会被编译器忽略，不会影响代码逻辑。
注解是给编译器、JVM看的，可以被编译器打包进class文件，被JVM保存，JVM在运行时可以获取到注解内容，配合其它‘工具’使用时可以影响代码逻辑。
```
**2.关于注解的理解**  
```
在SpringBoot中，SpringBoot自带的注解大大简化了配置的过程、节省了代码工作量。
自定义注解一般配合AOP使用。注解给被标注对象打一个标记，AOP根据这个标记找到需要切入的点，然后实现自定义的业务逻辑。
```
## 二、元注解
**Java中定义了四个元注解，用来修饰其他注解。**
#### 1.@Retention
表名被标注的注解怎么保存，描述此注解的声明周期(即被标注的注解在什么范围内有效)。有三种可选，可选值在枚举类**RetentionPolicy**中。
```java
public enum RetentionPolicy {
    SOURCE,    //被标注的注解只保存在源文件中，在编译时会被编译器忽略

    CLASS,     //

    RUNTIME    //
}
```
#### 2.@Target
#### 3.@Inherited
#### 4.@Documented
- @Retention，**标注此注解怎么保存**。有三种可选，可选值在枚举类**RetentionPolicy**中。
- @Documented，**标注生成Javadoc时是此注解否会被记录**，没有实际作用。
- @Target，**标注此注解可以作用于哪些对象**，可填一个数组。有十种可选，可选值在枚举类**ElementType**中。
- @Inherited，标注被此注解标注的类的子类也会被此注解标注(**此注解可以继承给子类**)，接口不受影响(实现接口的类不会受此注解影响)。  
`PS:`RetentionPolicy类中的三种类型：SOURCE(只保存在代码中)、CLASS(编入Class文件中)、RUNTIME(编入Class文件且保存在JVM中)。  
  
**2.Java自带注解**  
- @Override，检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。  
- @Deprecated，标记过时方法。如果使用该方法，会报编译警告。  
- @SuppressWarnings，指示编译器去忽略注解中声明的警告。  
- @SafeVarargs，Java7开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。  
- @FunctionalInterface，Java8开始支持，标识一个匿名函数或函数式接口。  
- @Repeatable，Java8开始支持，标识某注解可以在同一个声明上使用多次。
## 三、参数
## 四、实践
本次实践内容：使用自定义注解记录请求相关信息，将请求人uid、uname、请求的资源、执行的方法、返回码、返回信息、请求时间记录入库。  
#### 1.定义
定义注解@ResultRecorder，该注解作用于Controller类，需要传入的参数是此Controller所操作的资源名。  
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ResultRecorder {
    /**
     * RestFul中所操作的资源名，例：用户、订单、地址
     */
    String value();
}
```
#### 2.处理
#### 3.结果
## 五、总结