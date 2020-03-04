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
　Jenkins             2.89.3
　SonarQube Server    6.7.1
　JDK                 1.8                                                         下载地址1（aisy）
　Git                 2.6.1                                                       下载地址2（59uy）
　TortoiseSVN         1.10.6                                                      下载地址3（bvre）
　VS2015                              用于安装MSBuild及其他编译构建所必须的环境     下载地址4（vuvd）
　MSBuild             14.0.23107.0    用于执行编译构建，安装VS2015后会自动安装
　Sonar-scanner       4.6.1           用于执行代码扫描                             下载地址5（mprv）
```
　　　　　　　　　---[下载地址1](https://pan.baidu.com/s/1BRRYebT5FP9pSiiCG5cvNg&shfl=sharepset)---[下载地址2](https://pan.baidu.com/s/1Egm-KtuNakavYscR5mt8Bw&shfl=sharepset)---
[下载地址3](https://pan.baidu.com/s/18ms6b15iniZq5CtZwgm2ug&shfl=sharepset)---[下载地址4](https://pan.baidu.com/s/1eSZ3SfBahLwncQukATqyuQ&shfl=sharepset)---
[下载地址5](https://pan.baidu.com/s/1oGcSFoYAETvZR_n5Zymq9A&shfl=sharepset)---
### <span id="second">二、专属slave搭建</span>
　该slave可以是一台物理主机或虚拟机，是Jenkins用于执行.NET代码编译构建及扫描的节点。由于.NET本身限制，该slave**只能是windows**系统。  
**1.安装JDK**  
　slave连接至Jenkins需Java环境。  
　下载上文中相应的安装包进行安装。（[安装指南](https://blog.csdn.net/weixin_37601546/article/details/88623530)）  
**2.安装Git及TortoiseSVN**  
　用于从GitLab或SVN拉取代码。  
　下载上文中相应的安装包进行安装，双击运行后一直点击下一步即可，无特殊配置。    
**3.安装MSBuild及VS2015**  
　MSBuild是.NET编译构建的工具，类似Java的Maven；而VS2015则用于提供编译构建时所必须的环境。  
　下载上文中的vs2015安装包并解压，解压后运行"vs_community.exe"进行安装，按照提示安装即可，无特殊配置。  
![]({{ "/assets/img/20191108/dotnet1.jpg"}})  
**4.<span id="scanner">安装Sonar-scanner</span>**  
　用于执行.NET代码扫描。  
　下载上文中相应的安装包，下载完成后解压至合适的文件夹即可使用。  

### 三、Jenkins配置
#### 1.配置SonarQube Server  
　用于在扫描任务结束后将扫描报告上传至该Server。  
　1）在Jenkins首页点击　"系统管理" –> "系统设置"，找到"SonarQube Server"。  
　　![]({{ "/assets/img/20191108/dotnet2.jpg"}})  
　2）填入下图红框内的数据  
　　a."**Name**"： 随意填（也别太随意...），在有多个SonarQube Server时进行区分标识；  
　　b."**Server URL**"： 为sonar服务器的地址（域名IP都可）；  
　　c."**Server version**"： 根据sonar服务器版本进行选择；  
　　d."**Server authentication token**"： 在sonar界面上生成后复制过来（不建议使用账号密码形式）；（生成步骤）  
![]({{ "/assets/img/20191108/dotnet4.jpg"}})
#### 2.新增Jenkins节点
　将[第二步](#second)搭建的slave关联为Jenkins的运行节点，专门用于.NET代码扫描。  
　1）在Jenkins首页点击　"系统管理" –> "管理节点" –>"新建节点"。  
　　![]({{ "/assets/img/20191108/dotnet5.jpg"}})    
　2）填写节点名称，选择"固定代理"后点击OK    
　　![]({{ "/assets/img/20191108/dotnet7.jpg"}})  
　3）填写节点详情信息，填写完成后保存即可。  
　　a."**of executors**"：此节点可同时并行任务数，一般填4—8即可；  
　　b."**远程工作目录**"：此节点的工作目录，在第二步搭建的专属slave中创建一个空文件夹并将绝对路径复制过来即可（此目录要有足够的磁盘空间）；  
　　c."**<span id="label">标签</span>**"：用于在Jenkins job中指定特定节点执行任务，此处建议填写".net"；  
　　d."**用法**"：选择"只允许运行绑定到这台机器的job"；  
　　e."**启动方法**"：选择"通过Java Web启动代理"，其余不用填写；  
　　f."**Availability**"：选择"尽量保持代理在线"；  
　　g."**环境变量**"：在此输入第二步专属slave的JAVA_HOME；  
　　![]({{ "/assets/img/20191108/dotnet8.jpg"}})  
　4）连接slave至Jenkins  
　　创建成功后会显示下列信息，复制红框内命令在slave中用CMD执行即可  
![]({{ "/assets/img/20191108/dotnet13.jpg"}})  

#### 3.配置构建扫描工具
　将[第二步](#second)中安装的MSBuild及SonarQube Scanner for MSBuild关联到Jenkins，以便用户创建Jenkins job时可以使用。  
　1）在首页点击 "系统管理" –> "全局工具配置"，可以看到如下右图所示的两个插件；  
　![]({{ "/assets/img/20191108/dotnet10.jpg"}})  
　2）**MSBuild**  
　　点击"MSBuild安装"->"新增MSBuild"；请勿勾选自动安装，因为自动安装需连接github去下载安装包，机器不一定可以访问github。  
　　a."**<span id="name1">Name</span>**"：可随意填写；  
　　b."**Path to MSBuild**"：安装vs2015时自动安装至"C:/Program Files(x86)/MSBuild/14.0/Bin/amd64/MSBuild.exe"，此路径一般不会改变，填写之前先确认；
![]({{ "/assets/img/20191108/dotnet11.jpg"}})  
　3）**<span id="name2">SonarQube Scanner for MSBuild</span>**  
　　同MSBuild，"MSBUILD_SQ_SCANNER_HOME"为[安装Sonar-scanner](#scanner)的绝对路径。  
![]({{ "/assets/img/20191108/dotnet12.jpg"}})

### 四、创建Jenkins任务
1.新建任务  
　在Jenkins首页点击"新建"，输入任务名称后选择"**构建一个自由风格的软件项目**"，然后点击OK。  
![]({{ "/assets/img/20191108/dotnet14.jpg"}})  
2.配置任务  
　1）指定节点运行，填写新建Jenkins节点时[输入的标签](#label)
![]({{ "/assets/img/20191108/dotnet15.jpg"}})  
　2）绑定代码仓库。  
　　根据实际情况，选择Git或者Subversion，并填写仓库地址、指定仓库授信。  
![]({{ "/assets/img/20191108/dotnet16.jpg"}})  
　3）配置构建步骤  
　在"增加构建步骤中"依次选择（必须是这个顺序）  
　　"SonarQube Scanner for MSBuild - Begin Analysis" ->  
　　"Build a Visual Studio project or solution using MSBuild" ->  
　　"SonarQube Scanner for MSBuild - End Analysis"；  
![]({{ "/assets/img/20191108/dotnet17.jpg"}})  
　4）SonarQube Scanner for MSBuild - Begin Analysis    
![]({{ "/assets/img/20191108/dotnet18.jpg"}})  
　a."**SonarQube Scanner for MSBuild**"：选择在[这一步](#name1)填写的名字；  
　b."**Project key**"：在sonar中唯一的标识符；  
　c."**Project name**"：在sonar中显示的项目名称；  
　d."**Project version**"：此次扫描的版本（这次为1.0，下次为2.0，或者一直用1.0）；  
　e."**Additional arguments**"：扫描参数，可参考[此说明](https://docs.sonarqube.org/latest/analysis/analysis-parameters/)；  
　5）Build a Visual Studio project or solution using MSBuild  
![]({{ "/assets/img/20191108/dotnet19.jpg"}})   
　a."**MSBuild Version**"：选择在[这一步](#name2)填写的名字；  
　b."**MSBuild Build File**"：.NET项目工程文件（一般填.sln文件），类似Maven所需要的POM文件。("${WORKSPACE}"代表当前任务空间的路径)  
　c."**Command Line Arguments**"：编译构建参数，可参考[此说明](https://docs.microsoft.com/en-us/visualstudio/msbuild/msbuild-command-line-reference?view=vs-2019)；  
`Tips：`    
　Jenkins会为每个任务创建一个工作区（文件夹），名字为任务名。Jenkins在clone代码时会直接将仓库中的所有文件clone至工作区，而不是在工作区里创建以仓库名命名的文件夹再将仓库clone至该文件夹下。


