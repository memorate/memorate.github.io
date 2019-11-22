---
layout: post  
title: java-泛型原来是这么回事  
tags: 
- Java
- Study
categories: Java  
description: 关于泛型的个人学习及理解
---
#### 用于记录关于泛型的自我学习及理解成果

<!-- more -->

### 什么是泛型
　1）泛型是一种**语法糖**，方便程序员编程；  
　2）泛型的**本质是一种参数化类型**，将所操作的数据类型（类类型/引用类型）指定为一个参数，在调用时再传入具体的类型；  
　3）泛型的**原理**是在编译时Java编译器将泛型代码转换为普通代码，将类型T擦除，替换为Object类，并加入必要的强制类型转换；Java虚拟机在加载运行.class文件时并不知道存在泛型，只会接收到普通类及代码。   
　4）泛型只在**编译阶段**有效，在编译过程中正确验证泛型结果后，会将泛型相关信息擦除，并且会在对象进入和离开方法的边界处添加类型检查和类型转换的方法。因此，泛型信息不会进入到运行阶段。  
　5）泛型可用于类、接口、方法中，分别被称为**泛型类、泛型接口、泛型方法**；  


### 为什么使用泛型
　1）**类型安全**，在编译时进行类型检测，只有指定类型才可添加到集合/类中；   
　2）泛型将运行时的类型转换问题提前到了编译期，**避免了强制类型转换**；   
```java
   List list = new ArrayList();
   list.add("hello");
   String s = (String) list.get(0);   //强制类型转换
   -------------------------------------------------
   List<String> list = new ArrayList<String>();
   list.add("hello");
   String s = list.get(0);         //无需类型转换
```
　3）提高代码的**可读性**和**重用率**，降低程序的复杂度  
### 一、泛型类
##### 1.*定义泛型类*
　1）在普通类的类名后加`<T>`来定义该类为泛型类，其中T也可是E、K、V等任意字母。(本文中将\<T>称为**泛型标识符**)   
　2）T在该泛型类中可作为**属性的类型**、**方法的入参类型**、**方法的返回值类型**。  
```java
public class Demo<T>{           //类名后接泛型标识符"<T>"，T也可是E、K、V等任意字母

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
`注意：`字符串也可用于定义泛型(不推荐使用！！！)
```java
public class NotRecommend<String>{         //此处"String"与"T"的作用等同，代表一个参数化类型。然而容易造成误解

    private String value1;                 //此处的"String"代表参数化类型，而不是！不是！不是！"java.lang.String"类

    private java.lang.String value2;       //若要使用字符串类只能这样写。此处的"java.lang.String"代表字符串类
}
```
　3）泛型可定义多个
```java
public class MultiDemo<T,K,V>{          //类型参数可定义多个。类名后接"<T,K,V>"，个数不限

    private T value;
    private K key;

    public MultiDemo(T value, K key){       
        this.value = value;
        this.key = key;
    }

    public T getValue(){
        return value;
    }

    public K getKey(){
        return key;
    }

    public V transfer(){
        return (V)key;
    }
}
```
##### 2.*实例化泛型类*
**1）显式指定泛型类的类型：**  
```java
Demo<Integer> integerDemo1 = new Demo<Integer>(1024);　　　　　//显示指定泛型类integerDemo的参数化类型为Integer  
Demo<Integer> integerDemo2 = new Demo<>(1024);　　　　       　//Java7以后，支持省略后面的参数类型  

Demo<String> stringDemo1 = new Demo<>("string");　　        　//显示指定stringDemo类的参数化类型为String  

