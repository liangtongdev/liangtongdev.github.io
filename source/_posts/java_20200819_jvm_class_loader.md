---
layout:     post
title:      类加载器
date:       2020-08-22 16:54:13
author:     liangtong
catalog: true
categories: Java
tags: JVM

---



JVM 把描述类数据从Class文件加载到内存，然后经过校验、转换解析和初始化，最终形成能被JVM直接使用的Java类型，这就是JVM的类加载机制。



一般类从加载到内存 到 卸载出内存到整个过程分为 7 个阶段 ： `加载、 验证、 准备、 解析、 初始化、 使用 和 卸载`。



### 加载过程

 类加载全过程包括`加载、 验证、 准备、 解析、 初始化` ，其中 `验证、准备、解析` 统称为 `连接`。



#### 加载

加载指到是将类的Class文件加载到内存中，并将这些数据转换成方法中的运行数据（静态变量、静态代码块、常量池等），在堆中生成一个Class类对象，作为方法区类数据的访问入口。

> 注：Java虚拟机并没有明确要求一定要存储在堆区中，只是hotspot选择将Class对戏那个存储在方法区中。

加载过程分为三步：

+ 通过一个类的全限定名来获取定义此类的二进制字节流。
+ 将这个字节流代表的静态存储结构转化为方法区的运行时数据结构。
+ 在内存中生成这个类的Class类对象，作为方法区这个类的数据访问入口。



Class文件的来源：

+ 从本地磁盘中直接加载
+ 通过网络下载.class文件
+ 从zip、 jar 等归档文件中加载.class文件
+ 从数据库中
+ 其他文件生成（jsp）



#### 验证

验证是连接阶段的第一步，重要而非必要。主要作用是确保被加载类的正确性。

验证内容包括格式验证、语义分析（比如final类是否被继承）、操作验证（私有方法访问等）等。

验证的目的确保class文件的字节流信息符合JVM要求，不会危害虚拟机的安全。



#### 准备

**为类中的所有静态变量分配内存空间，并为其设置一个初始值。**

+ 类变量（static）会分配内存，但实例变量不会。
+ 这里的初始值是数据类型默认值，而不是代码中被显式赋予的值。

```Java
// 在准备阶段之后，value值为0，而不是1。赋值为1的动作发生在初始化阶段。
public static int value = 1;

//例外情况：在准备阶段之后，value值即为1.
//如果变量被 static 和 final 同时修饰，则准备阶段直接赋值为指定值。
public finally static int value = 1;
```



#### 解析

**符号引用 -> 直接引用**，将常量池中的符号引用转为直接引用，这个可以在初始化之后再执行。



#### 初始化

这是类加载机制的最后一步，在这个阶段，java程序代码才开始真正执行。

JVM初始化一个类一般包括以下几个步骤：

+ 如果这个类还没有被加载和连接，则程序先加载并连接该类。
+ 如果这个类的直接父类还没有被初始化，则先初始化其直接父类。
+ 如果类中有初始化语句，系统依次执行这些初始化语句。

类的初始化时机：

+ 创建类的实例，也就是new的方式
+ 访问类的静态变量
+ 调用类的静态方法
+ 反射
+ 初始化类的时候，其父类也会被初始化

Java程序的初始化顺序：

+ 静态变量/代码块
  + 父类的静态变量
  + 父类的静态代码块
  + 子类的静态变量
  + 子类的静态代码块
+ 父类
  + 非静态变量
  + 非静态代码块
  + 构造函数
+ 子类
  + 非静态变量
  + 非静态代码块
  + 构造函数



### 类加载器

虚拟机设计团队把加载动作放到JVM外部实现，以便让应用程序决定如何获取所需的类。程序调用时，JVM会初始化，初始化的过程中会生成多个类加载器，JVM调用指定的类加载器去加载类即可。

#### 类缓存

标准的Java SE类加载器可以按要求查找类，一旦某个类被加载到类加载器中，它将维持加载（缓存）一段时间。不过，JVM垃圾收集器可以回收这些Class对象。

#### 类加载器分类

系统提供有3个类加载器：

+ 引导类加载器（Bootstrap ClassLoader）： 最顶层的加载器，主要加载核心类库。负责加载　`JAVA_HOME/lib` 下的jar包和class文件
+ 扩展类加载器（extensions class loader）：负责加载　`JAVA_HOME/lib/ext` 下的jar包和class文件
+ 应用程序类加载器（application class loader）：加载当前应用的classpath的所有类。一般来说，Java应用的类都由他来加载完成的。

用户自定义类加载器

可以继承`java.lang.ClassLoader`的方式实现自己的类加载器。

类加载器的顺序（自上而下） ： **启动类加载器 > 扩展类加载器 > 应用类加载器 > 自定义类加载器**

ClassLoader的常用方法：

