---
layout:     post
title:      计数排序
date:       2017-05-18 21:05:37
author:     liangtong
catalog: true
categories: 算法
tags: Sort

---

* content
{:toc}

​	计数排序是一个非基于比较的整数排序算法，用空间换时间，优势在于在对一定范围内的整数排序时，它的复杂度为Ο(n+k)（其中k是整数的范围）。



### 基本思想及实现   
基本思想： 假设输入的线性表L的长度为n，L=L1,L2,..,Ln；线性表的元素属于有限偏序集S，|S|=k且k=O(n)，S={S1,S2,..Sk}；则计数排序可以描述如下：      
* ① 扫描整个集合S，对每一个Si∈S，找到在线性表L中小于等于Si的元素的个数T(Si)；
* ② 扫描整个线性表L，对L中的每一个元素Li，将Li放在输出线性表的第T(Li)个位置上，并将T(Li)减1。

使用Swift编写简单的代码实现，如下：    
```Swift
/**
*  计数排序
* 计数排序是一个非基于比较的排序算法，优势在于在对一定范围内的整数排序时，它的复杂度为Ο(n+k)（其中k是整数的范围），快于任何比较排序算法。当然这是一种牺牲空间换取时间的做法，而且当O(k)>O(n*log(n))的时候其效率反而不如基于比较的排序。
* ① 扫描整个集合S，对每一个Si∈S，找到在线性表L中小于等于Si的元素的个数T(Si)；
* ② 扫描整个线性表L，对L中的每一个元素Li，将Li放在输出线性表的第T(Li)个位置上，并将T(Li)减1。
**/
func countSort(sortArray:inout Array<Int>, maxValue:Int) -> Array<Int>{
let count = sortArray.count;
var retArray = Array<Int>();//用于存放返回值
if count > 0 {
var countArray = Array<Int>();//用于存放计数器
for _ in 0 ... maxValue {//初始化计数器
countArray.append(0);
}
for _ in 0 ..< count {
retArray.append(0);
}

//第一个步骤：统计数量
for i in 0 ..< count {//统计小于位置i的元素个数
countArray[sortArray[i]] = countArray[sortArray[i]] + 1;
}
for i in 1 ... maxValue {//统计出小于等于位置i的元素个数
countArray[i] = countArray[i] + countArray[i-1];
}

//第二个步骤：逆向扫描数组sortArray，将i位置元素放置到retArray中countArray对应值的位置上，同时修正countArray的值
for i in (0 ..< count).reversed() {
let value = sortArray[i];
let countValue = countArray[value];
retArray[countValue - 1] = value;
countArray[value] = countArray[value] - 1;
}
}
return retArray;
}
```

### 测试用例及结果

```Swift
var intArray =  [2, 5, 3, 0, 2, 3, 0, 3];
print(countSort(sortArray: &intArray,maxValue:5));

intArray =  [100,93,97,92,96,99,92,89,93,97,90,94,92,95];
print(countSort(sortArray: &intArray,maxValue:100));


//程序输出
[0, 0, 2, 2, 3, 3, 3, 5]
[89, 90, 92, 92, 92, 93, 93, 94, 95, 96, 97, 97, 99, 100]

```
