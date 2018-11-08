---
layout:     post
title:      ViewController Life-Circle
date:       2016-10-25 22:16:32
catalog: true
categories: 前端
tags: 杂谈 

---

* content
{:toc}

对ViewController生命周期进行介绍，同时会说一些曾经遇到过的坑及注意事项



### 前言
​	这部分其实没啥好说的，苹果UIViewController接口文档对各个接口的描述写的很详细。
​	在视图的生命周期中，我曾经出问题的地方包括是UI表现异常；通知监听与释放没有成对出现；异步闭包回调引起的循环引用的资源无法释放；`__weak`和`__strong`问题导致的程序闪退等。这里就是把这些坑给记下来，（如果你碰巧）遇到这类问题后不会感到莫名其妙。

### UIViewController的生命周期

> 视图出现的时候调用顺序

	- (void)viewDidLoad; // Called after the view has been loaded.just once
	
	- (void)viewWillAppear:(BOOL)animated;    // Called when the view is about to made visible. Default does nothing
	
	// Called just before the view controller's view's layoutSubviews method is invoked. Subclasses can implement as necessary. The default is a nop.
	- (void)viewWillLayoutSubviews NS_AVAILABLE_IOS(5_0);
	
	// Called just after the view controller's view's layoutSubviews method is invoked. Subclasses can implement as necessary. The default is a nop.
	- (void)viewDidLayoutSubviews NS_AVAILABLE_IOS(5_0);
	
	- (void)viewDidAppear:(BOOL)animated;     // Called when the view has been fully transitioned onto the screen. Default does nothing

---

> 视图离开屏幕的时候调用顺序

	- (void)viewWillDisappear:(BOOL)animated; // Called when the view is dismissed, covered or otherwise hidden. Default does nothing
	
	- (void)viewDidDisappear:(BOOL)animated;  // Called after the view was dismissed, covered or otherwise hidden. Default does nothing




### 说明及最佳实践

```Objective-C
- (void)viewDidLoad；  
```

> 很常用、很重要的方法，只是在APP页面加载的时候调用一次，以后不会再调用了。
> 通常做一些`初始化设置`，但是UI相关的设置不要放在这里
> 当然，iOS6之前早期系统上，没有AutoLayout，使用绝对布局在这里更新UI问题不大
>
> 


```Objective-C
- (void)viewWillAppear:(BOOL)animated；  
```

> 视图即将出现在屏幕上，会调用该方法。
> 重写该方法，可以做一些`UI相关设置`，以及KVO监听之类操作
> 当APP视图间进行切换的时候，也会调用该方法
>
> 

```Objective-C
- (void)viewWillLayoutSubviews；  
```

> 在viewWillAppare后调用，将要对子视图进行布局
>
> 


```Objective-C
- (void) viewDidLayoutSubviews；  
```

> 已经布局完成子视图
>
> 


```Objective-C
- (void)viewDidAppear:(BOOL)animated；  
```

> 视图出现在屏幕上，会调用该方法。
> 重写该方法，可以做一些*UI更新设置*
>
> 

```Objective-C
 - (void)viewWillDisappear:(BOOL)animated；  
```

> 视图变换，即将离开屏幕时，会调用该方法。
> 重写该方法，做一些善后的处理工作，比如持久化处理、移除KVO监听之类的
>
> 

```Objective-C
 - (void)viewDidDisappear:(BOOL)animated；  
```

> 视图变换，离开屏幕时，会调用该方法。
> 重写该方法，对已消失的视图做一些操作，比如释放资源
>
> 


### 补充

* UI表示异常的话，多半是在不正确的时机调用了UI相关设置。

* 资源释放，尤其涉及视频、监听相关内容。

* 异步操作，涉及闭包，协议等操作时，要谨慎。
  * 例如：使用`__weakSelf`和`__strongSelf`打破循环引用
  * 例如：使用@selector前要判断

* 其他

  ​

### 后记

​	在这里遇到问题的话，一定要静下心。举个例子，比如你在viewDidLoad方法内有关于某个对象的回调处理。

```Objective-C
    _obj.testBlock = ^(){
        //your test block process
    };
```

但是实际开发中没有触发。可以从以下几个方向去调查：
* _obj对应类/结构体中是否会调用（绑定）该闭包；
* _obj的作用域：是全局的还是堆栈的；
* 该闭包是否在其他地方进行了设置；
* 其他。


>
>
>
>



