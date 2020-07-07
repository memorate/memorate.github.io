---
layout: post
title: Java-导出Excel并下载
tags:
- Java 
- Tips
categories: Java
description: Controller返回Excel文件
---  
**SpringBoot导出Excel并下载**

<!-- more -->
## 先上代码
```java
/* Controller层代码 */
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
```java
/* ServiceImpl层代码(省略Service层代码) */
@Service
public class FileServiceImpl implements FileService {
    @Resource
    private UserService userService;

    @Resource
    private HttpServletResponse response;

    @Override
    public void exportAllUsers(){
        List<User> users = userService.getAllUser("id");    //获取源数据
        List<UserDTO> collect = users.stream().map(UserDTO::new).collect(Collectors.toList());   //转为DTO
        ExportParams params = new ExportParams(null, "用户信息", ExcelType.HSSF);
        try (Workbook wb = ExcelExportUtil.exportExcel(params, UserDTO.class, collect)) {
            //组装附件名称和格式
            this.response.setContentType("application/vnd.ms-excel");
            this.response.setHeader("Content-disposition", "attachment; filename=" + URLEncoder.encode("用户信息.xls", "UTF-8"));
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
使用EasyPoi生成Excel([EasyPoi官网](http://doc.wupaas.com/docs/easypoi/easypoi-1c0u4mo8p4ro8))。
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
import cn.afterturn.easypoi.excel.ExcelExportUtil;
import cn.afterturn.easypoi.excel.entity.ExportParams;
import cn.afterturn.easypoi.excel.entity.enmus.ExcelType;

ExportParams params = new ExportParams(null, "用户信息", ExcelType.HSSF);  //三个参数(Excel中标题，sheetName，Excel类型)
Workbook wb = ExcelExportUtil.exportExcel(params, UserDTO.class, data)
```
```java
public class UserDTO {

    @Excel(name = "序号", type = 10, width = 8)
    private Long id;

    @Excel(name = "姓名")                //name是列名
    private String name;

    @Excel(name = "年龄", type = 10)     //type = 10,代表类型是文本
    private Integer age;

    @Excel(name = "描述", width = 40)    //width用来设置列宽
    private String description;
}
```
## 二、导出