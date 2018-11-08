---
layout:     post
title:      二分查找
date:       2017-04-27 10:07:43
author:     liangtong
catalog: true
categories: 算法
tags: Search

---

* content
{:toc}

​	二分查找算法也称为折半搜索、二分搜索，是一种在有序数组中查找某一特定元素的搜索算法。算法是建立在有序数组基础上的。时间复杂度： T(n) = Θ(logn)。



### 原理及实现    
基本原理：使用递归、分治的思想
 * ① 当数组为空时，说明数组中不存在需查找的元素。
 * ② 用有序数组的中间位置元素与需查找元素进行大小比较，如果相等，则查找结束。
 * ③ 如果中间位置元素大于需查找元素，则从数组左侧位置进行递归查找。
 * ④ 如果中间位置元素小于需查找元素，则从数组右侧位置进行递归查找。

使用Swift编写简单的代码实现，如下：

```Swift
/**
 *  二分查找
 * 使用递归、分治的思想
 * ① 当数组为空时，说明数组中不存在需查找的元素。
 * ② 用有序数组的中间位置元素与需查找元素进行大小比较，如果相等，则查找结束。
 * ③ 如果中间位置元素大于需查找元素，则从数组左侧位置进行递归查找。
 * ④ 如果中间位置元素小于需查找元素，则从数组右侧位置进行递归查找。
 **/

func binarySearch<T:Comparable>(array:Array<T>, start:Int, end:Int, value:T) -> Int{
    let count = end - start + 1;
    if count <= 0 {
        return -1;//没有查找到
    }
    let midIndex = (start + end) / 2;
    let midValue = array[midIndex];
    if midValue > value {//在左侧查找
        return binarySearch(array: array, start: start, end: midIndex - 1, value: value);
    }else if midValue < value {//在右侧查找
        return binarySearch(array: array, start: midIndex + 1, end: end, value: value);
    }else{//位置查找到，即当前数组中间位置元素
        return midIndex;
    }
}
```

### 测试用例及结果

```Swift
//方法调用
func callBinarySearch<T:Comparable>(array: Array<T>, value:T){
    
    let index = binarySearch(array: array, start: 0, end: array.count - 1, value: value);
    
    print("Index Of Value \(value)  In Array \(array) is : \(index)")
    
}

var intArray =  [1, 4, 7, 8, 9, 11, 11, 15, 16, 24];
callBinarySearch(array: intArray , value: 3)
callBinarySearch(array: intArray , value: 16)

var stringArray =  ["Assembly", "BASIC", "C#", "Java", "Objective-C", "PHP", "Swift"];
callBinarySearch(array: stringArray,value:"Assembly")
callBinarySearch(array: stringArray,value:"PHP")
callBinarySearch(array: stringArray,value:"AAA")


//程序输出
Index Of Value 3  In Array [1, 4, 7, 8, 9, 11, 11, 15, 16, 24] is : -1
Index Of Value 16  In Array [1, 4, 7, 8, 9, 11, 11, 15, 16, 24] is : 8
Index Of Value Assembly  In Array ["Assembly", "BASIC", "C#", "Java", "Objective-C", "PHP", "Swift"] is : 0
Index Of Value PHP  In Array ["Assembly", "BASIC", "C#", "Java", "Objective-C", "PHP", "Swift"] is : 5
Index Of Value AAA  In Array ["Assembly", "BASIC", "C#", "Java", "Objective-C", "PHP", "Swift"] is : -1
```

