---
layout:     post
title:      可变参数函数
date:       2018-06-02 00:00:00
author:     liangtong
categories: iOS
tags: objc

---

当一个函数拥有不定的参数个数时，该函数为可变参数函数。例如NSString的stringWithFormat:方法；NSLog(NSString *format, ...)

<!-- more -->


#### 函数声明
多参数情况下，可变参数处于最后一个位置，其后紧跟 **,...** 修饰。
可以添加宏 **NS_REQUIRES_NIL_TERMINATION** 表示给该多参数传值的时候以 **nil** 结尾。
例如：

```Objective-C
/**
 * 特定位置展示Alert
 **/
+(void)showAlertOnViewController:(UIViewController*)viewController
                      sourceView:(UIView*)sourceView//iPad
                      sourceRect:(CGRect)sourceRect//iPad
                           style:(UIAlertControllerStyle)style
                           title:(NSString*)title
                         message:(NSString*)message
                         actions:(UIAlertAction*)action,... NS_REQUIRES_NIL_TERMINATION;
```

#### 函数实现
函数实现的时候需要理解几个关键字
 * va_list 指向变参的指针。
 * va_start 访问变长参数列表中的参数之前使用的宏。使用可变参数的第一个参数来初使化va_list指针。
 * va_arg va_list中参数的遍历，**不包括可变参数的第一个参数。**
 * va_end 清空参数列表，并置参数指针args无效
此处需要特别注意的就是**va_arg**宏是从可变参数列表的第二个参数开始的。
例如：
```Objective-C
+(void)showAlertOnViewController:(UIViewController*)viewController
                      sourceView:(UIView*)sourceView//iPad
                      sourceRect:(CGRect)sourceRect//iPad
                           style:(UIAlertControllerStyle)style
                           title:(NSString*)title
                         message:(NSString*)message
                         actions:(UIAlertAction*)action,... NS_REQUIRES_NIL_TERMINATION{

    
    UIAlertController *alertController = [UIAlertController alertControllerWithTitle:title message:message preferredStyle:style];
    UIPopoverPresentationController *popover = alertController.popoverPresentationController;
    if (popover) {
        popover.sourceView = sourceView;
        popover.sourceRect = sourceRect;
        popover.permittedArrowDirections = UIPopoverArrowDirectionAny;
    }
    
    if(action){//可变参数的第一个
        [alertController addAction:action];
        
        va_list list;//指向变参的指针
        va_start(list, action);//使用第一个参数来初使化list指针
        while (true) {
            UIAlertAction* otherAction = va_arg(list, UIAlertAction*);//后续参数遍历
            if (!otherAction) {
                break;
            }
            [alertController addAction:otherAction];
        }
        va_end(list);// 清空参数列表，并置参数指针args无效
    }
    
    dispatch_async(dispatch_get_main_queue(), ^{
        [viewController presentViewController:alertController animated:YES completion:nil];
    });
}
```

#### 函数调用

```Objective-C
    UIAlertAction* leaveAction = [UIAlertAction actionWithTitle:@"离开"
                                                          style:UIAlertActionStyleDefault
                                                        handler:^(UIAlertAction* action){}];
    UIAlertAction* stayAction = [UIAlertAction actionWithTitle:@"取消"
                                                         style:UIAlertActionStyleCancel
                                                       handler:nil];
    [LTxSipprPopup showAlertOnViewController:self
                                  sourceView:self.saveBtn
                                  sourceRect:self.saveBtn.frame
                                       style:UIAlertControllerStyleAlert
                                       title:@"提示"
                                     message:@"确定要离开当前页面吗？"
                                     actions:leaveAction,stayAction,nil];

```
