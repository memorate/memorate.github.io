---
layout: post  
title: java-泛型原来是这么回事  
tags: 
- Java
- StudyRecord
- 学习记录  
categories: Java  
description: 关于泛型的个人学习及理解
---
#### 用于记录关于泛型的自我学习及理解成果

<!-- more -->

### *什么是泛型*
　1）**泛型是一种语法糖；**  
　2）**泛型的本质是一种参数化类型，将所操作的数据类型（类类型/引用类型）指定为一个参数，在调用时再传入具体的类型；**  
　3）**泛型可用于类、接口、方法中，分别被称为泛型类、泛型接口、泛型方法；**  
　4）**泛型只在编译阶段有效，在编译过程中正确验证泛型结果后，会将泛型相关信息擦除，并且会在对象进入和离开方法的边界处添加类型检查和类型转换的方法。因此，泛型信息不会进入到运行阶段。**

### *为什么使用泛型*
　1）**类型安全，在编译时进行类型检测，只有指定类型才可添加到集合/类中；**   
　2）**泛型将运行时的类型转换问题提前到了编译期，避免了强制类型转换；**   
```java
   List list = new ArrayList();
   list.add("hello");
   String s = (String) list.get(0);   //强制类型转换
   -------------------------------------------------
   List<String> list = new ArrayList<String>();
   list.add("hello");
   String s = list.get(0);         //无需类型转换
```
　3）**提高代码的可读性和重用率，降低程序的复杂度** 

### 一、泛型类
##### 1.*定义泛型类*
　1）在普通类的类名后加`<T>`来定义该类为泛型类，其中T也可是E、K、V等任意字母。  
　2）T在该泛型类中可作为**属性的类型**、**方法的入参类型**、**方法的返回值类型**。  
```java
public class Demo<T>{           //类名后接"<T>"，T也可是E、K、V等任意字母

    private T value;             //泛型作为属性的类型
     
    public Demo(T value){        //泛型作为方法的入参类型
       this.value = value;
    }
     
    public T getValue(){         //泛型作为方法的返回值类型
       return value;
    }
}
```
```text
  一般通用命名方式：
  T : Type
  E : Element, 元素，Java 集合框架中广泛使用
  K ：Key
  V : Value
  N : Number
```
`注意：`字符串也可用于定义泛型(极度不推荐使用)
```java
public class NotRecommend<String>{          //此处"String"与"T"的作用等同，代表一个参数化类型。然而容易造成误解

    private String value1;                 //此处的"String"代表参数化类型，而不是！而不是！而不是！"java.lang.String"类

    private java.lang.String value2;       //若要使用字符串类只能这样写。此处的"java.lang.String"代表字符串类
}
```
　***3）泛型可声明多个***
```java
public class Demo<T,K,V>{          //一个类中声明多个泛型。类名Demo后接"<T,K,V>"，个数不限

    private T value;
    private K key;

    public Demo(T value){       
        this.value = value;
    }

    public T getValue(){
        return value;
    }

    public V transfer(){
        return (V)key;
    }
}
```
##### 2.*实例化泛型类*
**1）显式指定泛型类的类型：**  
```java
Demo<Integer> integerDemo = new Demo<Integer>(1024);　　　　　//显示指定泛型类integerDemo的参数化类型为Integer  

Demo<String> stringDemo = new Demo<String>("string");　　  　//显示指定stringDemo类的参数化类型为String  
```
    
**2）由编译器推断泛型类的类型：**  
```java
Demo integerDemo = new Demo(1024);            //编译器会推断出integerDemo的参数化类型是Integer
  
Demo stringDemo = new Demo("string");        //隐式指定stringDemo类的参数化类型为String
```
*测试获取泛型类中T的实际类型：*
```java
System.out.println("integerDemo 中value的类型为：" + integerDemo.getValue().getClass().getTypeName());

System.out.println("stringDemo 中value的类型为：" + stringDemo.getValue().getClass().getTypeName());
```
![]({{ "/assets/img/demoTest.jpg" | absolute_url }})

`注意：`  
　　１）泛型的类型参数只能是***类类型（引用类型）***，不能是简单类型。（如：只能是Integer，不能是int）  
```java
       Demo<int> intDemo = new Demo<int>(1024);　　　　　　　　　　 　//编译错误！！！，泛型不能是简单类型  
```
　　２）不能对***确切的泛型类***使用instanceof操作:  
　　　　`stringDemo instanceof Demo<String>`是***非法***的，但`stringDemo instanceof Demo`是***合法***的。

### 二、泛型接口
##### 1.*定义泛型接口*
　*1）接口名后接泛型标识符`<T>`*  
　*2）T可用于接口中方法的返回值和方法的入参类型*
```java
public interface IDemo<T>{       //申明IDemo为泛型接口。接口名后接"<T>"，T也可是E、K、V等任意字母

    public T algorithm(T value);
}
 ```
