---
layout:     post
title:      Runtime之objc_msgSend流程
date:       2019-01-23 18:50:12
author:     liangtong
catalog: true
categories: iOS
tags: note
cover: /img/objc.jpeg

---


### 总览

Objetive-C的消息发送，是通过objc_msgSend来实现的，具体执行过程，主要分三个阶段：

+ 1、消息发送；
+ 2、动态方法解析
+ 3、消息转发或重新签名



 <!-- more -->



----



#### 消息发送

Person类有两个方法 *sayHello* 和 *sayBye* ，Student继承Person，并重写 *sayHello* 方法。

```objective-c
@interface Person : NSObject
-(void)sayHello;
-(void)sayBye;
@end

@interface Student : Person
@end
    
@implementation Person
-(void)sayHello{
    NSLog(@"%s",__func__);
}
-(void)sayBye{
    NSLog(@"%s",__func__);
}
@end
    
@implementation Student
-(void)sayHello{
    NSLog(@"%s",__func__);
}
@end
```

现在，通过给Student实例对象发消息，来展示消息的调用顺序

```objective-c
    //1.1 如果接收者类的cache中能找到方法，则直接调用。
	//否则从接受者类的方法列表中查找方法，找到后添加到cache中
    Student* student = [[Student alloc] init];
    [student sayHello];
    //1.2 以上两个步骤均找不到的时候，从superClass的cache中查找，同 1.1
    [student sayBye];
```

结果如下：

```bash
2019-01-23 19:09:00.825000+0800 runtime_objc_msgSend[14364:5870808] -[Student sayHello]
2019-01-23 19:09:00.825100+0800 runtime_objc_msgSend[14364:5870808] -[Person sayBye]
```

通过对结果的分析，我们得到如下方法查找的顺序：

+ 1 接收者首先从接收者类的cache中查找方法
  + 1.1 如果能找到方法，直接调用，结束
  + 1.2 如果找不到方法，继续执行2
+ 2 从接收者类的方法列表中查找
  + 2.1 如果找到方法，调用并将方法添加到接收者类的cache中，结束
  + 2.2 如果找不到方法，则从其superClass的cache中查找
  + 递归2.1，直到最顶层类。
+ 3 如果找不到方法，则判断走以下两个步骤
  + 3.1 如果两个步骤均不涉及，则直接抛出异常 '**unrecognized selector sent to instance**'
  + 详细步骤参照以下阶段
    + 注：每个阶段结束会重新进入本阶段。



------

#### 动态方法解析

如果消息发送阶段，未找到匹配的方法，则开发者可以通过重写NSObject中的以下两个方法来对未匹配的方法进行解析。

```objective-c
+ (BOOL)resolveClassMethod:(SEL)sel
+ (BOOL)resolveInstanceMethod:(SEL)sel
```

为了测试代码，我们重写Student类，对其进行扩展，当方法dynamicAnalysisMethod不存在时，我们将Student类中方法dynamicAnalysisOther的实现添加给dynamicAnalysisMethod。代码如下

```objective-c
@interface Student : Person
-(void)dynamicAnalysisMethod;
@end
    
@implementation Student
+(BOOL)resolveInstanceMethod:(SEL)sel{
    if (sel == @selector(dynamicAnalysisMethod)) {
        Method method = class_getInstanceMethod([self class], @selector(dynamicAnalysisOther));
        //Adds a new method to a class with a given name and implementation.
        class_addMethod([self class],
                        sel,
                        method_getImplementation(method),
                        method_getTypeEncoding(method));
        return true;
    }
    return [super resolveInstanceMethod:sel];
}

-(void)dynamicAnalysisOther{
    NSLog(@"%s",__func__);
}
@end
```

此时，我们给Student实例对象发*dynamicAnalysisMethod*消息，代码如下

