---
layout:     post
title:      Swift Functions
date:       2016-11-20 00:20:11
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

​	函数是执行特定功能的独立代码块，可以通过函数名字调用函数执行其功能。Swift的函数统一语法可以很灵活地表达任何东西，无论是没有参数的C风格函数还是拥有多个名字和参数的ObjC风格函数。参数可以提供默认值以简化函数调用、当参数变量需要改变的时候，参数也可以作为输入输出(`in-out`)参数.
在Swift中，每个函数都有一个类型，包括参数类型和返回值类型。你可以像Swift中其他任何一个类型一样使用函数，这使得将函数作为函数参数、作为函数返回值变得非常简单。函数也可以写在其他函数内部以限制其作用域（内部函数）。



### 函数的声明和调用
当定义函数的时候，可以选择性的定义一个或多个名字，定义一些输入类型值作为参数，也可以选择性的定义一类值作为输出类型。每一个函数都有一个名字，用来表明函数的功能，可以通过调用该函数的名字来使用函数，并且通过传递输入值（参数）来匹配函数的参数，调用函数时，输入参数类型必须和函数声明时类型的参数类型顺序一致。

函数以关键字`func`开头，返回值类型紧跟符号`->`。说了这么多，还是直接看例子，函数名字为：`greet(person:)`参数和返回值是String类型，定义及调用如下：

```Swift
func greet(person: String) -> String {
    let greeting = "Hello, " + person + "!"
    return greeting
}

print(greet(person: "Anna"))
// Prints "Hello, Anna!"
print(greet(person: "Brian"))
// Prints "Hello, Brian!"
```

### 函数的参数和返回值
Swift中函数的参数和返回值比较灵活，无论是一个只有一个未命名参数的简单函数，还是包含不同类型参数的复杂函数，你可以随意定义。

#### 无参函数
函数参数是非必需的，例如：

```Swift
func sayHelloWorld() -> String {
    return "hello, world"
}
print(sayHelloWorld())
// Prints "hello, world"
```

#### 多参函数
函数可以有多个输入参数，这些参数写在函数的括号内，用逗号隔开。例如：

```Swift
func greet(person: String, alreadyGreeted: Bool) -> String {
    if alreadyGreeted {
        return greetAgain(person: person)
    } else {
        return greet(person: person)
    }
}
print(greet(person: "Tim", alreadyGreeted: true))
// Prints "Hello again, Tim!"
```

#### 无返回值函数
函数也不需要定义返回值类型，当没有返回值的时候，返回箭头（ - >）和返回类型可以省略。例如：

```Swift
func greet(person: String) {
    print("Hello, \(person)!")
}
greet(person: "Dave")
// Prints "Hello, Dave!"
```

 > * 严格的说，即使没有定义返回值，函数仍然会返回一个void类型的特殊值，他是一个简单的空的元组，可以写为()。

再调用函数时，其返回值可以被忽略，例如：

```Swift
func printAndCount(string: String) -> Int {
    print(string)
    return string.characters.count
}
func printWithoutCounting(string: String) {
    let _ = printAndCount(string: string)
}
printAndCount(string: "hello, world")
// prints "hello, world" and returns a value of 12
printWithoutCounting(string: "hello, world")
// prints "hello, world" but does not return a value
```

#### 多返回值函数
可以使用元组作为函数的返回值来一次返回多个值。例如：

```Swift
func minMax(array: [Int]) -> (min: Int, max: Int) {
    var currentMin = array[0]
    var currentMax = array[0]
    for value in array[1..<array.count] {
        if value < currentMin {
            currentMin = value
        } else if value > currentMax {
            currentMax = value
        }
    }
    return (currentMin, currentMax)
}
```

由于元组成员的类型是函数的返回值，可以通过如下方式使用，例如：

```Swift
let bounds = minMax(array: [8, -6, 2, 109, 3, 71])
print("min is \(bounds.min) and max is \(bounds.max)")
// Prints "min is -6 and max is 109"
```

### 可选的元组返回类型
如果一个函数的返回的元组是可选的，就可以使用一个可选的元组返回类型，例如：`(Int, Int)? `。

```Swift
func minMax(array: [Int]) -> (min: Int, max: Int)? {
    if array.isEmpty { return nil }
    var currentMin = array[0]
    var currentMax = array[0]
    for value in array[1..<array.count] {
        if value < currentMin {
            currentMin = value
        } else if value > currentMax {
            currentMax = value
        }
    }
    return (currentMin, currentMax)
}

if let bounds = minMax(array: [8, -6, 2, 109, 3, 71]) {
    print("min is \(bounds.min) and max is \(bounds.max)")
}
// Prints "min is -6 and max is 109"
```

>* 注：可选元组类型(Int, Int)? 和包含可选值的元组(Int?, Int?)不同，对于可选的元组类型，这个元组是可选的，而不是元组中的某个组成元素可选。

### 函数的参数标签和参数名
每个函数参数都包含一个标签名和一个参数名称，标签名在函数被调用的时候使用；函数中的每一个参数前都有一个标签名，参数名在函数体内使用，默认情况下，函数参数使用它们的参数名作为标签名。所有的参数名必须唯一，尽管参数的标签名可以不唯一，但唯一的标签名可以提高代码的易读性。

```Swift
func someFunction(firstParameterName: Int, secondParameterName: Int) {
    // In the function body, firstParameterName and secondParameterName
    // refer to the argument values for the first and second parameters.
}
someFunction(firstParameterName: 1, secondParameterName: 2)
```

#### 明确的参数标签
在参数名前写一个参数标签，用空格隔开，来明确参数标签，格式如下：

```Swift
func someFunction(argumentLabel parameterName: Int) {
    // In the function body, parameterName refers to the argument value
    // for that parameter.
}

```

