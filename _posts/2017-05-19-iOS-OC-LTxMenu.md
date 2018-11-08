---
layout:     post
title:      LTxMenu
date:       2017-05-19 18:10:12
author:     liangtong
catalog: true
categories: 前端
tags: PopMenu 

---

* content
{:toc}

**[ Update v2.0.0 ]** Similar to Facebook News Feed , Alipay Life , QZone and other social applications . click a drop-down button to display more functions



### Introduction

​         In Facebook News Feed , Alipay Life , QZone and other social applications , they all contain a function drop-down button which would show a list of more functions when taped . I didn’t find any on GitHub , so I wrote a similar UI controls myself using Objective-C. ：

![](https://raw.githubusercontent.com/l900416/LTxMenu/master/screenshots/1.png)

>>GitHub link : <a href="https://github.com/l900416/LTxMenu">LTxMenu</a>

### Get Start
> * Manually add the files into your Xcode project.
> * LTxMenu is available as LTxMenu in Cocoapods.


### How To Use
​	Create a LTxMenuView Object:

```Objective-C
/**
* class instance method with dataSource and delegate. you can also create with [[LTxMenuView alloc] init] then set the dataSource and the delegate.
**/
+ (instancetype)instanceWithDataSource:(id <LTxMenuViewDataSource>)dataSource delegate:(id <LTxMenuViewDelegate>)delegate;
/**
* show menuView in viewController from a special position.
* @param viewController the menuview 's container
* @param position the menuview 's arrow direction
**/
- (void)showMenu:(UIViewController*)viewController from:(CGRect)position;
/**
* hide the menuview. usually you did not need to call this method
**/
- (void)dismissMenu;
```
​	DataSource && Delegate

```Objective-C

#pragma mark LTxMenuViewDelegate
@protocol LTxMenuViewDelegate<NSObject>

@optional
/**
* called when a specified index was selected.
**/
-(void)didSelectRowAtIndex:(NSInteger)index;

/**
* called when a specified accessoryView was selected.
**/
-(void)didSelectAccessoryView:(UIView*)accessoryView
atIndex:(NSInteger)index;
@end

#pragma mark LTxMenuViewDataSource
@protocol LTxMenuViewDataSource<NSObject>

@required
/**
* Returns the number of rows
**/
- (NSInteger)numberOfRows;

@optional
/**
* Returns the height of specified index. default value is 50.
**/
- (CGFloat)heightForRowAtIndex:(NSInteger)index;

/**
* Returns the attributedTitle of specified index.
**/
- (NSAttributedString*)attributedTitleForRowAtIndex:(NSInteger)index;

/**
* Returns the image of specified index.
**/
- (UIImage*)imageForRowAtIndex:(NSInteger)index;

/**
* Returns the accessoryViews placed at the end of specified index.
**/
- (NSArray<UIView*>*)accessoryViewsAtIndex:(NSInteger)index;
@end;
```

### How it works
​	LTxMenu use **LTxMenuItem** class to config view showed in row. and **LTxMenuView** class with callback methods to calculate the UI performance .  draw arrow and the border use  **UIBezierPath** class . and the show/hide animation use **UIKit**.

### Reference

​	https://github.com/kolyvan/kxmenu









