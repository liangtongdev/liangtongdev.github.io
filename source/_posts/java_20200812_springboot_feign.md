---
layout:     post
title:      SpringBoot Feign Post 参数过长
date:       2020-08-12 12:14:13
author:     liangtong
catalog: true
categories: Java
tags: note

---



SpringCloud微服务(SpringBoot项目)之间使用`feign`进行服务调用，在使用`post`请求时出现参数长度过长的错误。目前的解决方法是在工程配置文件`yaml 或 yml`中添加配置。具体值自己计算配置：


```yml
server:
  max-http-header-size: 102400
```

原因大概是 Spring feign post请求把参数写到了url上，导致长度限制8k




