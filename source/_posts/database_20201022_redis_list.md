---
layout:     post
title:      Redis数据结构 - 链表
date:       2020-10-22 08:00:23
author:     liangtong
catalog: true
categories: 数据库
tags: redis

---


Redis使用的C语言中没有链表这种数据结构，所以Redis构建了自己的链表实现。
链表在Redis的应用非常广泛。比如列表键的底层实现之一就是链表。发布订阅、慢查询、监视器的功能也用到了链表，Redis服务器本身还用链表保存多个客户端的状态信息。

![](/post/db/20201022/redis_list.png)





### 链表和链表节点的实现

```C
//链表节点
typedef struct listNode{
  //前置节点
  struct listNode * prev;
  //后置节点
  struct listNode * next;
  //节点的值
  void * value;
}listNode;
```

多个listNode可以通过 prev指针 和 next指针 组成双向链表。

虽然仅仅使用多个listNode结构就能组成链表，Redis使用更加方便高效的list来持有链表。

```C
//链表
typedef struct list{
  //表头节点
  listNode * head;
  //表尾节点
  listNode * tail;
  //链表所包含的节点数量
  unsigned long len;
  //节点值复制函数
  void *(*dup)(void *ptr);
  //节点值释放函数
  void (*free)(void *ptr)
  //节点值比较函数
  int (*match)(void *ptr, void *key);
}list;
```

list结构为链表提供了表头指针，表尾指针，表长度，以及用于实现多态链表的函数。




Redis的链表实现的特征如下：

+ 双向： 每个节点有prev指针 和 next指针，获取某个节点的前置和后置节点的复杂度是O（1）。
+ 无环： 表头的prev指针 和表尾的next指针均指向NULL，对链表的访问以NULL为终点。
+ 表头指针和表尾指针：通过list结构的表头指针和表尾指针，程序获取表头节点和表尾节点的复杂度为O（1）。
+ 长度len：list结构中持有链表的长度，程序获取链表的节点数量的复杂度为O（1）。
+ 多态：链表节点中使用 void* 指针来存储节点值。通过list结构中的dump 、free、 match三个特定函数，链表可以存储不同类型的值。





