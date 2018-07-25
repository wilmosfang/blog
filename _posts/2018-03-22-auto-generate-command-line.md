---
layout: post
title: "Auto Generate Command Line"
author:  wilmosfang
date: 2018-03-22 16:42:47
image: '/assets/img/'
excerpt: '使用 Openpyxl 实现一个自动生成命令行的工具'
main-class: python
color: '#265277'
tags:
 - python
 - openpyxl
categories:
 - python
twitter_text: 'auto generate command line script by python'
introduction: 'simple script to generate command line automaticly by python Openpyxl'
---


## 前言

**[Openpyxl][openpyxl]** 是一个用来读写 **`Excel 2010 xlsx/xlsm/xltx/xltm`** 文件的开源库

> A Python library to read/write Excel 2010 xlsx/xlsm files

这里演示一下如何傅用 **[Openpyxl][openpyxl]** 来构建一个自动生成命令行的小脚本

> **Tip:** 当前的版本为 **openpyxl-2.5.1**

---

# 操作


## 环境

~~~
[root@56-201 ~]# hostnamectl 
   Static hostname: 56-201
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 33dc28f7e76c4903ad9b603b77e29a7c
           Boot ID: b278707b56304e11a4f30711cf56d76b
    Virtualization: kvm
  Operating System: CentOS Linux 7 (Core)
       CPE OS Name: cpe:/o:centos:centos:7
            Kernel: Linux 3.10.0-514.21.1.el7.x86_64
      Architecture: x86-64
[root@56-201 ~]# ip a 
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:0e:38:94 brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic enp0s3
       valid_lft 81534sec preferred_lft 81534sec
    inet6 fe80::2bb7:5b3:9584:d8eb/64 scope link 
       valid_lft forever preferred_lft forever
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP qlen 1000
    link/ether 08:00:27:bb:5d:54 brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.201/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:febb:5d54/64 scope link 
       valid_lft forever preferred_lft forever
4: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
       valid_lft forever preferred_lft forever
5: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc pfifo_fast master virbr0 state DOWN qlen 1000
    link/ether 52:54:00:16:5e:11 brd ff:ff:ff:ff:ff:ff
[root@56-201 ~]# 
~~~


## 表格

![openpyxl](/assets/img/openpyxl/openpyxl01.png)


## 安装 openpyxl

~~~
[root@56-201 ~]# pip install openpyxl
Collecting openpyxl
Collecting et-xmlfile (from openpyxl)
  Using cached et_xmlfile-1.0.1.tar.gz
Collecting jdcal (from openpyxl)
  Using cached jdcal-1.3.tar.gz
Building wheels for collected packages: et-xmlfile, jdcal
  Running setup.py bdist_wheel for et-xmlfile ... done
  Stored in directory: /root/.cache/pip/wheels/99/f6/53/5e18f3ff4ce36c990fa90ebdf2b80cd9b44dc461f750a1a77c
  Running setup.py bdist_wheel for jdcal ... done
  Stored in directory: /root/.cache/pip/wheels/0f/63/92/19ac65ed64189de4d662f269d39dd08a887258842ad2f29549
Successfully built et-xmlfile jdcal
Installing collected packages: et-xmlfile, jdcal, openpyxl
Successfully installed et-xmlfile-1.0.1 jdcal-1.3 openpyxl-2.5.1
[root@56-201 ~]# 
~~~


## 编写脚本


~~~
[root@56-201 sf_usb]# vim generate_option.py 
[root@56-201 sf_usb]# cat generate_option.py 
#!/usr/bin/env python3.6
# -*- coding: utf-8 -*-

#import
import os
from optparse import OptionParser
from openpyxl import load_workbook

#args parser
usage =  "Usage: %prog <-f xlsx> [options] arg"
parser = OptionParser(usage)
parser.add_option("-f","--xlsx",dest="xlsx",action="store",type="string",help="(mandatory)the xlsx file ready to read")
parser.add_option("-e","--command",dest="command",default="",action="store",type="string",help="command line before each option and its value, default is '%default'")
parser.add_option("-p","--prefix",dest="prefix",default="-",action="store",type="string",help="prefix of each option, default is '%default'")
parser.add_option("-c","--confix",dest="confix",default="=",action="store",type="string",help="confix between each option and its value, default is '%default'")
(options,args)=parser.parse_args()

#check args
if options.xlsx == None:
    exit("Error: you need to specified target xlsx file to be read")
