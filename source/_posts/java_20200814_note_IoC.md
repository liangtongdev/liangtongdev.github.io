---
layout:     post
title:      Spring IoC
date:       2020-08-14 17:24:12
author:     liangtong
catalog: true
categories: Java
tags: note

---



#### IoC

`Inversion of Control` 即 `控制反转`，不是什么技术，而是一种思想。是Spring的核心。**对Spring框架来说，是由Spring容器来负责控制对象的生命周期和对象之间的关系** 。IoC 意味着将你设计好的对象交给Spring容器控制，而不是由你的对象在内部直接控制。

> 传统的程序设计中，直接在对象内部通过new进行创建对象，是程序主动创建并控制对象，也就是正转。
>
> 而IoC是有专门的容器来创建这些对象，由容器来帮忙注入所依赖的对象。这就是反转。

`IoC`容器帮我们查找及注入依赖对象，对象只是别动的接受依赖对象，所以是反转。`IoC`主要从思想上对编程带来影响。原本程序获取资源都需要主动出击，但在`Ioc/DI`思想中，应用程序编程被动等待IOC容器来创建并注入他所需的资源了。



#### DI

`Dependency Injection` 即 `依赖注入`，组件之间的依赖关系由容器在运行期决定，由容器动态的将某个依赖关系注入到组件中。

> 应用程序依赖于IoC容器。由IoC容器来提供对象所需要的外部资源，并将依赖的资源（对象、资源、常量数据等）注入到应用程序的某个对象

**Ioc 和 DI 是同一个概念的不同角度描述**



#### Spring IoC

Spring IoC 容器的代表就是`org.springframework.beans`包中的`BeanFactory`接口，该接口提供了 `IoC` 容器的最基本功能。**BeanFactory 比较简单，通常只提供注册put和获取get两个功能**

而`org.springframework.context`包中的`ApplicationContext`接口扩展了`BeanFactory`，并集成了其他多个接口，因此具备高级的功能，比如资源获取。



+ XmlBeanFactory：BeanFactory实现，提供基本的IoC容器功能，可以从classpath或文件系统等获取资源；
  （1） File file = new File("fileSystemConfig.xml");
  Resource resource = new FileSystemResource(file);
  BeanFactory beanFactory = new XmlBeanFactory(resource);
  （2）
  Resource resource = new ClassPathResource("classpath.xml");
  BeanFactory beanFactory = new XmlBeanFactory(resource);

+ ClassPathXmlApplicationContext：ApplicationContext实现，从classpath获取配置文件；
  BeanFactory beanFactory = new ClassPathXmlApplicationContext("classpath.xml");

+ FileSystemXmlApplicationContext：ApplicationContext实现，从文件系统获取配置文件。
  BeanFactory beanFactory = new FileSystemXmlApplicationContext("fileSystemConfig.xml");



+ ApplicationContext接口获取Bean方法简介：
  + Object getBean(String name) 根据名称返回一个Bean，客户端需要自己进行类型转换；
  +  T getBean(String name, Class<T> requiredType) 根据名称和指定的类型返回一个Bean，客户端无需自己进行类型转换，如果类型转换失败，容器抛出异常；
  +  T getBean(Class<T> requiredType) 根据指定的类型返回一个Bean，客户端无需自己进行类型转换，如果没有或有多于一个Bean存在容器将抛出异常；
  +  Map<String, T> getBeansOfType(Class<T> type) 根据指定的类型返回一个键值为名字和值为Bean对象的 Map，如果没有Bean对象存在则返回空的Map。



















