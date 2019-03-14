---
layout:     post
title:      位置服务授权适配
date:       2018-05-30 00:00:00
author:     liangtong
catalog: true
categories: iOS
tags: objc

---


iOS11系统针对位置服务授权字段描述进行了变更。iOS11之后如果不进行配置，则在使用位置时会提示如下信息

 使用iBeacon技术，在程序不打开的情况下对区域进行监听时，需要将位置使用方式设置为Always。控制台出现以下log信息：

 > This app has attempted to access privacy-sensitive data without a usage description. The app's Info.plist must contain both NSLocationAlwaysAndWhenInUseUsageDescription and NSLocationWhenInUseUsageDescription keys with string values explaining to the user how the app uses this data


意思是说在plist中需要设置请求位置服务的描述信息。字段名已经提供，只需要将info.plist中响应的字段修改下就可以了。

若需要适配多个系统，则需要在info.plist中有针对性的添加如下信息：
 * iOS11及之后系统

     * NSLocationAlwaysAndWhenInUseUsageDescription

 * iOS11之前系统

     * NSLocationAlwaysUsageDescription - 「始终」
     * NSLocationWhenInUseUsageDescription - 「使用应用期间」
