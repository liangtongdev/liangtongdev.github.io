---
layout:     post
title:      全屏浏览网页
date:       2021-01-29 14:41:13
author:     liangtong
catalog: true
categories: 分享
tags: chrome

---


使用Chrome浏览器默认全屏浏览网页，配置开机启动打开网页

#### 桌面快捷方式

依次点击Chrome浏览器右上角 <更多> -> <更多工具> -> <创建快捷方式...>

#### 默认全屏浏览

快捷方式上右键点击，依次选择 <属性> -> <快捷方式> ， 在 <目标栏> 输入框内添加参数 `--kiosk "http://www.liangtong.site"` 
例如：`"C:\Program Files\Google\Chrome\Application\chrome_proxy.exe" --kiosk "http://www.liangtong.site"`

#### 开机启动

`Windows + R` 打开运行，输入 `%USERPROFILE%\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup` 打开文件夹。
将快捷方式复制到文件夹內即可。











