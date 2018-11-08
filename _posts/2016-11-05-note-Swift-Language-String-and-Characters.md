---
layout:     post
title:      Swift String and Characters
date:       2016-11-05 00:20:11
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

​	字符串`String`是一系列字符集合，Swift提供了一种快速、Unicode的方式的字符串和字符类型。和C语言的字符串字面值语法类似，Swift的字符串的创建和操作是轻量级、可读性的。可以通过`+`操作符连接两个字符串。Swift的字符串中的每个字符都是由`Unicode`编码的字符组成，并支持以各种方式访问这些字符。
    注：Swift中的String类型通过Foundation框架的NSString类型桥接，Foundation也对String类型对应扩展了NSString中定义的方法。这就意味着如果引用了Foundation，就可以不用转换直接使用NSString中的方法。



### 字符串的字面值
​	可以通过双引号`"`来定义字符串字面值，Swift会根据字符串字面值将对应类型推断为字符串类型。

```Swift
let someString = "Some string literal value"
```

### 字符串操作

#### 空字符串
​	初始化，检查字符串是否为空。

```Swift
var emptyString = ""               // empty string literal
var anotherEmptyString = String()  // initializer syntax

if emptyString.isEmpty {
print("Nothing to see here")
}
```

#### 字符串是值类型
​	字符串是值类型，创建一个字符串后，如果在函数或者方法中使用，或者该字符串会被赋值给常量或者变量，字符串值会被复制，源字符串数据不会变化。

#### 字符串与字符
​	可以使用`for-in`循环，通过使用字符串的`characters`属性来访问字符值，例如

```Swift
for character in "Dog!�".characters {
print(character)
}
// D
// o
// g
// !
// �
```

​	可以通过字符来创建字符串，例如：

```Swift
let exclamationMark: Character = "!"

let catCharacters: [Character] = ["C", "a", "t", "!", "�"]
let catString = String(catCharacters)
```

#### 字符串连接
​	可以通过操作符`+`、`+=`连接两个字符串，也可以通过使用String的append()方法给字符串追加一个字符值，例如：

```Swift
var welcome = "hello" + " there"

var instruction = "look over"
instruction += string2

let exclamationMark: Character = "!"
welcome.append(exclamationMark)
```

### Unicode
​	用的不多，不详细介绍

### 字符串访问和修改

​	不同的字符需要不同的空间来存储，所以为了确定哪个字符的特定位置，需要遍历字符串。正是这个原因，Swift字符串不能通过整型的索引来访问。
可以通过`startIndex`属性来访问字符串的第一个字符位置，`endIndex`属性对应最后一个字符位置。如果一个字符串的两个属性相同，则说明是一个空字符串。另外String还提供来几个方法来确认位置：` index(before:)`、`index(after:)`、`index(_:offsetBy:)`。如果访问越界，则会出现运行时错误：

```Swift
greeting[greeting.endIndex] // Error
```

字符的插入和移除可以通过方法`insert(_:at:)`和`remove(at:)`进行

### 字符串比较
​	Swift提供了3中方式比较文字值：字符串和字符等式、前缀等式、后缀等式

#### 字符串和字符等式
​	可以通过操作符：`==`和`!=`，例如：

```Swift
let quotation = "We're a lot alike, you and I."
let sameQuotation = "We're a lot alike, you and I."
if quotation == sameQuotation {
print("These two strings are considered equal")
}
// Prints "These two strings are considered equal"
```

#### 前缀、后缀等式
​	Swift提供了方法：`hasPrefix(_:)`和`hasSuffix(_:)`

### 字符串的Unicode表示

​	Swift提供了不同方式来访问不同版本的Unicode字符，你可以遍历字符串通过`for-in`语句，然后通过访问每个字符的Unicode编码。

> * 通过访问字符串的utf8属性，对于UTF-8格式
> * 通过访问字符串的utf16属性，对于UTF-16格式
> * 通过访问字符串的unicodeScalars属性，对于UTF-32格式

举个例子，对于一个字符串，需要访问不同格式的编码：

```Swift
let dogString = "Dog‼🐶"
```
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/UTF8_2x.png)
对于UTF-8格式表示，如下：

```Swift
for codeUnit in dogString.utf8 {
print("\(codeUnit) ", terminator: "")
}
print("")
// 68 111 103 226 128 188 240 159 144 182
```
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/UTF16_2x.png)
对于UTF-16格式表示，如下：

```Swift
for codeUnit in dogString.utf16 {
print("\(codeUnit) ", terminator: "")
}
print("")
// Prints "68 111 103 8252 55357 56374 "
```
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/UnicodeScalar_2x.png)
对于Unicode Scalar格式表示，如下：

```Swift
for codeUnit in dogString.utf16 {
for scalar in dogString.unicodeScalars {
print("\(scalar.value) ", terminator: "")
}
print("")
// Prints "68 111 103 8252 128054 "

for scalar in dogString.unicodeScalars {
print("\(scalar) ")
}
// D
// o
// g
// ‼
// 🐶
```

### 参照文档

>* <a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/StringsAndCharacters.html/">Strings and Characters</a> 

>
>
>
>
>
>
>​Copyright (c) liangtong. All rights reserved.
>


