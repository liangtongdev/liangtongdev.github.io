---
layout:     post
title:      iOS企业级应用ipa文件签名过期时间检查与重签名
date:       2021-09-24 08:43:12
author:     liangtong
catalog: true
categories: iOS
tags: 笔记

---


iOS企业级应用在启动时，苹果会对签名进行验证，如果验证失败，则会出现应用无法打开不可再用的提示。




####  ipa文件签名过期检查

XCode在进行ipa文件打包的时候，会优先使用本地的缓存证书。结果就是将本地证书的过期时间会打包到ipa文件中，这就会导致**明明刚打包的企业级ipa文件，就不能安装（或无法启动了）**

检查ipa文件过期时间步骤如下：

+ 重命名 ipa 文件，修改后缀为zip，例如：AAA.ipa -> AAA.zip
+ 解压文件（解压后为Payload）文件夹
+ 打开对应Payload目录中的App文件目录（例如：cd /Users/liangtong/Downloads/Payload/AAA.app）
+ 使用命令 'security cms -D -i embedded.mobileprovision' 查看签名情况
+ 查看证书过期时间（对应 ExpirationDate 项内容）

####  如何避免

+ 企业级开发证书续费后，重新打包发布应用
+ 每次打包前，清理证书缓存证书文件。位置：'~/Library/MobileDevice/Provisioning Profiles'




####  iOS企业帐号对ipa重新签名流程

前提：需要企业级应用发布证书（例如：AAA.mobileprovision），具体需要登录苹果开发者网站，新建 In House Dsitributation Profile；
应用的标识，例如：U4Y5XJ6M4W.com.xxx.AAA；
苹果发布证书。

+ 解压ipa文件，得到payload目录
+ 删除 Payload/AAA.app/_CodeSignature
+ 替换文件：cp AAA.mobileprovision Payload/AAA.app/embedded.mobileprovision
+ 准备Entitlements.plist文件，内容如下

``` xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">

<plist version="1.0"> 
  <dict> 
    <key>application-identifier</key>  
    <string>U4Y5XJ6M4W.com.xxx.AAA</string>  
    <key>com.apple.developer.team-identifier</key>  
    <string>U4Y5XJ6M4W</string>  
    <key>get-task-allow</key>  
    <false/>  
    <key>keychain-access-groups</key>  
    <array> 
      <string>U4Y5XJ6M4W.com.xxx.AAA</string> 
    </array> 
  </dict> 
</plist>

```
> 其中application-identifier的值要对应mobileprovision里的应用标识为： U4Y5XJ6M4W.com.xxx.AAA
com.apple.developer.team-identifier, 团队标识为： U4Y5XJ6M4W
keychain-access-groups, 数组的第一项，建议与 application-identifier 一样。

+ 重新签名ipa文件 `codesign -f -s "iPhone Distribution: xxxxx,Inc." --entitlements Entitlements.plist Payload/AAA.app/`
> 建议：苹果发布证书的名称 iPhone Distribution: xxxxx,Inc. 从钥匙串访问中复制，因为中间空格不同可能会引起错误

+ 压缩ipa包：zip -r new_AAA.ipa Payload

