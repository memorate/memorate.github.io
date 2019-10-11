---
layout: post
title: java-泛型的个人理解
tags: 
- java
- 学习记录
categories: java
description: 关于泛型的个人学习及理解
---
#### 用于记录关于泛型的自我学习及理解成果

<!-- more -->

### 泛型：
　1）**泛型是一种语法糖**  
　2）**泛型的本质是一种参数化类型，将所操作的数据类型指定为一个参数，在调用时再传入具体的类型**  
　3）**泛型可用于类、接口、方法中，分别被称为泛型类、泛型接口、泛型方法。**  
　4）**泛型只在编译阶段有效，在编译过程中正确验证泛型结果后，会将泛型相关信息擦除，并且会在对象进入和离开方法的边界处添加类型检查和类型转换的方法。因此，泛型信息不会进入到运行阶段。**

### 一、泛型类
##### 1.定义泛型类
在普通类的类名后加`<T>`来定义该类为泛型类，其中T也可是E、K、V等任意字母
```java
public class Demo<T>{    //类名后接"<T>"，T也可是E、K、V等任意字母
    private T name;
     
    public Demo(T name){
       this.name = name;
    }
     
    public T getName(){
       return name;
    }
}
```
##### 2.new泛型类
**1）显式指定泛型类型：**  
　　Demo&lt;Integer> demo = new Demo&lt;Integer>(1024);
    
**2）由编译器推断泛型类型：**  
　　Demo demo = new Demo(1024);   

**注意：** １）泛型的类型参数只能是类类型，不能是简单类型。（如：只能是Integer，不能是int）  
　　　２）不能对确切的泛型类型使用instanceof操作。（"example instanceof Demo<String>"是非法的）

### 二、泛型接口
```java
public interface Demo<T>{   //类名后接"<T>"，T也可是E、K、V等任意字母
    public T algorithm();
}
 ```
 **使用:**  
 1）未传入泛型实参时
```java
class SimpleDemo<T> implements Demo<T>{     //类名和接口名后必须有"<T>"
     @override
     public T algorithm(){
         return null;
     }
}
```
2）传入泛型实参时
```java
class AnotherDemo implements Demo<String>{   //确定泛型类型后，类名后可不接<String>
     @override
     public String algorithm(){
         return null;
     }
}
```
Tips：1）接口可extends多个接口，例：public interface Demo extends A,B,C{....}  
　　　2）类可implements多个接口，例：public class Demo implements A,B,C{....}
		  
### 三、泛型方法
```java
public class Demo<T>{
	
    public <T,K> K algorithm(T value){             //该方法是泛型方法，此处方法内的两个T代表同一种类型，他俩可以与泛型类中的T相同，也可不同。
         return null;
    }
	   
    public void method(T value){                   //该方法不是泛型方法，此处的T与泛型类中的T是同一种类型
         System.out.println(valus.toString());
    }

    public static <K> K staticMethod(K value){     //如果静态方法也使用泛型时，该静态方法必须申明为泛型方法，否则编译会报错
         return value;  
    }
    
    public static T staticMethod(T value){         //缺少<T>编译报错
        return value;  
    }
    
}
```
注意：1）只有方法的public与返回值之间有<T\>时该方法才是泛型方法（T可为E、K、V等）  
　　　2）泛型类，是在实例化类的时候指明泛型的具体类型；泛型方法，是在调用方法的时候指明泛型的具体类型。

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