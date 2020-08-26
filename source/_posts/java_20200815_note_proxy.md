---
layout:     post
title:      Java 动态代理
date:       2020-08-15 17:14:13
author:     liangtong
catalog: true
categories: Java
tags: 笔记

---



#### 代理模式

代理模式是一种常见的设计模式。特征是代理类与委托类有相同的接口，代理类主要负责为委托类预处理消息，过滤消息、转发消息给委托类，以及事后消息处理等。代理类与委托类之间通常会存在关联关系。代理类的对象本身不实现服务，而是通过委托类对象来访问。

![proxy_design.png](/post/java/20200814/proxy_design_pattern.png)


#### 静态代理

由程序员创建或特定工具自动生成源代码，在编译时，接口、被代理类和代理类均已确定。在程序运行前，代理类的 `.class` 文件已经生成。

假设需求，将现有类的特定方法执行前后打印日志。在不修改已有代码的前提下完成。那么通过静态代理的方式做法如下：

+ 为现有类编写一个对应的代理类，并且该代理类和现有类实现相同的接口
+ 在创建代理类对象时，通过构造器塞入一个现有类对象。然后通过代理类的内部方法调用现有类对象的方法，并在调用前后打印日志。 代理对象 = 增强代码 + 目标对象(现有对象)



总结下代理模式的三个主要的元素：

+ 公共接口
+ 具体类（委托类或被代理类）
+ 代理类

其中，代理类持有具体类的实例，代为执行具体类的实例方法。



**静态代理的缺陷： 需要手动创建目标类的代理类，当目标类比较多时，工作量太大。**



#### 复习

在开始介绍动态代理前，先复习下两个知识点：反射、对象的创建

##### 反射

在Java运行环境中，对于任意一个类，通过反射可以知道这个类有哪些属性和方法，对于任意一个对象，可以调用其任意的方法。Java的反射主要体现在以下几个方面

+ 在运行时，判断任意一个对象所属的类。
+ 在运行时，构造任意一个类的对象。
+ 在运行时，任意一个类所具有的成员变量和方法。
+ 在运行时，调用任意一个对象的方法。

##### 对象的创建

![new_class.png](/post/java/20200814/new_class.jpg)

要创建实例，最关键的是得到对应在 **JVM方法区的`Class对象`**。 Class对象中包含了一个类的所有信息，比如构造器、方法、属性等。在得到代理的Class对象后，通过反射可以直接创建其实例。


![class.png](/post/java/20200814/class.jpg)

#### JDK动态代理

代理类在程序运行时创建的代理方式叫动态代理。在动态代理中，代理类并不是在Java代码中定义的，而是在运行期间动态生成的。JDK动态代理只能对实现了接口的类进行代理，而不是针对类。

在Java的`java.lang.reflect`包下提供了一个Proxy类和一个InvocationHandler接口，这个类和接口相互配合，可以生成JDK动态代理类和动态代理对象。

在Proxy类中，有两个个静态方法，可以返回代理的Class对象和代理Class对象的实例。

```Java
    @CallerSensitive
    public static Class<?> getProxyClass(ClassLoader loader,
                                         Class<?>... interfaces)
        throws IllegalArgumentException
    {
        //通过传入类加载器和一组接口，他就可以返回给代理的Class对象
    }

    @CallerSensitive
    public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h)
        throws IllegalArgumentException
    {
      //通过传入类加载器、一组接口和一个回调处理对象，他就可以返回给代理的Class对象的实例
     }
```

因此，一旦我们明确了接口（**注意：JDK动态代理必须有接口**），就可以通过以下两种方式创建代理类的实例。实际工作中，使用第二种的方法比较多。

+ 通过Proxy 获取 代理类 Class对象，然后获取Class对象的构造函数，最后创建对象实例

