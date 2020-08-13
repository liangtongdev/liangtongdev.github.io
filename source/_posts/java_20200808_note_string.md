---
layout:     post
title:      String、StringBuffer和StringBuilder之间的区别
date:       2020-08-08 18:24:12
author:     liangtong
catalog: true
categories: Java
tags: note

---



### 三者对比

+ String

  + 不可变
  + 每次对String的操作都会产生新的String对象，效率低浪费内存

+ StringBuffer

  + 可变

  + 线程安全

  + 对他指向的字符串操作不会产生新的对象，每个StringBuffer对象都会有一定的缓冲区容量，当字符串容量超过该容量时，会自动增加容量。

    + 初始化方法中

      ```Java
          //部分初始化方法截取
      		/**
           * Constructs a string buffer with no characters in it and an
           * initial capacity of 16 characters.
           */
          public StringBuffer() {
              super(16);
          }
      
         /**
           * Constructs a string buffer initialized to the contents of the
           * specified string. The initial capacity of the string buffer is
           * {@code 16} plus the length of the string argument.
           *
           * @param   str   the initial contents of the buffer.
           */
          public StringBuffer(String str) {
              super(str.length() + 16);
              append(str);
          }
      ```

      

  + 多线程操作字符串

    + 方法中都加有 synchronized 关键字

      ```Java
          //部分源码截取
      		@Override
          public synchronized StringBuffer append(CharSequence s) {
              toStringCache = null;
              super.append(s);
              return this;
          }
          @Override
          public synchronized StringBuffer append(char c) {
              toStringCache = null;
              super.append(c);
              return this;
          }
      ```

      

+ StringBuilder

  + 可变
  + 速度快
  + 线程不安全，单线程操作字符串



### String拼接

默写特殊情况下，String对象的字符串拼接其实是被JVM解释成了StringBuffer对象的拼接，所以这些时候String对象的速度并不比StringBuffer对象慢。例如以下情况：

```Java
String S1 = "This is only a" + " simple" + " test";
StringBuffer Sb = new StringBuffer("This is only a").append(" simple").append(" test");
```



### String 对象的创建

```Java
//代码1  
String sa = new String("Hello world");            
String sb = new String("Hello world");      
System.out.println(sa == sb);  // false       
//代码2    
String sc = "Hello world";    
String sd = "Hello world";  
System.out.println(sc == sd);  // true  
```

+ 代码1中局部变量sa,sb中存储的是JVM在堆中new出来的两个String对象的内存地址。虽然这两个String对象的值(char[]存放的字符序列)都是"Hello world"。 因此"=="比较的是两个不同的堆地址。

+ 代码2中局部变量sc,sd中存储的也是地址，但却都是常量池中"Hello world"指向的堆的唯一的那个拘留字符串对象的地址 。自然相等了。

  > JVM 加载 验证 准备 解析 初始化

### String  '+'

```Java

//代码1  
String sa = "ab";                                          
String sb = "cd";                                       
String sab = sa+sb;                                      
String s = "abcd";  
System.out.println(sab == s); // false  
//代码2  
String sc = "ab"+"cd";  
String sd = "abcd";  
System.out.println(sc == sd); //true 
```

+ 代码1中局部变量sa,sb存储的是堆中两个拘留字符串对象的地址。而当执行sa+sb时，JVM首先会在堆中创建一个StringBuilder类，同时用sa指向的拘留字符串对象完成初始化，然后调用append方法完成对sb所指向的拘留字符串的合并操作，接着调用StringBuilder的toString()方法在堆中创建一个String对象，最后将刚生成的String对象的堆地址存放在局部变量sab中。而局部变量s存储的是常量池中"abcd"所对应的拘留字符串对象的地址。 sab与s地址当然不一样了。
+ 代码2中"ab"+"cd"会直接在编译期就合并成常量"abcd"， 因此相同字面值常量"abcd"所对应的是同一个拘留字符串对象，自然地址也就相同。































