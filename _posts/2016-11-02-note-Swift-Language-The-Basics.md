---
layout:     post
title:      Swift The Basics
date:       2016-11-02 22:06:31
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

​	Swift的很多地方类似C和Objective-C语言。在Swift中，用`Int`表示整型，用`Double`和`Float`表示浮点类型，用`Bool`表示布尔类型，用`String`表示文本类型。另外还提供有一些集合类型，比如`Array`，`Set`和`Dictionary`。是一种类型安全的语言。
​	Swift拥有和C语言类似变量和常量。还有一些高级的类型例如元组`tuples`（同PL/SQL中的类似）、不仅仅用在Class上的可选类型`optional types`(`Optionals say either “there is a value, and it equals x” or “there isn’t a value at all”.`)



### 常量和变量
​	常量和变量在使用前必须声明，使用`var`来声明变量，使用`let`来声明常量，例如

```Swift
let maximumNumberOfLoginAttempts = 10
var currentLoginAttempt = 0
var x = 0.0, y = 0.0, z = 0.0
```

​	可以通过`let(var) name: type`来明确常量或变量的类型，例如：

```Swift
var welcomeMessage: String
var red, green, blue: Double
```

​	变量和常量的名称可以包括绝大多数字符（包括Unicode），但不能包含空格、数学符号、箭头以及一些无效的Unicode，且不能以数字开头。变量或者常量一旦被定义，其名称不能就不能被重复使用，或者改变其类型，也不能将变量改变为常量（反之也不可以）。

```Swift
var friendlyWelcome = "Hello!"
let 你好 = "你好世界"
```

​	通过print函数来输出查看结果：

```Swift
print("The current value of friendlyWelcome is \(friendlyWelcome)")
```

### 注释
​	和C语言类似，可以通过`//`进行单行注释；使用`/*`和`*/`进行多行注释，与C语言不同的是，多行注释在Swift中可以嵌套。例如：

```Swift
// This is a comment.

/* This is also a comment
but is written over multiple lines. */

/* This is the start of the first multiline comment.
/* This is the second, nested multiline comment. */
This is the end of the first multiline comment. */

```

### 分号
​	和其他大部分语言不同的是，Swift不需要你在每个代码语句后边添加分号(;)，尽管写上也没问题

### 整型
​	包括无符号整型`signed`和有符号整型`unsigned`,Swift提供的整型是以8、16、32、64位格式的，例如：`UInt8`、`Int32`。
​	可以通过`min`和`max`两个属性值来查看边界值：

```Swift
let minValue = UInt8.min  // minValue is equal to 0, and is of type UInt8
let maxValue = UInt8.max  // maxValue is equal to 255, and is of type UInt8
```

​	默认情况下，不需要在代码中指明特定的整数，对于`Int`，在32位系统中，默认为`Int32`，64位系统中，默认为`Int64`；对于`UInt`，在32位系统中，默认为`UInt32`，64位系统中，默认为`UInt64`

### 浮点类型
​	Swift中提供两种有符号的浮点类型：用'Double'表示64位浮点类型数据，'Float'表示32位浮点类型。根据所需精度进行选择。

### 类型安全和类型推断
​	Swift是一个类型安全的语言，意味着对类型区分很清晰，类型检查在编译的时候能确保在你的代码中不会出现因失误导致的类型赋值错误。当你没有明确你需要的类型时，Swfit使用类型推断帮助你选择合适的类型，这样Swift会帮你减少很大的代码量。

```Swift
let meaningOfLife = 42// meaningOfLife is inferred to be of type Int

//对于浮点类型，类型推断会将其定义为Double
let pi = 3.14159// pi is inferred to be of type Double
```

### 数值常量
​	对于整型数值常量来说：十进制没有前缀，二进制以`0b`开头，八进制以`0o`开头，十六进制以`0x`开头

```Swift
let decimalInteger = 17
let binaryInteger = 0b10001       // 17 in binary notation
let octalInteger = 0o21           // 17 in octal notation
let hexadecimalInteger = 0x11     // 17 in hexadecimal notation
```

​	浮点数值常量可以是十进制或者十六进制。其中十进制的浮点数值常量中使用字符`e`，代表的是10的exp幂；十六进制数值常量中使用字符`p`，，代表的是2的exp幂

>* 1.25e2 means 1.25 x 10^2, or 125.0.
>* 1.25e-2 means 1.25 x 10^-2, or 0.0125.
>* 0xFp2 means 15 x 2^2, or 60.0.
>* 0xFp-2 means 15 x 2^-2, or 3.75.

​	

​	附小常识：数值常量可以通过添加额外的0或者下划线`_`来帮助阅读。不会影响具体值，例如：

```Swift
let oneMillion = 1_000_000
let justOverOneMillion = 1_000_000.000_000_1
let paddedDouble = 000123.456
```

### 数值类型转换
​	这部分需要注意值域问题，如果转换导致越界，会出现错误。别的没啥好说的，编译器会告诉你的。如果需要的话，转换过程举例如下（显示调用类型方法，比如Double(6) ）：
>>Int8 -> Int16 -> Int -> Double

### 类型别名
​	使用关键字`typealias`起别名，例如：

```Swift
typealias AudioSample = UInt16//起别名
var maxAmplitudeFound = AudioSample.min//使用
```

