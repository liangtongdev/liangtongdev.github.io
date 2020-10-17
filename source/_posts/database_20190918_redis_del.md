---
layout:     post
title:      Redis远程批量删除特定的keys
date:       2019-09-18 14:40:27
author:     liangtong
catalog: true
categories: 数据库
tags: redis

---

使用redis-cli批量删除远程服务器上特定的keys

#### 命令如下

```shell
./redis-cli -h IP -p PORT -a PASSWORD keys 'key*' | xargs  ./redis-cli -h IP  -p PORT -a PASSWORD del
```



+ IP : redis 服务器的ip地址
+ PORT ： redis服务器的端口
+ PASSWORD：redis服务的密码



例如，MAC系统上，命令如下：

```nginx config
$ redis-cli -h 192.168.1.76 -p 6379  keys 'andon_duration*' | xargs  redis-cli -h 192.168.1.76 -p 6379  del
```

