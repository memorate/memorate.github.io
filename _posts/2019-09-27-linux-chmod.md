---
layout: post
title: Linux权限记录
tags: Linux
categories: Linux
description: 记录Linux更改文件权限
---
### 权限说明
```text
r 读权限read  4
w 写权限write 2
x 操作权限execute  1
```
### 权限组合说明
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
```bash
chmod 777 aaa.txt
```