MultiDemo<Integer,Double,Number> multiDemo = new MultiDemo<>(108, 45.6);　　     //实例化多个类型参数的泛型类
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
　1）泛型的类型参数只能是**类类型（引用类型）**，不能是简单类型。（如：只能是Integer，不能是int）  
```java
   Demo<int> intDemo = new Demo<>(1024);　　　　　　　　　　 　//编译错误！！！，泛型不能是简单类型  
```
　2）不能对**确切的泛型类**使用instanceof操作:  
　　`stringDemo instanceof Demo<String>`是**非法**的，但`stringDemo instanceof Demo`是**合法**的。  
　3）Integer是Number的子类，但List<Integer\>**并不是**List<Number\>的子类！  
　4）泛型**不能**用于声明类中**静态变量**的类型  
　　原因：①理念不合。static的理念是确定、唯一，泛型的理念是可变、多用；  
　　　　　②当实例化多个类型不同的泛型类时，静态变量的类型无法确定。  
```java
   public class IllegalDemo<M>{
       private static M key;    //编译不通过！！！
   }
```  
```java
   IllegalDemo<Integer> integerDemo = new IllegalDemo<>(1024);      //不论实例化多少个IllegalDemo类，static变量key只会在内存中存在一份；
   IllegalDemo<String> stringDemo = new IllegalDemo<>("String");    //而key的类型却有三种Integer、String、Double，因此编译器无法确定
   IllegalDemo<Double> doubleDemo = new IllegalDemo<>(9.99);        //key的类型到底是哪个
```
### 二、泛型接口
##### 1.*定义泛型接口*
　1）接口名后接泛型标识符`<T>`  
　2）T可用于接口中方法的返回值和方法的入参类型
```java
public interface IDemo<T>{       //申明IDemo为泛型接口。接口名后接泛型标识符"<T>"，T也可是E、K、V等任意字母

    public T algorithm(T value);
}
 ```
　3）泛型可定义多个
```java
public interface IMultiDemo<T,E>{     //类型参数可以有多个，接口名后接"<T,E>"，个数不限

    public E algorithm(T value);
}
 ```
##### 2.*使用泛型接口*
 **1）引用的泛型接口未传入类型实参时**  
 　需要将泛型标识符也加在类名后。**类名**和**接口名**后必须都有泛型标识符`<T>`  
```java
class SimpleDemo<T> implements IDemo<T>{      //此时的SimpleDemo为泛型类，类名和接口名后必须都有泛型标识"<T>"

     @override
     public T algorithm(T value){             //此段代码中出现的四个泛型标识符必须相同（都是T）
         return value;
     }
}
```
**2）引用的泛型接口传入类型实参时**  
　在使用泛型接口的类中，需将所有使用到泛型的地方替换成传入的实参类型
```java
class AnotherDemo implements IDemo<String>{       //T替换为String

     @override
     public String algorithm(String value){       //T替换为String
         return value;
     }
}
```
**Tips：**  
　　1）接口可extends多个接口，例：public interface IDemo extends A,B,C{....}  
　　2）类可implements多个接口，例：public class Demo implements A,B,C{....}
		  
### 三、泛型方法
<br/>
　在方法的**权限修饰符**和**返回值**之间使用标识符`<T>`来定义泛型方法。  
<br/>
1）在**普通类**中定义泛型方法  
```java
public class normalDemo{
    public String normalMethod(String value){                       //普通方法
        return value;
    }
    
    public <T> String genericMethod(T value){                       //泛型方法，public和String之间有"<T>"
        return value.toString();
    }
    
    public T illegalMethod(T value){                                //编译错误！！！，无法识别类型T
        return value;
    }

    protected <E,K> void mutilGenericMethod(E param, K value){      //泛型方法，泛型可声明多个
        System.out.println(param.toString() + value.toString());
    }

    public static String staticMethod(String value){                //普通静态方法
        return value;
    }

    public static <T> T genericStaticMethod(T value){               //泛型静态方法，静态方法若要使用泛型，必须将静态方法申明为泛型方法
        return value;
    }
}
```
2）在**泛型类**中定义泛型方法  
```java
public class genericDemo<T>{
    public T normalMethod(T value){                 //普通方法，两个T与泛型类中的T相同
        return value;                               //此方法可以理解为是具有泛型特性（类型参数化）的普通方法，因为它可以使用泛型类的参数化类型
    }

    public <T> T genericMethod(T value){            //泛型方法，此处T是一种全新的类型，可以与泛型类中的T相同，也可不同。
        return value;                               //不建议在这里使用已使用的字母T，应换一个字母来声明
    }

    public <E,K> void gmutilGenericMethod(E param, K value){          //泛型方法，泛型可声明多个;E、K可与T相同，也可不同
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
　1）**一个方法是不是泛型的，与它所在的类（或接口）是不是泛型的没有关系。**  
　2）只有方法的访问修饰符（public等）与返回值之间有泛型标识符<T\>时该方法才是泛型方法（T可为E、K、V等），否则它只是个普通方法。  
　3）泛型类，是在实例化类的时候指明泛型的具体类型；泛型方法，是在调用方法的时候指明泛型的具体类型。  
　4）泛型类中的参数化类型T的作用域是整个类（类中任意地方可用T），泛型方法中的类型E仅能用于这个方法。  
　**例：**`public class genericDemo<T>{}`，T可用于整个genericDemo类；`public <T> T genericMethod(){}`，T只能用于genericMethod方法。

### 四、泛型范围限定
<br/>
　在**定义**泛型类、泛型接口、泛型方法时，可以使用关键字`extends`来限定参数化类型T的上界。  
　**使用方法：**`<T extends ClassName>`，ClassName可以是一个类或是一个接口，也可是其他参数化类型（泛型）。  
<br/>
1）上界为某个**具体类**  
　当ClassName为一个具体类时，代表T只能是该类或其子类。  
　指定边界后，泛型擦除时就不会转换成Object类，而是会转换成边界类型。  
```java
public class ClassDemo<T extends Number>{          //限定T的上界为Number类，T只能是Number类或者Number的子类
    private T value;

