---
layout:     post
title:      Runtime之@dynamic关键字
date:       2019-01-25 10:50:12
author:     liangtong
catalog: true
categories: iOS
tags: note

---

讲述@dynamic之前，需要了解几个名词。最后再重点介绍@dynamic的用法

+ @property

  + 原子性

  + 存取控制

  + 内存管理

+ @synthesize



 <!-- more -->



----



### @property

使用 **@property** 声明的属性，可以方便快捷的为实例变量创建存取器（getter 和 setter）。默认以下划线  *_* 开头。比如

```objective-c
@interface Person : NSObject
@property(nonatomic, readwrite, strong) NSString *name;
-(void)sayHello;
@end
    
@implementation Person
@synthesize name = _name;//默认情况，可以不写。除非想通过其他的单词（非_name）来使用
-(void)sayHello{
    NSLog(@"%@ says hello",_name);
}
@end
```

此时，我们可以通过点访问符 **.** 来给对象的name属性进行存取。 

```objective-c
    //设置
    Person* person1 = [[Person alloc] init];
	person1.name = @"zhangsan";
    [person1 sayHello];
    //获取
	NSLog(@"person1's name is %@",[person1.name sayBye]);
```

##### 原子性

​	在上述例子中，我们看到了声明语句括号内的**nonatomic**关键字。具体原子性包含两个关键字：

 + atomic
    + 默认。原子的，意味着只有一个线程访问实例变量(的getter和setter)。
    + 线程安全，至少在当前的存取器上是安全的
    + 影响效率，使用较少
 + nonatomic
    + 非原子的，可以同时被多个线程访问
    + 效率高
    + 多线程下不安全，使用较多

##### 存取控制

​	在上述例子中，我们看到了声明语句括号内的**readwrite**关键字。表示属性的存取特征。存取控制具体包含以下几个：

+ readwrite
  + 默认属性，系统会自动给属性生成getter和setter存取器
+ readonly
  + 只读属性，只会生成getter；不能通过点运算符(setter)给属性赋值

##### 内存管理

​	在上述例子中，我们看到了声明语句括号内的**strong**关键字。表示属性内存管理的关键字，有以下几个：

+ assign
  + 默认，适用于值类型。如int、NSInteger、CGFloat、float等。
+ retain
  + 在setter方法中，会对传入的对象的引用计数器加1（理解ARC）。
+ strong
  + strong是iOS引入ARC之后的关键字，是retain的一个可选的替代。
+ weak
  + 与retain相比，对引入对象的引用计数器，不进行加1操作。经常用于delegate等。
+ copy
  + 与strong类似，区别是copy是创建了一个新对象。通常NSString类属性会使用copy。



----

### @synthesize

​	@synthesize默认会给属性添加一个别名，比如上个例子中的**_name** 。默认情况下，对于@property的变量都会生成相关别名和存取器

#### @dynamic

​	如果某属性已实现了自己的getter和setter（当然对于 readonly 的属性只有 getter ），可以通过@dynamic关键字来阻止自动生成getter和setter的覆盖。@dynamic关键字有两个作用：

 + 让编译器不要创建实现属性所用的实例变量；
 + 让编译器不要创建该属性的get和setter方法。

**编译时没问题，在运行时才执行相应的方法，这就是所谓的动态绑定。**

 > 假如一个属性被声明为@dynamic var，然而没有提供对应的getter和setter方法，编译的时候不会报错，但是当程序运行到 *instance.var = xxx* 时，由于缺少setter方法会导致程序崩溃



----

### @dynamic的使用

这里介绍三种

+ 通过私有变量来实现@dynamic
+ Category扩展
+ 动态解析和函数重签名

#### 通过私有变量来实现@dynamic

通过私有变量来实现@dynamic，可以达到隐藏某些信息的目的。
例如DemoClass1中有个dynamicVar属性，我们将其设置为@dynamic，在程序运行期间，我们想通过他来存取某个私有(对象的)属性，简单起见，我们将对应的私有属性命名为privateVar。
**.h** 文件：

```Objective-C
/***
 * 用来展示通过私有变量来实现@dynamic
 * 用于隐藏某些信息
 ***/
@interface DemoClass1 : NSObject
@property (nonatomic , strong) NSString* dynamicVar;
@end
```
**.m** 文件：

```Objective-C
@interface DemoClass1()
@property (nonatomic, strong) NSString* privateVar;
@end

@implementation DemoClass1
@dynamic dynamicVar;
@synthesize privateVar = _privateVar;

- (void)setDynamicVar:(NSString *)dynamicVar{
    self.privateVar = dynamicVar;
}
-(NSString*)dynamicVar{
    return self.privateVar;
}

-(void)setPrivateVar:(NSString *)privateVar{
    _privateVar = privateVar;
}
-(NSString*)privateVar{
    if (!_privateVar) {
        _privateVar = @"This is private var";
    }
    return _privateVar;
}
```

方法的调用，我们通过调用*dynamicVar*的getter和setter，然后看下对应值

