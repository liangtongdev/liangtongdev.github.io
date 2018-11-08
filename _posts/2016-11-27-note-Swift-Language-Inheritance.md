---
layout:     post
title:      Swift Inheritance
date:       2016-11-26 00:20:11
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

​	一个类可以继承另一个类的方法，属性和其它特性。当一个类继承其它类时，继承类叫子类，被继承类叫超类（或父类）。在 Swift 中，继承是区分「类」与其它类型的一个基本特征。
在 Swift 中，类可以调用和访问超类的方法，属性和下标，并且可以重写这些方法，属性和下标来优化或修改它们的行为。Swift 会检查你的重写定义在超类中是否有匹配的定义，以此确保你的重写行为是正确的。
可以为类中继承来的属性添加属性观察器，这样一来，当属性值改变时，类就会被通知到。可以为任何属性添加属性观察器，无论它原本被定义为存储型属性还是计算型属性。



### 基类
不继承于其它类的类，称之为基类。
	Swift 中的类并不是从一个通用的基类继承而来。如果你不为你定义的类指定一个超类的话，这个类就自动成为基类。

下面的例子定义了一个叫Vehicle的基类。包含一个currentSpeed存储属性，一个description的只读计算属性，一个makeNoise方法，方法什么都不做，子类会重写该方法。

```Swift
class Vehicle {
    var currentSpeed = 0.0
    var description: String {
        return "traveling at \(currentSpeed) miles per hour"
    }
    func makeNoise() {
        // do nothing - an arbitrary vehicle doesn't necessarily make a noise
    }
}
```

实例化对象，并访问其属性：

```Swift
let someVehicle = Vehicle()

print("Vehicle: \(someVehicle.description)")
// Vehicle: traveling at 0.0 miles per hour
```

### 子类
子类是在一个已有类的基础上创建一个新的类，你可以重新定义子类继承已有类的特性，也可以为子类添加新特性来完善子类。类继承使用冒号分割，将超类写在冒号后边：

```Swift
class SomeSubclass: SomeSuperclass {
    // subclass definition goes here
}
```

下一个例子，定义一个叫Bicycle的子类，继承成父类Vehicle：

```Swift
class Bicycle: Vehicle {
    var hasBasket = false
}
```

新的Bicycle类自动获得Vehicle类的所有特性，Bicycle类还定义了一个默认值为false的存储型属性hasBasket（属性推断为Bool）。

```Swift
let bicycle = Bicycle()
bicycle.hasBasket = true

bicycle.currentSpeed = 15.0
print("Bicycle: \(bicycle.description)")
// Bicycle: traveling at 15.0 miles per hour
```

子类还可以继续被其它类继承，下面的示例为Bicycle创建了一个名为Tandem（双人自行车）的子类：

```Swift
class Tandem: Bicycle {
    var currentNumberOfPassengers = 0
}

let tandem = Tandem()
tandem.hasBasket = true
tandem.currentNumberOfPassengers = 2
tandem.currentSpeed = 22.0
print("Tandem: \(tandem.description)")
// Tandem: traveling at 22.0 miles per hour
```

### 重写
子类可以为继承来的实例方法，类方法，实例属性，或下标提供自己定制的实现。我们把这种行为叫重写。
如果要重写某个特性，你需要在重写定义的前面加上 `override` 关键字。这么做，你就表明了你是想提供一个重写版本，而非错误地提供了一个相同的定义。任何缺少override关键字的重写都会在编译时被诊断为错误。override关键字会提醒 Swift 编译器去检查该类的超类（或其中一个父类）是否有匹配重写版本的声明。这个检查可以确保你的重写定义是正确的。

#### 访问超类的方法，属性及下标
当你在子类中重写超类的方法，属性或下标时，有时在你的重写版本中使用已经存在的超类实现会大有裨益。比如，你可以完善已有实现的行为，或在一个继承来的变量中存储一个修改过的值。
你可以通过使用 `super` 前缀来访问超类版本的方法，属性或下标：

 * 在方法someMethod()的重写实现中，可以通过super.someMethod()来调用超类版本的someMethod()方法。
 * 在属性someProperty的 getter 或 setter 的重写实现中，可以通过super.someProperty来访问超类版本的someProperty属性。
 * 在下标的重写实现中，可以通过super[someIndex]来访问超类版本中的相同下标。

