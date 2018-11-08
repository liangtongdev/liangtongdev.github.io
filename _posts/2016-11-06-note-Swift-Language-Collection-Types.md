---
layout:     post
title:      Swift Collection Types
date:       2016-11-06 00:20:11
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

​	同其他语言类型类似，Swift提供了3个集合类型`Array`、`Set`和`Dictionary`来存储集合值。与ObjC不同的是，Swift中的集合属于范型，不能不同类型的数据添加的同一个集合类型中，且Swift中的集合类型变量都支持修改。
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/CollectionTypes_intro_2x.png)



### Array
数组类型是存储相同类型数据的有序列表，其中数组不同位置存储的数可以相同。Swift中的`Array`类型是Foundation框架中的`NSArray`类型的桥接.Swift中Array属于范型类，具体的写法是`Array<Element>`或者`[Element]`，两种写法相同。

#### 数组的创建
可以通过`[Element](), []`的形式创建空数组，可以通过Array提供方法进行创建，当然也可以通过字面值`[value 1, value 2, value 3]`的形式创建并初始化一个数组

```Swift
var someInts = [Int]()//创建一个空的Int数组
someInts.append(3)//数组中添加一个元素
someInts = []//数组被清空

// threeDoubles is of type [Double], and equals [0.0, 0.0, 0.0]
var threeDoubles = Array(repeating: 0.0, count: 3)

var shoppingList: [String] = ["Eggs", "Milk”]
```

#### 数组的组合
可以通过操作符`+`将两个形同类型元素的数组合并到一起。

```Swift
var anotherThreeDoubles = Array(repeating: 2.5, count: 3)
// anotherThreeDoubles is of type [Double], and equals [2.5, 2.5, 2.5]
 
var sixDoubles = threeDoubles + anotherThreeDoubles
// sixDoubles is inferred as [Double], and equals [0.0, 0.0, 0.0, 2.5, 2.5, 2.5]
```

#### 数组元素的访问和修改
可以通过调用数组的方法和属性来访问和修改数组元素，或者通过下标语法，比如：

```Swift
//通过数组的count属性来确定数组的元素
print("The shopping list contains \(shoppingList.count) items.")

//通过数组的Bool属性isEmpty来判断数组是否为空
if shoppingList.isEmpty {
    print("The shopping list is empty.")
} else {
    print("The shopping list is not empty.")
}

//通过调用数组的append方法来给数组添加元素
shoppingList.append("Flour")

//当然也可以通过操作符+=给数组添加元素（合并数组）
shoppingList += ["Baking Powder"]
shoppingList += ["Chocolate Spread", "Cheese", "Butter"]

//通过下标访问和修改数组元素：如果越界会导致运行时错误
var firstItem = shoppingList[0]
shoppingList[0] = "Six eggs"

//当然也可以通过下标来同时修改一个范围元素，长度不相同也无所谓
shoppingList[4...6] = ["Bananas", "Apples"]

//可以通过insert方法向数组中插入元素
shoppingList.insert("Maple Syrup", at: 0)

//可以通过remove方法向移除数组特定位置的元素
let mapleSyrup = shoppingList.remove(at: 0)
```

#### 数组的遍历
通过使用`for-in`循环来遍历数组，当然如果需要数组对应下标值的话，可以通过`enumerated()`方法（对于数组的`enumerated()`方法，会返回一个包含整形索引和具体值的元组）。例如：

```Swift
for item in shoppingList {
    print(item)
}

for (index, value) in shoppingList.enumerated() {
    print("Item \(index + 1): \(value)")
}
```

### Set
Set是用来存储不重复的相同类型元素`无序`列表；Swift中的`Set`类型是Foundation框架中的`NSSet`类型的桥接。

#### Set类型的哈希值
存储在Set类型中的元素必须是`hashable`的（用来保证存储在Set中的元素唯一）。Swift中Set类型的创建语法格式为`Set[Element]`。与其他语言一样，不再赘述

 > 注：可以通过实现Swfit中的`Hashable`协议来自定义存储在Set集合中的元素

#### Set类型的创建
可以通过`Set<Element>(), []`语法创建一个Set类型对象，当然也可以通过字面值`[value 1, value 2, value 3]`的形式创建并初始化一个Set对象，例如：

```Swift
//创建一个空的Character类型的Set对象
var letters = Set<Character>()

//通过调用inset方法，向Set中添加元素
letters.insert("a")

//Set被清空
letters = []

//通过字面值创建并初始化Set集合
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]
//也可以这样写，Swift存在类型推断
var favoriteGenres: Set = ["Rock", "Classical", "Hip hop"]
```

#### Set对象的访问和修改
可以通过调用Set的方法和属性来访问和修改Set元素，例如：

