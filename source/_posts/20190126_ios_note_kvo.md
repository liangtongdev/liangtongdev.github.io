---
layout:     post
title:      KVO的使用及原理
date:       2019-01-26 17:50:12
author:     liangtong
catalog: true
categories: iOS
tags: note

---

#### 概述

`KVO` 全称`KeyValueObserving`键值监听，是苹果提供的一套事件通讯机制。允许对象监听另一个对象特定属性的改变，并在改变时接收到事件。一般继承自`NSObject`的对象都默认支持`KVO`。

对象的属性是否发生变化肯定会调用其setter方法，而KVO的本质是**监听对象有没有调用被监听属性的setter方法**。

#### 常用方法

使用KVO分三个步骤：

+ 添加观察者(注册)。
+ 观察者中实现回调。
+ 移除观察者(删除)。

```objective-c
/*
注册监听器
监听器对象为observer，被监听对象为消息的发送者即方法的调用者在回调函数中会被回传
监听的属性路径为keyPath支持点语法的嵌套
监听类型为options支持按位或来监听多个事件类型
监听上下文context主要用于在多个监听器对象监听相同keyPath时进行区分
添加监听器只会保留监听器对象的地址，不会增加引用，也不会在对象释放后置空，因此需要自己持有监听对象的强引用，该参数也会在回调函数中回传
*/
- (void)addObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath options:(NSKeyValueObservingOptions)options context:(nullable void *)context;

/*
删除监听器
监听器对象为observer，被监听对象为消息的发送者即方法的调用者，应与addObserver方法匹配
监听的属性路径为keyPath，应与addObserver方法的keyPath匹配
监听上下文context，应与addObserver方法的context匹配
*/
- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath context:(nullable void *)context API_AVAILABLE(macos(10.7), ios(5.0), watchos(2.0), tvos(9.0));

/*
与上一个方法相同，只是少了context参数
推荐使用上一个方法，该方法由于没有传递context可能会产生异常结果
*/
- (void)removeObserver:(NSObject *)observer forKeyPath:(NSString *)keyPath;

/*
监听器对象的监听回调方法
keyPath即为监听的属性路径
object为被监听的对象
change保存被监听的值产生的变化
context为监听上下文，由add方法回传
*/
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context;
```

#### 简单实现

我们创建一个 `Person`类，然后在类中添加一个`name`属性和`sex`属性。

```objective-c
@interface Person : NSObject
@property (nonatomic, strong) NSString* name;
@property (nonatomic, strong) NSString* sex;
@end
 
@implementation Person

-(instancetype)init{
    self = [super init];
    if (self) {
        _name = @"liangtong";
        _sex = @"M";
    }
    return self;
}
```

然后我们观察这个Person实例对象的`name`属性

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    // Do any additional setup after loading the view, typically from a nib.
    _person = [[Person alloc] init];
    NSKeyValueObservingOptions options = NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld;
    [_person addObserver:self forKeyPath:@"name" options:options context:@"123"];
    _person.name = @"joker";
}

// 当监听对象的属性值发生改变时，就会调用
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context{
    NSLog(@"监听到%@的%@属性值改变了 - %@ - %@", object, keyPath, change, context);
}

-(void)dealloc{
    [_person removeObserver:self forKeyPath:@"name"];
    _person = nil;
}
```

执行程序，然后log信息如下

```bash
2019-01-26 15:53:08.545313+0800 runtime_kvo[5666:1724236] 监听到isa : NSKVONotifying_Person 
 superclass : Person 
 setName IMP : 0x1096ea63a 
 setSex IMP: 0x1093914d0的name属性值改变了 - {
    kind = 1;
    new = joker;
    old = liangtong;
} - 123
```

注册观察者后，当我们通过`.`给对象属性进行赋值时，最终会通知观察者具体的改变。例子中，我们会在监听回调中得到`keypath 为 name，context为123的object变更`。

#### 实现原理

为了能够看到更多的细节，我们重写`Person`类的`description`方法。

```objective-c
/****
 * 重写description，展示更多信息
 ***/
