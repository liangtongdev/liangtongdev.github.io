---
layout:     post
title:      commands
date:       2019-03-19 13:51:43
author:     liangtong
catalog: true
categories: 操作系统
tags: shell

---

  

常用命令

+ `type`  说明怎样解释一个命令名
+ `which`  显示会执行哪个可执行程序
+ `man`   显示命令手册页
+ `apropos` 显示一系列适合的命令
+ `info`  显示命令 info
+ `whatis` 显示一个命令的简洁描述
+ `alias` 创建命令别名







#### 命令的形式



+ 是一个可执行程序，就像我们所看到的位于目录/usr/bin 中的文件一样。 这一类程序可以是用诸如 C 和 C++语言写成的程序编译的二进制文件, 也可以是由诸如shell，perl，python，ruby等等脚本语言写成的程序 。
+ 是一个内建于 shell 自身的命令。bash 支持若干命令，内部叫做 shell 内部命令 (builtins)。例如，cd 命令，就是一个 shell 内部命令。
+ 是一个 shell 函数。这些是小规模的 shell 脚本，它们混合到环境变量中。
+ 是一个命令别名。我们可以定义自己的命令，建立在其它命令之上。



#### 识别命令

**type - 显示命令的类别**

`type command`

**which - 显示一个可执行程序的位置**

**help - 得到shell内建命令的帮助文档**

注意表示法：出现在命令语法说明中的方括号，表示可选的项目。一个竖杠字符 表示互斥选项。在上面 cd 命令的例子中：

```shell
cd [-L|-P] [dir]
```



**--help - 显示用法信息**

```bash
liangdeiMac:bin liangtong$ cd --help
-bash: cd: --: invalid option
cd: usage: cd [-L|-P] [dir]
liangdeiMac:bin liangtong$ 
```



**man - 显示程序手册页**

**apropos - 显示适当的命令**

**whatis - 显示非常简洁的命令说明**

**info - 显示程序info条目**



安装程序的文档文件，位于`usr/share/doc`目录下，大多数以文本文件的形式存储。可以用less阅读来浏览。



**用别名（alias）创建自己的命令**

*命令行小技巧。可以把多个命令放在同一行上，命令之间 用”;”分开。*

```bash
command1; command2; command3...
```

```bash
liangdeiMac:usr liangtong$ cd /usr;ls;cd -
bin		libexec		sbin		standalone
lib		local		share
/usr
```

```bash
liangdeiMac:usr liangtong$ type liangtong-test
-bash: type: liangtong-test: not found
liangdeiMac:usr liangtong$ alias liangtong-test='cd /usr;ls;cd -'
liangdeiMac:usr liangtong$ liangtong-test
bin		libexec		sbin		standalone
lib		local		share
/usr
liangdeiMac:usr liangtong$ type liangtong-test
liangtong-test is aliased to `cd /usr;ls;cd -'
liangdeiMac:usr liangtong$ 
```

删除别名，使用 unalias 命令

`unalias command`



