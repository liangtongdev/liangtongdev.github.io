---
layout:     post
title:      Mysql blocked
date:       2019-08-23 09:07:43
author:     liangtong
catalog: true
categories: 数据库
tags: mysql

---


​	使用mysql数据库进行开发的时候，控制台突然出现以下错误

```bash
[ERROR] 2019-08-23 09:27:32 [Druid-ConnectionPool-Create-1704872741] c.alibaba.druid.pool.DruidDataSource - create connection SQLException, url: jdbc:mysql://xxxx:3306/xxxx?characterEncoding=utf-8&useSSL=false, errorCode 1129, state HY000
java.sql.SQLException: null,  message from server: "Host '192.168.xx.xx' is blocked because of many connection errors; unblock with 'mysqladmin flush-hosts'"
```

是同一IP的connection errors超出默认的最大值了。

解决方法：

+ 重启mysql服务即可

+ 命令行：flush hosts


