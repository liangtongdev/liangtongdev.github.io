---
layout:     post
title:      Redis数据结构 - 跳表skiplist
date:       2020-10-24 18:00:00
author:     liangtong
catalog: true
categories: 数据库
tags: redis
---



跳表是一种有序的数据结构，通过在每个节点中维持多个指向其他节点的指针，从而达到快速访问节点的目的。平均O（logN）、最坏O（N）复杂度。跳表由跳表节点zskiplistNode 和 跳表zskiplist 两个结构定义

![](/post/db/20201024/redis_skiplist.png)


### 跳表节点

```C
//跳表节点
typedef struct zskiplistNode{
  //层
  struct zskiplistLevel {
    //前进指针
    struct zskiplistNode * forward;
    //跨度
    unsigned int span
  } level[];
  //后退指针
  struct zskiplistNode *backward;
  //分值
  double score；
  //成员对象
  robj *obj;
} zskiplistNode;
```

+ level： 层数组，每个层带有两个属性。一般来说，层数越多，访问其他节点的速度就越快
  + forward 前进指针：用于访问位于表尾方向的其他跳表节点
  + span 跨度：记录了前进指针指向的节点和当前节点的距离，跨度越大，节点距离就越远。
+ 后退指针：指向当前节点的前一个节点。从表尾向表头遍历时使用
+ score： 分值，节点按分值从小到大排列
+ obj： 成员对象（**Redis对象 redisObject 结构后续介绍**）

### 跳表

```C
typedef struct zskiplist {
  //表头、表尾节点
  struct zskiplistNode *header, *tail;
  //表中节点的数量
  unsigned long length;
  //表中层数最大的节点的层数
  int level;
} zskiplist;
```

+ header : 指向跳表的表头节点。定位表头节点的复杂度为O（1）
+ tail： 指向跳表的表尾节点。定位表尾节点的复杂度为O（1）
+ length： 记录跳表的长度。即跳表中目前包含节点的数量（表头不算在内）。获取跳表长度的复杂度为O（1）
+ level： 记录目前跳表内，层数最大的那个节点的层数（表头节点的层数不算在内）

**注：表头节点和其他节点的构造是一样的，也有后退指针、分值及成员对象。不过表头节点的这些属性不会被用到**