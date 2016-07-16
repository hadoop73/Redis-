---
title: Java 总结
tags: 
grammar_cjkRuby: true
---

[TOC]


[Java 面试题全集（上）][1]



##  反射

[深入研究 java.lang.Class 类][2]

[Java 反射机制(包括组成、结构、示例说明等内容)][3]

通俗来讲呢，就是在运行状态中，我们可以根据“类的部分已经的信息”来还原“类的全部的信息”。这里“类的部分已经的信息”，可以是“类名”或“类的对象”等信息

Java 运行时系统对所有的对象进行了运行时类型标识，记录了每个对象所属的类，虚拟机通常使用运行时类型信息选准正确方法去执行，用来保存这些类型信息的类是 Class 类。当装载类时，Class 类型的对象自动创建

* 通过对象获取类型
```java
MyObject x;
Class c1 = x.getClass();
```
* 通过字符串获取类型
```java
 Class c2=Class.forName("MyObject");
 Class c2="MyObject".getClass();
 // Employee必须是接口或者类的名字。
```
* 通过 Class 获取类型
```java
Class cl = Manager.class;
```
**创建对象**
```java
 Object obj = Class.forName(s).newInstance();
```

##  

  [1]: http://blog.csdn.net/jackfrued/article/details/44921941
  [2]: http://lavasoft.blog.51cto.com/62575/15433
  [3]: http://wangkuiwu.github.io/2012/03/04/reflection/
