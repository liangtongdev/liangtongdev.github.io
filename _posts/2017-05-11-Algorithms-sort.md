---
layout:     post
title:      快速排序
date:       2017-05-11 17:25:00
author:     liangtong
catalog: true
categories: 算法
tags: Sort

---

* content
{:toc}

​	快速排序是采用分治法（Divide and Conquer）的应用。和归并排序不同的是，她不需要额外的存储空间，时间复杂度： T(n) = Θ(nlgn)，最坏情况下是：T(n) = Θ(n²)。



### 基本原理及实现   
基本原理： 使用递归、分治的思想，每次调用排序方法，随机选取一个key值，结果保证：key值所在数组中的位置，且能保证key左侧位置的数据总小于key值；右侧位置数据总大于key值      
* ① 当数组为空时，数组不需要排序，算法结束。
* ② 选取一个key值，可以是带排序数组中任意位置的元素，这里选第一个元素。
* ③ 遍历数组(角标j)，将所有比key值小的元素移动到数组前边(角标i)。遍历结束时，i即是中间位置。
* ④ 将key值与角标i位置元素位置互换。
* ⑤ 递归对i位置左侧、右侧数组进行快速排序。

使用Swift编写简单的代码实现，如下：    
```Swift
/**
*  快速排序
* 使用递归、分治的思想，每次调用排序方法，选取一个key值，结果保证：key值所在数组中的位置，且能保证key左侧位置的数据总小于key值；右侧位置数据总大于key值
* ① 当数组为空时，数组不需要排序，算法结束。
* ② 选取一个key值，可以是带排序数组中任意位置的元素，这里选第一个元素。
* ③ 遍历数组(角标j)，将所有比key值小的元素移动到数组前边(角标i)。遍历结束时，i即是中间位置。
* ④ 将key值与角标i位置元素位置互换。
* ⑤ 递归对i位置左侧、右侧数组进行快速排序。
**/
func quickSort<T:Comparable>(array:inout Array<T>, start:Int, end:Int){
if end <= start {
return;
}
let key = array[start];//随机选取key值 - 这里我们固定选择第一个元素
var i = start;
var j = i + 1;
while j <= end {//从key后开始遍历，查找所有比key小的元素，移动到数组前边
let aj = array[j];
if key > aj {
//交换元素
let tempValue = array[i+1];
array[i+1] = aj;
array[j] = tempValue;
//重新设置角标
i += 1;
}
j += 1;//数组遍历
}
//i位置即是最后一个比key值小的元素位置，然后交换key值和i位置元素
let ai = array[i];
array[i] = key;
array[start] = ai;

print("sorting：key：\(key)  values: \(array[start...end])");

//对i位置左侧、右侧的数组进行排序方法调用
quickSort(array: &array, start: start, end: i - 1)
quickSort(array: &array, start: i+1, end: end);
}
```

### 测试用例及结果

```Swift
var intArray =  [6,10,13,5,8,3,2,11];
quickSort(array: &intArray,start:0,end:intArray.count-1);
print("排序结束：\(intArray)");


//程序输出
sorting：key：6  values: [2, 5, 3, 6, 8, 13, 10, 11]
sorting：key：2  values: [2, 5, 3]
sorting：key：5  values: [3, 5]
sorting：key：8  values: [8, 13, 10, 11]
sorting：key：13  values: [11, 10, 13]
sorting：key：11  values: [10, 11]
排序结束：[2, 3, 5, 6, 8, 10, 11, 13]


var stringArray =  ["Java","C#","Python","Objective-C","Swift","PHP","BASIC","Pasical","Assembly"];
quickSort(array: &stringArray,start:0,end:stringArray.count-1);
print("排序结束：\(stringArray)");


//程序输出
sorting：key：Java  values: ["Assembly", "C#", "BASIC", "Java", "Swift", "PHP", "Python", "Pasical", "Objective-C"]
sorting：key：Assembly  values: ["Assembly", "C#", "BASIC"]
sorting：key：C#  values: ["BASIC", "C#"]
sorting：key：Swift  values: ["Objective-C", "PHP", "Python", "Pasical", "Swift"]
sorting：key：Objective-C  values: ["Objective-C", "PHP", "Python", "Pasical"]
sorting：key：PHP  values: ["PHP", "Python", "Pasical"]
sorting：key：Python  values: ["Pasical", "Python"]
排序结束：["Assembly", "BASIC", "C#", "Java", "Objective-C", "PHP", "Pasical", "Python", "Swift"]
```
