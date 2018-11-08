---
layout:     post
title:      Swift Closures
date:       2016-11-22 00:20:11
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

​	闭包是一种可以在代码中传递和使用的自包含代码块。Swift中的闭包盒C／ObjC中的闭包与其它语言中的lambdas表达式类似。闭包可用捕获其被定义的上下文中的变量或常量。Swift会为你处理所有的通过捕获的内存管理。函数章节介绍的全局／嵌套函数，是闭包的特殊形式，闭包采取以下三种形式之一

 > * 全局函数是一个有名字且不捕获任何值的闭包
 > * 嵌套函数是一个有名字且可用捕获其封闭函数作用域内值的闭包
 > * 闭包表达式是一个利用轻量级语法所写的可用捕获所在上下文中值（变量／常量）的没有名字的闭包

Swift中的闭包表达式拥有清晰、简洁的风格，在以下常见的场景中实现闭包优化：

 > * 根据上下文怼参数和返回值进行类型推断
 > * 单表达式闭包可用省略return关键字
 > * 参数名称可用缩写
 > * Trailing闭包语法



### 闭包表达式
嵌套函数是一种在较复杂函数中方便进行命名和定义自包含代码模块的方式，但是有时候编写一个不需要名字和参数的简洁版本更有用，尤其是在将函数作为参数传递给另一个函数的时候。
闭包表达式是一种利用简洁语法构建内联闭包的方式。闭包表达式提供了一些语法优化，使得撰写闭包变得简单明了。下面闭包表达式的例子通过使用几次迭代展示了 sort 函数定义和语法优化的方式。 每一次迭代都用更简洁的方式描述了相同的功能。

#### 排序方法
Swift标准库提供的一个排序方法`sorted(by:)`会根据你提供的排序闭包对已知类型的数组的值进行排序，排序完成后会返回一个同源数组大小相同的排序后的数组。例如一个原始数组如下：

```Swift
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
```

排序方法`sorted(by:)`接受一个包含两个数组内值相同类型参数并返回一个Bool值来表明顺序的闭包，闭包回返回true如果要求第一个值在排序的结果中靠前，否则返回false。例子中闭包表达式类型为：

```Swift
(String, String) -> Bool
```

可以通过普通函数，作为参数传递，例如：

```Swift
func backward(_ s1: String, _ s2: String) -> Bool {
    return s1 > s2
}
var reversedNames = names.sorted(by: backward)
// reversedNames is equal to ["Ewa", "Daniella", "Chris", "Barry", "Alex"]
```

#### 闭包表达式语法
闭包表达式一般语法如下：

```Swift
{ (parameters) -> return type in
    statements
}
```

闭包表达式的参数可以是`in-out`类型，但是不能有默认值，也可以是可变参数，也支持元组类型参数或者返回值，例如上述操作可以简化为：

```Swift
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in
    return s1 > s2
})
```

#### 根据上下文推断类型
因为排序闭包是作为参数传递的，Swift可以推断出闭包的参数类型和返回值类型。因为所有的类型都可以被推断，所以这里具体类型没有必要提供。返回值和参数周围的括号也可以被省略，例如：

```Swift
reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 } )
```

实际上，在内连闭包表达式构造的闭包作为参数传递给函数时，都可以推断出参数和返回值类型，这意味着你几乎不需要写完整的闭包格式。然而，你仍然可以使用明确的类型来消除阅读的歧义，并且这是被建议使用的方式。

#### 单行闭包表达式可以省略return
单行表达式闭包可以通过隐藏 return 关键字来隐式返回单行表达式的结果，如上版本的例子可以改写为：

```Swift
reversed = sort(names, { s1, s2 in s1 > s2 } ) 
```

#### 参数名缩写
Swift自动提供了参数名称缩写功能，可以直接使用`$0, $1, $2,`等来引用闭包中的参数。数量和类型可以被推断，当时用参数名称缩写的时候，参数列表定义部分可以省略，in关键字也可以省略。此事闭包表达式完全由闭包函数体构成：

```Swift
reversed = sort(names, { $0 > $1 } ) 
```

#### 运算符方法
有一种更简洁的方法来写上述例子中的闭包表达式。Swift中的String类型定义了操作符`>`的实现。因此可以简单的传递一个符号。例如：

```Swift
reversed = sort(names, { $0 > $1 } ) 
```

### Trailing闭包
当闭包作为一个拥有很多参数函数的最后一个参数时，可以使用Trailing闭包来增强函数的可读性。Trailing闭包是一个书写在函数括号之外的闭包表达式（尽管闭包仍然是函数的参数）。当使用Trailing闭包语法时，不写闭包对应的函数参数标签。例如：

```Swift
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // function body goes here
}
 
// Here's how you call this function without using a trailing closure:
 
someFunctionThatTakesAClosure(closure: {
    // closure's body goes here
})
 
// Here's how you call this function with a trailing closure instead:
 
someFunctionThatTakesAClosure() {
    // trailing closure's body goes here
}
```

上述排序例子可以简写为：

```Swift
reversedNames = names.sorted() { $0 > $1 }
```

除了闭包外无其他参数，小括号也可以省略，例如：

```Swift
reversedNames = names.sorted { $0 > $1 }
```

