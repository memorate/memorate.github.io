---
layout: post
title: GitLab-IP解禁最佳方案
tags:
- GitLab
categories: GitLab
description: GitLab用户IP被封最佳解禁方案
---
#### 由高并发引起的GitLab用户IP被封的最佳解禁方案

<!-- more -->
#### **现象**
浏览器访问gitlab出现“Forbidden”。  
![]({{ "/assets/img/ip1.png" | absolute_url }})

#### **原因**
GitLab并发访问过多引起IP被封。  

#### **解决方案**
解决方案有三种，一是删除黑名单，二是添加白名单，三是等待一小时自动解禁。下文是删除黑名单方案，不推荐使用添加白名单方案。  
![]({{ "/assets/img/ip2.png" | absolute_url }})  
（上图文件是/etc/gitlab/gitlab.rb，安装后未修改默认配置"bantime"为3600秒，60分钟，意思是IP被封禁时间为1小时）

#### **解决步骤**
**1.找到gitlab的embedded文件夹**  
　若找不到，使用该命令查找（前提是已安装locate）
```bash
locate embedded
```
![]({{ "/assets/img/ip3.png" | absolute_url }})  
<br/>
**2.进入embedded/bin文件夹**  
```bash
cd /opt/gitlab/embedded/bin
```
<br/>
**3.在上一步的目录中执行该命令（直接copy执行）** 
```bash 
./redis-cli -s /var/opt/gitlab/redis/redis.socket
```
/var/opt/gitlab/redis/redis.socket在gitlab的安装目录下，若不知路径在哪，使用命令"locate redis.socket"查找  
注意`使用`/gitlab/redis/redis.socket  `别使用`/gitlab.bak/redis/redis.socket  
![]({{ "/assets/img/ip4.png" | absolute_url }})  
<br/>
**4.执行完上一会进入以下命令界面**  
![]({{ "/assets/img/ip5.png" | absolute_url }})  
<br/>
**5.输入" keys \*attack* "查看被禁ip**  
```bash
keys *attack*
```
![]({{ "/assets/img/ip6.png" | absolute_url }})  
<br/>
**6.输入以下命令解禁** 
```bash
del cache:gitlab:rack::attack:allow2ban:ban:10.74.131.47
```
del后面部分为第5步图片中显示的数据，若第5步显示多条根据ip人工找到要解禁的那条数据
![]({{ "/assets/img/ip7.png" | absolute_url }})  
若出现下图内容，则表示删除成功
![]({{ "/assets/img/ip8.png" | absolute_url }})  
<br/>
**7.浏览器访问网站验证是否解禁成功**  
<br/>  
<br/>
`Tips：`　统计IP被403次数
```bash
cat /var/log/gitlab/nginx/gitlab_access.log | grep 403 | grep "10.74.131.59" | wc -l
```
