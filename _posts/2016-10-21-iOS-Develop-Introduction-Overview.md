---
layout:     post
title:      iOS移动App开发综述
date:       2016-10-21 23:50:12
catalog: true
categories: 前端
tags: 杂谈 

---

* content
{:toc}

对iOS端移动App使用的开发语言、框架等信息进行介绍



### 环境及工具
* OS
  * macOS , OSX
* TOOL
  * XCode： `Version 8.0+` , 早期版本

> 建议：鉴于苹果的习惯，如果没有特殊需求，保持最新的系统和工具版本。

### 开发语言选择
​		首先要对开发的应用的功能有一定的了解，结合应用需要的功能及部署系统版本选择合适的开发语言。可以使用的主要开发语言包括：

* Objective-C：

  * 优点：老牌iOS开发语言，很多成熟稳定的开源代码可以使用；可以用来开发各个iOS系统上的应用。

  * 缺点：苹果已经开始主推Swift；语言较复杂，比如虽然引入了ARC概念，但是部分内存管理还是需要自己来搞定；

* Swift：苹果在2014年WWDC上发布的语言（函数式编程Monad，最低支持iOS7.0系统)，建议多使用结构体。

  * 优点：苹果主推的取代Objective-C语言，*开源*、*易学易用*、*借鉴了好多主流开发语言(很多概念比如：泛型，闭包，元组，协议等)，如果熟悉数据库，JavaScript、Objective-C等的话，可以快速入手*

  * 缺点：新语言，对旧系统不支持、多语言混合编译存在一些桥接问题；

* 其它：C/C++​


---
> *  如果想快速开发应用，建议使用Objective-C，毕竟有很多现成的东西；
> *  长远的看的话，建议使用Swift，新东西还是很让人兴奋的；
> *  Swift语言选择3.0以上的版本，因为XCode8.0及后续工具不兼容3.0之前的语言版本；
> *  如果项目中必须引入C++混合编译，建议使用Objective-C，Swift调用C代码还好，调用C++目前来说不可行。
> *  没啥想法，建议前期使用Objective-C和Swift混合开发，最终全面向Swift过渡。


---

### 程序框架选择
​	这里主要介绍两个框架MVC和MVVM，以及最终程序开发中使用的MVMCV

##### MVC
​	MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。
![](http://hi.csdn.net/attachment/201105/19/0_1305768550zP1b.gif)

* View：负责数据展示。
  * 应用程序中用户可以看见的对象。在iOS应用程序开发中，所有的控件、窗口等都继承自 UIView，UIView及其子类主要负责UI的实现，而UIView所产生的事件都可以采用委托的方式，交给UIViewController实现。

* Controller：处理输入输出。
  * View和Model之间交互的媒介，控制器对象解释在视图对象中进行的用户操作，并将新的或更改过的数据传达给模型对象。模型对象更改时，一个控制器对象会将新的模型数据传达给视图对象，以便视图对象可以显示它。对于不同的UIView，有相应的UIViewController，例如在iOS上常用的UITableView，它所对应的Controller就是UITableViewController。

* Model：持有数据，比如本地数据库，本地文件。
  ​

  ​	iOS应用中，对应UIViewController其实可以理解为ViewContainer（View + Controler）；如果使用MVC框架进行移动App开发的话，ViewController会显得特别臃肿，并且如果更换Controller策略就会显得比较麻烦。对于功能复杂的页面，其ViewController结构将会非常庞大，后期维护代价明显会比较高。

##### MVVM
​		MVVM代表的是Model View View-Model，她的出现可以理解成是给MVC的视图控制器瘦身。
![](http://images.cnitblog.com/i/380707/201403/152139487613221.png)

* View：负责数据展示。
  * 包含实际 UI 本身(不论是 UIView 代码, storyboard 和 xib), 任何视图特定的逻辑, 和对用户输入的反馈. 在 iOS 中这不仅需要 UIView 代码和那些文件, 还包括很多需由 UIViewController 处理的工作。

* View-Model ：处理输入输出。
  * 它的职责之一就是作为一个表现视图显示自身所需数据的静态模型;但它也有收集, 解释和转换那些数据的责任. 这留给了 view (controller) 一个更加清晰明确的任务: 呈现由 view-model 提供的数据。

* Model：持有数据，比如本地数据库，本地文件。
  ​

  ​	说下View-Model：它从必要的资源(数据库, 网络服务调用, 等)中获取原始数据, 运用逻辑, 并处理成 view (controller) 的展示数据. 它(通常通过属性)暴露给视图控制器需要知道的仅关于显示视图工作的信息(理想地你不会暴漏你的 data-model 对象)。 它还负责对上游数据的修改(比如更新模型/数据库, API POST 调用)。

##### MVC与MVVM的对比
​		经过上边对MVC和MVVM的介绍，相信大家对两种模式简单的映射关系有了一定的了解。通过对比，我们会发现视图控制器是非常大的。
![](http://cc.cocimg.com/api/uploads/20150525/1432542173109354.png)

##### MVMCV的产生
​		如果将两者结合，会是什么样子呢
![](http://cc.cocimg.com/api/uploads/20150525/1432542301497557.gif)
​		结果就说MVMCV产生了。
![](http://cc.cocimg.com/api/uploads/20150525/1432542328653234.png)

​		view-model会在视图控制器上以一个属性的方式存在. 视图控制器知道 view-model 和它的公有属性, 但是 view-model 对视图控制器一无所知. 你早就该对这个设计感觉好多了因为我们的关注点在这儿进行更好地分离.最后我们用一个应用构建模块层级图来结束这节介绍：
![](http://cc.cocimg.com/api/uploads/20150525/1432542362309133.png)


### 第三方应用的使用
* 手动将别人的代码拖拽到自己工程中。
  * 优点：随时可以修改别人的代码。
  * 缺点：需要自己维护该代码。
* 通过pod引用
  * 优点：大家一起维护，代码质量高；代码更新只要一句`pod update`一切都搞定 
  * 缺点：如果发现问题，修改可能会比较慢

>建议：如果不涉及UI，尽量使用pod来管理；使用GitHub第三方代码的时候需要注意几点：star、watch数量，还有活跃度。别选那种没人用的，不然后期出了问题你就很可能只能靠自己了

### 发布相关
* 签名Code Sign
  * App标识符
  * 开发证书
  * 发布证书
* APNS
  * 开发证书
  * 发布证书
* 其他

### 学习建议
* 如果有一定的语言基础，不用看「xx从入门到精通」之类的书。因为技术更新很快，书上很多东西都过时或者被弃用了
* 多看看苹果开发者文档，接口文档介绍的非常详细。
* 多逛逛GitHub，学习GitHub上一些牛人的思想，编码风格。链接 ： <a href="https://github.com/search?l=Objective-C&o=desc&q=stars%3A%3E1&s=stars&type=Repositories">Objective-C 排行</a>

>
> 
>
> 
