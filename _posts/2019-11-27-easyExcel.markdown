---
layout: post
title: java工具 - EasyExcelの初体验
tags:
- Java
- Tools
categories: Java
description: 阿里团队EasyExcel的初步使用
---
**Excel解析之阿里团队EasyExcel的初步使用**

<!-- more -->

### 一、EasyExcel简介  
　EasyExcel是阿里团队基于poi开发的操作Excel的开源项目。  
　其优势在于上手容易、操作简单、节约内存。（[GitHub地址](https://github.com/alibaba/easyexcel)、[官方指南](https://alibaba-easyexcel.github.io/index.html)）
```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>2.1.2</version>         <!--本文中所有相关代码使用此版本Java包-->
</dependency>
```

### 二、EasyExcel读取   
　　![]({{ "/assets/img/20191127/excel1.jpg"}})（模板数据，共4列10行）  
　1）使用EasyExcel读取**首先要创建`Listener`类**，该类会**按行**解析Excel（需继承抽象类`AnalysisEventListener<T>`。T为期望将Excel转换成的数据类型）。  
　2）继承AnalysisEventListener类后**必须**实现两个方法`invoke()`和`doAfterAllAnalysed()`。  
　　①invoke()：用于处理读取到的每一行数据；  
　　②doAfterAllAnalysed()：在读完当前sheet中所有数据后会自动调用此方法；  
##### *1.使用Map读取*  
使用EasyExcel类调用read方法，需要两个参数：Excel**路径**和解析Excel的**Listener**。直接上代码：    
```java
NormalListener listener = new NormalListener();
EasyExcel.read("D:\\MyData\\Administrator\\Desktop\\sample.xlsx", listener).sheet().doRead();
System.out.println(listener.getData());    //取出并打印数据
```
**运行结果：**  
![]({{ "/assets/img/20191127/excel3.jpg"}})（只解析到了最后一行数据）  
`原因：`不论将T的类型声明为HashMap或Map，EasyExcel都会将其转换为LinkedHashMap。
且invoke()中data的key每次始终都是0、1、2、3（列的index），因此最终读取到的dataList只是Excel中最后一行数据（Map的key不可重复）。
因此应将NormalListener类中dataList声明为**List<Map<Integer, String>>**类型。  
![]({{ "/assets/img/20191127/excel2.png"}})              
**Listener类：**
```java
public class NormalListener extends AnalysisEventListener<Map<Integer, String>> {

    //用于装载解析Excel得到的数据，Map会导致获取数据不完整，应使用List<Map<Integer, String>>
    private Map<Integer, String> dataList = new HashMap<>(); 
    private int row = 0;

    @Override
    public void invoke(Map<Integer, String> data, AnalysisContext context) {
        dataList.putAll(data);         //将每一行数据放入dataList
        row++;
    }

    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        System.out.println("Total row :" + row);    //输出解析了多少行数据
    }

    public Map<Integer, String> getData(){          //用于获取解析到的数据
        return dataList;
    }
}
```
##### *2.使用Java类读取*
**读取代码写法：**  与之前不同的是使用head()来指定映射的Java类  
```java
SheetListener listener = new SheetListener();
EasyExcel.read("D:\\MyData\\Administrator\\Desktop\\sample.xlsx", listener).head(Sheet.class).sheet().doRead();
List<Sheet> dataList = listener.getData();     //取出数据
System.out.println(dataList);
```
**Java类：**使用`@ExcelProperty`注解来指定读取的列，可以使用列名或者列序来读取。一般只使用列名。  
```java
public class Sheet {
    @ExcelProperty(index = 0)      //使用列序读取，第1列index为0
    private int index;

    @ExcelProperty("姓名")         //使用列名读取
    private String name;

    @ExcelProperty("年龄")         //推荐使用列名读取
    private int age;

    @ExcelProperty(value = "祖籍", index = 3)   //同时使用列名和列序来读取
    private String hometown;
}                                  //省略get和set方法，实际读取时必须有get和set方法，否则读不到数据
```  
`注：`当Java类中默认参构造函数被覆盖时，需增加无参构造函数，否则无法读取数据。   
**Sheet1Listener类：**  
```java
public class SheetListener extends AnalysisEventListener<Sheet> {

    List<Sheet> dataList = new ArrayList<>();
    private int row = 0;

    @Override
    public void invoke(Sheet data, AnalysisContext context) {
        dataList.add(data);
        row++;
    }

    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        System.out.println("Total row :" + row);
    }

    public List<Sheet> getData() { return dataList; }

}
```
##### *3.读取指定sheet*  
sheet()有三个重载方法：  
　1）sheet(int sheetNo)　　　　　　　　　　　　//指定sheet的**序号**来读，序号从0开始  
　2）sheet(String sheetName)　　　　　　　　　//指定sheet的**名称**来读，区分大小写  
　3）sheet(int sheetNo, String sheetName)　　　//同时使用，一般只使用sheetName  
**1）一次读取单个sheet**  
之前所有的读取默认是读Excel中第一个sheet,现在使用sheet()来指定sheet读。  
**读取指定sheet**
```java
SheetListener listener = new SheetListener();
EasyExcel.read("D:\\MyData\\Administrator\\Desktop\\sample.xlsx", listener)
        .sheet(1).head(Sheet.class).doRead();           //指定读取第二个sheet
System.out.println(listener.getData());
```
**2）一次读取多个sheet**    
```java
SheetListener listener = new SheetListener();
//打开Excel
ExcelReader reader = EasyExcel.read("D:\\MyData\\Administrator\\Desktop\\sample.xlsx").build(); 
//指定sheet名称读取第一个sheet
ReadSheet sheet1 = EasyExcel.readSheet("Sheet1").head(Sheet.class).registerReadListener(listener).build();
//指定sheet序号读取第二个sheet
ReadSheet sheet2 = EasyExcel.readSheet(1).head(Sheet.class).registerReadListener(listener).build();
reader.read(sheet1, sheet2);                //真实去读
reader.finish();                            //必须关闭
System.out.println(listener.getData());     //取出两个sheet的总数据并打印
```
实际情况中每个sheet对应的Java类可能是不同的（`head()`中class类型不同，相应的`registerReadListener()`中listener也不同）。  
### 三、EasyExcel写入    
##### *1.写入一个sheet*   
写入时会根据`@ExcelProperty(value = "列名", index = n)`注解中的value值来生成表头，若未使用value则会将该注解所在的属性名当做表头。  
在write()方法中指定写入文件**路径**和写入的**Java类**，在doWrite()中指定写入的数据，该数据是List\<T>类型。  
```java
List<Sheet> dataList = new ArrayList<>();
add some data to dataList...
EasyExcel.write("D:\\MyData\\Administrator\\Desktop\\output.xlsx", Sheet.class).sheet().doWrite(dataList);
``` 
`注意：`  
　①write()中指定的文件若**不存在**EasyExcel会**创建**，若文件**存在**EasyExcel会用dataList中数据**覆盖**文件中的所有内容。  
　②若Sheet存在父类，且其父类中属性未使用注解指明index，写入数据时父类中属性会排在列序的**最后**。可使用@ExcelProperty(index = n)指定顺序。  
　③可使用`@ColumnWidth`注解指明**列宽**，使用`@ContentRowHeight`指明**行高**，这两个注解都可放在**类上**和**属性上**，放在类上作用于类中所有属性，放在属性上优先级高于放在类上。  
##### *2.写入多个sheet*   
```java
List<Sheet> dataList = new ArrayList<>();
add some data to dataList...
ExcelWriter writer = EasyExcel.write("D:\\MyData\\Administrator\\Desktop\\output.xlsx").build();   //打开文件
WriteSheet sheet1 = EasyExcel.writerSheet(0).head(Sheet.class).build();          //根据sheet序号写入第一个sheet
WriteSheet sheet2 = EasyExcel.writerSheet("Sheet2").head(Sheet.class).build();   //根据sheet名称写入第二个sheet
writer.write(dataList, sheet1);       //真实向sheet1中写数据
writer.write(dataList, sheet2);       //真实向sheet2中写数据
writer.finish();                      //必须关闭
```