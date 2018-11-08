---
layout:     post
title:      Swift Control Flow
date:       2016-11-17 00:20:11
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

​	Swift提供了很多流程控制语法，包括可以通过使用`while`循环来重复执行一个任务，可以通过`if`、`guard`和`switch`来进行条件判断从而执行不同代码分支，可以通过`break`和`continue`来跳转到特定代码位置。
​	Swift也提供了诸如可以通过`for-in`循环来遍历数组，字典，区间(range)，字符串和其他一些序列(sequences)。
​	Swift的`switch`语句相对比C语言的`强大`(该关键字与其他多数语言不同，至于算不算优点，意见保留)。在Swift中，该语句中的`case`语法不会自动流转到下一个`case`，这避免了像C语言中因为缺失`break`语句导致的错误。而Swift中的Case语句也相对比较强大，可以匹配多种包括整型、元组、转换的特殊类型等形式。在Case语句中匹配的值可以被绑定到一个临时的常量或者变量中，从而在`case`语句对一个的代码块(body)中使用，一些复杂的匹配可以通过使用`where`表达式进行。



### for-in 循环
可以使用`for-in`循环来遍历一个序列，比如一个区间(range)中的数字、数组中的元素、字符串的字符，例如：

```Swift
//使用index变量来访问range中的具体值。注意操作符`...`的用法
for index in 1...5 {
    print("\(index) times 5 is \(index * 5)")
}
//不关心range中的具体值，可以使用下划线’_’
let base = 3
let power = 10
var answer = 1
for _ in 1...power {
    answer *= base
}
print("\(base) to the power of \(power) is \(answer)")
//遍历数组
let names = ["Anna", "Alex", "Brian", "Jack"]
for name in names {
    print("Hello, \(name)!")
}
//字典，是以元组的形式来匹配的，当然也可以使用下划线’_’
let numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]
for (animalName, legCount) in numberOfLegs {
    print("\(animalName)s have \(legCount) legs")
}

```
### while循环
while的代码块会一直被执行，知道条件变为flase。这类循环主要用在不确定遍历次数的情况下，Swift提供了两种while循环语法：`while`和`repeat-while`

#### while
while循环是以一个简单的bool条件式开始，当该条件为true的时，循环体会一直被执行。知道条件变为false。语法如下：

```Swift
while condition {
    statements
}
```

#### repeat-while 
与`while`不同的是，`repeat-while`循环会首先执行以下循环体，然后判断条件是否为true。和其他语言中的`do-while`循环类似。语法如下：

```Swift
repeat {
    statements
} while condition
```

### 分支判断
根据特定的条件执行对应的处理代码是很常用的，Swift提供了两种分支判断语法：`if`和`switch`，通常情况下`if`用在简单的分支判断，`switch`用在比较复杂的场合。

#### if 
if语句包含一个条件，如果条件为true，则执行紧接着的语句集合。否则执行else (if) 对应的语句集合（如果不关心else (if) 的情况，该部分可以不写）。

#### switch
switch语句将一个值和多种匹配的模式进行比较，然后根据第`一`个合适的配结果执行对应的代码块，语法格式如下：

```Swift
switch some value to consider {
case value 1:
    respond to value 1
case value 2,
     value 3:
    respond to value 2 or 3
default:
    otherwise, do something else
}
```
switch语句包含多个情况，每个case语句都代表一个分支。switch语句必须列举出所有情况，也就是说case分支必须能够匹配所有可能的情况，如果case不能完全匹配，则需要通过关键字`default`在最后位置定义一个默认的分支，例如：

```Swift
let someCharacter: Character = "z"
switch someCharacter {
case "a":
    print("The first letter of the alphabet")
case "z":
    print("The last letter of the alphabet")
default:
    print("Some other character")
}
```

##### 不存在隐式穿透(No Implicit Fallthrough)
和C与ObjC语言中的Switch语法不同的是，Swift中的switch语法默认情况下在case语句执行完成后不会自动匹配下一个case语句，而是当第一个匹配case分支执行之后马上结束，你不需要像其他语言那样为每个case写上一个break关键字。（尽管break在swift中switch中是非必需的，但你可以使用它来中断一些代码的执行）。如果想自动匹配下一个，则需要使用关键字`fallthrough`。
每一条case分支必须包含至少一个可执行语句，比如以下代码会引起编译时错误

```Swift
let anotherCharacter: Character = "a"
switch anotherCharacter {
case "a": // Invalid, the case has an empty body
case "A":
    print("The letter A")
default:
    print("Not the letter A")
}
// This will report a compile-time error.
```
一条case分支可以匹配多个值，用逗号`,`隔开，比如上述代码会引起编译时错误可以通过以下写法来解决

```Swift
let anotherCharacter: Character = "a"
switch anotherCharacter {
case "a", "A":
    print("The letter A")
default:
    print("Not the letter A")
}
// Prints "The letter A"
```

