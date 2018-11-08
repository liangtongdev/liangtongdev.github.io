---
layout:     post
title:      Swift Classes and Structures
date:       2016-11-24 00:20:11
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

​	类和结构体是人们构建代码所用的一种通用且灵活的构造体。你可以使用几乎完全相同的语法规则给类和结构体定义属性、添加方法。与其他语言不同的是，Swift不需要你为自定义类和结构体去独立创建接口和实现文件。在Swwift中，你只需要使用一个单独的文件来定义类和结构体，系统会自动生成面向其他代码的外部接口。
	通常类的实例被称为对象，Swift中类和结构体的关系比在其他语言中要密切



### 类和结构体的对比
类和结构体在Swift中有很多共同点，例如：

 * 定义属性用于存储值
 * 定义方法用于提供功能
 *  定义下标操作涌来通过下标语法来访问实例存储的值
 *  定义构造函数用于初始化值
 *  通过扩展来增加其功能的实现
 *  实现协议以提供某种标准功能

与结构体相比，类还有如下额外功能：

 *  继承允许一个类继承另一个类的特征
 *  类型转换允许在运行时检查和解释一个类实例等类型
 *  析构器允许一个类释放其所被分配到资源
 *  引用计数允许一个类被多次引用。

	注意：结构体通常是通过值传递，而不使用引用计数

#### 定义语法
类和结构体有相似的定义语法，通过关键字`class`定义类，`struct`定义结构体，在一对大括号中定义它们的具体内容：

```Swift
class SomeClass {
    // class definition goes here
}
struct SomeStructure {
    // structure definition goes here
}
```

当你定义了类或者结构体的时候，确切的讲是定义了一个新类型。使用UpperCamelCase这种方式来命名（如SomeClass和SomeStructure等），相反的，请使用lowerCamelCase这种方式为属性和方法命名（如framerate和incrementCount），以便和类型名区分。

以下是定义结构体和类的示例：

```Swift
struct Resolution {
    var width = 0
    var height = 0
}
class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}
```

#### 类和结构体实例
生成类和结构体实例等语法很相似，例如：


```Swift
let someResolution = Resolution()
let someVideoMode = VideoMode()
```

结构体和类都使用构造器语法来生成新的实例。构造器语法的最简单形式是在结构体或者类的类型名称后跟随一对空括号，如Resolution()或VideoMode()。通过这种方式所创建的类或者结构体实例，其属性均会被初始化为默认值。

#### 属性访问
通过使用点语法，你可以访问实例的属性。其语法规则是，实例名后面紧跟属性名，两者通过点号(.)连接，不能有空格：


```Swift
print("The width of someResolution is \(someResolution.width)")
// Prints "The width of someResolution is 0"

print("The width of someVideoMode is \(someVideoMode.resolution.width)")
// Prints "The width of someVideoMode is 0"
```

也可以使用点语法为变量属性赋值，例如：

```Swift
someVideoMode.resolution.width = 1280
print("The width of someVideoMode is now \(someVideoMode.resolution.width)")
// Prints "The width of someVideoMode is now 1280"
```

	与 Objective-C 语言不同的是，Swift 允许直接设置结构体属性的子属性。上面的最后一个例子，就是直接设置了someVideoMode中resolution属性的width这个子属性，以上操作并不需要重新为整个resolution属性设置新值。

#### 结构体成员逐一构造函数
所有结构体都有一个自动生成的成员逐一构造器，用于初始化新结构体实例中成员的属性。新实例中各个属性的初始值可以通过属性的名称传递到成员逐一构造器之中：

```Swift
let vga = Resolution(width: 640, height: 480)
```

	与结构体不同的是，类实例没有默认的成员逐一构造器

### 结构体和枚举是值类型
值类型被赋予给一个变量、常量或者被传递给一个函数的时候，其值会被拷贝。
在Swift中，所有的基本类型：整数（Integer）、浮点数（floating-point）、布尔值（Boolean）、字符串（string)、数组（array）和字典（dictionary），都是值类型，并且在底层都是以结构体的形式所实现。
在 Swift 中，所有的结构体和枚举类型都是值类型。这意味着它们的实例，以及实例中所包含的任何值类型属性，在代码中传递的时候都会被复制。例如：

```Swift
let hd = Resolution(width: 1920, height: 1080)
var cinema = hd
```

声明一个名为hd的常量，初始值宽高分别为1920和1080，然后声明一个名为cinema的变量，并将hd赋值给它。由于Resolution是结构体类型，所以cinema的值其实是hd的一个拷贝副本，而不是hd本身。尽管它们两个的值相同，但是是两个不同的实例。例如以下修改后，hd的值不变：

