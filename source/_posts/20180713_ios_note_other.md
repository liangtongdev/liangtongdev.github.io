---
layout:     post
title:      iOS程序目录结构
date:       2018-07-13 15:00:00
author:     liangtong
catalog: true
categories: iOS
tags: note

---

使用**MVMCV**模式，记录初始目录结构。
虽然后来的开发过程中，借助**Pods**工具，将共通部分进行了分离统一维护。


<!-- more -->


#### Common

 > 存放系统宏定义、配置文件等

#### View

+ View
 > 业务相关自定义View
+ Storyboard
 > Storyboard文件
+ Common
 > 共通的部分，比如消息相关

#### Container

+ Controller
 > 业务页面容器
+ Base
 > 基类
+ Common
 > 共同部分，比如文件、网页容器等

#### ViewModel

+ Util
 > 比如错误码处理
+ ViewModel
 > 各业务对应的ViewModel
+ Common
 > 网络访问，数据解析等

#### Model

+ Database
 > 数据库所有操作
+ FileManager
 > 沙盒操作

#### Verdor

 > 除pod外的第三方代码，优先使用pod维护

#### Resources

 > 资源文件