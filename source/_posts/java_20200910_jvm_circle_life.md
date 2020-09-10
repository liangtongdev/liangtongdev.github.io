---
layout:     post
title:      JVM 生命周期
date:       2020-09-10 20:54:13
author:     liangtong
catalog: true
categories: Java
tags: JVM

---

​	**JVM实例对应一个独立运行的Java程序（进程级别）**。当启动一个Java程序时，一个JVM实例诞生；当程序关闭/退出时，这个JVM也随之消亡。如果一台机器上同时运行多个Java程序，将产生多个JVM实例。

+ JVM实例的诞生：启动Java程序，任何一个由main函数的class都可以作为JVM实例运行的起点。
+ JVM实例的运行：main函数作为初始线程的起点，其他线程均由该线程启动。
+ JVM实例的消亡：当程序中所有的非守护线程都终止时，JVM才退出。程序也可以使用Runtime类或者exit退出。