```Swift
cinema.width = 2048

print("cinema is now \(cinema.width) pixels wide")
// Prints "cinema is now 2048 pixels wide"

print("hd is still \(hd.width) pixels wide")
// Prints "hd is still 1920 pixels wide"
```

枚举也遵循相同的行为标准

```Swift
enum CompassPoint {
    case north, south, east, west
}
var currentDirection = CompassPoint.west
let rememberedDirection = currentDirection
currentDirection = .east
if rememberedDirection == .west {
    print("The remembered direction is still .west")
}
// Prints "The remembered direction is still .west"
```

### 类是引用类型
与值类型不同，引用类型在被赋予到一个变量、常量或者被传递到一个函数时，其值不会被拷贝。因此，引用的是已存在的实例本身而不是其拷贝。
例如，实例化一个类VideoMode的对象，并为其属性赋值：

```Swift
let tenEighty = VideoMode()
tenEighty.resolution = hd
tenEighty.interlaced = true
tenEighty.name = "1080i"
tenEighty.frameRate = 25.0
```

然后将其赋值给一个新变量，同时对新变量进行修改

```Swift
let alsoTenEighty = tenEighty
alsoTenEighty.frameRate = 30.0
```

因为类是引用类型，所以tenEight和alsoTenEight实际上引用的是相同的VideoMode实例。换句话说，它们是同一个实例的两种叫法。

```Swift
print("The frameRate property of tenEighty is now \(tenEighty.frameRate)")
// Prints "The frameRate property of tenEighty is now 30.0"
```

	需要注意的是tenEighty和alsoTenEighty被声明为常量而不是变量。然而你依然可以改变tenEighty.frameRate和alsoTenEighty.frameRate，因为tenEighty和alsoTenEighty这两个常量的值并未改变。它们并不“存储”这个VideoMode实例，而仅仅是对VideoMode实例的引用。所以，改变的是被引用的VideoMode的frameRate属性，而不是引用VideoMode的常量的值。

#### 恒等运算符
因为类是引用类型，有可能有多个常量和变量在幕后同时引用同一个类实例。（对于结构体和枚举来说，这并不成立。因为它们作为值类型，在被赋予到常量、变量或者传递到函数时，其值总是会被拷贝。）
如果能够判定两个常量或者变量是否引用同一个类实例将会很有帮助。为了达到这个目的，Swift 内建了两个恒等运算符：

 *  === 等价于
 *  !== 不等价于

运用这两个运算符可以检查两个变量或者常量是否引用同一个实例：

```Swift
if tenEighty === alsoTenEighty {
    print("tenEighty and alsoTenEighty refer to the same VideoMode instance.")
}
// Prints "tenEighty and alsoTenEighty refer to the same VideoMode instance."
```

	注意`=== 等价于`和`== 等于`的区别：一个是判断引用相同的类实例，一个是判断值相等

#### 指针
在C、C++、ObjC中，使用指针类引用内存中的地址，Swift中某个引用类型的常量或者变量和其他语言中的指针类似，但不是直接指向内存地址，也不要求你使用星号（*）来表明你在创建一个引用。Swift 中的这些引用与其它的常量或变量的定义方式相同。

### 类和结构体的选择
你可以使用类或结构体来定义你自己的数据类型。但是结构体总是通过值传递，类是引用传递。这意味着两者适用不同的任务。当你在考虑一个工程项目的数据结构和功能的时候，你需要决定每个数据结构是定义成类还是结构体。

按照通用的准则，如果符合以下一条或者多条时，可以使用结构体：

 *  该数据结构的主要目的是用来封装少量相关简单数据值。
 *  有理由预计该数据结构的实例在被赋值或传递时，封装的数据将会被拷贝而不是被引用。
 *  该数据结构中储存的值类型属性，也应该被拷贝，而不是被引用。
 *  该数据结构不需要去继承另一个既有类型的属性或者行为。

### 字符串、数组、和字典类型的赋值与复制行为
Swift 中，许多基本类型，诸如String，Array和Dictionary类型均以结构体的形式实现。这意味着被赋值给新的常量或变量，或者被传入函数或方法中时，它们的值会被拷贝。

Objective-C 中NSString，NSArray和NSDictionary类型均以类的形式实现，而并非结构体。它们在被赋值或者被传入函数或方法时，不会发生值拷贝，而是传递现有实例的引用。

以上是对字符串、数组、字典的“拷贝”行为的描述。在你的代码中，拷贝行为看起来似乎总会发生。
然而，Swift 在幕后只在绝对必要时才执行实际的拷贝。Swift 管理所有的值拷贝以确保性能最优化，所以你没必要去回避赋值来保证性能最优化。	








### 参照文档

>* <a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/ClassesAndStructures.html/">ClassesAndStructures</a> 


>
>
>Copyright (c) liangtong. All rights reserved.
>


