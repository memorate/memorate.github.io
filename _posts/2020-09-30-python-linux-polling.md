---
layout: post
title: Python - 批量巡检Linux的CPU、内存、硬盘
tags:
- Python
- Linux
categories: Python
description: Linux机器巡检
---  
**Python脚本查询Linux机器的CPU、内存、硬盘**

<!-- more -->
## 前言
1.每逢大的节假日前公司都要对主要应用所在的机器做一次巡检，确保假期期间机器不会出问题。  
2.其实所有机器都已经装了Zabbix监控，并且配置了报警，但是奈何形式大于一切，因此在领导的示意下开发了此脚本。  
3.只巡检三个指标：**CPU使用率**、**内存使用率**、**硬盘使用率**。  
## 脚本
使用前需要使用命令`pip install paramiko`安装paramiko模块
```python
import re

import paramiko
import xlwt

ip_file = 'C:/Users/anchor/Desktop/test.txt'
log_file = 'C:/Users/anchor/Desktop/log.txt'
sheet_name = 'test'
excel_name = 'C:/Users/anchor/Desktop/巡检结果.xls'


def ssh_connect(ip, password):
    cpu_per = memory_per = storage_per = ''
    print('polling ' + ip)
    port = 22
    user_name = 'apps'
    command = 'cd /apps;df -h /apps;free;sar -u 1 1'

    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(ip, port, user_name, password)
    stdin, stdout, stderr = ssh.exec_command(command)
    lines = stdout.readlines()

    for t_line in lines:
        if 'Average' in str(t_line):
            cpu_per = read_cpu(str(t_line))
            continue
        if 'Mem:' in str(t_line):
            memory_per = read_memory(str(t_line))
            continue
        if '/apps' in str(t_line):
            storage_per = read_storage(str(t_line))
            continue
    ssh.close()
    return cpu_per, memory_per, storage_per


def read_cpu(c_line):
    c_list = []
    for c_each in c_line.split(' '):
        if c_each == '':
            continue
        else:
            c_list.append(c_each)
    r_cpu = c_list[2] + '%'
    return r_cpu


def read_memory(m_line):
    m_list = []
    for m_each in m_line.split(' '):
        if m_each == '':
            continue
        else:
            m_list.append(m_each)
    memory = (int(m_list[2]) / int(m_list[1])) * 100
    r_memory = str(round(memory, 1)) + '%'
    return r_memory


def read_storage(s_line):
    storage = str(re.findall('[0-9]+%', s_line))
    r_storage = storage.strip('[').strip(']').strip('\'')
    return r_storage


def execute():
    source = open(ip_file)
    log = open(log_file, 'a')
    result_array = []
    error_array = []
    total = success = fail = 0
    for f_line in source.readlines():
        total = total + 1
        info = str(f_line).strip().split(' ')
        ip = info[0]
        psd = info[1]
        try:
            cpu, mem, sto = ssh_connect(ip, psd)
        except BaseException as e:
            error_array.append(ip)
            log.write(ip + ' error info: \n' + str(e) + '\r\n')
            fail = fail + 1
            continue
        item = [ip, cpu, mem, sto]
        result_array.append(item)
        success = success + 1
    source.close()
    log.close()
    return result_array, error_array, total, success, fail


def write_xls(array):
    write_book = xlwt.Workbook()
    sheet = write_book.add_sheet(sheet_name)
    i = j = 0
    for line in array:
        for item in line:
            sheet.write(i, j, item)
            j = j + 1
        j = 0
        i = i + 1
    write_book.save(excel_name)


if __name__ == '__main__':
    print('—————————Starting polling———————')
    result, error, total_num, success_num, fail_num = execute()
    write_xls(result)
    print('——————————End polling———————————')
    print('———————————Statistic————————————')
    print('success: ' + str(success_num) + '   fail: ' + str(fail_num) + '   total: ' + str(total_num))
    if len(error) != 0:
        print('——————————Failed ips————————————')
        for each in error:
            print(each)
    print('————————————————————————————————')

```
## 核心
解析此行命令返回的数据即可获得三个巡检值
```shell
cd /apps;df -h /apps;free;sar -u 1 1
```  
返回数据展示
```text
Filesystem                   Size  Used Avail Use% Mounted on
/dev/mapper/data01vg-appslv  197G   74G  113G  40% /apps
              total        used        free      shared  buff/cache   available
Mem:        8010808     5288156      609260      162016     2113392     1311712
Swap:       8388604      605336     7783268
Linux 3.10.0-514.16.1.el7.x86_64 (mvxl9136) 	10/09/2020 	_x86_64_	(4 CPU)

02:45:41 PM     CPU     %user     %nice   %system   %iowait    %steal     %idle
02:45:42 PM     all      0.50      0.00      0.00      0.00      0.00     99.50
Average:        all      0.50      0.00      0.00      0.00      0.00     99.50
```
## 解析
### CPU
### 内存
### 硬盘
## 小知识
Linux在一行执行多条命令
![]({{ "/assets/img/20200930/20200930.jpg"}})
