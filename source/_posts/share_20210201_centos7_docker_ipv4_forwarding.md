---
layout:     post
title:      IPv4 forwarding is disabled. Networking will not work. 
date:       2021-02-01 14:49:21
author:     liangtong
catalog: true
categories: 分享
tags: docker

---



CentOS 7.5私有化服务部署环境，服务器突然断电（据说是断电，也可能是服务器做了什么配置策略调整）后。服务不能正常访问，使用docker启动镜像服务提输出以下日志：

```bash
WARNING: IPv4 forwarding is disabled. Networking will not work.
```

网上查了下需要做以下配置调整：

+ **/usr/lib/sysctl.d/00-system.conf** 文件中，追加 **net.ipv4.ip_forward=1**
+ 重启docker和网络服务 **systemctl restart network && systemctl restart docker**。 如果配置了开机启动，也可以直接进行**reboot**操作