##### 区间匹配
switch中的值可以进行区间匹配，比如：

```Swift
let approximateCount = 62
let countedThings = "moons orbiting Saturn"
var naturalCount: String
switch approximateCount {
case 0:
    naturalCount = "no"
case 1..<5:
    naturalCount = "a few"
case 5..<12:
    naturalCount = "several"
case 12..<100:
    naturalCount = "dozens of"
case 100..<1000:
    naturalCount = "hundreds of"
default:
    naturalCount = "many"
}
print("There are \(naturalCount) \(countedThings).")
// Prints "There are dozens of moons orbiting Saturn."
```

##### 元组
可以通过使用下划线`_`来匹配任意一个元素，如果存在多个匹配，默认情况下执行第一个匹配的case，其他case会被忽略。代码比较清楚：

```Swift
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    print("(0, 0) is at the origin")
case (_, 0):
    print("(\(somePoint.0), 0) is on the x-axis")
case (0, _):
    print("(0, \(somePoint.1)) is on the y-axis")
case (-2...2, -2...2):
    print("(\(somePoint.0), \(somePoint.1)) is inside the box")
default:
    print("(\(somePoint.0), \(somePoint.1)) is outside of the box")
}
// Prints "(1, 1) is inside the box"
```

##### 值绑定
在switch的case语句中，可以将匹配的值绑定给一个临时的常量或者变量，在case的body中使用。例如：

```Swift
let anotherPoint = (2, 0)
switch anotherPoint {
case (let x, 0):
    print("on the x-axis with an x value of \(x)")
case (0, let y):
    print("on the y-axis with a y value of \(y)")
case let (x, y):
    print("somewhere else at (\(x), \(y))")
}
// Prints "on the x-axis with an x value of 2"
```

##### where
一个case语句中可以使用where语法进行额外的匹配检查，比如：

```Swift
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
case let (x, y) where x == y:
    print("(\(x), \(y)) is on the line x == y")
case let (x, y) where x == -y:
    print("(\(x), \(y)) is on the line x == -y")
case let (x, y):
    print("(\(x), \(y)) is just some arbitrary point")
}
// Prints "(1, -1) is on the line x == -y"
```

##### 复合匹配
在case中通过逗号`,`隔开匹配内容。复合匹配也可以包含值绑定，在复合匹配中，无论哪个部分匹配上，case的body都会被执行。例如：

```Swift
let stillAnotherPoint = (9, 0)
switch stillAnotherPoint {
case (let distance, 0), (0, let distance):
    print("On an axis, \(distance) from the origin")
default:
    print("Not on an axis")
}
// Prints "On an axis, 9 from the origin"
```

### 控制转移语句
swift提供了5个流程控制转移语句：`continue`、`break`、`fallthrough`、`return`、`throw`。
除了`fallthrough`，其他四个关键字与C等语言中的基本相同，swift中的switch语句默认情况下不存在隐式穿透，如果想要支持，则需要在case语句后添加`fallthrough`，（注：该关键字不检查即将进入的case条件）例如：

```Swift
let integerToDescribe = 5
var description = "The number \(integerToDescribe) is"
switch integerToDescribe {
case 2, 3, 5, 7, 11, 13, 17, 19:
    description += " a prime number, and also"
    fallthrough
default:
    description += " an integer."
}
print(description)
// Prints "The number 5 is a prime number, and also an integer."
```

### 标签语法
循环或条件语句可以嵌套，可以通过break语法结束当前代码块的执行，如果有多个嵌套的循环，明确哪个循环是非常有用的，为了实现这个目标，你可以使用标签语法。标签语法是由名字和冒号`:`组成。例如一个while循环的标签如下：

```Swift
label name: while condition {
    statements
}
```

### 提前退出
和`if`语法类似，`guard`语法语句的执行取决于一个bool表达式。可以使用`guard`语法来确保在特定条件成立的情况下执行某些代码。和`if`不同的是，`guard`语法在条件表达式为false时会执行`else`块中的内容。例如：

```Swift
guard let name = person["name"] else {
        return
    }
    
    print("Hello \(name)!")
```

### API有效性检查
可以在`if`或`guard`语句中使用`availability`条件。其中平台比如：`iOS`, `macOS`, `watchOS`, 和 `tvOS`，语法格式如下

```Swift
if #available(platform name version, ..., *) {
    statements to execute if the APIs are available
} else {
    fallback statements to execute if the APIs are unavailable
}
```
具体写法例如：

```Swift
if #available(iOS 10, macOS 10.12, *) {
    // Use iOS 10 APIs on iOS, and use macOS 10.12 APIs on macOS
} else {
    // Fall back to earlier iOS and macOS APIs
}
```

### 参照文档

>* <a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/ControlFlow.html/">Control Flow</a> 

>
>
>Copyright (c) liangtong. All rights reserved.
>


