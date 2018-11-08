---
layout:     post
title:      Swift 学习笔记
date:       2016-11-01 22:06:31
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

Swift3 发布有段时间了，考虑到需要将项目中的Swift2.x中的语言转换到3.x版本，就打算把苹果官方文档给看一遍，顺便记录下来



### 前言

​	Swift builds on the best of C and Objective-C, without the constraints of C compatibility.Swift adopts safe programming patterns and adds modern features to make programming easier, more flexible, and more fun. is an opportunity to reimagine how software development works.


​	Swift采用安全的编程模式，并添加了使编程更加简单、灵活、有趣的特色，重新定义软件开发过程！和ObjC语言类似，采用可读性的参数命名方式，是一款友好的脚本语言。




### Release Log

| 开始时间       |   耗时 |                    内容                    |
| ---------- | ---: | :--------------------------------------: |
| 2016／11／02 |   4H | <a href="https://l900416.github.io/2016/11/02/iOS-Swift-Language-The-Basics/">基础知识</a> |
| 2016／11／04 |   2H | <a href="https://l900416.github.io/2016/11/04/iOS-Swift-Language-Basic-Operators/">基本操作符</a> |
| 2016／11／05 |   3H | <a href="https://l900416.github.io/2016/11/05/iOS-Swift-Language-String-and-Characters/">字符串和字符</a> |
| 2016／11／06 |   3H | <a href="https://l900416.github.io/2016/11/06/iOS-Swift-Language-Collection-Types/">集合类型</a> |
| 2016／11／17 |   4H | <a href="https://l900416.github.io/2016/11/17/iOS-Swift-Language-Control-Flow/">流程控制</a>  |
| 2016／11／20 |   6H | <a href="https://l900416.github.io/2016/11/20/iOS-Swift-Language-Functions/">函数</a>  |
| 2016／11／22 |   6H | <a href="https://l900416.github.io/2016/11/22/iOS-Swift-Language-Closures/">闭包</a>  |
| 2016／11／23 |   6H | <a href="https://l900416.github.io/2016/11/23/iOS-Swift-Language-Enumerations/">枚举</a>  |
| 2016／11／24 |   4H | <a href="https://l900416.github.io/2016/11/24/iOS-Swift-Language-Classes-and-Structures/">类和结构体</a>  |
| 2016／11／25 |   2H | <a href="https://l900416.github.io/2016/11/25/iOS-Swift-Language-Properties/">属性</a>  |
| 2016／11／25 |   2H | <a href="https://l900416.github.io/2016/11/25/iOS-Swift-Language-Methods/">方法</a>  |
| 2016／11／26 |   2H | <a href="https://l900416.github.io/2016/11/25/iOS-Swift-Language-Subscripts/">下标</a>  |
| 2016／11／26 |   2H | <a href="https://l900416.github.io/2016/11/26/iOS-Swift-Language-Inheritance/">继承</a>  |
| 2016／11／28 |   8H | <a href="https://l900416.github.io/2016/11/27/iOS-Swift-Language-Initialization/">构造过程</a>  |
| 2016／11／28 |   2H | <a href="https://l900416.github.io/2016/11/28/iOS-Swift-Language-Deinitialization/">析构过程</a>  |
| 2016／11／29 |   4H | <a href="https://l900416.github.io/2016/11/29/iOS-Swift-Language-ARC/">ARC</a>  |
| 2016／11／30 |   3H | <a href="https://l900416.github.io/2016/11/30/iOS-Swift-Language-Optional-Chaining/">可选链</a>  |
| 2016／12／01 |   2H | <a href="https://l900416.github.io/2016/11/30/iOS-Swift-Language-Error-Handling/">错误处理</a>  |
| 2016／12／01 |   2H | <a href="https://l900416.github.io/2016/11/30/iOS-Swift-Language-Type-Casting/">类型转换</a>  |
| 2016／12／02 |   2H | <a href="https://l900416.github.io/2016/12/01/iOS-Swift-Language-Extensions/">扩展</a>  |
| 2016／12／02 |   2H | <a href="https://l900416.github.io/2016/12/01/iOS-Swift-Language-Nested-Types/">嵌套类型</a>  |
| 2016／12／03 |   5H | <a href="https://l900416.github.io/2016/12/02/iOS-Swift-Language-Protocols/">协议</a>  |
| 2016／12／03 |   2H | <a href="https://l900416.github.io/2016/12/02/iOS-Swift-Language-Generics/">泛型</a>  |
| 2016／12／05 |   3H | <a href="https://l900416.github.io/2016/12/05/iOS-Swift-Language-Access-Control/">访问控制</a>  |
|   |     |    |
|   |     |    |





### 后记
历经一个月，终于把Swift3.x文档看了一遍，鉴于个人英文阅读能力，后期在阅读官方文档的同时参照了<a href="http://wiki.jikexueyuan.com/project/swift">极客学院Swift翻译教程</a>。<br><br>
在阅读的过程中，也思考并查阅了很多在使用Objective-C的过程中没有注意到的问题，比如闭包；也学习到了一些与其他语言不同的东西，比如构造过程；对于一些问题的理解更加深刻了，比如ARC等。
<br>总体来说，这次收获还是蛮多的。

<p>
毕竟语言只是一种表达自己想法的途径，接下来的时间，就通过实践来慢慢消化吧。<br><br><br>


—    后记于 2016/12/05晚

</p>



>

>
>
>
>
>
>​Copyright (c) liangtong. All rights reserved.
>


