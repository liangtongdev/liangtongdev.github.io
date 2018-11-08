---
layout:     post
title:      基数排序
date:       2017-06-13 11:25:37
author:     liangtong
catalog: true
categories: 算法
tags: Sort

---

* content
{:toc}

​	基数排序（radix sort）属于“分配式排序”（distribution sort），基数排序法是属于稳定性的排序，其时间复杂度为O (nlog(r)m)，其中r为所采取的基数，而m为堆数，在某些时候，基数排序法的效率高于其它的稳定性排序法。参照斯坦福大学算法公开课视频



### 基本思想及实现   
基本思想： 对于一个整型数组：      
* ① 将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列。
* ② 排序算法使用计数排序思想

使用Swift编写简单的代码实现，如下：    
```Swift
/**
*  基数排序
* 分布式排序算法
* 将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列。
* 基数排序的方式可以采用LSD（Least significant digital）或MSD（Most significant digital），LSD的排序方式由键值的最右边开始，而MSD则相反，由键值的最左边开始。
* 排序使用 - 计数排序
**/


/**
*
*  @brief 基数排序 - LSD
*  @param  sortArray 待排序整形数组
*  @param  loop    排序轮次
*  @param  step    步长
*
**/
func radixSort(sortArray:inout Array<Int>, loop:Int,step:Int){

    let length = sortArray.count;//待排序数组长度

    /**
    * 根据最高位和最低位，确定每次进行计数排序的中间位数
    **/
    var msb = 1;//最高位
    var minimum = 1;//最低位
    for _ in 0 ..< loop * step {
        msb = msb * 10;
    }
    for _ in 0 ..< step {
        minimum = minimum * 10;
    }

    /**
    * 计数排序 - 非基于比较的排序算法 。 可参照上一篇文章
    */
    var countArray = Array<Int>();
    var buckets = Array<Int>()
    for _ in 0 ... minimum {
        countArray.append(0);
    }
    for _ in 0 ..< length{
        buckets.append(0);
    }
    //第一个步骤：统计数量
    for i in 0 ..< length {//统计小于位置i的元素个数
        let sortValue = sortArray[i] / msb % minimum;
        countArray[sortValue] = countArray[sortValue] + 1;
    }
    for i in 1 ... minimum {//统计出小于等于位置i的元素个数
        countArray[i] = countArray[i] + countArray[i - 1];
    }
    //第二个步骤：逆向扫描数组sortArray，将i位置元素放置到retArray中countArray对应值的位置上，同时修正countArray的值
    for i in (0 ..< length).reversed() {
        let value = sortArray[i] / msb % minimum;
        let countValue = countArray[value];
        buckets[countValue - 1] = sortArray[i];
        countArray[value] = countArray[value] - 1;
    }

    //对本次基数排序结果进行排序
    for i in 0 ..< length {
        sortArray[i] = buckets[i];
    }
}
```

### 测试用例及结果

```Swift
    var intArray =  [14, 22, 28, 39, 43, 81, 93, 100, 92, 96, 99, 95, 10563, 55, 7, 65, 73, 999, 1024, 20490, 1];

    var maxValue = 0;
    var maxLength = 1;
    var step = 2;
    var loopTimes = 0;

    for i in 0 ..< intArray.count{
        if intArray[i] > maxValue{
            maxValue = intArray[i];
        }
    }

    while(maxValue / 10 != 0){
        maxLength = maxLength + 1;
        maxValue = maxValue / 10;
    }

    if maxLength % step > 0{
        loopTimes = 1;
    }
    loopTimes = loopTimes + maxLength / step;

    print("待排序数组\(intArray)");
    print("步长：\(step)");
    for i in 0 ..< loopTimes{
        radixSort(sortArray: &intArray, loop: i, step: step);
        print("第\(i)次基数排序后结果：\(intArray)");
    }

//程序输出
待排序数组[14, 22, 28, 39, 43, 81, 93, 100, 92, 96, 99, 95, 10563, 55, 7, 65, 73, 999, 1024, 20490, 1]
步长：2
第0次基数排序后结果：[100, 1, 7, 14, 22, 1024, 28, 39, 43, 55, 10563, 65, 73, 81, 20490, 92, 93, 95, 96, 99, 999]
第1次基数排序后结果：[1, 7, 14, 22, 28, 39, 43, 55, 65, 73, 81, 92, 93, 95, 96, 99, 100, 20490, 10563, 999, 1024]
第2次基数排序后结果：[1, 7, 14, 22, 28, 39, 43, 55, 65, 73, 81, 92, 93, 95, 96, 99, 100, 999, 1024, 10563, 20490]

```

### 相关推荐

>* <a href="https://l900416.github.io/2017/05/18/Algorithms-sort/"> 计数排序 </a>