当闭包非常长的时候，Trailing 闭包就变得非常有用。例如Swift的Array类型有一个map方法`map(_:)`，其接收一个闭包表达式作为唯一的参数，对于数组中的每一个元素会调用该闭包一次，并返回该元素映射的值，具体映射方式和返回值类型由闭包来指定，当提供给数组该闭包函数后，map方法将返回一个新的包含了与原数组一一对应映射关系的数组。例如以下例子讲述如何使用该闭包：

```Swift
let digitNames = [
    0: "Zero", 1: "One", 2: "Two",   3: "Three", 4: "Four",
    5: "Five", 6: "Six", 7: "Seven", 8: "Eight", 9: "Nine"
]
let numbers = [16, 58, 510]

let strings = numbers.map {
    (number) -> String in
    var number = number
    var output = ""
    repeat {
        output = digitNames[number % 10]! + output
        number /= 10
    } while number > 0
    return output
}
// strings is inferred to be of type [String]
// its value is ["OneSix", "FiveEight", "FiveOneZero"]
```

### 捕获值
闭包可以捕获其定义的上下文中的常量或变量，即使定义这些变量或者常量的作用域已经不存在，闭包仍然可以在闭包体内引用(和修改)这些值。在Swift中，最简单的闭包形式是嵌套函数，也就是定义在函数体内的函数。嵌套函数可以捕获外部函数的参数以及所有定义在外部函数体内的变量和常量。
下边是一个嵌套函数的例子：

```Swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}
```

注意：Swift会`自动持有被截获的变量的引用`，这样就可以在block内部直接修改变量。Swift会自动判断你是否在闭包中或闭包外改变了变量。如果没有改变，闭包会直接持有变量，即使你没有显式的把它卸载捕获列表中。以下摘自苹果官方文档：

```Swift
As an optimization, Swift may instead capture and store a copy of a value if that value is not mutated by a closure, and if the value is not mutated after the closure is created.
```

### 闭包是引用类型
如果将闭包赋值给一个变量和常量，那么赋值后的变量和常量会引用同一个闭包，例如

```Swift
let alsoIncrementByTen = incrementByTen
alsoIncrementByTen()
```


### 逃避闭包(Escaping Closures)
当闭包作为函数的参数传入时，很有可能这个闭包在函数返回之后才会被执行，这就是逃逸闭包。在Swift中可以在参数名前标注 @noescape 来指明这个闭包是不允许逃逸出这个函数的。因为非逃逸闭包只能在函数体中被执行，不能脱离函数体执行，所以这使得编译器可以明确的知道运行时的上下文环境，进而做出优化。
一般情况下，一些异步函数会使用逃逸闭包。这类函数会在异步操作开始之后立刻返回，但是闭包直到异步操作结束后才会被调用。比如网络请求中处理服务器返回请求的闭包。例如：

```Swift
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
```

如果需要引用自身self，因为编译器知晓非逃逸闭包的上下文环境，所以非逃逸闭包中可以不写 self (隐式的存在)。但是逃逸闭包必须明确写self。例如：

```Swift
class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}
```

### 自动闭包(Autoclosures)
自动闭包是自动创建的包含一个语句的闭包，作为参数传递给函数。自动闭包不接受任何参数，被调用时会返回被包装在其中的表达式的值。
自动闭包是延迟求值，因为闭包中的代码直到你调用闭包的时候才会执行。延迟求值对一些可能存在副作用的代码来说很有用，通过延迟求值你可以控制代码的执行时机。例如：

```Swift
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)
// Prints "5"
 
let customerProvider = { customersInLine.remove(at: 0) }
print(customersInLine.count)
// Prints "5"
 
print("Now serving \(customerProvider())!")
// Prints "Now serving Chris!"
print(customersInLine.count)
// Prints "4"
```

即使闭包代码中将数组中的第一个元素移除了，但是闭包代码不会执行直到闭包被调用。如果闭包不被调用，那么闭包内的移除代码就不会调用。当你做微函数参数传递的时候你也是同样的道理

```Swift
// customersInLine is ["Alex", "Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: { customersInLine.remove(at: 0) } )
// Prints "Now serving Alex!"
```

以上例子中的serve函数接受一个明确的闭包（返回一个顾客的名字），下边这个例子和上边效果一样，不过是接收一个自动闭包。

```Swift
// customersInLine is ["Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))
// Prints "Now serving Ewa!"
```

如果一个自动闭包需要逃逸，使用两个修饰符`@autoclosure`和`@escaping`：

```Swift
// customersInLine is ["Barry", "Daniella"]
var customerProviders: [() -> String] = []
func collectCustomerProviders(_ customerProvider: @autoclosure @escaping () -> String) {
    customerProviders.append(customerProvider)
}
collectCustomerProviders(customersInLine.remove(at: 0))
collectCustomerProviders(customersInLine.remove(at: 0))
 
print("Collected \(customerProviders.count) closures.")
// Prints "Collected 2 closures."
for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
// Prints "Now serving Barry!"
// Prints "Now serving Daniella!"
```


### 参照文档

>* <a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/ControlFlow.html/">Control Flow</a> 

>* <a href="http://www.jianshu.com/p/d0d7b519fec1/">OC与Swift闭包对比总结</a> 

>* <a href="http://www.tuicool.com/articles/MRJ3me7/">浅谈iOS中的闭包</a> 

>
>
>Copyright (c) liangtong. All rights reserved.
>