    public double transform(){
        return value.doubleValue();                //因为限定了T的类型，所以可以使用Number类里的方法（子类可调用父类中的方法）
    }
}
```
```java
ClassDemo<Integer> demo = new ClassDemo<>();       //Integer是Number的子类，编译ok

ClassDemo<String> illegal = new ClassDemo<>();     //String不是Number的子类，编译不通过！！！
```
2）上界为某个**接口**  
　当ClassName为一个接口时，代表T必须实现了ClassName接口。（**接口可限定多个**）  
```java
public class InterfaceDemo<T extends Serializable>{      //限定T的上界为Serializable接口，T必须是实现了Serializable的类或接口
    private T value;

    public T getValue(){
        return value;  
    }
}
```
```java
public class Mutile<T extends SomeClass & Interface1 & Interface2>{     //上界可同时为类和接口，类只能有一个，接口可有多个，
     private T variable                                                 //类需放在extends之后第一位，中间用"&"连接。
}                                                                       //表示T必须是SomeClass或其子类，且需实现Interface1、Interface1接口。
```
```java
public <T extends Compatable<T>> T method(T param){      //递归类型限制，表示T必须实现了Comparable接口且可与同类型的元素进行比较
    return param;
}
```
3）上界为其他**类型参数**  
　泛型的上界可限定为其他泛型。
```java
public class AdvancedDemo<T extends E>{}     //限定T的上界为其他类型参数，当E是类时，T必须是E或E的子类；当E为接口时，T必须实现了E
```
`注意：`  
　1）泛型范围限定`<T extends ClassName>`可用在泛型类、泛型接口、泛型方法的**定义**中。可理解为`<T>`是普通泛型标识符，`<T extends ClassName>`是高级泛型标识符。  
　2）不存在泛型的下界！！！`<T super ClassName>`是**不存在**的。
### 五、泛型通配符
<br/>
**1.无边界泛型通配符**  
　写法：`<?>`，代表任意（未知）类型
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
**1.固定上边界泛型通配符**  
**1.固定下边界泛型通配符**  
Tips：一般用于方法的入参，当具体类型不确定时，可使用泛型通配符"?"；使用泛型通配符后无法使用类的具体功能，只能使用Object类中的功能。
 
注意：1）无边界泛型通配符"?"是类型实参，是一种真实的类型，可以看做所有类的父类，而不是类型形参。  
　　　2）不能同时申明泛型通配符的上界和下界。  
　　　3）<? extends Demo>代表Demo及其子类，<? super Demo>代表Demo及其父类。

原则：1）PECS（Producer extends，Consumer super）即生产者使用extends，消费者使用super。  
　　　2）需要读取数据以使用时，使用extends，固定上边界，将对象当做一个只读对象。  
　　　3）需要将数据写入对象时，使用super，固定下边界，将对象当做一个只写对象。  
　　　4）需要既写又读时，请勿使用泛型通配符。 　