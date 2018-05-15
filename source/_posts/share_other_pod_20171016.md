---
layout:     post
title:      CocoaPods个人代码组件管理
date:       2017-10-16 14:25:19
author:     liangtong
categories: share
tags: CocoaPods

---

[Cocoapods](https://cocoapods.org/)是非常好用的一个iOS依赖管理工具，使用它可以方便的管理和更新项目中所使用到的第三方库，本节介绍如何将自己的组件代码交由它去管理。

### 创建Git仓库

  根据GitHub提示操作即可，建立自己的Git仓库。 

### 创建podspec文件

  终端直接执行以下命令，创建Pod项目工程文件，例如要创建一个叫做 **LTChat** 的Pod工程，命令执行成功后，会看到一个叫做 **LTChat.podspec** 的文件。

```Shell
pod spec create "LTChat"
```

![](/post/share/github_pod_create_1.png)


<!-- more -->

### 编辑podspec文件  

编辑文件时注意编辑工具的使用，Mac上默认的文本编辑文件编辑时很容易出现编码问题。
修改Pod文件的配置信息，摘要如下：

```Shell
  # ―――  Spec Metadata  ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  s.name         = "LTChat"
  s.version      = "0.0.1"
  #  summary 总结
  s.summary      = "Chat use XMPP framework and WebRTC framework."
  #  description 描述，与summary不能相同
  s.description  = <<-DESC
基于XMPP和WebRTC框架进行聊天组件。
                   DESC

  s.homepage     = "https://github.com/l900416/LTChat"
  # ―――  Spec License  ――――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  s.license      = "MIT"
  # ――― Platform Specifics ――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  s.platform     = :ios, "8.0"

  #  When using multiple platforms
  s.ios.deployment_target = "8.0"
  # ――― Source Location ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  s.source       = { :git => "https://github.com/l900416/LTChat.git", :tag => "#{s.version}" }
  # ――― Source Code ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  s.source_files  = "LTChat", "LTChat/**/*.{h,m}"
  s.exclude_files = "LTChat/Exclude"

  s.public_header_files = "LTChat/**/*.h"
  # ――― Resources ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  s.resources = "LTChat/**/*.{xib}","LTChat/**/*.{png}"

  # ――― Project Linking ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  # s.frameworks = "SomeFramework", "AnotherFramework"
  # s.libraries = "iconv", "xml2"

  # ――― Project Settings ――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  # s.requires_arc = true
  s.dependency "XMPPFramework", "~> 3.7.0"
  s.dependency "GoogleWebRTC"

```

### 建立TAG

使用 **git tag** 和 **git push** 命令建立TAG

![](/post/share/github_pod_create_2.png)


### 验证Pod文件

使用 **pod lib lint** 或 **pod spec lint**  验证Pod文件格式是否正确，可以添加 **--verbose** 获取更多的信息，添加 **--no-clean** 不对验证的工程进行清理，方便查看工程验证错误原因。两者的区别：

  * pod lib lint 是只从本地验证pod是否能够通过验证
  * pod sepc lint 是从本地和远程来验证pod是否通过验证。

### 发布

使用 **pod trunk** 发布程序，使用到的命令如下：

  * pod trunk register 邮箱 '用户名'：注册，之后在邮件中进行验证。
  * pod trunk me ： 查看个人信息，包括该邮件下已经注册的代码组件、个人信息、登陆session等。
  * pod trunk push *.podspec : 发布podspec
  * 等待审核
  * pod search： 检查是否发布成功

### 更新

建立TAG，并发布。
![](/post/share/github_pod_update_1.png)
![](/post/share/github_pod_update_2.png)

### 问题记录

#### pod lib lint验证失败

如果出现验证失败，则需要添加参数 **--verbose** 查看详细错误原因 。例如

```Objective-C
The following build commands failed:
Ld /Users/liangtong/Library/Developer/Xcode/DerivedData/App-eowwjkrvluwvfdhhkljszfucgimo/Build/Intermediates.noindex/Pods.build/Release-iphonesimulator/LTChat.build/Objects-normal/i386/LTChat normal i386
(1 failure)


[!] LTChat did not pass validation, due to 1 error and 71 warnings.
```

原因是由于谷歌官方iOS版本的 **GoogleWebRTC** 仅支持处理器**x86_64 armv7 arm64**导致

#### Swift Language Version

根据失败的log日志，确认失败原因，比如LTChat依赖于XMPPFramework，而XMPPFramework又依赖了KissXML。此时编译的时候出现了Swift版本不明确的错误。

```Objecitve-C
=== BUILD TARGET KissXML OF PROJECT Pods WITH CONFIGURATION Release ===

Check dependencies
The “Swift Language Version” (SWIFT_VERSION) build setting must be set to a supported value for targets which use Swift. This setting can be set in the build settings editor.
The “Swift Language Version” (SWIFT_VERSION) build setting must be set to a supported value for targets which use Swift. This setting can be set in the build settings editor.

** BUILD FAILED **

The following build commands failed:
Check dependencies
```

解决办法，指定 **Swift Language** 版本
>  pod lib lint --swift-version=4.0


#### 上传后查询不到最新版本

解决办法：清理缓存，重新setup

```Shell
  rm -rf ~/.cocoapods/repos/master
  pod setup
```


