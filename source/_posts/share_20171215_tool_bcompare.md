---
layout:     post
title:      Beyond Compare for Mac免License方法
date:       2017-12-15 14:25:19
author:     liangtong
categories: 分享
tags: tool

---

本文操作仅供参考，如果感觉使用方面建议购买正版[Beyond Compare链接](http://www.scootersoftware.com/)。


具体操作 

 * 下载Beyond Compare，然后依次 *包含内容 -> Contents -> MacOS*
 * 重命名 BCompare -> BCompare.real
 * 新建脚本，命名 *BCompare* ，修改权限
 * 打开，可以用了


```base
bogon:MacOS liangtong$ echo "" > BCompare
bogon:MacOS liangtong$ chmod a+x BCompare
```
其中，新建脚本的内容如下：

```base
#!/bin/bash
rm "/Users/$(whoami)/Library/Application Support/Beyond Compare/registry.dat"
"`dirname "$0"`"/BCompare.real $@

```

