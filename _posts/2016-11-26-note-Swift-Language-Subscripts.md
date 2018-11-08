---
layout:     post
title:      Swift Subscripts
date:       2016-11-26 00:20:11
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

​	下标可以定义在类、结构体和枚举中，为访问集合，列表或序列中元素提供便捷。可以使用下标的索引，设置和获取值，而不需要再调用对应的存取方法。举例来说，用下标访问一个Array实例中的元素可以写作someArray[index]，访问Dictionary实例中的元素可以写作someDictionary[key]。



### 语法
下标允许你通过在实例化对象后边的中括号中添加一个或多个索引值来查询或修改实例。语法类似于实例方法语法和计算型属性语法。通过关键字 `subscript` 定义，并指定一个或多个输入参数和返回类型；与实例方法不同的是。下标语法可以设定为读写( `read-write` )或只读( `read-only` )。这种行为由 getter 和 setter 实现，有点类似计算型属性：

```Swift
subscript(index: Int) -> Int {
    get {
        // return an appropriate subscript value here
    }
    set(newValue) {
        // perform a suitable setting action here
    }
}
```

newValue的类型和下标的返回类型相同。如同计算型属性，可以不指定 setter 的参数（newValue）。如果不指定参数，setter 会提供一个名为newValue的默认参数。如同只读计算型属性，可以省略只读下标的get关键字：

```Swift
subscript(index: Int) -> Int {
    // return an appropriate subscript value here
}
```

下面代码演示了只读下标的实现：

```Swift
struct TimesTable {
    let multiplier: Int
    subscript(index: Int) -> Int {
        return multiplier * index
    }
}
let threeTimesTable = TimesTable(multiplier: 3)
print("six times three is \(threeTimesTable[6])")
// Prints "six times three is 18"
```

### 用法
下标的含义通常取决于使用场景，通常在集合、列表和序列中快速访问用。你也可以根据自己的特定类型实现下标功能。
例如Swift中的字典Dictionary使用下标语法对实例中储存的值进行存取操作。

```Swift
var numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]
numberOfLegs["bird"] = 2
```

	Swift 的Dictionary类型的下标接受并返回可选类型的值。因为不是每个键都有个对应的值，同时这也提供了一种通过键删除对应值的方式，只需将键对应的值赋值为nil即可。


### 选项
下标可以接受任意数量的入参，并且这些入参可以是任意类型。下标的返回值也可以是任意类型。下标可以使用变量参数和可变参数，但不能使用输入输出参数，也不能给参数设置默认值。
一个类或结构体可以根据自身需要提供多个下标实现

```Swift
struct Matrix {
    let rows: Int, columns: Int
    var grid: [Double]
    init(rows: Int, columns: Int) {
        self.rows = rows
        self.columns = columns
        grid = Array(repeating: 0.0, count: rows * columns)
    }
    func indexIsValid(row: Int, column: Int) -> Bool {
        return row >= 0 && row < rows && column >= 0 && column < columns
    }
    subscript(row: Int, column: Int) -> Double {
        get {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            return grid[(row * columns) + column]
        }
        set {
            assert(indexIsValid(row: row, column: column), "Index out of range")
            grid[(row * columns) + column] = newValue
        }
    }
}
```

你可以通过传入合适的row和column的数量来构造一个新的Matrix实例：

```Swift
var matrix = Matrix(rows: 2, columns: 2)
```

将row和column的值传入下标来为矩阵设值，下标的入参使用逗号分隔：

```Swift
matrix[0, 1] = 1.5
matrix[1, 0] = 3.2
```

访问

```Swift
let rightTopValue = matrix[0, 1]
// 1.5

let someValue = matrix[2, 2]
// this triggers an assert, because [2, 2] is outside of the matrix bounds
```


### 参照文档

>* <a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Subscripts.html/">Subscripts</a> 


>
>
>Copyright (c) liangtong. All rights reserved.
>


