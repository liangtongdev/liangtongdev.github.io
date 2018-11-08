---
layout:     post
title:      LTxMenu
date:       2016-10-20 15:10:12
author:     liangtong
catalog: true
categories: 前端
tags: PopMenu 

---

* content
{:toc}

**[ Update v1.0.0 ]** Similar to Facebook News Feed , Alipay Life , QZone and other social applications . click a drop-down button to display more functions



### Introduction

​         In Facebook News Feed , Alipay Life , QZone and other social applications , they all contain a function drop-down button which would show a list of more functions when taped . I didn’t find any on GitHub , so I wrote a similar UI controls myself using Objective-C. ：

![](https://raw.githubusercontent.com/l900416/LTxMenu/64a7706ae5c6fbde8e7b5f2eb6706b6f56795b05/screenshots/1.gif)

>>GitHub link : <a href="https://github.com/l900416/LTxMenu">LTxMenu</a>

### Get Start
> * Manually add the files into your Xcode project.
> * LTxMenu is available as LTxMenu in Cocoapods.


### How To Use
​	Create a LTxMenuView Object:

```Objective-C
    @property (nonatomic, strong)LTxMenuView* menuView;
```
​	I was too lazy to write a protocol 😄，use callback methods：

```Objective-C
    _menuView = [[LTxMenuView alloc] init];//init
    __weak __typeof(self) weakSelf = self;
    _menuView.numberOfRows = ^(){//row numbers
        return (int)weakSelf.menuItems.count;
    };
    _menuView.heightForRow = ^(NSInteger row){//height of a row
        return 50.f;
    };
    _menuView.rowAtIndex = ^(NSInteger row){//the content of a row
        NSDictionary* menuItem = [weakSelf.menuItems objectAtIndex:row];
        return [LTxMenuItem menuItemWithImage:[menuItem objectForKey:@"image"]
                                        title:[menuItem objectForKey:@"title"]
                                rightBtnItems:[menuItem objectForKey:@"more"]//An array contains subClass of UIView
                                     tapBlock:^(NSString *identifier) {
                                         NSLog(@"tap at %@",identifier);
                                         __strong __typeof(weakSelf)strongSelf = weakSelf;
                                         [strongSelf.menuView dismissMenu];
                                     }];
    };
    [_menuView showMenuInView:self.view
                     fromRect:sender.frame];
```

### How it works
​	LTxMenu use **LTxMenuItem** class to config view showed in row. and **LTxMenuView** class with callback methods to calculate the UI performance .  draw arrow and the border use  **UIBezierPath** class . and the show/hide animation use **UIKit**.

### Reference

​	https://github.com/kolyvan/kxmenu