-(NSString*)description{
    
    NSString* className = NSStringFromClass(object_getClass(self));
    NSString* superclass = NSStringFromClass(class_getSuperclass(object_getClass(self)));

    IMP setNameIMP = [self methodForSelector:@selector(setName:)];
    IMP setSexIMP = [self methodForSelector:@selector(setSex:)];
    
    NSString* desc = [NSString stringWithFormat:@"isa : %@ \n  \
                      superclass : %@ \n \
                      setName IMP : %p \n \
                      setSex IMP: %p",
                      className,superclass,setNameIMP,setSexIMP];
    return desc;
}
```

在注册观察者前后分别打印`_person`实例对象的信息，如下

```objective-c
2019-01-26 15:59:23.528750+0800 runtime_kvo[5734:1745208] Before Observe---------------->
 isa : Person 
                        superclass : NSObject 
                       setName IMP : 0x100dca430 
                       setSex IMP: 0x100dca490
2019-01-26 15:59:23.529090+0800 runtime_kvo[5734:1745208] After Observe---------------->
 isa : NSKVONotifying_Person 
                        superclass : Person 
                       setName IMP : 0x10112363a 
                       setSex IMP: 0x100dca490
2019-01-26 15:59:23.529278+0800 runtime_kvo[5734:1745208] 监听到isa : NSKVONotifying_Person 
                        superclass : Person 
                       setName IMP : 0x10112363a 
                       setSex IMP: 0x100dca490的name属性值改变了 - {
    kind = 1;
    new = joker;
    old = liangtong;
} - 123
```

我们可以看到，注册KVO监听后，`_person`对象的`isa`指针由**Person类**变成了**NSKVONotifying_Person类**。而`superclass`由**NSObject**变成了**Person类**。`setName:`的方法实现发生了变更（由**0x100dca430**变成了**0x10112363a**）而`setSex:`的未发生变更。

我们大致可以猜到**`KVO`是通过`isa-swizzling`技术实现的**。

+ 在运行期间根据原类创建一个中间类（**NSKVONotifying_xxx**），这个中间类是原类的子类。
+ 动态修改了对象的`isa`指向中间类。
+ 中间类重写了被监听属性的`setter`方法，没有监听的属性setter方法则不会被重写。
  + 重写属性的`setter`方法在修改之前会调用`willChangeValueForKey:`方法。
  + 重写属性的`setter`方法在修改之后会调用`didChangeValueForKey:`方法。
  + 通过添加断点，我们可以看到在修改之后，会调用`NSKeyValueNotifyObserver`。
  + 最终会调用到`observeValueForKeyPath:ofObject:change:context:`方法中。
+ 重写delloc方法，销毁新生成的**NSKVONotifying_Person类**。



#### 猜测与验证

通过以上，我们猜测如果阻止系统自动调用属性的`willChangeValueForKey:`和`didChangeValueForKey:`方法，可能会阻止KVO的事件传递。于是我们在`Person`类中重写以下方法

```objective-c
/**
 * 当key未name时候，不自动触发相关的setter
 **/
+ (BOOL) automaticallyNotifiesObserversForKey:(NSString *)key {
    if ([key isEqualToString:@"name"]) {
        return NO;
    }
    return [super automaticallyNotifiesObserversForKey:key];
}
```

继续运行刚才的代码，结果如下

```objective-c
2019-01-26 16:28:54.434002+0800 runtime_kvo[6071:1853052] Before Observe---------------->
 isa : Person 
                        superclass : NSObject 
                       setName IMP : 0x104ea4400 
                       setSex IMP: 0x104ea4460
2019-01-26 16:28:54.434228+0800 runtime_kvo[6071:1853052] After Observe---------------->
 isa : Person 
                        superclass : NSObject 
                       setName IMP : 0x104ea4400 
                       setSex IMP: 0x104ea4460
