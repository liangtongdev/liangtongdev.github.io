---
layout:     post
title:      mqtt之apache-apollo服务搭建
date:       2019-03-14 17:50:12
author:     liangtong
catalog: true
categories: Java
tags: mqtt

---


#### MQTT

MQTT是工作在TCP/IP协议族上的发布/订阅范式消息的协议。
跟HTTP协议一样，不能直接拿来使用，需要找实现这个协议的库或者服务器来运行。
本文介绍服务器apache-apollo的配置。



 <!-- more -->



#### 下载

从官网http://activemq.apache.org/apollo/index.html下载服务器安装软件。下载后解压，如下图所示

![unzip.png](https://upload-images.jianshu.io/upload_images/16014538-073a65cb93526141.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)


#### 安装

##### JAVA_HOME

如果已配置过 `JAVA_HOME`, 本步骤可以省略。
具体步骤如下图，若`.bash_profile`文件不存在，则可以通过`touch .bash_profile`命令创建
![bash_profile_1.png](https://upload-images.jianshu.io/upload_images/16014538-a8b2ce17b108ed40.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

![bash_profile_2.png](https://upload-images.jianshu.io/upload_images/16014538-37d6a65515bd9a47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

保存文件后，使用命令让配置的环境变量生效
```bash
source .bash_profile  
```

##### broker服务
bin 目录下包含两个文件，运行 `apollo`文件，创建服务器实例。
服务器实例包含了所有的配置，运行时数据等，并且和一个服务器进程关联。
![apollo create.png](https://upload-images.jianshu.io/upload_images/16014538-05be2475d221674b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

create mybroker之后会在bin目录下生成mybroker文件夹。 


#### 配置

##### 登录名和密码

在`/Users/xxx/Documents/w_dev/tool/apache-apollo-1.7.1/bin/mybroker/etc/users.properties`中进行配置：

![users-properties.png](https://upload-images.jianshu.io/upload_images/16014538-79c3af8e20880ae3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

##### 启动服务

使用命令`./apollo-broker run`启动

![apollo-broker run.png](https://upload-images.jianshu.io/upload_images/16014538-2dcec9c3fe0c20bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)

##### 登录服务验证

![login-success.png](https://upload-images.jianshu.io/upload_images/16014538-f6de6840682fb6ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/840)


OK，搞定。下一篇介绍SpringBoot集成MQTT消息推送功能！


