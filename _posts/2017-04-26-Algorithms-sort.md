---
layout:     post
title:      归并排序
date:       2017-04-26 14:25:00
author:     liangtong
catalog: true
categories: 算法
tags: Sort

---

* content
{:toc}

​	归并排序是建立在归并操作上的一种有效的排序算法，该算法是采用分治法（Divide and Conquer）的一个非常典型的应用。将已有序的子序列合并，得到完全有序的序列；即先使每个子序列有序，再使子序列段间有序。若将两个有序表合并成一个有序表，称为二路归并。需要额外的存储空间，时间复杂度： T(n) = Θ(nlgn)。



### 基本原理及实现   
基本原理：使用递归、分治思想，将待排序数组分割成字数组，将排序后的字数组合并。    
 * ① 当数组只包含一个元素时，直接返回该元素即可。
 * ② 取数组中间位置元素，递归调用左侧和右侧数组，进行归并排序。
 * ③ 将左侧和右侧排序后的有序数组进行合并。     

使用Swift进行功能实现，如下：

```Swift
/**
 * 归并排序 - 算法思想
 * 使用递归、分治的思想。
 * ① 当数组只包含一个元素时，直接返回该元素即可。
 * ② 取数组中间位置元素，递归调用左侧和右侧数组，进行归并排序。
 * ③ 将左侧和右侧排序后的有序数组进行合并。
 *
 **/
func mergeSort<T:Comparable>(array: Array<T>, start:Int , end:Int) -> Array<T>{
    if start >= end {//① 只有一个元素，直接返回即可
        return [array[start]];
    }else{
        let middle = (start + end) / 2;
        //② 递归调用，给左侧数组排序
        let leftSortArray = mergeSort(array:array,start:start,end:middle);
        //② 递归调用，给右侧数组排序
        let rightSortedArray = mergeSort(array: array, start: middle + 1, end: end);
        
        //③ 合并有序数组
        var leftIndex = 0;
        var rightIndex = 0;
        let leftCount = leftSortArray.count;
        let rightCount = rightSortedArray.count;
        var retArray:Array<T> = [];
	// 两个有序数组均存在值时，依次插入最小值
        while leftIndex < leftCount && rightIndex < rightCount {
            let leftValue = leftSortArray[leftIndex];
            let rightValue = rightSortedArray[rightIndex];
            if leftValue < rightValue {
                retArray.append(leftValue);
                leftIndex += 1;
            }else{
                retArray.append(rightValue);
                rightIndex += 1;
            }
        }
        // 将左侧有序数组剩余部分依次插入到排序结果中
        while leftIndex < leftCount {
            let leftValue = leftSortArray[leftIndex];
            retArray.append(leftValue);
            leftIndex += 1;
        }
        // 将右侧有序数组剩余部分依次插入到排序结果中
        while rightIndex < rightCount {
            let rightValue = rightSortedArray[rightIndex];
            retArray.append(rightValue);
            rightIndex += 1;
        }
        
        return retArray;
    }
}

//调用方法
func callMergeSort<T:Comparable>(array:Array<T>)->Array<T>{
    let count = array.count;
    if count > 0 {
        return mergeSort(array: array, start: 0, end: count - 1);
    }else{
        return [];
    }

}
```

### 测试用例及结果

```Swift
//测试用例
var intArray =  [16,4,7,9,8,11,11,15,24,1];
print("待排序数组：\(intArray)");
let sortedArray = callMergeSort(array: intArray);
print("排序后数组：\(sortedArray)");

//程序输出
待排序数组：[16, 4, 7, 9, 8, 11, 11, 15, 24, 1]
排序后数组：[1, 4, 7, 8, 9, 11, 11, 15, 16, 24]
```

```Swift
//测试用例
var stringArray =  ["Java","C#","Python","Objective-C","Swift","PHP","BASIC","Pasical","Assembly"];
print("待排序数组：\(stringArray)");
let sortedStringArray = callMergeSort(array: stringArray);
print("排序后数组：\(sortedStringArray)");

//程序输出
待排序数组：["Java", "C#", "Python", "Objective-C", "Swift", "PHP", "BASIC", "Pasical", "Assembly"]
排序后数组：["Assembly", "BASIC", "C#", "Java", "Objective-C", "PHP", "Pasical", "Python", "Swift"]
```

