---
layout:     post
title:      重定向
date:       2017-08-29 09:51:43
author:     liangtong
catalog: true
categories: 操作系统
tags: shell

---

在linux系统下，一切皆是文件，对文件的操作，一般要用到文件标识符。

#### 常用的FD(File Descriptor)

linux启动后，会默认打开3个文件描述符。分别是：   

 + 0 - 标准输入，/dev/stdin，可以理解为键盘
 + 1 - 标准输出，/dev/stdout，正确的输出，可以理解为屏幕
 + 2 - 标准错误输出，/dev/stderr，错误输出

对于任何一条linux命令，都会有以上默认的3个描述符。也可以简单的理解为每个程序(函数)都会有输入、正确／错误输出。

<!-- more -->

#### 重定向

重定向分为输入重定向和输出重定向。

##### 输入重定向

即不从键盘读入、而是从文件或者其他。
 > *<* 表示输入重定向运算符。

例如命令先删除b.txt文件中内容，然后将a.txt文件中内容复制到b.txt文件中。

```shell
liangtongdeiMac:playground liangtong$ > b.txt 
liangtongdeiMac:playground liangtong$ cat > b.txt  < a.txt 
```

等同与

```shell
liangtongdeiMac:playground liangtong$ > b.txt 
liangtongdeiMac:playground liangtong$ less a.txt >b.txt 
```

> *<<* 表示当前标准输入来自命令行，符号后边跟随的是『结束的输入字符』。

```shell
liangtongdeiMac:playground liangtong$ > b.txt 
liangtongdeiMac:playground liangtong$ less b.txt 
liangtongdeiMac:playground liangtong$ > b.txt 
liangtongdeiMac:playground liangtong$ cat > b.txt << eof
> hello
> world
> eof
liangtongdeiMac:playground liangtong$ 
liangtongdeiMac:playground liangtong$ cat >>b.txt  << endofinput
> hahaha
> endofinput
liangtongdeiMac:playground liangtong$ cat >>b.txt  << endofinput
> adsfasdf
> endofinput
liangtongdeiMac:playground liangtong$ less b.txt 

```


##### 输出重定向

将一条命令执行结果（标准输出，或者错误输出，本来都要打印到屏幕上面的）  重定向其它输出设备（文件，打开文件操作符，或打印机等等）1,2分别是标准输出，错误输出

 + *>* 会判断右边文件是否存在，如果存在就先删除，并且创建新文件。不存在直接创建。
 + *>>*  判断右边文件，如果不存在，先创建。以添加方式打开文件。

例如以下命令对应3种操作。 
 + 创建c.txt文件或者清除c.txt文件内容。
 + 将a.txt内容写入c.txt中。
 + 将b.txt内容追加写入c.txt中。

```shell
liangtongdeiMac:playground liangtong$ > c.txt
liangtongdeiMac:playground liangtong$ less a.txt > c.txt 
liangtongdeiMac:playground liangtong$ less b.txt  >> c.txt 

```

 +  命令执行完后，绑定文件的描述符也自动失效。0,1,2又会空闲

####  典型案例分析

经过上述介绍，下面我们看一些典型的例子，这些尤其在makefile中很常见。 **特别注意的是，多个重定下复合时，按照顺序执行，例如 1>/dev/null 2>&1 和 2>&1 1>/dev//null 执行结果不一样**

 > & 是一个描述符，如果1或2前不加&，会被当成一个普通文件
 
##### 1>&2和2>&1和&>filename

1>&2 意思是标准输出重定向到标准错误。比如以下命令将a.txt
2>&1 意思是把标准错误输出重定向到标准输出
&>filename 意思是把标准输出和标准错误输出都重定向到文件filename中

比如，以下命令分别 *ls -l *命令的标准输出、错误输出重定向到right.txt和error.txt文件中

```Shell
liangtongdeiMac:playground liangtong$ ls -l not_exist.txt a.txt 1>right.txt 2>error.txt
liangtongdeiMac:playground liangtong$ less right.txt 
liangtongdeiMac:playground liangtong$ less error.txt 
```

以下命令将标准输出重定向到right.txt文件中，将标准错误输出重定向到标准输出（此时是right.txt文件），此时right.txt文件将存放命令执行的标准输出和标准错误输出。

```Shell
liangtongdeiMac:playground liangtong$ ls -l not_exist.txt a.txt 1>right.txt 2>&1
liangtongdeiMac:playground liangtong$ less right.txt 
```

##### /dev/null

/dev/null是一个垃圾箱，是一个无底洞，表示的含义为不显示。直接丢弃。
比如以下命令效果一样，对标准输入、标准错误输出均丢弃

```Shell
liangtongdeiMac:playground liangtong$ ls -l not_exist.txt a.txt 1>/dev/null 2>/dev/null
liangtongdeiMac:playground liangtong$ ls -l not_exist.txt a.txt 1>/dev/null 2>&1
```

##### 2>&1 1>/dev/null 

将标准错误输出在屏幕上（标准输出）表示，将标准输出丢弃，此时屏幕上只输出标准错误日志。

##### 1>/dev/null 2>&1

将标准输出丢弃，将标准错误重定向到标准输出，此时无任何输出(均被丢弃)。
