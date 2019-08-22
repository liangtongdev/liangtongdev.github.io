---
layout:     post
title:      使用nib自定义tableHeaderView的高度设置
date:       2019-08-08 10:50:12
author:     liangtong
catalog: true
categories: iOS
tags: tableView

---


#### 问题描述

使用nib自定义view的形式设置UITableview的tableHeaderView，headerView的高度显示有问题。



#### 解决方法

在header和footer 外层再套一层view 用来适配高度



#### 示例

```objective-c
    StickView* headerView = (StickView*)[UIView ltx_loadInstanceFromNibWithName:@"StickView"];
    [headerView setupStickView];
    headerView.backgroundColor = [UIColor clearColor];
    headerView.translatesAutoresizingMaskIntoConstraints = false;
    
    UIView * header = [[UIView alloc] init];
    [header addSubview:headerView];
    [header ltx_pinAllEdgesOfSubview:headerView];
    header.frame = CGRectMake(0, 0, 3, 130);
    self.tableView.tableHeaderView = header;
```


![断点.png](/post/oc/20190808/tableview_nib_header.png)


#### 备注

**直接通过设置tableView.tableHeaderView的形式进行设置，header会随tableview滚动**