```Swift
//通过Set的count属性来确定Set的元素
print("I have \(favoriteGenres.count) favorite music genres.")

//通过Set的Bool属性isEmpty来判断Set是否为空
if favoriteGenres.isEmpty {
    print("As far as music goes, I'm not picky.")
} else {
    print("I have particular music preferences.")
}

//通过调用Set的insert方法来给Set添加元素
favoriteGenres.insert("Jazz")

//可以通过remove方法向移除Set中的特定元素
if let removedGenre = favoriteGenres.remove("Rock") {
    print("\(removedGenre)? I'm over it.")
} else {
    print("I never much cared for that.")
}

//通过调用Set的contains方法来判断Set是否包含某个元素
if favoriteGenres.contains("Funk") {
    print("I get up on the good foot.")
} else {
    print("It's too funky in here.")
}
// Prints "It's too funky in here."

```

#### Set的遍历
通过使用`for-in`循环来遍历Set，Set对象是无序列表，如果想要顺序访问具体对象值，可以调用Set的`sorted()`方法。例如：

```Swift
for genre in favoriteGenres {
    print("\(genre)")
}
// Jazz
// Hip hop
// Classical

for genre in favoriteGenres.sorted() {
    print("\(genre)")
}
// Classical
// Hip hop
// Jazz
```

#### Set常用运算
常用操作如下：

 > * operator (==) 判断是非相等
 > * isSubset(of:) 判断Set是否子集
 > * isSuperset(of:) 判断是否超级
 > * isStrictSubset(of:) 或 isStrictSuperset(of:) 判断是否是子集或者超集，但不相等
 > * isDisjoint(with:) 判断两个Set是否有相同元素

再放一张图吧
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/setVennDiagram_2x.png)



### Dictionary
Dictionary存储<Key,Value>键值对，其中键key值必须唯一，Swift中的`Dictionary `类型是Foundation框架中的`NSDictionary `类型的桥接。

#### Dictionary的创建
可以通过`[Key:Value](), []`的形式创建空Dictionary，也可以通过字面值`[key 1: value 1, key 2: value 2, key 3: value 3]`的形式创建。例如：

```Swift
//创建一个空字典
var namesOfIntegers = [Int: String]()

//字典的赋值
namesOfIntegers[16] = "sixteen"

//字典的清空
namesOfIntegers = [:]

//通过字面值创建
var airports: [String: String] = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]
//类型推断
var airports = ["YYZ": "Toronto Pearson", "DUB": "Dublin"]

```

#### Dictionary对象的访问和修改
可以通过调用Dictionary的方法和属性来访问和修改Dictionary元素，或者通过下标语法，例如：

```Swift
//通过Dictionary的count属性来确定Dictionary的元素个数
print("The airports dictionary contains \(airports.count) items.")

//通过Dictionary的Bool属性isEmpty来判断Dictionary是否为空
if airports.isEmpty {
    print("The airports dictionary is empty.")
} else {
    print("The airports dictionary is not empty.")
}

//通过下标语法添加或者修改Dictionary中的元素
airports["LHR"] = "London"

//可以通过updateValue(_:forKey:) 方法添加或者修改Dictionary中的元素。
//需要注意的是如果执行更新操作，该方法会返回旧的Value值
if let oldValue = airports.updateValue("Dublin Airport", forKey: "DUB") {
    print("The old value for DUB was \(oldValue).")
}


//可以通过下标语法来访问某个key对应的value
if let airportName = airports["DUB"] {
    print("The name of the airport is \(airportName).")
} else {
    print("That airport is not in the airports dictionary.")
}

//可以通过下标语法来移除某个<key,value>元素
airports["APL"] = "Apple International"
airports["APL"] = nil

//可以通过removeValue(forKey:) 方法移除某个<key,value>元素
if let removedValue = airports.removeValue(forKey: "DUB") {
    print("The removed airport's name is \(removedValue).")
} else {
    print("The airports dictionary does not contain a value for DUB.")
}
```

#### Dictionary的遍历
通过使用`for-in`循环来遍历Dictionary，每个Dictionary元素是以(key,value)元组的形式，例如：

```Swift
for (airportCode, airportName) in airports {
    print("\(airportCode): \(airportName)")
}
// YYZ: Toronto Pearson
// LHR: London Heathrow

for airportCode in airports.keys {
    print("Airport code: \(airportCode)")
}
// Airport code: YYZ
// Airport code: LHR
 
for airportName in airports.values {
    print("Airport name: \(airportName)")
}
// Airport name: Toronto Pearson
// Airport name: London Heathrow

```


### 参照文档

>* <a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/CollectionTypes.html/">CollectionTypes</a> 

>
>
>
>
>
>
>​Copyright (c) liangtong. All rights reserved.
>