```Java
    @Test
    public void  test_listProxy1(){
        List<String> list = new ArrayList<>();
        try{
            //1.1 通过创建代理Class对象
            Class listProxyClazz = Proxy.getProxyClass(list.getClass().getClassLoader(), List.class);
            //1.2 得到参数构造器
            Constructor constructor = listProxyClazz.getConstructor(InvocationHandler.class);
            //1.3 反射创建对象实例
            List listProxyImpl = (List)constructor.newInstance(new InvocationHandler() {
                @Override
                public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                    log.info("invoke method " + method);
                    return method.invoke(list, args);
                }
            });
            listProxyImpl.add(" insert into list");
            log.info(list.toString());
        }catch (Exception e){
            log.info(e.toString());
        }
    }
```

运行结果如下：

```bash
[INFO ] 2020-08-15 16:00:35 [main] xxx.xx.xxxxx.xxxxxx - invoke method public abstract boolean java.util.List.add(java.lang.Object)
[INFO ] 2020-08-15 16:00:35 [main] xxx.xx.xxxxx.xxxxxx - [ insert into list]
```

+ 通过代理对象实例、接口直接创建代理对象实例。

```Java
    @Test
    public void test_listProxy2() {
        List<String> list = new ArrayList<>();
        try {
            //2.1 通过创建代理Class对象实例
            List listProxyImpl = (List) Proxy.newProxyInstance(list.getClass().getClassLoader(),
                    list.getClass().getInterfaces(),
                    new InvocationHandler() {
                        @Override
                        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                            log.info("invoke method " + method);
                            return method.invoke(list, args);
                        }
                    });

            listProxyImpl.add(" insert into list");
            log.info(list.toString());
        } catch (Exception e) {
            log.info(e.toString());
        }
    }
```

运行结果如下：

```bash
[INFO ] 2020-08-15 16:06:12 [main] xxx.xx.xxxxx.xxxxxx - invoke method public abstract boolean java.util.List.add(java.lang.Object)
[INFO ] 2020-08-15 16:06:12 [main] xxx.xx.xxxxx.xxxxxx - [ insert into list]
```

最后总结下JDK动态代理的几个关键步骤

+ 创建一个 `InvocationHandler` 对象，可以匿名，也可以创建XXProxy类实现该接口。
+ 使用Proxy类的getProxyClass静态方法生成动态代理 Class对象
+ 获取Class对象的带有InvocationHandler参数的构造器
+ 通过构造器创建动态实例。

以上过程也可以直接使用Proxy类的 `newProxyInstance` 方法来简化。

动态代理的优势在于对代理类的函数进行统一的处理（`InvocationHander` 中的 `invoke`），而不用修改每个代理类中的方法。



#### CGLIB动态代理

CGLIB是针对类实现的代理，主要是制定的类生成一个子类，覆盖其中的方法。

同JDK动态代理一样，我们只需要实现`org.springframework.cglib.proxy.MethodInterceptor`接口，然后设置`Enhancer`的callback对象，即可通过`enhancer.create()`生成动态代理类。

```Java
    @Test
    public void test_listCGLib(){
        List<String> list = new ArrayList<>();
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(list.getClass());
        enhancer.setCallback(new MethodInterceptor() {
            @Override
            public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                log.info("invoke method " + method);
//                return method.invoke(list, objects);
                return methodProxy.invokeSuper(o, objects);
            }
        });
        List listCGLibProxy = (List) enhancer.create();
        listCGLibProxy.add("insert into list");
        log.info(listCGLibProxy.toString());
    }
```

程序运行结果

```bash
[INFO ] 2020-08-17 13:55:52 [main] xxx.xx.xxxxx.xxxxxx - invoke method public boolean java.util.ArrayList.add(java.lang.Object)
[INFO ] 2020-08-17 13:55:52 [main] xxx.xx.xxxxx.xxxxxx - [insert into list]
```





#### Spring 选择使用动态代理方式？

  当Bean实现接口时，Spring会使用JDK动态代理；没有实现接口的话，在使用CGLib类实现。

  可以配置文件中强制使用CGLib的形式