以下是一个使用明确标签名的例子：

```Swift
func greet(person: String, from hometown: String) -> String {
    return "Hello \(person)!  Glad you could visit from \(hometown)."
}
print(greet(person: "Bill", from: "Cupertino"))
// Prints "Hello Bill!  Glad you could visit from Cupertino."
```

#### 省略的参数标签
如果不需要参数标签，可以使用下划线(_)来代替参数标签。如果参数前有标签的话，函数调用的时候就必须使用标签。

```Swift
func someFunction(parameterWithoutDefault: Int, parameterWithDefault: Int = 12) {
    // If you omit the second argument when calling this function, then
    // the value of parameterWithDefault is 12 inside the function body.
}
someFunction(parameterWithoutDefault: 3, parameterWithDefault: 6) // parameterWithDefault is 6
someFunction(parameterWithoutDefault: 4) // parameterWithDefault is 12
```

#### 默认的参数值
可以为任何参数设定默认值来作为函数的定义的一部分。如果默认值已经定义，调用函数时就可以省略该参数的传值。将使用默认值的参数放在函数列表的末尾，在函数中，没有默认值的参数通常有重要的意义。例如

```Swift
func someFunction(_ firstParameterName: Int, secondParameterName: Int) {
    // In the function body, firstParameterName and secondParameterName
    // refer to the argument values for the first and second parameters.
}
someFunction(1, secondParameterName: 2)
```

#### 可变参数
一个可变参数的参数接受零个或多个指定类型的值。当函数被调用时，您可以使用一个可变参数的参数来指定该参数可以传递不同数量的输入值。写可变参数的参数时，需要参数的类型名称后加上点字符（...）。传递一个可变参数的参数的值时，函数体中是以提供适当类型的数组的形式存在。例如

```Swift
func arithmeticMean(_ numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
arithmeticMean(1, 2, 3, 4, 5)
// returns 3.0, which is the arithmetic mean of these five numbers
arithmeticMean(3, 8.25, 18.75)
// returns 10.0, which is the arithmetic mean of these three numbers
```

 > * 一个函数最多只有一个可变参数。

#### 输入-输出参数
默认情况下，函数的参数是常量（只读）形式，在函数方法体内修改参数会导致编译错误。如果想在函数体内修改参数，则需要将参数类型设置为`in-out`来代替。通过在参数类型前添加关键字`inout`来表明输入-输出参数。一个在输入-输出参数都有一个传递给函数的值，由函数修改后，并从函数返回来替换原来的值。

 > * 输入输出参数类型必须是变量，不能有默认值。不能说常量，不能是可变参数。当你把一个变量传给输入输出参数时，在变量前添加一个连接符&，表明它在函数体内可以修改

例如：

```Swift
func swapTwoInts(_ a: inout Int, _ b: inout Int) {
    let temporaryA = a
    a = b
    b = temporaryA
}
```

调用的时候，注意添加前缀符号&，例如：

```Swift
var someInt = 3
var anotherInt = 107
swapTwoInts(&someInt, &anotherInt)
print("someInt is now \(someInt), and anotherInt is now \(anotherInt)")
// Prints "someInt is now 107, and anotherInt is now 3"
```

### 函数类型
每一个函数都有一个特定的函数类型，由参数类型和返回值类型。例如，以下例子中的函数类型是：(Int, Int)->Int

```Swift
func addTwoInts(_ a: Int, _ b: Int) -> Int {
    return a + b
}
func multiplyTwoInts(_ a: Int, _ b: Int) -> Int {
    return a * b
}
```

#### 使用函数类型
在Swift中可以像其他类型一样使用函数类型，例如可以定义一个函数类型的常量或者变量和调用该变量：

```Swift
var mathFunction: (Int, Int) -> Int = addTwoInts

print("Result: \(mathFunction(2, 3))")
// Prints "Result: 5"
}
```

#### 函数类型的参数
可以使用函数类型（例如：(Int, Int)->Int）作为另一个函数的参数类型，例如：

```Swift
func printMathResult(_ mathFunction: (Int, Int) -> Int, _ a: Int, _ b: Int) {
    print("Result: \(mathFunction(a, b))")
}
printMathResult(addTwoInts, 3, 5)
// Prints "Result: 8"
```

#### 函数类型的返回值
可以使用函数类型作为一个函数的返回值。例如：

```Swift
func stepForward(_ input: Int) -> Int {
    return input + 1
}
func stepBackward(_ input: Int) -> Int {
    return input - 1
}
```

函数类型的返回值

```Swift
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
    return backward ? stepBackward : stepForward
}
```

使用

```Swift
var currentValue = 3
let moveNearerToZero = chooseStepFunction(backward: currentValue > 0)
// moveNearerToZero now refers to the stepBackward() function
```

### 嵌套函数

嵌套函数是定义在函数内的函数，对外是隐藏的，但是可以调用和使用其内部的函数。例如

```Swift
func chooseStepFunction(backward: Bool) -> (Int) -> Int {
    func stepForward(input: Int) -> Int { return input + 1 }
    func stepBackward(input: Int) -> Int { return input - 1 }
    return backward ? stepBackward : stepForward
}
var currentValue = -4
let moveNearerToZero = chooseStepFunction(backward: currentValue > 0)
// moveNearerToZero now refers to the nested stepForward() function
while currentValue != 0 {
    print("\(currentValue)... ")
    currentValue = moveNearerToZero(currentValue)
}
print("zero!")
// -4...
// -3...
// -2...
// -1...
// zero!
```

### 参照文档

>* <a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Functions.html/">Functions</a> 

>
>
>Copyright (c) liangtong. All rights reserved.
>


