---
layout:     post
title:      MYSQL 给用户单个表权限
date:       2020-08-17 16:34:21
author:     liangtong
catalog: true
categories: 数据库
tags: mysql

---

​	

​	MySQl 创建用户，并给用户单独授权表权限（账号：wcs_wms ； 密码：xxx@yy； 数据库：siiri_aa_bb； 具体表：mm_nn_table； 权限类别：CURD ），具体如下：



```MYSQL
-- 创建用户： 用户名 wcs_wms ，密码 xxx@yy
create user wcs_wms identified by 'xxx@yy';

-- 分配表权限：这里是CURD权限（读、插、更新、删除） 。对应事例数据库表 siiri_aa_bb.mm_nn_table
grant select, insert, update, delete on siiri_aa_bb.mm_nn_table to wcs_wms@'%';

```



​	配置完成后，使用 `wcs_wms` 账号，密码 `xxx@yy` 连接数据库，会发现对 `siiri_aa_bb` 数据库的 `mm_nn_table` 表，可以有`CURD`权限。