```

果真未自动触发KVO！！

那么问题来了，我们可以通过手动出发KVO吗？如果我们手动调用被阻止的两个方法，可以出发KVO吗？为了测试我们的猜想，我们给`Person`类的`sex`属性的setter中，添加相关代码。

```objective-c
-(void)setSex:(NSString *)sex{
    _sex = sex;
    //通过手动调用setter，测试能否触发KVO
    [self willChangeValueForKey:@"name"];
    _name = @"Hello Ketty";
    [self didChangeValueForKey:@"name"];
}
```

修改下运行的代码，之前是通过`name`的setter进行操作，现在我们换成调用`sex`的setter，如下

```objective-c
    _person.sex = @"F";
```

结果如下：

```bash
2019-01-26 16:35:28.746294+0800 runtime_kvo[6174:1872068] Before Observe---------------->
 isa : Person 
                        superclass : NSObject 
                       setName IMP : 0x10b9c2400 
                       setSex IMP: 0x10b9c2320
2019-01-26 16:35:28.746496+0800 runtime_kvo[6174:1872068] After Observe---------------->
 isa : Person 
                        superclass : NSObject 
                       setName IMP : 0x10b9c2400 
                       setSex IMP: 0x10b9c2320
2019-01-26 16:35:28.746700+0800 runtime_kvo[6174:1872068] 监听到isa : Person 
                        superclass : NSObject 
                       setName IMP : 0x10b9c2400 
                       setSex IMP: 0x10b9c2320的name属性值改变了 - {
    kind = 1;
    new = "Hello Ketty";
    old = liangtong;
} - 123
```

在`sex`的`setter`方法中，我们手动调用`willChangeValueForKey:`和`didChangeValueForKey:`方法对`name`进行设置，成功触发了KVO操作！



#### 总结

经过以上代码，我们大致了解了KVO的本质。

+ 1、`isa-swizzling`，利用RuntimeApi动态生成一个子类（**NSKVONotifying_xxx**），并让instance对象的`isa`指向这个全新的子类。
+ 2、当修改对象的被监听属性时候，会依次调用子类（**NSKVONotifying_xxx**）的以下方法
  + `willChangeValueForKey:`
  + 父类原来的setter
  + `didChangeValueForKey:`
  + 最终触发`observeValueForKeyPath:ofObject:change:context:`

![断点.png](https://upload-images.jianshu.io/upload_images/16014538-0129cd6f93af2985.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/540)

当我们重写`automaticallyNotifiesObserversForKey:`方法，对name`的相关自动调用`willChangeValueForKey:`和`didChangeValueForKey:`方法返回**NO**时，KVO未触发，表明直接修改成员变量的值不会触发KVO。



经过以上猜测部分，我们也知道了如何手动触发KVO。手动调用以下两个方法：

+ 手动调用 `willChangeValueForKey:`
+ 手动调用`didChangeValueForKey:`

KVO通知依赖以上两个方法，在属性变更之前通过调用 `willChangeValueForKey:`记录旧值；而属性发生改变之后通过调用`didChangeValueForKey:`保存新值。继而 `observeValueForKey:ofObject:change:context: `也会被调用。



KVO的监听移除

+ 添加与移除成对出现

+ 不移除会造成内存泄漏

+ 多次移除会造成崩溃

  + ```objective-c
       @try {
          [object removeObserver:self forKeyPath:@"keyPath"];
       }
       @catch (NSException * __unused exception) {}
    ```

  + 系统为了实现KVO，为NSObject添加了一个名为NSKeyValueObserverRegistration的Category，KVO的add和remove的实现都在里面。在移除的时候，系统会判断当前KVO的key是否已经被移除，如果已经被移除，则主动抛出一个NSException的异常


#### Demo
https://github.com/liangtongdev/Demo-runtime_kvo

#### 参照

KVO原理分析及使用进阶：https://www.jianshu.com/p/badf5cac0130
KVO ：https://github.com/SunshineBrother/JHBlog/blob/master/iOS知识点/iOS底层/3、KVO.md
iOS-KVO 实现原理：https://www.jianshu.com/p/0e75d99c3480

