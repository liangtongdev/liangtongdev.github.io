---
layout:     post
title:      文件的查看
date:       2017-06-19 21:25:19
author:     liangtong
catalog: true
categories: 前端
tags: 笔记

---

* content
{:toc}

​	使用第三方应用预览文件。



### 前言   

对于特定格式（比如DWG等）文件的预览，没有必要耗费代价去自行开发功能，此时可能需要使用第三方应用来打开特定格式的文件。本文中记录三种方式：UIDocumentInteractionController、UIActivityViewController和特定第三方分享方式。

### UIDocumentInteractionController    

界面表现上和AirDrop长得很像，这个也可以进行AirDrop蓝牙分享很强大的功能，与之不同的是，在与文件类型关联的备选APP列表中，会有类似 **[拷贝至 微信]** 应用的选择，如下：   
![](https://l900416.github.io/img/post/iOS/Objective-C/file_preview_1.png)    
#### 创建对象   
```Objective-C
//引用
@property (nonatomic, strong) UIDocumentInteractionController* docInteractionController;


//初始化及动作定义
NSURL* filePath = xxx;//文件路径
_docInteractionController = [UIDocumentInteractionController  interactionControllerWithURL:filePath];
_docInteractionController.delegate = self;
[_docInteractionController presentOpenInMenuFromRect:CGRectZero
                                              inView:self.view
                                            animated:YES];
```

#### 实现常用协议
```Objective-C
#pragma mark - UIDocumentInteractionControllerDelegate
-(void)documentInteractionController:(UIDocumentInteractionController *)controller willBeginSendingToApplication:(NSString *)application {//将要发送的应用
}

//下面是他的代理方法

-(void)documentInteractionController:(UIDocumentInteractionController *)controller didEndSendingToApplication:(NSString *)application{//已经发送的应用

}

-(void)documentInteractionControllerDidDismissOpenInMenu:(UIDocumentInteractionController *)controller{//dismiss
}
```


### UIActivityViewController    
与UIDocumentInteractionController不同的是，UIActivityViewController无类似 **[拷贝至 微信]** 应用的选择。其界面和备选应用等可自行定制(屏蔽)，如下：    

![](https://l900416.github.io/img/post/iOS/Objective-C/file_preview_2.png)    

```Objective-C
    NSURL* fileURL = [NSURL URLWithString:@"http://xxx.dwg"];

    //注：activityItems可以是包含文字、图片、URL地址的数组，至少包含一项；applicationActivities参数可以对平台进行自定义
    UIActivityViewController *activity = [[UIActivityViewController alloc] initWithActivityItems:@[_fileURL]
                                                                           applicationActivities:nil];
    activity.excludedActivityTypes = @[UIActivityTypeAirDrop];//可用于屏蔽掉的应用列表，参照UIActivityType


    UIPopoverPresentationController *popover = activity.popoverPresentationController;
    if (popover) {
        popover.sourceView = self.shareBtn;
        popover.permittedArrowDirections = UIPopoverArrowDirectionUp;
    }
    [self presentViewController:activity animated:YES completion:nil];

```   


![](https://l900416.github.io/img/post/iOS/Objective-C/file_preview_3.png) 


### 分享   

第三方平台比如微信、新浪微博等平台的分享，集成其提供的SDK即可。此处不再赘述。若需要多分享平台集成，建议使用第三方平台比如友盟、ShareSDK等