### 布尔类型
​	Swift中的布尔类型为`Bool`，两个值`true`和`false`。布尔值在`if`语句中很常用。与其他语言不同的是，`if`语句中必须为`Bool`值，否则会出现编译错误。

### 元组
​	将多个（不同类型的）数据组合成一个数据类型，可以使用任何你喜欢类型创建元组`(Int, Int, Int)`,或者`(String, Bool)`，没有任何限制，一个元组的创建例子如下：

```Swift
let http404Error = (404, "Not Found")
// http404Error is of type (Int, String), and equals (404, "Not Found")
```

​	可以将元组的内容分解成独立的常量或变量来单独访问；当你只需要元组中的部分值时，其它部分可以通过下划线(`_`)忽略；当然也可以通过下标的形式来访问(`下标从0开始`)。例如：

```Swift
let (statusCode, statusMessage) = http404Error
print("The status code is \(statusCode)")
// Prints "The status code is 404"
print("The status message is \(statusMessage)")
// Prints "The status message is Not Found"

let (justTheStatusCode, _) = http404Error
print("The status code is \(justTheStatusCode)")
// Prints "The status code is 404"

print("The status code is \(http404Error.0)")
// Prints "The status code is 404"
print("The status message is \(http404Error.1)")
// Prints "The status message is Not Found"
```

​	定义元组的时候可以给单独的元素起名字，然后可以通过元素名字来访问具体的值，例如：

```Swift
let http200Status = (statusCode: 200, description: "OK")

print("The status code is \(http200Status.statusCode)")
// Prints "The status code is 200"
print("The status message is \(http200Status.description)")
// Prints "The status message is OK"
```

>> 当函数有多个返回值时，元组显得非常适用；但元组不适合于复杂的数据结构创建。

###  可选类型
​	当一个值可能不存在时，使用可选类型(`optionals`)，可选表示两种可能：要么存在一个值，你可以解包并访问其值；要么不存在。

>> ​	注：可选的概念在C或者Objective-C中是不存在的，ObjC中一个相似点是一个方法要么返回一个对象，要么返回nil，但是在ObjC中仅适用于对象，不适用于结构体、基本C类型或者枚举类型。但是Swift中的可选适用于所有类型。

```Swift
let possibleNumber = "123"
let convertedNumber = Int(possibleNumber)
// convertedNumber is inferred to be of type "Int?", or "optional Int"
```

#### nil
​	可以将nil赋值给一个可选类型变量。对于可选类型变量，如果声明的时候没有提供初始值，则默认被设置为nil。

>> ​	注：nil不能被可选类型以外的变量或者常量适用，如果代码中一个变量需要处理不存在的情况，通常将其定义为可选类型。Swift中的nil和ObjC中的nil不同：ObjC中的nil是一个指向不存在的对象的指针；Swift中的nil不是指针，它是一个特定类型不存在的值，适用于所有Swift类型

```Swift
var serverResponseCode: Int? = 404
// serverResponseCode contains an actual Int value of 404
serverResponseCode = nil
// serverResponseCode now contains no value

var surveyAnswer: String?
// surveyAnswer is automatically set to nil
```

#### if语句和强制解包
​	可以通过使用If语句来确定一个可选类型是一个值或者是nil(使用操作符`==`或`!=`)。如果一个可选类型存在一个值，就可以通过在参数名后边添加`!`来强制解包该可选类型值。

>> 注：[重要] 使用`!`的时候一定要确保可选类型的值是存在的，否则会出现运行时错误，导致程序闪退。


```Swift
if convertedNumber != nil {
    print("convertedNumber has an integer value of \(convertedNumber!).")
}
```

#### 可选绑定
​	在if和while语句中，可以通过使用可选绑定来检查可选值。if语句中使用可选绑定格式如下

```Swift
if let constantName = someOptional {//if var constantName
    non-nil-statements
} else {
    nil-statement
}
```

#### 可选类型的隐式解包
​	通常情况下，对于一个可选类型，如果在设置初始值后能够`确定包含一个值`，就没有必要每次通过可选绑定来进行检查、解包。这时可以通过在可选类型名后边添加`!`来隐式解包

```Swift
let possibleString: String? = "An optional string."
let forcedString: String = possibleString! // requires an exclamation mark
 
let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString // no need for an exclamation mark
```

### 错误处理
在程序执行过程中，可以通过错误处理来响应程序运行期间遇到的错误。当一个函数遇到错误的时候，可以抛出，函数调用者捕捉该错误并做相应的相应处理。函数通过关键字`throws`来表明会抛出错误。当调用该类型的函数时，使用`try`关键字

```Swift
func canThrowAnError() throws {
    // this function may or may not throw an error
}

do {
    try canThrowAnError()
    // no error was thrown
} catch {//默认error，当然你也可以使用多个catch，并明确具体的错误类型 比如：catch SandwichError.outOfCleanDishes
    // an error was thrown
}


```


### 断言
​	在一些特定的情况下，代码没有必要继续执行。这时候可以通过触发断言来终止代码的执行，并告知原因。

​	通常情况下，断言的使用时机：
>* 在Debug的时候使用；

>* 使用下标访问数据时；

>* 函数入口参数检查时；

>* 可选类型要求非nil时；




### 参照文档

`https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html`

>
>
>
>
>
>
>​Copyright (c) liangtong. All rights reserved.
>


