---
layout:     post
title:      Mybatis中Example的or写法
date:       2019-07-03 17:50:12
author:     liangtong
catalog: true
categories: Java
tags: mybatis

---



近期的Java后端项目中遇到Mybatis的Example的or的用法，写法不是太直接。此处记录以下。



#### where A or B

```Java
        KanbanDeviceAuthorizeExample example = new KanbanDeviceAuthorizeExample();
        KanbanDeviceAuthorizeExample.Criteria criteria1 = example.createCriteria();
        KanbanDeviceAuthorizeExample.Criteria criteria2 = example.createCriteria();
        if (filterName != null && !filterName.trim().isEmpty()){
            criteria1.andNameLike("%" + filterName.trim() + "%");
            criteria2.andMacLike("%" + filterName.trim() + "%");
        }
        example.or(criteria2);
```

#### where (条件1 and 条件2) or （条件3 and 条件4）

```Java
        KanbanDeviceAuthorizeExample example = new KanbanDeviceAuthorizeExample();
        KanbanDeviceAuthorizeExample.Criteria criteria1 = example.createCriteria();
        KanbanDeviceAuthorizeExample.Criteria criteria2 = example.createCriteria();
        criteria1.andNameLike("%11%").andandMacEqualTo("22");
        criteria2.andMacLike("%33%").andandMacEqualTo("44");
        example.or(criteria2);
```

#### where  条件1 and (条件2 or 条件3) ==> (条件1 and 条件3) or ( 条件1 and 条件3 )

```Java
        KanbanDeviceAuthorizeExample example = new KanbanDeviceAuthorizeExample();
        KanbanDeviceAuthorizeExample.Criteria criteria1 = example.createCriteria();
        KanbanDeviceAuthorizeExample.Criteria criteria2 = example.createCriteria();
        criteria1.andNameLike("%11%").andandMacEqualTo("22");
        criteria2.andMacLike("%11%").andandMacEqualTo("44");
        example.or(criteria2);
```

