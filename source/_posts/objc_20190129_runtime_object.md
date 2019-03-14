---
layout:     post
title:      RunTime之NSObject解析
date:       2019-01-29 19:50:12
author:     liangtong
catalog: true
categories: iOS
tags: runtime

---


#### 写在前边

上篇文章中，介绍了Objc对象的分类：实例对象、类对象、元类对象；也介绍了对象分类中通过`isa`或`superclass`进行方法查找的流程。今天通过对苹果官方源码`objc4-750`对其详细说明。



 <!-- more -->
 


#### 源码

我们知道，`Objective-C`中，通常情况下，我们新建类都会继承于`NSObject`。那么，我们就从NSObject开始吧。

![NSObject.png](https://upload-images.jianshu.io/upload_images/16014538-80e58f2fc9f3ce8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)



我们看到在`NSObject`中，包含一个`Class isa`成员变量；

##### Class

接上图，然后我们点击能进入查看`Class`的定义。发现她是一结构体`objc_class`指针。

![Class.png](https://upload-images.jianshu.io/upload_images/16014538-22bddedaa8f6f332.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)


同时，我们也看到了我们常用的`id`其实是一个叫`objc_object`的结构体指针。继续，查看`objc_class`的实现

![objc_class.png](https://upload-images.jianshu.io/upload_images/16014538-e87d21a6da6c8b97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

我们看到，`objc_class`继承于`objc_object`，而且还包含了其他一些信息，稍后再详细介绍这些字段具体作用。

而从前边的截图中，我们也看到了`objc_object`就是`id`类型！也就是说 **`Class`继承于`id`**。

那么`objc_object`中包含什么信息呢?

##### id

继续，查看`objc_object`信息。

![objc_object.png](https://upload-images.jianshu.io/upload_images/16014538-6c63d74798c11808.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

我们在`objc_object`中看到了一个`isa_t`类别的成员变量，终于找到了传说中的`isa`（**异常兴奋！**）

##### isa_t

那么，`isa_t`具体是什么类型呢，继续我们发现，她原来是一个`union 共用体`，不仅可以用来存储类的指针信息，还可以用来表示`位域`（或者`位段`，可以用来存储更多的信息）。

![isa_t-1.png](https://upload-images.jianshu.io/upload_images/16014538-cac74cdac7caf670.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)


`位域`详情定义如下：

![isa_t-2.png](https://upload-images.jianshu.io/upload_images/16014538-54fc7ec8b2f6c85c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)


这里介绍下各个位的作用

+ nonpointer：0，代表普通的指针，存储着Class、Meta-Class对象的内存地址；1，代表优化过，使用位域存储更多的信息
+ has_assoc：是否有设置过关联对象，如果没有，释放时会更快
+ has_cxx_dtor：是否有C++的析构函数（.cxx_destruct），如果没有，释放时会更快
+ shiftcls：存储着Class、Meta-Class对象的内存地址信息
+ magic：用于在调试时分辨对象是否未完成初始化
+ weakly_referenced：是否有被弱引用指向过，如果没有，释放时会更快
+ deallocating：对象是否正在释放
+ extra_rc：里面存储的值是引用计数器
+ has_sidetable_rc：引用计数器是否过大无法存储在isa中；如果为1，那么引用计数会存储在一个叫SideTable的类的属性中

#### 分析

##### isa_t的位域

首先，使用使用`位域`可以节约资源。例如，我们比较两个类实例占用的空间。

```objective-c
@interface Person1 : NSObject
@property (nonatomic, assign) BOOL tall;
@property (nonatomic, assign) BOOL handsome;
@property (nonatomic, assign) NSInteger age;
@end

@interface Person2 : NSObject{
    union{
        NSInteger bits;
        struct {
            char tall : 1;
            char handsome: 1;
            NSInteger age: 7;
        } ;
    }tallHandsomeAge;
}
-(void)setTall:(BOOL)tall;
-(BOOL)tall;
-(void)setHandsome:(BOOL)handsome;
-(BOOL)handsome;
-(void)setAge:(NSInteger)age;
-(NSInteger)age;
-(NSInteger)bits;
@end
```

实现代码，如下：

```objective-c
@implementation Person1

@end

#define Tall_Mask (1<<0)
#define handsome_Mask (1<<1)
#define age_offset 2
#define age_Mask (1<<8)

@implementation Person2

-(void)setTall:(BOOL)tall{
    if (tall) {
        tallHandsomeAge.bits |= Tall_Mask;
    }else{
        tallHandsomeAge.bits &= ~Tall_Mask;
    }
}
-(BOOL)tall{
    return !!(tallHandsomeAge.bits & Tall_Mask);
}
-(void)setHandsome:(BOOL)handsome{
    if (handsome) {
        tallHandsomeAge.bits |= handsome_Mask;
    }else{
        tallHandsomeAge.bits &= ~handsome_Mask;
    }
}
-(BOOL)handsome{
    return !!(tallHandsomeAge.bits & handsome_Mask);
}
-(void)setAge:(NSInteger)age{
    tallHandsomeAge.bits &= (age_Mask | Tall_Mask | handsome_Mask);
    tallHandsomeAge.bits |= (age << age_offset);
}
-(NSInteger)age{
    return tallHandsomeAge.bits & age_Mask;
}
-(NSInteger)bits{
    return tallHandsomeAge.bits;
}
@end
```

首先，占用空间大小比较优势很明显；其次，`位域`使用的`位运算`对处理器来说，效率是很高的


![bit_field.png](https://upload-images.jianshu.io/upload_images/16014538-a977d3c48ef471ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

##### isa_t的Class(objc_class)

首先，`objc_class`继承自 `objc_object`(objc_object包含一个isa_t的成员变量isa)。`objc_class`结构体中包含的主要信息如下：

```objective-c
    // Class ISA;
    Class superclass;
    cache_t cache;             // formerly cache pointer and vtable
    class_data_bits_t bits;    // class_rw_t * plus custom rr/alloc flags

    class_rw_t *data() { 
        return bits.data();
    }
```



![class_rw_t.png](https://upload-images.jianshu.io/upload_images/16014538-00ce0e9217be04f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)

![class_ro_t.png](https://upload-images.jianshu.io/upload_images/16014538-91d1dd77e9089774.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/640)


+ ISA指针
+ superclass
+ cache方法缓存
+ bits具体的类信息（结构体class_rw_t）
  + 方法列表
  + 属性列表
  + 协议列表
  + class_ro_t *ro
    + instance对象占用的内存大小等
    + 成员变量列表

###### cache方法缓存

`Class`中有个方法缓存`cache_t`,用哈希表来缓存曾经调用过的方法，以提高方法的查找速度。

```objective-c
struct cache_t {
    struct bucket_t *_buckets;//散列表
    mask_t _mask;//总长度
    mask_t _occupied;//已缓存的方法数量
 
    //other information
}


struct bucket_t {
private:
    // IMP-first is better for arm64e ptrauth and no worse for arm64.
    // SEL-first is better for armv7* and i386 and x86_64.
#if __arm64__
    MethodCacheIMP _imp;//函数的内存地址
    cache_key_t _key;//SEL
#else
    cache_key_t _key;//SEL
    MethodCacheIMP _imp;//函数的内存地址
#endif
    //other information
}
```

之前，我们介绍过`objc_msgSend`消息对应方法查找流程。这里再简单介绍下：

+ 先查找自己类的缓存中是否有对应的方法，如果能找到直接发消息调用
+ 查找自己类中的bits类信息中的方法列表，如果能找到，则发消息，将方法放入方法缓存中。
+ 遍历父类的方法。

```objective-c
bucket_t * cache_t::find(cache_key_t k, id receiver)
{
    assert(k != 0);

    bucket_t *b = buckets();
    mask_t m = mask();
    mask_t begin = cache_hash(k, m);
    mask_t i = begin;
    do {
        if (b[i].key() == 0  ||  b[i].key() == k) {
            return &b[i];
        }
    } while ((i = cache_next(i, m)) != begin);

    // hack
    Class cls = (Class)((uintptr_t)this - offsetof(objc_class, cache));
    cache_t::bad_cache(receiver, (SEL)k, cls);
}

#if __arm__  ||  __x86_64__  ||  __i386__
// objc_msgSend has few registers available.
// Cache scan increments and wraps at special end-marking bucket.
#define CACHE_END_MARKER 1
static inline mask_t cache_next(mask_t i, mask_t mask) {
    return (i+1) & mask;
}

#elif __arm64__
// objc_msgSend has lots of registers available.
// Cache scan decrements. No end marker needed.
#define CACHE_END_MARKER 0
static inline mask_t cache_next(mask_t i, mask_t mask) {
    return i ? i-1 : mask;
}

#else
#error unknown architecture
#endif
```



###### bits具体的类信息class_rw_t

这里主要介绍的是方法列表 `method_array_t methods` 。具体method_t的结构如下：

```objective-c
struct method_t {
    SEL name;//函数名称
    const char *types;//函数编码，在swazzing中用的比较多
    MethodListIMP imp;//函数指针（函数地址）

    struct SortBySELAddress :
        public std::binary_function<const method_t&,
                                    const method_t&, bool>
    {
        bool operator() (const method_t& lhs,
                         const method_t& rhs)
        { return lhs.name < rhs.name; }
    };
};
```


#### 参照

https://opensource.apple.com/tarballs/objc4/