```objective-c
    Student* student = [[Student alloc] init];
    //针对类和实例对象方法。
    //2.1重写NSObject的方法 + (BOOL)resolveClassMethod:(SEL)sel 
	//  	或 + (BOOL)resolveInstanceMethod:(SEL)sel
    //2.2在方法中对方法进行动态解析。
    [student dynamicAnalysisMethod];
```

结果如下：

```bash
2019-01-23 19:09:00.825300+0800 runtime_objc_msgSend[14364:5870808] -[Student dynamicAnalysisOther]
```

这样，我们实现了消息的动态解析。

+ 针对未匹配的方法，我们可以通过 class_addMethod 给类添加新的方法和实现
+ 重新进入消息发送阶段



----

#### 消息转发

如果在以上两个阶段均没有找到相关方法，此时就进入了消息转发阶段。消息转发主要有两个类别

+ 直接转发
+ 方法重签名，转发

此时，我们新建一个Worker类，详细代码如下：

```objective-c
@interface Worker : NSObject
-(void)sayHello;
-(void)reSignature;
@end
    
@implementation Worker
-(id)forwardingTargetForSelector:(SEL)aSelector{
    if (aSelector == @selector(sayHello)) {
        return [[Student alloc] init];
    }
    return nil;
}

-(NSMethodSignature*)methodSignatureForSelector:(SEL)sel{
    if (sel == @selector(reSignature)) {
        NSMethodSignature* signature = [[[Student alloc] init] methodSignatureForSelector:@selector(reSignatureMethod)];
        return signature;
    }
    return [super methodSignatureForSelector:sel];
}
- (void)forwardInvocation:(NSInvocation *)anInvocation{
    NSLog(@"%s",__func__);
    if (anInvocation.selector == @selector(reSignatureMethod)) {
        anInvocation.target = [[Student alloc] init];
        [anInvocation invoke];
        //或者如下形式
        //[anInvocation invokeWithTarget:[[Student alloc] init]];
    }
}
@end
```

对Worker类，有两个方法申明*sayHello* 和*reSignature* ， 但是并不对其进行实现。此时我们给Worker的实例对象发送消息，如下：

```objective-c
    Worker* worker = [[Worker alloc] init];
    //直接转发
    //3.1 重写NSObject的方法 - (id)forwardingTargetForSelector:(SEL)aSelector
    //返回 消息接收者对象
    [worker sayHello];
    
    //方法重签名。
    //如果3.1转发方法返回的是nil。则可以通过重新签名的方式来实现
    //4.1 重写NSObject的类/实例方法 
	//	  - (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
	//  	或 + (NSMethodSignature *)instanceMethodSignatureForSelector:(SEL)aSelector
    //4.2 在方法中，返回新方法的方法签名
    //4.3 重写NSObject的方法 - (void)forwardInvocation:(NSInvocation *)anInvocation
    //根据签名等信息，对NSInvocation的target进行赋值。然后invoke唤醒
    [worker reSignature];
```

方法均没有实现，我们通过消息转发，结果如下：

```objective-c
2019-01-23 19:09:00.825449+0800 runtime_objc_msgSend[14364:5870808] -[Student sayHello]
2019-01-23 19:09:00.825554+0800 runtime_objc_msgSend[14364:5870808] -[Worker forwardInvocation:]
```



消息直接转发

+ 重写NSObject的 -(id)forwardingTargetForSelector:(SEL)aSelector方法
+ 直接返回接收消息的对象实例。



方法重新签名

+ 通过重写NSObject的方法

  + ```objective-c
    - (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector 
    + (NSMethodSignature *)instanceMethodSignatureForSelector:(SEL)aSelector 
    ```

  + 在方法中，我们针对reSignature selector进行了重新签名

+ 重写NSObject方法

  + ```objective-c
    - (void)forwardInvocation:(NSInvocation *)anInvocation 
    ```

  + 方法中，对 reSignatureMethod selector的target进行了重新赋值

  + 唤醒

+ 进入方法发送阶段



