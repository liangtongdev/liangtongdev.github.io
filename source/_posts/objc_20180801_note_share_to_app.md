---
layout:     post
title:      打开移动应用
date:       2018-08-01 00:00:00
author:     liangtong
catalog: true
categories: iOS
tags: objc

---


场景描述： 

 > 应用A打开应用B，（并向应用B传递参数）。应用B监听相应方法（并处理相应参数）

 说明：**应用分享需要真机环境，模拟器不可用**

 ### 约定

 约定打开应用标识，比如：ccs.eep.sippr

 ### 应用A

 #### 配置程序

 需要在程序 **Info.plist** 文件中添加以下字段

 ```Objective-C
    <key>LSApplicationQueriesSchemes</key>
    <array>
        <string>ccs.eepm.sippr</string>
    </array>
 ```

 #### 打开方法

 URL前缀，必须以 **ccs.eepm.sippr://** 开头。

 iOS 10及之后使用以下方法打开

  ```Objective-C
      NSURL *url  = [NSURL URLWithString:@"ccs.eepm.sippr://"];
      if ([[UIApplication sharedApplication] canOpenURL:url]) {
           [[UIApplication sharedApplication] openURL:url options:@{} completionHandler:^(BOOL success) {
                NSLog(@"open status: %d",success);
           }];
      }else{
           NSLog(@"can not open ccs ...");
      }
  ```

 iOS 10 之前使用以下方法打开

   ```Objective-C
      NSURL *url  = [NSURL URLWithString:@"ccs.eepm.sippr://"];
      if ([[UIApplication sharedApplication] canOpenURL:url]) {
           BOOL success = [[UIApplication sharedApplication] openURL:url];
           NSLog(@"open status: %d",success);
      }else{
           NSLog(@"can not open ccs ...");
      }
   ```

   ####  参数传递

   参数传递类似HTTP GET请求的形式，例如传递的URL形式如下

```Objective-C
      ccs.eepm.sippr://com.liangtong/module1/?from=liangtong&value=test
```

 ### 应用B

#### 配置程序

 需要在程序 **Info.plist** 文件中添加以下字段

 ```Objective-C
    <key>CFBundleURLTypes</key>
    <array>
        <dict>
            <key>CFBundleURLSchemes</key>
            <array>
                <string>ccs.eepm.sippr</string>
            </array>
            </dict>
    </array>
 ```

 #### 处理请求

 需要在 **AppDelegate** 协议回调方法进行如下实现，例如

 ```Objective-C
 - (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary *)options{
     if ([[url scheme] isEqualToString:@"ccs.eepm.sippr"]) {
         NSLog(@"host = %@",[url host]);
         NSLog(@"path = %@",[url path]);
         NSLog(@"query = %@",[url query]);
         NSLog(@"relativePath = %@",[url relativePath]);
         NSLog(@"resourceSpecifier = %@",[url resourceSpecifier]);
     }
     return true;
 }
 ```

 控制台log信息如下：
 ```Objective-C
 2018-08-01 15:00:22.885475+0800 ALEForSH[1215:546607] host = com.liangtong
 2018-08-01 15:00:26.863281+0800 ALEForSH[1215:546607] path = /module1
 2018-08-01 15:00:26.863571+0800 ALEForSH[1215:546607] query = from=liangtong&value=test
 2018-08-01 15:00:26.863627+0800 ALEForSH[1215:546607] relativePath = /module1
 2018-08-01 15:00:26.863701+0800 ALEForSH[1215:546607] resourceSpecifier = //com.liangtong/module1/?from=liangtong&value=test
 ```

 #### 验证

为了防止被恶意打开，应用B可以对应用A的身份进行验证。比如通过 **url scheme** 的形式进行。

+ 首先验证应用来源URL的 **url scheme**
+ 其次验证应用来源Info.plist文件的 **CFBundleURLSchemes**

感觉微信就是这么搞的（猜测😄）


 ### 参数传递
 
复制粘贴板 -（淘宝😄）
 
 
