---
layout:     post
title:      CCS中地图模式的切换
date:       2016-11-21 15:33:19
author:     liangtong
categories: summary
tags: ccs

---


#### 描述
ALE中，地图定位模式包括普通导览模式、普通定位模式、普通指向模式、导航模式、导航指向模式、导航导览模式。
模式切换的时机有两种：点击模式切换按钮、操作地图画面。

 + 点击模式切换按钮时，地图模式如下变换、
    + 普通导览模式 ⇀ 普通定位模式 ⇌ 普通指向模式
    + 导航导览模式 ⇀ 导航模式 ⇌ 导航指向模式
 + 操作地图时，地图模式如下变换
    + 普通定位模式/普通指向模式 ⇀ 普通导览模式
    + 导航模式/导航指向模式 ⇀ 导航导览模式

状态流转参照下图

![](/post/summary/ccs_map_state_20161121.png)

#### 方案

使用 **状态模式** 来实现状态流转。
使用 **观察者模式** 来监听状态变化，并根据具体的地图模式进行特定的UI展现。

地图基础状态中除了包含一个表示地图模式的属性外，还包含具体流转定义两种操作方法，分别对应点击模式切换按钮和操作地图。

```Objective-C
@class EepMSVGMapMode;
//地图模式基类
@interface EepMSVGMapDisplayModeState : NSObject
@property (nonatomic, assign) EepMSVGMapDisplayMode mapMode;//地图模式的属性
-(void)handleMapWithContext:(EepMSVGMapMode*)context;//操作地图
-(void)modeSwitchWithContext:(EepMSVGMapMode*)context;//按钮切换模式
@end
```

业务中的具体模式继承基础状态模式，重写两种操作对应的流转方法，修改相应的属性值。比如导航普通模式的实现细节如下：
```Objective-C
//导航－普通模式
@implementation EepMSVGMapDisplayModeNavigationNormalState
-(instancetype)init{
    self = [super init];
    self.mapMode = EepMSVGMapDisplayModeNavigationNormal;
    return self;
}
-(void)handleMapWithContext:(EepMSVGMapMode*)context{
    context.mapState = [[EepMSVGMapDisplayModeNavigationTourGuideState alloc] init];
}
-(void)modeSwitchWithContext:(EepMSVGMapMode*)context{
    context.mapState = [[EepMSVGMapDisplayModeNavigationDirectionState alloc] init];
}
@end
```

 > 此处需要注意的是普通导览模式和导航导览模式下，操作地图的时候，地图模式不会发生变更，两个状态对应的实现类可以将 **handleMapWithContext: **方法hook，或者干脆不重写该方法


对外通过以下访问形式；**调用者不需要关心当前模式是什么，只需要在点击模式切换时和操作地图时，调用特定名称的方法即可。** 

```Objective-C
/**
 *  地图展示
 **/
@interface EepMSVGMapMode : NSObject

@property (nonatomic,strong) EepMSVGMapDisplayModeState* mapState;
+(instancetype)sharedInstance;
-(void)setMapMode:(EepMSVGMapDisplayMode)mapMode;
//当前地图模式
-(EepMSVGMapDisplayMode)currentMapMode;
//操作地图
-(void)handleMap;
//点击了模式切换按钮
-(void)modeSwitch;
@end
```

实现相当简单，比如对于操作地图接口，只需要调用当前状态的 **handleMapWithContext:** 方法即可。

```Objective-C
-(void)handleMap{
    [self.mapState handleMapWithContext:self];
    [[NSNotificationCenter defaultCenter] lt_postNotificationOnMainThreadName:ALEMapDisplayModeChangeKey object:nil];
}
```





