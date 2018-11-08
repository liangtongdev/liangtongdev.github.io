---
layout:     post
title:      插入排序
date:       2017-04-25 21:22:00
author:     liangtong
catalog: true
categories: 算法
tags: Sort

---

* content
{:toc}

​	每次将一个待排序的记录，按其大小插入到前面已经排好序的子序列中的适当位置，直到全部记录插入完成为止。时间复杂度 T(n) = Θ(n²)。



### 原理及实现   
基本原理：对待排序数组a[n]，假设第i(i∈[1,n])个元素左侧已排序，右侧未排序。    
 * ① 从i位置左侧已排序数组位置中，逆序遍历，查找元素a[i]的目标位置，即第一个比a[i]小的元素后边targetPos（或者第一个位置）。
 * ② 移动元素：将数组元素[targetPos + 1 .. i - 1] 整体向后移动一位。
 * ③ 将第i(i∈[1,n])个元素a[i]放置到[targetPos + 1]位置。
 * ④ 遍历执行①、②、③操作，直到最后一个元素a[n]。     

使用Swift进行功能实现，如下：

```Swift
/**
 * 插入排序 - 算法思想
 * 对待排序数组a[n]，假设第i(i∈[1,n])个元素左侧已排序，右侧未排序。
 * ① 从i位置左侧已排序数组位置中，逆序遍历，查找元素a[i]的目标位置，即第一个比a[i]小的元素后边targetPos（或者第一个位置）。
 * ② 移动元素：将数组元素[targetPos + 1 .. i - 1] 整体向后移动一位。
 * ③ 将第i(i∈[1,n])个元素a[i]放置到[targetPos + 1]位置。
 * ④ 遍历执行①、②、③操作，直到最后一个元素a[n]。
 *
 **/
func insertSort<T: Comparable>(array: inout Array<T>){
    let count = array.count;
    if count > 1 {
        //第i个元素的左侧为有序，右侧为无序。遍历执行操作④
        for i in 1 ..< count {
            let ai = array[i];
            
            var targetPos = i - 1;
            var aj = array[targetPos];
            //如果当前元素不小于前一个元素，表明数据已经是有序的，无需后续操作
            if ai < aj {
                //操作①，查找第一个比a[i]小的元素位置targetPos
                while  ai < aj && targetPos >= 0 {
                    targetPos = targetPos - 1;
                    if targetPos >= 0 {
                        aj = array[targetPos];
                    }
                }
                //操作②，移动元素
                for index in (targetPos + 1  ..< i).reversed() {
                    array[index + 1] = array[index];
                }
                //操作③，赋值
                array[targetPos + 1] = ai;
            }
            print("第\(i)次处理结果： \(array)")
        }
    }
}

```

### 测试用例及结果

```Swift
//测试用例
var intArray =  [16,4,7,9,8,11,11,15,24,1];
print("待排序整形数组：\(intArray)");
insertSort(array: &intArray);

//程序输出
待排序整形数组：[16, 4, 7, 9, 8, 11, 11, 15, 24, 1]
第1次处理结果： [4, 16, 7, 9, 8, 11, 11, 15, 24, 1]
第2次处理结果： [4, 7, 16, 9, 8, 11, 11, 15, 24, 1]
第3次处理结果： [4, 7, 9, 16, 8, 11, 11, 15, 24, 1]
第4次处理结果： [4, 7, 8, 9, 16, 11, 11, 15, 24, 1]
第5次处理结果： [4, 7, 8, 9, 11, 16, 11, 15, 24, 1]
第6次处理结果： [4, 7, 8, 9, 11, 11, 16, 15, 24, 1]
第7次处理结果： [4, 7, 8, 9, 11, 11, 15, 16, 24, 1]
第8次处理结果： [4, 7, 8, 9, 11, 11, 15, 16, 24, 1]
第9次处理结果： [1, 4, 7, 8, 9, 11, 11, 15, 16, 24]
```

```Swift
//测试用例
var stringArray =  ["Java","C#","Python","Objective-C","Swift","PHP","BASIC","Pasical","Assembly"];
print("待排序字符串数组：\(stringArray)");
insertSort(array: &stringArray);

//程序输出
待排序字符串数组：["Java", "C#", "Python", "Objective-C", "Swift", "PHP", "BASIC", "Pasical", "Assembly"]
第1次处理结果： ["C#", "Java", "Python", "Objective-C", "Swift", "PHP", "BASIC", "Pasical", "Assembly"]
第2次处理结果： ["C#", "Java", "Python", "Objective-C", "Swift", "PHP", "BASIC", "Pasical", "Assembly"]
第3次处理结果： ["C#", "Java", "Objective-C", "Python", "Swift", "PHP", "BASIC", "Pasical", "Assembly"]
第4次处理结果： ["C#", "Java", "Objective-C", "Python", "Swift", "PHP", "BASIC", "Pasical", "Assembly"]
第5次处理结果： ["C#", "Java", "Objective-C", "PHP", "Python", "Swift", "BASIC", "Pasical", "Assembly"]
第6次处理结果： ["BASIC", "C#", "Java", "Objective-C", "PHP", "Python", "Swift", "Pasical", "Assembly"]
第7次处理结果： ["BASIC", "C#", "Java", "Objective-C", "PHP", "Pasical", "Python", "Swift", "Assembly"]
第8次处理结果： ["Assembly", "BASIC", "C#", "Java", "Objective-C", "PHP", "Pasical", "Python", "Swift"]
```

