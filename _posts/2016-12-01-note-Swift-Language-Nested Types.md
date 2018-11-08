---
layout:     post
title:      Swift Nested Types
date:       2016-12-01 19:20:11
author:     liangtong
catalog: true
categories: Swift
tags: 学习 

---

* content
{:toc}

​	枚举常被用于为特定类或结构体实现某些功能。类似的，枚举可以方便的定义工具类或结构体，从而为某个复杂的类型所使用。为了实现这种功能，Swift 允许你定义嵌套类型，可以在支持的类型中定义嵌套的枚举、类和结构体。
要在一个类型中嵌套另一个类型，将嵌套类型的定义写在其外部类型的{}内，而且可以根据需要定义多级嵌套。



### 嵌套类型实践
下面这个例子定义了一个结构体BlackjackCard，用来模拟BlackjackCard中的扑克牌点数。BlackjackCard结构体包含两个嵌套定义的枚举类型Suit和Rank。
在BlackjackCard中，Ace牌可以表示1或者11，Ace牌的这一特征通过一个嵌套在Rank枚举中的结构体Values来表示：

```Swift
struct BlackjackCard {
    
    // nested Suit enumeration
    enum Suit: Character {
        case spades = "♠", hearts = "♡", diamonds = "♢", clubs = "♣"
    }
    
    // nested Rank enumeration
    enum Rank: Int {
        case two = 2, three, four, five, six, seven, eight, nine, ten
        case jack, queen, king, ace
        struct Values {
            let first: Int, second: Int?
        }
        var values: Values {
            switch self {
            case .ace:
                return Values(first: 1, second: 11)
            case .jack, .queen, .king:
                return Values(first: 10, second: nil)
            default:
                return Values(first: self.rawValue, second: nil)
            }
        }
    }
    
    // BlackjackCard properties and methods
    let rank: Rank, suit: Suit
    var description: String {
        var output = "suit is \(suit.rawValue),"
        output += " value is \(rank.values.first)"
        if let second = rank.values.second {
            output += " or \(second)"
        }
        return output
    }
}
```

Suit枚举用来描述扑克牌的四种花色，并用一个Character类型的原始值表示花色符号。
Rank枚举用来描述扑克牌从Ace~10，以及J、Q、K，这13种牌，并用一个Int类型的原始值表示牌的面值。（这个Int类型的原始值未用于Ace、J、Q、K这4种牌。）
如上所述，Rank枚举在内部定义了一个嵌套结构体Values。结构体Values中定义了两个属性，用于反映只有Ace有两个数值，其余牌都只有一个数值：

* first, of type Int
* second, of type Int?, or “optional Int”

Rank还定义了一个计算型属性values，它将会返回一个Values结构体的实例。这个计算型属性会根据牌的面值，用适当的数值去初始化Values实例。对于J、Q、K、Ace这四种牌，会使用特殊数值。对于数字面值的牌，使用枚举实例的原始值。

BlackjackCard结构体拥有两个属性——rank与suit。它也同样定义了一个计算型属性description，description属性用rank和suit中的内容来构建对扑克牌名字和数值的描述。该属性使用可选绑定来检查可选类型second是否有值，若有值，则在原有的描述中增加对second的描述。

因为BlackjackCard是一个没有自定义构造器的结构体,有默认的成员构造器，所以你可以用默认的构造器去初始化新常量theAceOfSpades：

```Swift
let theAceOfSpades = BlackjackCard(rank: .ace, suit: .spades)
print("theAceOfSpades: \(theAceOfSpades.description)")
// Prints "theAceOfSpades: suit is ♠, value is 1 or 11"
```

尽管Rank和Suit嵌套在BlackjackCard中，但它们的类型仍可从上下文中推断出来，所以在初始化实例时能够单独通过成员名称（.Ace和.Spades）引用枚举实例。在上面的例子中，description属性正确地反映了黑桃A牌具有1和11两个值。

### 引用嵌套类型
在外部引用嵌套类型时，在嵌套类型的类型名前加上其外部类型的类型名作为前缀：

```Swift
let heartsSymbol = BlackjackCard.Suit.hearts.rawValue
// heartsSymbol is "♡"
```

对于上面这个例子，这样可以使Suit、Rank和Values的名字尽可能的短，因为它们的名字可以由定义它们的上下文来限定。







### 参照文档

>* <a href="https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/NestedTypes.html/"> NestedTypes </a> 


>
>
>Copyright (c) liangtong. All rights reserved.
>


