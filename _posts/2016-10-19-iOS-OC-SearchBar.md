---
layout:     post
title:      UISearchBar自定义
date:       2016-10-19 17:50:00
author:     liangtong
catalog: true
categories: 前端
tags:  笔记

---

* content
{:toc}

参照一些常用应用对UISearchBar进行UI展示自定义




#### 前言

​        系统默认的搜索栏（UISearchBar）默认样子真心不好看，而平时iOS移动项目中又缺不了搜索功能，参照一些常用应用对UISearchBar进行UI展示自定义，效果图如下：

![](https://l900416.github.io/img/post/iOS/Objective-C/rendering.gif)

#### UISearchBar介绍

​        如图所示，包括placeholder、textfiled、clearButton、bookmarkButton、leftView等：

![](https://l900416.github.io/img/post/iOS/Objective-C/default_1.png)

>> * ① SearchBar的TextField的leftView；
>> * ② SearchBar的TextField的placeholder；
>> * ③ SearchBar的BookmarkIcon，默认不显示；



![](https://l900416.github.io/img/post/iOS/Objective-C/default_2.png)

>> * ④ SearchBar的搜索Text；
>> * ⑤ SearchBar的ClearButton；

#### 实现步骤

* **1、修改SearchBar背景图**

```objective-c
//修改SearchBar背景（透明）图片，去除默认的黑线
[self setBackgroundImage:[UIImage new]];
```

* **2、修改所搜索框背景图片，并切圆角**

```objective-c
//输入框背景图片
[self setSearchFieldBackgroundImage:[self searchFieldBackgroundImage] forState:UIControlStateNormal];
//将输入框切圆角
[self setSearchTextBackgroundCorner];
```

![](https://l900416.github.io/img/post/iOS/Objective-C/process_2.png)

* **3、显示BookmarkButton，并设置其显示图标**

```objective-c
//显示BookmarkButton，并设置其图标（照相机）
self.showsBookmarkButton = YES;
[self setImage:[UIImage imageNamed:@"ic_searchbar_camera"] forSearchBarIcon:UISearchBarIconBookmark  state:UIControlStateNormal];
```

![](https://l900416.github.io/img/post/iOS/Objective-C/process_3.png)

* **4、设置「清空」图标图标**

```objective-c
//文本发生变更的时候，修改「清空」图标
[self setImage:[UIImage imageNamed:@"ic_searchbar_clear"] forSearchBarIcon:UISearchBarIconClear  state:UIControlStateNormal];
[self setImage:[UIImage imageNamed:@"ic_searchbar_clear"] forSearchBarIcon:UISearchBarIconClear  state:UIControlStateHighlighted];
```

![](https://l900416.github.io/img/post/iOS/Objective-C/process_4.png)

* **5、修改搜索框左侧的图标**

```objective-c
//搜索框左侧的搜索图标修改（默认是灰色）
UITextField* searchField = [self valueForKey:@"_searchField"];
UIImageView* searchIV = [[UIImageView alloc] initWithImage:[UIImage imageNamed:@"ic_searchbar_search"]];
searchField.leftView = searchIV;
searchField.leftViewMode = UITextFieldViewModeAlways;
```

* **6、修改输入框文本颜色**

```objective-c
//改变searcher的textcolor
searchField.textColor = [UIColor whiteColor];
//改变placeholder的颜色
[searchField setValue:[UIColor whiteColor] forKeyPath:@"_placeholderLabel.textColor"];
```

![](https://l900416.github.io/img/post/iOS/Objective-C/process_6.png)

* **7、附上调用方法**

```objective-c
//搜索框的背景图片
- (UIImage*) searchFieldBackgroundImage
{
    CGFloat height = 32.f;
    UIColor* bgColor = [UIColor colorWithRed:47/255.0 green:123/255.0 blue:200/255.0 alpha:0.5];
    CGRect r= CGRectMake(0.0f, 0.0f, 1.0f, height);
    UIGraphicsBeginImageContext(r.size);
    CGContextRef context = UIGraphicsGetCurrentContext();
    CGContextSetFillColorWithColor(context, [bgColor CGColor]);
    CGContextFillRect(context, r);
    UIImage *img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return img;
}

//切圆角
-(void)setSearchTextBackgroundCorner{
    for (UIView *subview in self.subviews) {
        for(UIView* grandSonView in subview.subviews){
            if ([grandSonView isKindOfClass:NSClassFromString(@"UISearchBarBackground")]) {
                grandSonView.alpha = 0.0f;
            }else if([grandSonView isKindOfClass:NSClassFromString(@"UISearchBarTextField")] ){
                grandSonView.layer.cornerRadius = 5.0f;
                grandSonView.layer.masksToBounds = YES;
            }else{
                grandSonView.alpha = 0.0f;
            }
        }
    }
}
```



#### 源码下载

https://pan.baidu.com/s/1i5JiW3z



#### 参考资料

​	https://developer.apple.com/reference/uikit/uisearchbar

​        http://stackoverflow.com/questions/2139115/uisearchbar-clear-background-color-or-set-background-image-iphone-sdk/5557255#5557255









