---
layout: post
title: Linux权限记录
tags: Linux
categories: Linux
description: 记录Linux更改文件权限
---
### 简介

用于记录Linux更改文件（文件夹）权限操作

<!-- more -->

### 权限说明
Linux系统中有三种权限：读、写、执行
```text
r 读权限read  4

w 写权限write 2

x 操作权限execute  1
```
### 权限组合说明
一般使用chmod命令进行赋权时使用一下数字代表的权限组合：
```text
-rw------- (600) 只有所有者才有读和写的权限

-rw-r--r-- (644) 只有所有者才有读和写的权限，组群和其他人只有读的权限

-rwx------ (700) 只有所有者才有读，写，执行的权限

-rwxr-xr-x (755) 只有所有者才有读，写，执行的权限，组群和其他人只有读和执行的权限

-rwx--x--x (711) 只有所有者才有读，写，执行的权限，组群和其他人只有执行的权限

-rw-rw-rw- (666) 每个人都有读写的权限

-rwxrwxrwx (777) 每个人都有读写和执行的权限
```

### Linux命令
更改aaa.text**文件**的权限为所有人具有读写和执行的权限：
```bash
chmod 777 aaa.txt
```
更改bbb**文件夹**的权限为所有人具有读写和执行的权限：
```bash
chmod -R 777 bbb
```
