---
layout: post
title: Jenkins-集成Sonar进行.NET代码扫描
tags:
- Jenkins
- Sonar  
- .Net
categories: Jenkins
description: Jenkins集成Sonar进行.NET代码扫描
---
**在已有Jenkins的基础上集成Sonar对.NET项目进行静态代码扫描**

<!-- more -->
### 一、简单说明
1）**说明：**本文所有操作在Windows 10和Windows Server 2019 Datacenter上均已测试成功。  
2）**适用范围：**已有Jenkins、Sonar服务器。若无，请先自行搭建！  
3）**需求描述：**使用Jenkins对.Net代码进行常规代码扫描，并将扫描结果上传至Sonar服务器。  
4）**环境准备：**  
```text
　Jenkins                  2.89.3
　SonarQube Server         6.7.1
　JDK                      1.8                                            下载地址1（aisy）
　Git                      2.6.1                                          下载地址2（59uy）
　TortoiseSVN              1.10.6                                         下载地址3（bvre）
　VS2015                   用于安装MSBuild及其他编译构建所必须的环境        下载地址4（vuvd）
　MSBuild 14.0.23107.0     用于执行编译构建，安装VS2015后会自动安装
　Sonar-scanner 4.6.1      用于执行代码扫描                                下载地址5（mprv）
```
　　　　　　　　　---[下载地址1](https://pan.baidu.com/s/1BRRYebT5FP9pSiiCG5cvNg&shfl=sharepset)---[下载地址2](https://pan.baidu.com/s/1Egm-KtuNakavYscR5mt8Bw&shfl=sharepset)---
[下载地址3](https://pan.baidu.com/s/18ms6b15iniZq5CtZwgm2ug&shfl=sharepset)---[下载地址4](https://pan.baidu.com/s/1eSZ3SfBahLwncQukATqyuQ&shfl=sharepset)---
[下载地址5](https://pan.baidu.com/s/1oGcSFoYAETvZR_n5Zymq9A&shfl=sharepset)---
### 二、专属slave搭建
　该slave可以是一台物理主机或虚拟机，是Jenkins用于执行.NET代码编译构建及扫描的节点。由于.NET本身限制，该slave**只能是windows**系统。  
**1.安装JDK**  
　slave连接至Jenkins需Java环境。  
　下载上文中相应的安装包进行安装。（[安装指南](https://blog.csdn.net/weixin_37601546/article/details/88623530)）  
**2.安装Git及TortoiseSVN**  
　用于从GitLab或SVN拉取代码。  
　下载上文中相应的安装包进行安装，双击运行后一直点击下一步即可，无特殊配置。    
**3.安装MSBuild及VS2015**  
　MSBuild是.NET编译构建的工具，类似Java的Maven；而VS2015则用于提供编译构建时所必须的环境。  
　下载上文中的vs2015安装包并解压，解压后运行“vs_community.exe”进行安装，按照提示安装即可，无特殊配置。  
![]({{ "/assets/img/dotnet1.jpg"}})  
**4.安装Sonar-scanner**  
　用于执行.NET代码扫描。  
　下载上文中相应的安装包，下载完成后解压至合适的文件夹即可使用。  

### 三、Jenkins配置
#### 1.配置SonarQube Server  
　1）扫描任务结束后将报告上传至该Server；  
　　在Jenkins首页点击　"系统管理" –> "系统设置"，找到"SonarQube Server"。  
　　![]({{ "/assets/img/dotnet2.jpg"}})![]({{ "/assets/img/dotnet3.jpg"}})  
　2）填入红框内的数据  
　　a."**Name**"： 随意填（也别太随意...），在有多个SonarQube Server时进行区分标识；  
　　b."**Server URL**"： 为sonar服务器的地址（域名IP都可）；  
　　c."**Server version**"： 根据sonar服务器版本进行选择；  
　　d."**Server authentication token**"： 在sonar界面上生成后复制过来（不建议使用账号密码形式）；（生成步骤）  
![]({{ "/assets/img/dotnet4.jpg"}})
#### 2.新增Jenkins节点
　1）新增Jenkins节点用于.NET代码扫描；在Jenkins首页点击　"系统管理" –> "管理节点" –>"新建节点"。  
　　![]({{ "/assets/img/dotnet5.jpg"}})![]({{ "/assets/img/dotnet6.jpg"}})  
　 2）填写节点名称，选择"固定代理"后点击OK  
　　![]({{ "/assets/img/dotnet7.jpg"}})  
　2）填写节点详情信息，填写完成后保存即可。  
　　a."**of executors**"：此节点可同时并行任务数，一般填4—8即可；  
　　b."**远程工作目录**"：此节点的工作目录，在第二步搭建的专属slave中创建一个空文件夹并将绝对路径复制过来即可（此目录要有足够的磁盘空间）；  
　　c."**标签**"：用于在Jenkins job中指定特定节点执行任务，此处建议填写".net"；  
　　d."**用法**"：选择"只允许运行绑定到这台机器的job"；  
　　e."**启动方法**"：选择"通过Java Web启动代理"，其余不用填写；  
　　f."**Availability**"：选择"尽量保持代理在线"；  
　　g."**环境变量**"：在此输入第二步专属slave的JAVA_HOME；  
　　![]({{ "/assets/img/dotnet8.jpg"}})  
#### 3.配置构建扫描工具



### 四、创建扫描任务