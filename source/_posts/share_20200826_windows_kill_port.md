---
layout:     post
title:      Windows 系统下关闭被占用端口的进程
date:       2020-08-26 19:54:13
author:     liangtong
catalog: true
categories: 分享
tags: shell

---



查看端口被占用情况，然后直接根据进程号杀死进程即可。涉及命令:

+ netstat -ano ： 查看所有端口号 
+ netstat -ano|findstr 端口号 ： 查看特定端口号使用情况
+ taskkill /pid 进程号 /f


```bash
# 查看19001端口被占用情况
netstat -ano|findstr  19001


# 关闭端口号应用，假设上步查询得到的进程号为5048
taskkill /pid 5048 /f

```









