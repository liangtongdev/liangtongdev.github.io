---
layout:     post
title:      Objective-C 知识普及
date:       2016-10-30 18:00:00
catalog: true
categories: 前端
tags: 杂谈 

---

* content
{:toc}

讲述闭包、协议、常用传值方式，交流时会结合参与的项目进行



### 闭包

​	一种带有局部变量的匿名函数。语法结构：`^` `返回值类型` `参数列表` `表达式`

比如：

```objective-c
^int(int count){return count+1;}
```

block的声明及使用

```Objective-C
//声明
int(^blk) (int) = ^(int count){return count+1;};

//使用 - 作为参数
void function(int(^blk)(int))

//使用 - 作为返回值
int (^func())(int) {
return ^(int count){return count+1;}
}

```

使用 `typedef` 简化

```objective-c
typedef int (^blk_t) (int);

//原来的写法
void func(int (^blk)(int))
//新的写法
void func(blk_t blk)

//原来的写法
int (^func()(int))
//新的写法
blk_t func()
```

Objective-C代码中常用的写法举例

```objective-c
//点击课程更多按钮菜单的回调
@property (nonatomic,copy) void(^courseMoreActionBlock)(EMoocCourseMoreActionType actionType, NSString* courseId);

///#begin
/**
 *	@brief	用户提交登录请求
 *
 *	@param 	userName 	登录名字
 *  @param  password    登录密码
 *	@param 	complete 	回调函数
 */
///#end
+(void)loginWithName:(NSString*)userName
            password:(NSString*)password
            progress:(void (^)(NSProgress *progress))progress
            complete:(void (^)(id data,NSInteger stateCode))complete;

```

##### 注意点

block 只读block外的属性，不可修改。
__block可破坏闭包，其修饰的属性可以在block内修改。
使用block的时候，特别要注意循环引用的问题



### 协议

​	类似Java中的interface。在ObjC中使用@protocol定义一组方法规范，实现此协议的类必须实现对应的方法。
例如常用的UITableViewDelegate协议格式：

```Objective-C
@protocol UITableViewDelegate<NSObject, UIScrollViewDelegate>//可以扩展多个协议

@required //必须实现的方法
- (NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section;

@optional //可选实现的方法
- (void)tableView:(UITableView *)tableView willDisplayCell:(UITableViewCell *)cell forRowAtIndexPath:(NSIndexPath *)indexPath;

@end
```

##### 注意点

* 协议方法的实现。
  * @required //必须实现的方法(或者属性)
  * @optional //可选实现的方法(或者属性)
* 调用delegate（实现协议的委托对象）方法时，如果是可选方法，一定要通过@selector进行判断


### 常用传值方式
​		这里说的传值方式，不包括View<->ViewController之间的Outlet、Target、Action等

##### 闭包
一言不合就上代码：注意`__weak`和`__strong`的使用

```Objective-C
//闭包作为属性的例子
@property (nonatomic,copy) void(^didTapCallBack)(id tapedItem);
//使用
__weak __typeof(self) weakSelf = self;
obj1.didTapCallBack = ^(NSDictionary* tapedItem){
	__strong __typeof(weakSelf)strongSelf = weakSelf;
   	[strongSelf performSegueWithIdentifier:@"showNextSegue” sender: tapedItem];
};


//闭包作为参数使用的例子：
-(void)functionNameWithParamName1:(id)paramName1
                    completeBlock:(NSDictionary* (^)(NSString* blockParam))completeBlock
//使用
[obj2 functionNameWithParamName1:@“” completeBlock:^NSDictionary*(NSString* blockParam) {
   	if (blockParam){
		return @[];
   	}else{
		return nil;
	}
}];



```

##### 协议
​	很常用的形式，比如UITableView、UIScrollView等UIKit框架中都是使用Protocol。

* 代码写起来清晰易懂

* 相对闭包来说代码量变大，第三者不容易理解

  ​

##### 共有方法(属性)

​	这块没啥说的，就是直接在`.h`文件中写属性，方法，对象直接调用（设置）即可

##### KVO
​	比较常用，一般Model、ViewModel层通过发广播的形式进行消息发送(数据传输)。


>
>
>
>



