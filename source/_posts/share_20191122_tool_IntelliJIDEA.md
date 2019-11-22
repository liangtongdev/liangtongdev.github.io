---

layout:     post
title:      IntelliJ IDEA Activation
date:       2019-11-22 13:25:19
author:     liangtong
categories: 分享
tags: tool


---

**针对IntelliJ IDEA 2019.2版本**，本文操作仅供学习交流，如果感觉使用方面建议购买正版[IntelliJ IDEA](https://www.jetbrains.com/idea/)。



### 前期准备

获取文件**jetbrains-agent.jar**和**对应的license**。（详细可留言mail索取）

### 具体操作 

 * 下载**IntelliJ IDEA**，安装。
 * 然后打开**应用程序 -> IntelliJ IDEA -> *包含内容 -> Contents -> bin**
 * 试用文件编译器打开文件`idea.vmoptions`，在文件末尾添加一下内容

```idea.vmoptions
-javaagent:/xxxx/jetbrains-agent.jar # xxxx为具体的路径
```

> **注：jetbrains-agent.jar路径一定要填写正确**

 * 打开工具**IntelliJ IDEA**，然后在弹框中依次选择 **Configure -> Manage License...** （也可通过其他路径，总之是打开license认证窗口）
 * 选择 **Activation code** 选项，然后输入框中输入具体License即可

> 具体License内容请参照文件，需要注意的是，如果之前已经有license，请先点击移除然后重新认证

+ 点击OK，见证奇迹的时刻！



### 附

**此方法同样适用于Windows版本的IntelliJ IDEA！区别是mac下`idea.vmoptions` 文件，对应windows下的是`idea.exe.vmoptions`文件**



**GoLand工具也是同样的思路**



