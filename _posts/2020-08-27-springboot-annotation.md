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
#### 1.注释与注解  
```
注释是给程序员看的，用于给其它程序员传递被注释代码的相关信息，会被编译器忽略，不会影响代码逻辑。
注解是给编译器、JVM看的，可以被编译器打包进class文件，被JVM保存，JVM在运行时可以获取到注解内容，配合其它‘工具’使用时可以影响代码逻辑。
```
#### 2.注解的理解
```
在SpringBoot中，SpringBoot自带的注解大大简化了配置的过程、节省了代码工作量。
自定义注解一般配合AOP使用。注解给被标注对象打一个标记，AOP根据这个标记找到需要切入的点，然后实现自定义的业务逻辑。
```
#### 3.注解的定义
使用**@interface**自定义注解。  
```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
    String value() default "hello";
}
```
Test是此注解的名称，@Retention是修饰注解的**元注解**，value是注解的**参数**(**默认值**是hello)。
## 二、元注解
**Java中定义了四个元注解，用来修饰其他注解。**
#### 1.@Retention
指明注解怎么保存；描述注解的声明周期(即被标注的注解在什么范围内有效)。有三种可选，可选值在枚举类**RetentionPolicy**中。
```java
public enum RetentionPolicy {
    SOURCE,    //注解只保存在源文件中，在编译时会被编译器忽略
    CLASS,     //注解保存到Class文件中，但在JVM加载Class文件时遗弃
    RUNTIME    //注解保存到Class文件中，并且会被JVM加载
}
```
#### 2.@Target
指明注解可以作用于哪些对象。其参数可填单个值或数组，有十种可选，可选值在枚举类**ElementType**中。  
`PS:`**不使用**@Target时**默认**注解可作用于任何地方。  
```java
public enum ElementType {
    TYPE,                //类、接口、注解、枚举类
    FIELD,               //字段，包括枚举值
    METHOD,              //方法
    PARAMETER,           //方法的参数
    CONSTRUCTOR,         //构造方法
    LOCAL_VARIABLE,      //局部变量
    ANNOTATION_TYPE,     //注解
    PACKAGE,             //包
    TYPE_PARAMETER,      //Java1.8支持，泛型
    TYPE_USE             //Java1.8支持，任何地方
}
```
#### 3.@Inherited
指明某注解A所标注类的子类也会被A注解标注(**换而言之注解可以继承给子类**)。接口不受影响(实现接口的类也不会受此注解影响)。
#### 4.@Documented
指明生成Javadoc时是此注解否会被记录，没有实际作用。  
#### 5.内置注解
Java中除了元注解，还自带了以下几个内置的注解，可直接使用：  
- @Override，检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。  
- @Deprecated，标记过时方法。如果使用该方法，会报编译警告。  
- @SuppressWarnings，指示编译器去忽略注解中声明的警告。  
- @SafeVarargs，Java7开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。  
- @FunctionalInterface，Java8开始支持，标识一个匿名函数或函数式接口。  
- @Repeatable，Java8开始支持，标识某注解可以在同一个声明上使用多次。
## 三、参数
在注解中可以定义参数及其默认值，参数类型可以是以下六种：
```text
1.所有基本类型       2.String             3.enum
4.Class             5.annotation         6.以上五种类型的数组
```
```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Test {
    int intValue() default 1;                             //基本数据类型
    String value();                                       //String
    Class classValue() default Object.class;              //class
    ElementType element_type() default ElementType.TYPE;  //enum
    Retention target();                                   //annotation
    long[] array() default {1L, 2L};                      //数组
}
```
**注意：**  
1.()之前的是参数名；使用注解时没有默认值的参数必须赋值，key-value形式赋值，例：@Test(intValue = 2)。  
2.如果注解中存在名为value的参数，且其他参数都有默认值，那么使用该注解时可以省略key，例：@Test("aa")。  
## 四、实践
本次实践内容：  
使用自定义注解记录请求相关信息，将请求人uid、uname、请求的资源、执行的方法、返回码、返回信息、请求时间记录入库。  
#### 1.定义
定义注解@ResultRecorder，该注解作用于Controller类，需要传入的参数是此Controller所操作的资源名。
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface ResultRecorder {
    //RestFul中所操作的资源名，例：用户、订单、地址
    String value();
}
```
#### 2.实现
```java
public class OperationLog {          //操作日志entity
    private String uid;
    private String uname;
    private String resource;           //注解@ResultRecorder中的参数值
    private String method;             //方法名，例：xxxController.xxxMethod()
    private Integer code;              //方法返回状态码
    private String message;            //方法返回的信息
    private LocalDateTime time;        //请求开始时间
}
```
```java
public class CommonPointcut {         //切点类
    @Pointcut("@within(ResultRecorder)")
    public void resultRecorder(){}
}
```
```java
@Aspect
@Component
public class ResultRecorderAspect {    //切面类
    @Resouce
    private OpeartionLogMapper mapper;
    
    @Around("CommonPointcut.resultRecorder()")
    public Object resultRecord(ProceedingJoinPoint joinPoint) {
        SysUser user = SystemUserHolder.getCurrent();       //获取用户信息，需自己实现
        OperationLog operation = new OperationLog()
                .setTime(LocalDateTime.now())               //记录请求进入的时间
                .setUid(user.getId)
                .setUname(user.getName);
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();    //获取方法签名
        operation.setMethod(signature.toShortString());     //shortString格式为：xxxController.xxxMethod(..)
        ResultRecorder annotation = joinPoint.getTarget().getClass().getAnnotation(ResultRecorder.class);  //获取注解
        operation.setResource(annotation.value());          //获取注解中参数value的值
        try {
            Object result = joinPoint.proceed();            //执行Aspect之后的流程
            if (result instanceof BaseResponse) {           //项目所有Controller中方法的返回值都是之前定义的统一返回类
                BaseResponse response = (BaseResponse) result;
                operation.setCode(response.getCode().code()); //提取BaseResponse中的code、message
                operation.setMessage(response.getMessage());
            } else {
                operation.setCode(500);
                operation.setMessage("Can't recognize the method's response.");
            }
            return result;
        } catch (Throwable throwable) {   
            operation.setCode(500);
            operation.setMessage(throwable.getMessage());
            throw throwable;                                 //异常要抛出给下一流程
        } finally {
            mapper.save(operation);                          //入库
            System.out.println(operation);                   //打印入库的数据
        }
    }
}
```
`PS:`实际项目中请勿用System.out.println来打印数据！
#### 3.结果
在CommonController类上做实验：
```java
@ResultRecorder("公共资源")           //自定义注解
@RestController
@RequestMapping("/common")
public class CommonController {
    @GetMapping
    public BaseResponse<Boolean> annotationTest(){
        System.out.println("Executing annotationTest()...");
        return new BaseResponse<>(true);
    }
}
```
`PS:`实际项目中请勿用System.out.println来打印数据！  
**实验结果：**
![]({{ "/assets/img/20200827/20200827001.png"}})
## 五、总结
1.注解(@interface)由元注解和注解参数构成，使用注解时若某个参数无默认值，则必须给它赋值。  
2.注解本身并不能影响代码逻辑，需要配合AOP使用。  
3.一般而言项目开发中基本没有自定义注解的需求，常用的注解是Spring自带的各类注解。