+ **getParent**() ： 返回该类加载器的父类加载器。
+ loadClass(String name) ：表示根据类名进行双亲委托模型进行类加载并返回类对象。**此方法负责加载指定名字的类，首先会从已加载的类中去寻找，如果没有找到；从parent ClassLoader[ExtClassLoader]中加载；如果没有加载到，则从Bootstrap ClassLoader中尝试加载(findBootstrapClassOrNull方法), 如果还是加载失败，则自己加载。如果还不能加载，则抛出异常ClassNotFoundException。**
+ **findClass**(String name) ：查找名称为 name的类，返回的结果是java.lang.Class类的实例。
+ **findLoadedClass**(String name)： 查找名称为 name的已经被加载过的类，返回的结果是 java.lang.Class类的实例。
+ **defineClass**(String name, byte[] b, int off, int len) 把字节数组 b中的内容转换成 Java 类，返回的结果是java.lang.Class类的实例。这个方法被声明为 final的。



#### 双亲委派模式

**自底而上的检查类是否被加载，然后自顶而下的尝试加载类**。

某个特定的类加载器接收到类加载的请求时，会将加载任务委托给自己的父类，直到最高级父类**引导类加载器（bootstrap class loader）**，如果父类能够加载就加载，不能加载则返回到子类进行加载。如果都不能加载则报错。**ClassNotFoundException**

> 注：它们之间的父子关系并不是通过继承关系来实现的，而是使用组合关系来复用父加载器中的代码。

双亲委托机制是为了保证 Java 核心库的类型安全。这种机制保证不会出现用户自己能定义java.lang.Object类等的情况。例如，用户定义了java.lang.String，那么加载这个类时最高级父类会首先加载，发现核心类中也有这个类，那么就加载了核心类库，而自定义的永远都不会加载。

+ 可以避免重复加载，父类已经加载了，子类就不需要再次加载。
+ 更加安全，很好的解决了各个类加载器的基础类的统一问题。



#### 自定义类加载器

自定义类加载器主要有两种方式：

+ 继承ClassLoader， 重写 findClass 方法 - 遵守双亲委派模型
+ 继承ClassLoader， 重写 loadClass 方法 - 破坏双亲委派模型

推荐使用第一种，最大程度上遵循双亲委派模式，流程如下：

+ 首先检查请求的类型是否已经被这个类装载器装载到命名空间中了，如果已经装载，直接返回。
+ 委派类加载请求给父类加载器，如果父类加载器能够完成，则返回父类加载器加载的Class实例。
+ 调用本类加载器的findClass（…）方法，试图获取对应的字节码，如果获取的到，则调用defineClass（…）导入类型到方法区；如果获取不到对应的字节码或者其他原因失败，返回异常给loadClass（…）， loadClass（…）转抛异常，终止加载过程。



为方便理解，截取网上自定义类加载器代码

```Java
import java.io.*;

/**
 * @ClassName FileSystemClassLoader
 * @Description 自定义文件类加载器
 * @Author xwd
 * @Date 2018/10/24 9:23
 */
public class FileSystemClassLoader extends ClassLoader {
    private String rootDir;//根目录

    public FileSystemClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    /**
     * @MethodName findClass
     * @Descrition 加载类
     * @Param [name]
     * @return java.lang.Class<?>
     */
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        Class<?> loadedClass = findLoadedClass(name);//查询该类是否已经被加载过
        if(loadedClass != null){  //该类已经被加载过了,直接返回
            return loadedClass;
        }else{  //该类还没有被加载过
            ClassLoader classLoader = this.getParent();//委派给父类加载
            try {
                loadedClass = classLoader.loadClass(name);
            } catch (ClassNotFoundException e) {
//                e.printStackTrace();
            }
            if(loadedClass != null){  //父类加载成功,返回
                return loadedClass;
            }else{
                byte[] classData = getClassData(name);
                if(classData == null){
                    throw new ClassNotFoundException();
                }else{
                    loadedClass = defineClass(getName(),classData,0,classData.length);
                }
            }
        }
        return loadedClass;
    }

    /**
     * @MethodName getClassData
     * @Descrition 根据类名获得对应的字节数组
     * @Param [name]
     * @return byte[]
     */
    private byte[] getClassData(String name) {
        //pri.xiaowd.test.A  -->  D:/myjava/pei/xiaowd/test/A.class
        String path = rootDir + "/" + name.replace('.','/') + ".class";
//        System.out.println(path);
        InputStream is = null;
        ByteArrayOutputStream baos = new ByteArrayOutputStream();

        try {
            is = new FileInputStream(path);

            byte[] bytes = new byte[1024];
            int temp = 0;
            while((temp = is.read(bytes)) != -1){
                baos.write(bytes,0,temp);
            }
            return baos.toByteArray();
        } catch (FileNotFoundException e) {
            e.printStackTrace();
            return null;
        } catch (IOException e) {
            e.printStackTrace();
            return null;
        } finally {
            try {
                if(baos != null){
                    baos.close();
                }
                if(is != null){
                    is.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```












