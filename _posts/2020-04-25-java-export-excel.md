---
layout: post
title: SpringBoot - 导出Excel并下载
tags:
- Java 
- Tips
categories: Java
description: Controller返回Excel文件
---  
**SpringBoot导出Excel并下载**

<!-- more -->
## 先上代码
**1.Controller层代码**
```java
@RestController
@RequestMapping("/file")
public class FileController {
    @Resource
    private FileService fileService;

    @GetMapping("/UsersExcel")
    public void exportUsers(){
        fileService.exportAllUsers();
    }
}
```
**2.ServiceImpl层代码(省略Service层代码)**
```java
@Slf4j
@Service
public class FileServiceImpl implements FileService {
    @Resource
    private UserService userService;

    @Resource
    private HttpServletResponse response;

    @Override
    public void exportAllUsers(){
        List<User> users = userService.getAllUser("id");    //从数据库获取源数据
        List<UserDTO> collect = users.stream().map(UserDTO::new).collect(Collectors.toList());   //转为DTO
        ExportParams params = new ExportParams(null, "用户信息", ExcelType.HSSF);    //三个参数(Excel中标题，sheetName，Excel类型)
        try (Workbook wb = ExcelExportUtil.exportExcel(params, UserDTO.class, collect)) {
            //组装附件名称和格式
            this.response.setContentType("application/vnd.ms-excel");
            this.response.setHeader("Content-disposition", "attachment; filename=" + URLEncoder.encode("用户信息.xls", "UTF-8"));
            //将文件转为二进制流进行传输
            ServletOutputStream out = this.response.getOutputStream();
            wb.write(out);
            out.flush();
            out.close();
        } catch (Exception e) {
            log.error("Export error!", e);
        }
    }
}
```
## 一、Excel
**使用EasyPoi生成Excel([EasyPoi官网](http://doc.wupaas.com/docs/easypoi/easypoi-1c0u4mo8p4ro8))。**
#### 1.引入
```xml
<dependency>
    <groupId>cn.afterturn</groupId>
    <artifactId>easypoi-spring-boot-starter</artifactId>
    <version>4.2.0</version>
</dependency>
```
#### 2.生成
```java
public class UserDTO {

    @Excel(name = "序号", type = 10, width = 8)    //使用此注解来映射 类的属性 与 Excel的列
    private Long id;

    @Excel(name = "姓名")                //name是列名
    private String name;

    @Excel(name = "年龄", type = 10)     //type = 10,代表Excel中age列中数据类型是文本
    private Integer age;

    @Excel(name = "描述", width = 40)    //width用来设置列宽
    private String description;
}
```
```java
import cn.afterturn.easypoi.excel.ExcelExportUtil;
import cn.afterturn.easypoi.excel.entity.ExportParams;
import cn.afterturn.easypoi.excel.entity.enmus.ExcelType;

ExportParams params = new ExportParams(null, "用户信息", ExcelType.HSSF);
Workbook wb = ExcelExportUtil.exportExcel(params, UserDTO.class, data)    //data一般为一个List
```
## 二、导出
①通过**HttpServletResponse**进行导出。([HttpServletResponse介绍](https://www.jianshu.com/p/8bc6b82403c5))  
②简单来说整个过程是这样：后端接收请求方的HttpRequest，然后返回一个HttpResponse，我们将Excel转换成二进制流塞进这个response并设置好格式等就OK了。
```text
import javax.servlet.http.HttpServletResponse;
```
```java
try (Workbook wb = ExcelExportUtil.exportExcel(params, UserDTO.class, collect)) {
    this.response.setContentType("application/vnd.ms-excel");    //response是注入的HttpServletResponse对象
    this.response.setHeader("Content-disposition", "attachment; filename=" + URLEncoder.encode("用户信息.xls", "UTF-8"));
    ServletOutputStream out = this.response.getOutputStream();
    wb.write(out);
    out.flush();
    out.close();
} catch (Exception e) {
    log.error("Export error!", e);
}
```
## 三、下载
![]({{ "/assets/img/20200425/export.jpeg"}})