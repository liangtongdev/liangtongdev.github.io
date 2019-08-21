---
layout:     post
title:      Objective-C对象的分类
date:       2019-01-26 13:50:12
author:     liangtong
catalog: true
categories: iOS
tags: objc

---


![分类.png](/post/oc/20190126/分类.png)

#### Ojbective-C对象

- instance 实例对象：就是通过类alloc出来的对象，每次调用alloc都会产生出来一个新对象
  - isa指针
  - 其他成员变量的值
- class 类对象：每个类在内存中有且只有一个类对象
  - isa指针
  - superclass指针
  - 类的属性信息（@property）
  - 类的对象方法信息（instance method）
  - 类的协议信息（protocol）
  - 类的成员变量信息（ivar）
- meta-class 元类对象：每个类在内存中有且只有一个。元类对象元类对象的结构和类对象的结构相同，但是用途不一样。元类对象在内存中存储的信息主要包括
  - isa指针
  - 类方法信息（class method）





<!-- more -->





---

![流程.png](/post/oc/20190126/流程.png)


#### isa指针

注：实例对象isa地址不是直接对应类对象的地址，类对象的isa地址也不是直接对应元类对象 的地址，在64bit开始，isa需要进行一次位运算才能计算出真正地址。



- instance的isa指向class
  - 当调用对象方法(instance method)时，通过instance的isa找到class。
    - 如果找到对象方法的实现进行调用。
    - 如果找不到，则通过class的superclass找到父类，然后找父类的方法实现
- class的isa指向meta-class
  - 当调用类方法(class method)时，通过class对象的isa找到元类对象，
    - 如果找到类方法调用。
    - 如果找不到，则通过superclass找父类，然后找父类的类方法
- meta-class的isa指向基类的meta-class



#### superclass

- class的superclass指向父类的class，如果没有父类，则superclass指向nil
- meta-class的superclass指向父类的meta-class。基类的meta-class的superclass指向基类的class



#### OC的类信息存放位置

- 成员变量的具体的值，存放在实例对象instance里。
- 属性信息、对象方法、协议信息、成员变量信息，存放在类对象class里。
- 类方法信息，存放在元类对象meta-class里。
