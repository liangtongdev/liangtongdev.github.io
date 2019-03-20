---
layout:     post
title:       I/O 重定向
date:       2019-03-20 08:00:00
author:     liangtong
catalog: true
categories: 操作系统
tags: shell

---

  

+ `cat`  连接文件
+ `sort`  排序文本行
+ `uniq`   报道或省略重复行
+ `grep` 打印匹配行
+ `wc`  打印文件中换行符，字，和字节个数
+ `head` 输出文件第一部分
+ `tail` 输出文件最后一部分
+ `tee` 从标准输入读取数据，并同时写到标准输出和文件







#### 标准输入、输出和错误

程序输出包括两个类别

+ 程序运行结果
+ 运行过程中的状态和错误信息

**万物皆文件**，默认情况下

+ 标准输出和标准错误的文件都连接到屏幕
+ 标准输入连接到键盘



#### 标准输出重定向

使用 `>`重定向符，可以将标准输出重定向到屏幕以外的文件。例如

```bash
liangdeiMac:draft liangtong$ ls /usr/bin/ > output.txt
liangdeiMac:draft liangtong$ 
```

#### 标准错误重定向

文件内容清除与追加。

假如我们使用一个不存在的目录，默认会产生一个标准错误。而标准错误默认连接的是屏幕

```bash
liangdeiMac:draft liangtong$ ls /usr/liangtong > output.txt
ls: /usr/liangtong: No such file or directory
liangdeiMac:draft liangtong$ 
```

我们打开重定向生成的output.txt文件：空白，大小为0

**小技巧：我们可以使用`> output.txt`这种重定向命令清空文件**

**文件内容追加使用`>>`重定向符**



**对于文件流的前 三个称作标准输入、输出和错误，shell 内部分别将其称为文件描述符0、1和2**，即

+ 0 标准输入
+ 1 标准输出
+ 2 标准错误

我们可以用以下语法来重定向标准错误

```bash
liangdeiMac:draft liangtong$ ls /usr/liangtong 2> output.txt
liangdeiMac:draft liangtong$ 
```



#### 重定向标准输出和错误到同一个文件

可能有这种情况，我们希望捕捉一个命令的所有输出到一个文件。为了完成这个，我们 必须同时重定向标准输出和标准错误。有两种方法来完成任务。第一个，传统的方法， 在旧版本 shell 中也有效：

```bash
ls /usr/liangtong > output.txt 2>&1
```

注意重定向的顺序安排非常重要。标准错误的重定向必须总是出现在标准输出 重定向之后，要不然它不起作用。

现在的 bash 版本提供了第二种方法，更精简合理的方法来执行这种联合的重定向。

```bash
liangdeiMac:draft liangtong$ ls /usr/liangtong &> output.txt
liangdeiMac:draft liangtong$ 
```



#### 不需要的输出 /dev/null

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



#### cat -  连接文件

使用`<`重定向符，我们可以把标准输入从键盘改到其他文件。例如

```bash
liangdeiMac:draft liangtong$ cat < output.txt 
ls: /usr/liangtong: No such file or directory
liangdeiMac:draft liangtong$ 
```



#### 管道线

命令从标准输入读取数据并输送到标准输出的能力被一个称为管道线的 shell 特性所利用。使用管道操作符**`|`**（竖杠），一个命令的标准输出可以通过管道送至另一个命令的标准输入：

```bash
command1 | command2
```

例如

```bash
liangdeiMac:draft liangtong$ cat < output.txt | less
liangdeiMac:draft liangtong$ 
```

管道线经常用来对数据完成复杂的操作。有可能会把几个命令放在一起组成一个管道线。 通常，以这种方式使用的命令被称为**过滤器**。过滤器接受输入，以某种方式改变它，然后 输出它。



把目录/bin 和/usr/bin 中 的可执行程序都联合在一起，再把它们排序，然后浏览执行结果：

```bash
liangdeiMac:draft liangtong$ ls /bin /usr/bin | sort | less
liangdeiMac:draft liangtong$ 
```



#### uniq - 报道或忽略重复行

uniq 命令经常和 sort 命令结合在一起使用。uniq 从标准输入或单个文件名参数接受数据有序 列表（详情查看 uniq 手册页），默认情况下，从数据列表中删除任何重复行。

```bash
liangdeiMac:draft liangtong$ ls /bin /usr/bin | sort | uniq | less
liangdeiMac:draft liangtong$ 
```



#### wc － 打印行数、字数和字节数



除了打印出行数、单词数和字节数，我们还可以通过添加`-l` 参数，只打印出行数。

```bash
liangdeiMac:draft liangtong$ ls /bin /usr/bin | sort | uniq | less | wc
    1007    1006    8278
liangdeiMac:draft liangtong$ ls /bin /usr/bin | sort | uniq | less | wc -l
    1007
liangdeiMac:draft liangtong$ 
```



#### grep － 打印匹配行

grep 是个很强大的程序，用来找到文件中的匹配文本。这样使用 grep 命令：

```bash
grep pattern [file...]
```

简单操作例如

```bash
liangdeiMac:draft liangtong$ ls /bin /usr/bin | sort | uniq | grep zip
bunzip2
bzip2
bzip2recover
funzip
gunzip
gzip
unzip
unzipsfx
zip
zipcloak
zipdetails
zipdetails5.18
zipgrep
zipinfo
zipnote
zipsplit
liangdeiMac:draft liangtong$ 
```



#### head / tail － 打印文件开头部分/结尾部分

有时候你不需要一个命令的所有输出。可能你只想要前几行或者后几行的输出内容。 head 命令打印文件的前十行，而 tail 命令打印文件的后十行。默认情况下，两个命令 都打印十行文本，但是可以通过”-n”选项来调整命令打印的行数。

```bash
liangdeiMac:draft liangtong$ head -n 3 os_20190319_shell_io_redirection.md 
---
layout:     post
title:       I/O 重定向
liangdeiMac:draft liangtong$ tail -n 3 os_20190319_shell_io_redirection.md 

#### head / tail － 打印文件开头部分/结尾部分

liangdeiMac:draft liangtong$ 
```



也可以结合管道使用

```bash
liangdeiMac:draft liangtong$ ls /bin /usr/bin | sort | uniq | grep zip | head -n 4
bunzip2
bzip2
bzip2recover
funzip
liangdeiMac:draft liangtong$ 
```



#### tee － 从 Stdin 读取数据，并同时输出到 Stdout 和文件

为了和我们的管道隐喻保持一致，Linux 提供了一个叫做 tee 的命令，这个命令制造了 一个”tee”，安装到我们的管道上。tee 程序从标准输入读入数据，并且同时复制数据 到标准输出（允许数据继续随着管道线流动）和一个或多个文件。当在某个中间处理 阶段来捕捉一个管道线的内容时，这很有帮助。

