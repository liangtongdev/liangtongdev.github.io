---
layout:     post
title:      接受其他应用分享的文件
date:       2018-08-23 16:00:00
author:     liangtong
catalog: true
categories: iOS
tags: objc

---

当某个应用无法查看/编辑某个文件时，可能会考虑使用其他应用打开。但是iOS沙盒机制前提下，我们对文件操作变得比较麻烦，本文介绍如何注册系统，打开其他应用分析的文件。

![](/post/note/share_file_20180823.png)

如何分享，可参照之前的文章。附上[链接](https://liangtongdev.github.io/2018/08/01/ios_node_share_to_app_20180801/)



 <!-- more -->

 ### 配置程序plist文件

 添加如下以下项

 ```Objective-C
    <key>CFBundleDocumentTypes</key>
    <array>
        <dict>
            <key>CFBundleTypeName</key>
            <string>Office</string>
            <key>LSItemContentTypes</key>
            <array>
                <string>public.item</string>
                <string>public.content</string>
                <string>public.text</string>
            </array>
        </dict>
    </array>
 ```

  ### 实现协议回调

  实现 **AppDelegate** 的方法。

```Objective-C
 
- (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options{
    if (self.window) {
        if (url) 
            // path 比如：file:///private/var/mobile/Containers/Data/Application/xxx/Documents/Inbox/xxx.pdf
            NSString *path = url.absoluteString; // 完整的url字符串
//            NSString* decodePath = [path stringByRemovingPercentEncoding];
            if ([path hasPrefix:@"file://"]) { // 通过前缀来判断是文件
                /**
                 * 处理文件，比如保存，展示等
                 **/
                
                return YES;
            }
        }
    }
    return YES;
}
```
