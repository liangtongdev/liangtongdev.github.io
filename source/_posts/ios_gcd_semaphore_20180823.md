---
layout:     post
title:      semaphore的使用
date:       2018-08-23 11:00:00
author:     liangtong
catalog: true
categories: iOS
tags: GCD

---

本文主要记录iOS semaphore（信号量）的使用。
首先描述下应用场景（类似生产者消费者模式）：一个文件上传功能。首先存在一个待上传文件队列，用户在选择照片/视频后，文件会进入该队列。同时还有一个上传任务，从上传队列中获取N (N > 0) 个待上传的文件进行上传操作。 单个时间点上只能有一个上传任务执行。


当然，你可以通过标签或者锁的方式来实现功能，此处主要讲解使用 **semaphore** 实现。

#### 准备

开始之前，首先需要了解下 **信号量** ：是一种可用来控制访问资源的数量的标识，设定了一个信号量，在线程访问之前，加上信号量的处理，则可告知系统按照我们指定的信号量数量来执行多个线程。


信号量操作的顺序一般是：**创建  ->  等待 -> 唤醒 -> 等待 -> 唤醒 ...**

+ 信号量:dispatch_semaphore_t
  + 创建：dispatch_semaphore_create(long value)
  + 参数：信号量值，大于0
+ 等待（**降低信号值**）
  + 等待：dispatch_semaphore_wait(dispatch_semaphore_t dsema, dispatch_time_t timeout)
  + 超时时间：
      + DISPATCH_TIME_NOW 
          +  Indicates a time that occurs immediately.
      + DISPATCH_TIME_FOREVER
          +   Indicates a time that means infinity.
+ 唤醒 （**提高信号值**）
  + 唤醒：dispatch_semaphore_signal(dispatch_semaphore_t dsema) 




 + 队列：dispatch_queue_t

#### 实现

一个上传任务，相当于只有一个消费者。将信号量的信号值设置为1即可。
当向文件上传队列中添加文件时，通知上传任务等待；任务上传完成后，唤醒信号量，待上传的任务获取信号量后将队列中的全部文件上传，完成后释放（唤醒）信号量

代码片段如下：

```Objective-C
@property (nonatomic, strong) dispatch_semaphore_t semaphore;//信号量，用于单个文件上传附件
@property (nonatomic, strong) dispatch_queue_t queue;//信号队列
```

```Objective-C
    if(!_semaphore){
        _semaphore = dispatch_semaphore_create(1);
    }
    if(!_queue){
        _queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    }
```


核心上传服务

```Objective-C
    dispatch_async(_queue, ^{
	//信号量等待。
        dispatch_semaphore_wait(self.semaphore, DISPATCH_TIME_FOREVER);
	//检查上传队列是否存在待上传的任务，如果不存在，则直接释放（唤醒）信号量。
        if ([self.uploadPhotoPathList count] == 0) {
            dispatch_semaphore_signal(self.semaphore);
        }else{
            //存在下载任务，直接执行服务即可
            NSArray* filePathArray = [self.uploadPhotoPathList copy];
            [self.uploadPhotoPathList removeAllObjects];
            __weak __typeof(self) weakSelf = self;
            
            [XXViewModel addAttachment:key files:filePathArray progress:^(NSProgress *progress) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    //进度更新
                });
            } complete:^(NSString *errorTips) {
                __strong __typeof(weakSelf)strongSelf = weakSelf;
                if (errorTips) {
                    //出现错误
                }
                dispatch_async(dispatch_get_main_queue(), ^{
                    //任务执行完成。释放（唤醒）信号量。
                    dispatch_semaphore_signal(strongSelf.semaphore);
                });
            }];
        }
    });
```


#### 扩展

使用场景是针对单任务进行的，如果要求同时有2个上传任务，则需要将信号值设置为2即可。为了更加直观的看到结果。此处使用网上一些前辈的代码进行测试。分别对value进行1、2、3进行赋值

```Objective-C
//信号量值
const int value = 1;
-(void)dispatchSignal{
    dispatch_semaphore_t semaphore = dispatch_semaphore_create(value);
    dispatch_queue_t quene = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    
    //TASK 1
    dispatch_async(quene, ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"run task 1");
        sleep(1);
        NSLog(@"complete task 1");
        dispatch_semaphore_signal(semaphore);
    });
    //TASK 2
    dispatch_async(quene, ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"run task 2");
        sleep(1);
        NSLog(@"complete task 2");
        dispatch_semaphore_signal(semaphore);
    });
    //TASK 3
    dispatch_async(quene, ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"run task 3");
        sleep(1);
        NSLog(@"complete task 3");
        dispatch_semaphore_signal(semaphore);
    });
}
```

信号量值为1时

```Objective-C
2018-08-23 10:28:16.395233+0800 test[3152:350929] run task 1
2018-08-23 10:28:17.399579+0800 test[3152:350929] complete task 1
2018-08-23 10:28:17.399737+0800 test[3152:350930] run task 2
2018-08-23 10:28:18.403436+0800 test[3152:350930] complete task 2
2018-08-23 10:28:18.403612+0800 test[3152:350931] run task 3
2018-08-23 10:28:19.407994+0800 test[3152:350931] complete task 3
```

信号量值为2时

```Objective-C
2018-08-23 10:29:37.039559+0800 test[3175:354691] run task 1
2018-08-23 10:29:37.039559+0800 test[3175:354692] run task 2
2018-08-23 10:29:38.043809+0800 test[3175:354692] complete task 2
2018-08-23 10:29:38.043809+0800 test[3175:354691] complete task 1
2018-08-23 10:29:38.043960+0800 test[3175:354690] run task 3
2018-08-23 10:29:39.049032+0800 test[3175:354690] complete task 3
```

信号量值为3时

```Objective-C
2018-08-23 10:29:59.676944+0800 test[3192:356706] run task 1
2018-08-23 10:29:59.676950+0800 test[3192:356707] run task 3
2018-08-23 10:29:59.676951+0800 test[3192:356708] run task 2
2018-08-23 10:30:00.682263+0800 test[3192:356706] complete task 1
2018-08-23 10:30:00.682253+0800 test[3192:356708] complete task 2
2018-08-23 10:30:00.682263+0800 test[3192:356707] complete task 3
```

