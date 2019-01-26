---
layout:     post
title:      沙盒及文件的读写
date:       2018-06-10 21:00:00
author:     liangtong
catalog: true
categories: iOS
tags: note

---


这是一篇几年前就应该记的笔记，涉及到iOS沙盒内文件管理，今天给补上。

### 沙盒 
iOS系统为了保证安全，文件夹使用沙盒机制。如果你试图通过记录绝对路径的形式来访问沙盒内的文件，结果可能会让你失望。
通常情况下，沙盒路径可以通过代码 **NSHomeDirectory()**获取。
沙盒目录文件下，包含三个文件夹。分别是
 
  + Documents
  + Library
  + tmp
  
  
  <!-- more -->
  

#### Documents

一般情况下存放程序产生的数据，比如程序内处理的图片。iTunes备份和恢复的时候会包括此目录。可以通过以下代码获取路径：

```Objective-C
//宏定义的形式
#define BIM_DOCUMENT_PATH ([NSHomeDirectory() stringByAppendingPathComponent:@"Documents"])
//方法调用的形式
NSString *path = NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES).lastObject;
```

> 此处需要注意的是，该目录下不要保存从网络上下载的文件，否则上不了架


#### Library

Library目录下包含两个子目录。路径获取方法同Documents类似，使用 **NSLibraryDirectory**。
 + Caches
 + Preferences
 
 ##### Caches
 此目录用来保存程序需要持久化的数据，比如下载的数据、网络请求的临时数据等。这些数据需要用户负责删除。iTunes备份时，不备份该目录。
 
##### Preferences
此目录保存程序等偏好设置，系统的「设置」应用会在该目录下查找应用的设置信息。iTunes会同步备份该目录。在该目录下不能直接创建文件，而是通过系统的方法来获取并设置文件。

```Objective-C
NSString *path = [NSSearchPathForDirectoriesInDomains(NSLibraryDirectory, NSUserDomainMask, YES).lastObject stringByAppendingPathComponent:@"Preferences"];
```

#### tmp
此目录下保存程序运行时产生的临时数据，比如文件的下载，使用完毕后将对应的数据删除即可。通过**NSTemporaryDirectory()** 获取。程序没有运行时，系统也会清理该目录下的文件。iTunes同步备份时不会备份该目录。

> 如果写过断点续下载，对该目录应该比较熟悉。


### 文件读写

沙盒内文件读写使用**NSFileManager**来管理。

#### 创建目录

```Objective-C
NSString* path = ...;
NSFileManager *fileManager = [NSFileManager defaultManager];
BOOL fileExists = [fileManager fileExistsAtPath:path];
if (!fileExists) {//判断文件是否存在
    [fileManager createDirectoryAtPath:path withIntermediateDirectories:YES attributes:nil error:nil];//创建文件夹
}
```

#### 获取目录下文件

```Objective-C
NSFileManager *fileManager = [NSFileManager defaultManager];
NSString* folderPath = ...;
NSArray* fileList =[fileManager contentsOfDirectoryAtPath:folderPath error:nil];
NSMutableArray* pathArray = [[NSMutableArray alloc] init];
for (NSString* fileName in fileList) {
    NSString* filePath = [folderPath stringByAppendingPathComponent:fileName];
    if (filePath) {
        [pathArray addObject:filePath];
        }
}
```

#### 删除文件

```Objective-C
NSString* path = ...;
NSFileManager *fileManager = [NSFileManager defaultManager];
[fileManager removeItemAtPath:filePath error:nil];
```

#### 移动文件

```Objective-C
[[NSFileManager defaultManager] moveItemAtPath:fromPath toPath:destPath error:&error];
```

>  注：fromPath是位于沙盒内的路径。如果试图从Bundle中移动文件，那么你是没有权限滴！！

#### 文件大小

```Objective-C
NSString* path = ...;
NSFileManager *fileManager = [NSFileManager defaultManager];
long long size = [[fileManager attributesOfItemAtPath:filePath error:nil] fileSize];
```