```Objective-C
    DemoClass1* demo1 = [[DemoClass1 alloc] init];
    NSLog(@"Before - DemoClass1's dynamicVar is :%@", demo1.dynamicVar);
    demo1.dynamicVar = @"Set dynamicVar";
    NSLog(@"After - DemoClass1's dynamicVar is :%@", demo1.dynamicVar);
```

输出信息如下：

```bash
2019-01-25 09:19:13.253753+0800 property_dynamic[18448:7311996] Before - DemoClass1's dynamicVar is :This is private var
2019-01-25 09:19:13.253893+0800 property_dynamic[18448:7311996] After - DemoClass1's dynamicVar is :Set dynamicVar
```

我们可以看到，当我们访问*dynamicVar*的getter方法时，最终展示的私有变量privateVar。而访问setter，最终设置的也是privateVar。
通过@dynamic关键字最终实现了将私有信息暴漏的目的。

#### Category扩展
另一个较常用的用法就是Category，例如，我们想对**UILabel**进行行间距和列间距的设置，我们想通过两个属性来设置
**.h** 文件：

```Objective-C
@interface UILabel (Demo2)
//行间距
@property (nonatomic, assign) CGFloat ltx_lineSpace;
//字间距
@property (nonatomic, assign) CGFloat ltx_wordSpace;
@end
```
**.m** 文件：

```Objective-C
#import "UILabel+Demo2.h"
#import <objc/runtime.h>

NSString const *ltx_label_lineSpace = @"ltx_label_lineSpace";
NSString const *ltx_label_wordSpace = @"ltx_label_wordSpace";
@implementation UILabel (Demo2)
@dynamic ltx_lineSpace;
@dynamic ltx_wordSpace;

-(void)ltx_updateLabelLineSpace{
    NSString* text = self.text;
    NSInteger lineSpace = self.ltx_lineSpace;
    NSInteger wordSpace = self.ltx_wordSpace;
    if ([text length] > 0) {
        NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:text];
        if (lineSpace > 0) {
            NSMutableParagraphStyle *paragraphStyle = [[NSMutableParagraphStyle alloc] init];
            [paragraphStyle setLineSpacing:lineSpace];
            [attributedString addAttribute:NSParagraphStyleAttributeName value:paragraphStyle range:NSMakeRange(0, [text length])];
        }
        if (wordSpace > 0) {
            [attributedString addAttribute:NSKernAttributeName value:[NSNumber numberWithFloat:wordSpace] range:NSMakeRange(0, [text length])];
        }
        self.attributedText = attributedString;
    }
}


#pragma mark - Getter && Setter
-(void)setLtx_lineSpace:(CGFloat)ltx_lineSpace{
    NSNumber* number = [NSNumber numberWithFloat:ltx_lineSpace];
    objc_setAssociatedObject(self, &ltx_label_lineSpace, number, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    [self ltx_updateLabelLineSpace];
}
-(CGFloat)ltx_lineSpace{
    NSNumber* number =  objc_getAssociatedObject(self, &ltx_label_lineSpace);
    return [number floatValue];
}

-(void)setLtx_wordSpace:(CGFloat)ltx_wordSpace{
    NSNumber* number = [NSNumber numberWithFloat:ltx_wordSpace];
    objc_setAssociatedObject(self, &ltx_label_wordSpace, number, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    [self ltx_updateLabelLineSpace];
}
-(CGFloat)ltx_wordSpace{
    NSNumber* number =  objc_getAssociatedObject(self, &ltx_label_wordSpace);
    return [number floatValue];
}
```

方法的调用，我们通过调用*ltx_lineSpace* 和 *ltx_wordSpace* 来对labe的列间距和字间距进行设置

```Objective-C
    //设置UILabel的行间距和字间距
    self.label.ltx_lineSpace = 8;
    self.label.ltx_wordSpace = 4;
```




更多扩展，请移步参照：https://github.com/liangtongdev/LTxCategories

#### 动态解析和函数重签名

即通过对@dynamic动态属性的getter和setter进行以下操作。
 + 动态解析
    + 重写NSObject的方法 + (BOOL)resolveInstanceMethod:(SEL)sel
    + 利用class_addMethod方法，将其他方法的实现添加给属性的getter和setter方法。
 + 函数重签名
    + 消息转发
       + 重写NSObject的方法-(id)forwardingTargetForSelector:(SEL)aSelector
       + 在方法内部，对动态属性的getter和setter方法进行转发，由其他对象来实现具体细节。
    + 方法重签名
       + 重写NSObject的方法 - (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector 
       + 在方法内，返回动态属性的getter和setter方法的新的方法签名
       + 重写NSObject的方法- (void)forwardInvocation:(NSInvocation *)anInvocation
       + 在方法内部，对重新签名的方法的target进行赋值，并唤醒

具体细节，请移步参照文章：[Runtime之objc_msgSend执行流程](https://www.jianshu.com/p/7994ae5d4c8a)
