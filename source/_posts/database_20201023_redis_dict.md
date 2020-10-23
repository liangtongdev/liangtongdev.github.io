---
layout:     post
title:      Redis数据结构 - 字典
date:       2020-10-23 08:00:00
author:     liangtong
catalog: true
categories: 数据库
tags: redis
---



字典是一种存储健值对(key-value)的抽象数据结构。在字典中，一个Key和一个Value进行关联，这些关联的键和值就成为健值对。字典中每个Key都是独一无二的，程序通过Key来更新对应的Value。

![](/post/db/20201023/redis_dic.png)

C语言中没有内置字典，Redis构建了自己的字典实现。字典在Redis中应用相当广泛，比如Redis数据库就是使用字典作为底层实现的，对数据库的CURD也是构建在对字典的操作之上。字典还是哈希健的底层实现之一。

### 字典的实现

Redis的字典使用哈希表作为底层实现，一个哈希表里边可以有多个哈希表节点，每个哈希表节点就保存字典中的一个健值对。

#### 哈希表

```C
//哈希表
typedef struct dictht{
  //哈希表数组
  dicEntry **table;
  //哈希表大小
  unsigned long size;
  //哈希表大小掩码，用于计算索引值。总是等于size - 1
  unsigned long sizemask;
  //该哈希表已有的节点数量
  unsigned long used;
}dictht;
```

+ table 属性是一个数组，数组中每个元素都指向dictEntity（哈希表节点）结构的指针，每个dictEntity结构保存着一个健值对。
+ size 属性记录哈希表的大小，也是table数组的大小。
+ used 属性记录了哈希表目前已有的健值对数量。
+ sizemask 属性的值总是等于size-1，这个属性和哈希值一起决定一个健应该放在table数组的哪个索引上边。


![](/post/db/20201023/redis_dictht.png)

#### 哈希表节点

```C
//哈希表节点
typedef struct dictEntry {
  // 健
  void *key;
  // 值
  union{
    void *val;
    unit64_t u64;
    int64_t s64;
  } v;
  // 指向下个哈希节点，形成链表
  struct dictEntry *next;
} dictEntry;
```

key 属性保存着健值对应的健，v属性保存对应的值。其中v属性可以是指针，或者是unit64_t整数，又或者是int64_t整数。next属性是指向下个哈希节点的指针，用以解决健冲突。

![](/post/db/20201023/redis_dictht_dicEntry.png)



#### 字典

```C
//字典
typedef struct dict{
  //类型特定函数
  dicType *type;
  //私有数据
  void *privdata；
  //哈希表
  dictht ht[2];
  //rehash 索引，当处于非rehash时期，值为-1
  int rehashidx;
} dict;
```

type 属性 和 privdata属性是针对不同类型的健值对，为创建多态字典而设置的：

+ type 属性是一个指向dicType结构的指针，每个dicType结构保存了一簇用于操作特定类型健值对的函数，Redis会为用途不同的字典设置不同的类型特定函数。
+ privdata 属性保存了需要传给那些特定类型函数的可选参数。

```C
typedef struct dicType {
  //计算哈希值的函数
  unsigned int (*hashFunction)(const void *key);
  //复制健的函数
  void *(*keyDup)(void *privdata, const void *key);
  //复制值的函数
  void *(*valDup)(void *privdata, const void *obj);
  //对比健的函数
  int (*keyCompare)(void *privdata, const void *key1, const void *key2);
  //销毁健的函数
  void (*keyDestructor)(void *privdata, void *key);
  //销毁值的函数
  void (*valDestructor)(void *privdata, void *obj);
} dicType;
```

+ ht 属性包含2个哈希表，一般情况下字典只使用ht[0]哈希表， 而ht[1]哈希表只会在对字典进行rehash的时候使用。
+ rehashidx 是一个记录当前rehash进度的属性，如果没有rehash则为-1。

![](/post/db/20201023/redis_dic_demo.png)



### 哈希算法

当要将一个新的健值对添加到字典里时，程序首先根据健值对的健计算出哈希值和索引值。然后根据索引值将包含健值对的哈希节点放到哈希数组指定索引上。

+ 使用字典设置的哈希函数，计算出健key的哈希值 
  + hash = dict->type->hashFunction(key);
+ 使用哈希表的sizemask属性和哈希值，计算出索引值。具体使用的哈希表由字典的rehashidx决定
  + index = hash & dict->ht[0].sizemask;



### 解决健冲突

当有2个或2个以上数量的健被分配到哈希表同一个索引上时，称为冲突。Redis使用链地址法来解决冲突。

每个哈希节点都有一个next指针，多个哈希节点可以通过next指针构成一个单向链。被分配到同一个索引上的多个哈希节点使用单向链连接起来，这就解决了健冲突。

考虑到速度，程序总是将新的哈希节点添加到链表的表头位置。

![](/post/db/20201023/redis_key_collision.png)



### rehash

随着对字典不断的操作，哈希表存储的健值对数量会发生变化，为了让哈希表的负载因子维持在一个合理的范围，程序需要对哈希表进行扩容或收缩。

Redis对字典的rehash步骤如下

+ 为字典的ht[1]哈希表分配空间。
  + 如果执行的扩容操作，那么ht[1]的大小为第一个大于等于 ht[0].used*2 的 2的n次幂。（例如，当ht[0].used = 9，即哈希表中有9个哈希节点扩容时。扩容大小为大于等于18的第一个2的n次幂，为：32）
  + 如果执行的是收缩操作，那么ht[1]的大小为第一个大于等于 ht[0].used 的2的n次幂。（例如，当ht[0].used = 9，即哈希表中有9个哈希节点收缩时。扩容大小为大于等于9的第一个2的n次幂，为：16）

+ 将保存在ht[0]中的所有健值对rehash到ht[1]上边。即重新计算索引值，然后将对应的哈希节点放知道ht[1]哈希表对应的索引上。
+ 当ht[0]中包含的所有健值对都迁移到ht[1]之后，释放ht[0]，将ht[1]设置为ht[0]，并在ht[1]新创建一个空白哈希表，为下次rehash作准备。

![](/post/db/20201023/redis_rehash.png)



### 渐进式rehash

当数据量比较大时，为避免rehash对服务器造成性能问题，服务器不是一次性的将ht[0]中的所有健全部rehash到ht[1]中，而是分多次、渐进式进行操作。

+ 为ht[1]分配空间，让字典同时持有ht[0]、ht[1]两个哈希表
+ 维持字典中的rehashidx索引，并将值设置为0，表示rehash正是开始。
+ rehash期间，每次对字典执行添加、删除、查找或更新操作时，除了执行操作外，还会顺带将ht[0]在rehashidx索引上的所有健值对rehash到ht[1]。当对应索引上rehash工作完成后，程序将rehashidx属性增一
+ 随着操作的进行，当ht[0]的所有健值对都被rehash到ht[1]上后，程序将rehashidx属性设置为-1，表示rehash操作完成。

**渐进式rehash的好处是采用分治方式，将rehash健值所需的操作分摊到对字典的每个添加、删除、查找和更新操作上。从而避免集中式rehash带来的性能影响。**



在执行渐进式rehash的过程中，字典会同时使用ht[0]和ht[1]两个哈希表，所以在渐进式rehash期间，字典的操作会在两个哈希表上进行。比如需要查找某个健的话，现在ht[0]上进行，如果没有找到，则在ht[1]上进行。

另外新添加的字典健值对一律会保存在ht[1]上，不在ht[0]上操作保证了ht[0]中的健值对数量只减不增，最终变成空表。