if not os.path.exists(options.xlsx):
    exit("Error: %s not found"%options.xlsx)
    
#prepare args
prefix=str(options.prefix).strip()
confix=str(options.confix)
command=str(options.command).strip()

#read only mode load workbook
wb=load_workbook(filename=options.xlsx, read_only=True)

#get active work sheet (first work sheet)
ws=wb.active

#generate command lines
def generate_cli(worksheet):
    ws=worksheet
    for row in ws[2:ws.max_row]:
        if command != '':
            print('%s'%command,end=" ")
        for cv in map(lambda x,y: '%s%s%s"%s"'%(prefix,x,confix,y),[str(x.value).strip() for x in ws[1]],[str(y.value).strip() for y in row]):
            print('%s'%cv,end=" ")
        print("") 

#main
if __name__ == "__main__":
    generate_cli(ws)
[root@56-201 sf_usb]# 
~~~

## 测试脚本

~~~
[root@56-201 sf_usb]# ./generate_option.py -h
Usage: generate_option.py <-f xlsx> [options] arg

Options:
  -h, --help            show this help message and exit
  -f XLSX, --xlsx=XLSX  (mandatory)the xlsx file ready to read
  -e COMMAND, --command=COMMAND
                        command line before each option and its value, default
                        is ''
  -p PREFIX, --prefix=PREFIX
                        prefix of each option, default is '-'
  -c CONFIX, --confix=CONFIX
                        confix between each option and its value, default is
                        '='
[root@56-201 sf_usb]# ./generate_option.py --help
Usage: generate_option.py <-f xlsx> [options] arg

Options:
  -h, --help            show this help message and exit
  -f XLSX, --xlsx=XLSX  (mandatory)the xlsx file ready to read
  -e COMMAND, --command=COMMAND
                        command line before each option and its value, default
                        is ''
  -p PREFIX, --prefix=PREFIX
                        prefix of each option, default is '-'
  -c CONFIX, --confix=CONFIX
                        confix between each option and its value, default is
                        '='
[root@56-201 sf_usb]# ./generate_option.py 
Error: you need to specified target xlsx file to be read
[root@56-201 sf_usb]# ./generate_option.py -f /tmp/test.xlsx 
-a="1" -b="2" -c="3" -d="4" 
-a="5" -b="6" -c="7" -d="8" 
[root@56-201 sf_usb]# ./generate_option.py -f /tmp/test1.xlsx 
Error: /tmp/test1.xlsx not found
[root@56-201 sf_usb]# ./generate_option.py -f /tmp/test.xlsx -e cli
cli -a="1" -b="2" -c="3" -d="4" 
cli -a="5" -b="6" -c="7" -d="8" 
[root@56-201 sf_usb]# ./generate_option.py -f /tmp/test.xlsx -e cli -p --
cli --a="1" --b="2" --c="3" --d="4" 
cli --a="5" --b="6" --c="7" --d="8" 
[root@56-201 sf_usb]# ./generate_option.py -f /tmp/test.xlsx -e cli -p -- -c ' '
cli --a "1" --b "2" --c "3" --d "4" 
cli --a "5" --b "6" --c "7" --d "8" 
[root@56-201 sf_usb]# ./generate_option.py -f /tmp/test.xlsx -e cli -p -- -c :
cli --a:"1" --b:"2" --c:"3" --d:"4" 
cli --a:"5" --b:"6" --c:"7" --d:"8" 
[root@56-201 sf_usb]# ./generate_option.py -f /tmp/test.xlsx -e '   cli   '    -p '  ++    ' -c :
cli ++a:"1" ++b:"2" ++c:"3" ++d:"4" 
cli ++a:"5" ++b:"6" ++c:"7" ++d:"8" 
[root@56-201 sf_usb]# ./generate_option.py -f /tmp/test.xlsx -e cli  -c ' '
cli -a "1" -b "2" -c "3" -d "4" 
cli -a "5" -b "6" -c "7" -d "8" 
[root@56-201 sf_usb]# 
~~~

分别针对每个选项做了不同测试，结果都符合预期

可以使用此脚本轻松地批量生成命令了


---

# 总结

这个脚本是在将第一行作为参数名，后面每行作为参数值，然后接合指定的命令来拼接出一个完整的命令


* TOC
{:toc}


---



[openpyxl]:https://openpyxl.readthedocs.io/en/stable/