#### 重写方法
在子类中，你可以重写继承来的实例方法或类方法，提供一个定制或替代的方法实现。
下面的例子定义了Vehicle的一个新的子类，叫Train，它重写了从Vehicle类继承来的makeNoise()方法：

```Swift
class Train: Vehicle {
    override func makeNoise() {
        print("Choo Choo")
    }
}

let train = Train()
train.makeNoise()
// Prints "Choo Choo"
```

#### 重写属性
你可以重写继承来的实例属性或类型属性，提供自己定制的 getter 和 setter，或添加属性观察器使重写的属性可以观察属性值什么时候发生改变。

##### 重写属性的 Getters 和 Setters
你可以提供定制的 getter（或 setter）来重写任意继承来的属性，无论继承来的属性是存储型的还是计算型的属性。子类并不知道继承来的属性是存储型的还是计算型的，它只知道继承来的属性会有一个名字和类型。你在重写一个属性时，必需将它的名字和类型都写出来。这样才能使编译器去检查你重写的属性是与超类中同名同类型的属性相匹配的。
你可以将一个继承来的只读属性重写为一个读写属性，只需要在重写版本的属性里提供 getter 和 setter 即可。但是，你不可以将一个继承来的读写属性重写为一个只读属性。
	如果你在重写属性中提供了 setter，那么你也一定要提供 getter。如果你不想在重写版本中的 getter 里修改继承来的属性值，你可以直接通过super.someProperty来返回继承来的值，其中someProperty是你要重写的属性的名字。

以下的例子定义了一个新类，叫Car，它是Vehicle的子类。这个类引入了一个新的存储型属性叫做gear，默认值为整数1。Car类重写了继承自Vehicle的description属性，提供包含当前档位的自定义描述：

```Swift
class Car: Vehicle {
    var gear = 1
    override var description: String {
        return super.description + " in gear \(gear)"
    }
}
```

如果你创建了Car的实例并且设置了它的gear和currentSpeed属性，你可以看到它的description返回了Car中的自定义描述：

```Swift
let car = Car()
car.currentSpeed = 25.0
car.gear = 3
print("Car: \(car.description)")
// Car: traveling at 25.0 miles per hour in gear 3
```

##### 重写属性观察器
你可以通过重写属性为一个继承来的属性添加属性观察器。这样一来，当继承来的属性值发生改变时，你就会被通知到，无论那个属性原本是如何实现的。
	你不可以为继承来的常量存储型属性或继承来的只读计算型属性添加属性观察器。这些属性的值是不可以被设置的，所以，为它们提供willSet或didSet实现是不恰当。
此外还要注意，你不可以同时提供重写的 setter 和重写的属性观察器。如果你想观察属性值的变化，并且你已经为那个属性提供了定制的 setter，那么你在 setter 中就可以观察到任何值变化了。

下面的例子定义了一个新类叫AutomaticCar，它是Car的子类。AutomaticCar表示自动挡汽车，它可以根据当前的速度自动选择合适的挡位:

```Swift
class AutomaticCar: Car {
    override var currentSpeed: Double {
        didSet {
            gear = Int(currentSpeed / 10.0) + 1
        }
    }
}

let automatic = AutomaticCar()
automatic.currentSpeed = 35.0
print("AutomaticCar: \(automatic.description)")
// AutomaticCar: traveling at 35.0 miles per hour in gear 4
```

### 防止重写
你可以通过把方法，属性或下标标记为final来防止它们被重写，只需要在声明关键字前加上 `final` 修饰符即可（例如：final var，final func，final class func，以及final subscript）。

如果你重写了带有 `final` 标记的方法，属性或下标，在编译时会报错。在类扩展中的方法，属性或下标也可以在扩展的定义里标记为 `final` 的。

你可以通过在关键字 `class` 前添加 `final` 修饰符（final class）来将整个类标记为 `final` 的。这样的类是不可被继承的，试图继承这样的类会导致编译报错。



### 参照文档

>* <a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Inheritance.html/"> Inheritance </a> 


>
>
>Copyright (c) liangtong. All rights reserved.
>


