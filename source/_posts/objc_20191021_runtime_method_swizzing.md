---
layout:     post
title:      iOS Method Swizzling应用
date:       2019-10-21 10:50:12
author:     liangtong
catalog: true
categories: iOS
tags: runtime

---


利用runtime `Method Swizzling` 解决实际问题的场景。 **慎用，慎用！**




####  iOS13上，模态弹窗 ViewController 默认样式改变。

模态弹窗属性 UIModalPresentationStyle 在 iOS 13 下默认被设置为 UIModalPresentationAutomatic新特性，展示样式更为炫酷，同时可用下拉手势关闭模态弹窗。

```objective-c
#import <objc/runtime.h>
@implementation UIViewController (ChangePresent)

+(void)load{
    static dispatch_once_t onceTokenChangePresent;
    dispatch_once(&onceTokenChangePresent, ^{
        Class class = [self class];
        Method method1 = class_getInstanceMethod(class, @selector(presentViewController:animated:completion:));
        Method method2 = class_getInstanceMethod(class, @selector(ltx_presentViewController:animated:completion:));
        method_exchangeImplementations(method1, method2);
        //以下效果相同
//        IMP imp1 = method_getImplementation(method1);
//        IMP imp2 = method_getImplementation(method2);
//        method_setImplementation(method1, imp2);
//        method_setImplementation(method2, imp1);
    });
}

- (void)ltx_presentViewController:(UIViewController *)viewControllerToPresent animated: (BOOL)flag completion:(void (^)(void))completion{
    viewControllerToPresent.modalPresentationStyle = UIModalPresentationFullScreen;
    [self ltx_presentViewController:viewControllerToPresent animated:flag completion:completion];
}
@end
```


####  App换肤方案

修改默认的`imageNamed:`方法，比如对图片整体添加前缀

```objective-c
#import <objc/runtime.h>
@implementation UIImage (ChangeNamed)

+ (void)load {
    static dispatch_once_t onceTokenChangeNamed;
    dispatch_once(&onceTokenChangeNamed, ^{
        Class class = [self class];
        Method oldMethod1 = class_getClassMethod(class, @selector(imageNamed:));
        Method newMethod1 = class_getClassMethod(class, @selector(ltx_imageNamed:));
        method_exchangeImplementations(oldMethod1, newMethod1);//替换方法
    });      
}

+(UIImage*) ltx_imageNamed:(NSString*) imageName{
    NSString* grayImageName = [NSString stringWithFormat:@"gray_%@",imageName];
    return [self ltx_imageNamed:grayImageName];
}
@end
```