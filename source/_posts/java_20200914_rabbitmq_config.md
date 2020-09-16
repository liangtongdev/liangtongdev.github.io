---
layout:     post
title:      RabbitMQ的安装与配置
date:       2020-09-14 18:54:13
author:     liangtong
catalog: true
categories: Java
tags: mq

---

​	**RabbitMQ** 是一个在AMQP协议标准基础上完整、可复用的消息系统。采用 Erlang 实现的工业级消息队列。

![rabbitmq.png](/post/java/20200915/rabbitmq.png)

### Windows上安装

#### 安装Erlang

下载地址：https://www.erlang.org/downloads
安装完成后设置系统变量 `ERLANG_HOME`

![erlang_home.png](/post/java/20200915/erlang_home.png)

修改环境变量path，增加Erlang变量至path，` %ERLANG_HOME%\bin;` 

#### 安装RabbitMQ

下载地址：https://www.rabbitmq.com/install-windows.html
安装完成后设置环境变量 `RABBITMQ_SERVER`

![rabbit_server.png](/post/java/20200915/rabbit_server.png)

修改环境变量path，增加rabbitmq变量至path， `%RABBITMQ_SERVER%\sbin;` 

切换到RabbitMQ安装目录sbin下，常用命令如下：

+ 通过 `rabbitmqctl status` 查看启动状态。

+ 通过 `rabbitmq-plugins.bat enable rabbitmq_management` 安装管理插件

+ 通过 `rabbitmq-server.bat` 启动服务


服务启动后，可以通过http://localhost:15672 进入控制管理页面，默认账号及密码均为guest。登陆之后，点击导航栏 「Admin」添加用户及配置`Virtual Host`权限

![rabbit_add_user.png](/post/java/20200915/rabbit_add_user.png)

可以简单通过管理端窗口直接添加用户（也可以通过命令行）

![rabbit_virtual_host_1.png](/post/java/20200915/rabbit_virtual_host_1.png)

配置 `virtual host` 权限

![rabbit_virtual_host_2.png](/post/java/20200915/rabbit_virtual_host_2.png)

最后通过 `rabbitmqctl.bat stop` 停止服务


### Mac上安装

#### 通过 homebrew 安装

```bash
> brew update
> brew install rabbitmq
```

#### 启动

RabbitMQ服务的脚本和CLI工具安装在`/usr/local/Cellar/rabbitmq`目录下。启动时可以切换到对应sbin目录下执行命令即可

```bash
> cd /usr/local/Cellar/rabbitmq/3.8.2/sbin
> ./rabbitmq-server
```

可以将rabbitmq工作sbin目录添加到环境变量文件` ~/.bash_profile`中，方便使用。