　***3）泛型可声明多个***
```java
public interface IDemo<T,E>{     //一个接口中声明多个泛型，接口名IDemo后接"<T,E>"，个数不限

    public T algorithm(E value);
}
 ```
##### 2.*使用泛型接口*
 **1）引用的泛型接口未传入实参时**  
 　*需要将泛型的申明也加到类中。`类名`和`接口名`后必须都有泛型标识符`<T>`*  
```java
class SimpleDemo<T> implements IDemo<T>{     //此时的SimpleDemo为泛型类，类名和接口名后必须有泛型标识"<T>"

     @override
     public T algorithm(T value){             //此段代码中出现的四个泛型标识符必须相同（都是T）
         return value;
     }
}
```
**2）引用的泛型接口传入实参时**  
　*在使用泛型接口的类中，需将所有使用到泛型的地方替换成传入的实参类型*
```java
class AnotherDemo implements IDemo<String>{     //实参类型为String

     @override
     public String algorithm(String value){       //实参类型为String
         return value;
     }
}
```
**Tips：**  
　　1）接口可extends多个接口，例：public interface IDemo extends A,B,C{....}  
　　2）类可implements多个接口，例：public class Demo implements A,B,C{....}
		  
### 三、泛型方法
<br/>
　**在方法的权限修饰符和返回值之间使用标识符`<T>`来定义泛型方法。**  
<br/>
**1）在*普通类*中定义泛型方法**  
```java
public class normalDemo{
    public String normalMethod(String value){                         //普通方法
        return value;
    }

    public <T> T genericMethod1(T value){                             //泛型方法，public和String之间有"<T>"
        return value;
    }

    protected <E,K> void genericMethod2(E param, K value){            //泛型方法，泛型可声明多个
        System.out.println(param.toString() + value.toString());
    }

    public static String normalGenericMethod(String value){            //普通静态方法
        return value;
    }

    public static <T> T staticGenericMethod(T value){                 //泛型静态方法，静态方法若要使用泛型，必须将静态方法申明为泛型方法
        return value;
    }
}
```
**2）在*泛型类*中定义泛型方法**  
```java
public class genericDemo<T>{
    public T normalMethod(T value){                 //普通方法，两个T与泛型类中的T相同
        return value;                               //这个方法可以理解为是具有泛型特性（类型参数化）的普通方法，因为它可以使用泛型类的参数化类型
    }

    public <T> T genericMethod1(T value){           //泛型方法，此处T是一种全新的类型，可以与泛型类中的T相同，也可不同。
        return value;                               //不建议在这里使用已使用的字母T，应换一个字母来声明
    }

    public <E,K> void genericMethod2(E param, K value){             //泛型方法，泛型可声明多个;E、K可与T相同，也可不同
        System.out.println(param.toString() + value.toString());
    }

    public static <K> K genericStaticMethod(K value){     //如果静态方法也使用泛型时，该静态方法必须申明为泛型方法，否则编译会报错
        return value;
    }

    public static T illegalStaticMethod(T value){         //缺少泛型标识符<T>，编译报错！！！
        return value;
    }
}
```
`注意：`  
　1）只有方法的访问修饰符（public等）与返回值之间有泛型类标识符<T\>时该方法才是泛型方法（T可为E、K、V等），否则只是个普通方法。  
　2）泛型类，是在实例化类的时候指明泛型的具体类型；泛型方法，是在调用方法的时候指明泛型的具体类型。  
　3）泛型类标识符<T\>中的参数化类型T的作用域是整个类（类中任意地方可用T），泛型方法标识符<E\>中的类型E仅能用于这个方法。  
　**例：**`public class genericDemo<T>{}`，T可用于整个genericDemo类；`public <T> T genericMethod1(){}`，T只能用于genericMethod1方法。

### 四、泛型通配符
```java
public void algorithm(List<?> value){                 //无边界泛型通配符"<?>"
    System.out.println(valus.toString());
}
    
public void algorithm(List<? extends Demo> value){    //固定上边界泛型通配符"<? extends Demo>"
    System.out.println(valus.toString());
}
 
public void algorithm(List<? super Demo> value){      //固定下边界泛型通配符"<? super Demo>"
    System.out.println(valus.toString());
}
```
Tips：一般用于方法的入参，当具体类型不确定时，可使用泛型通配符"?"；使用泛型通配符后无法使用类的具体功能，只能使用Object类中的功能。
 
注意：1）无边界泛型通配符"?"是类型实参，是一种真实的类型，可以看做所有类的父类，而不是类型形参。  
　　　2）不能同时申明泛型通配符的上界和下界。  
　　　3）<? extends Demo>代表Demo及其子类，<? super Demo>代表Demo及其父类。

原则：1）PECS（Producer extends，Consumer super）即生产者使用extends，消费者使用super。  
　　　2）需要读取数据以使用时，使用extends，固定上边界，将对象当做一个只读对象。  
　　　3）需要将数据写入对象时，使用super，固定下边界，将对象当做一个只写对象。  
　　　4）需要既写又读时，请勿使用泛型通配符。 　