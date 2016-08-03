---
title: Java 总结
tags: 
grammar_cjkRuby: true
---

[TOC]


[Java 面试题全集（上）][1]

[Java研发方向如何准备BAT技术面试][2]

[115个Java面试题和答案——终极列表][3]

##  反射

[深入研究 java.lang.Class 类][4]

[Java 反射机制(包括组成、结构、示例说明等内容)][5]

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

##  final 关键词

[浅析Java中的final关键字][6]

* 修饰类
类不能被继承
* 修饰方法
防止继承类修改它的含义,类的 private 方法会隐式地被指定为 final 方法
* 修饰变量
对于一个 final 变量，如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改；如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象。



##  多态性
* 对象多态性
	* 向上转型:程序自动完成
	* 父类  父类对象 = 子类实例
* 向下转型:强制类型转换
	* 子类  子类对象 = (子类)父类实例



# 总结
     最近BAT等各大互联网巨头们的校招陆陆续续都准备开始了，可能对于在校的大多数学生来说，不知道如何正确衡量自己掌握的技术，更不知道BAT这样的公司会要求自己必须具备什么样的技术能力。对于Java研发方向的技术面试，这里总结了一些你必须要掌握的技术知识点，考察的内容会比这里总结的多，所以如果想要有一个很不错的offer，下面的知识点需要都具备。社招考察的内容会在此基础上增加项目经验、技术实战经验、热门技术的使用及理解。

## Java基础：

 1. 面向对象和面向过程的区别
- `面向过程是一件事“该怎么做“，面向对象是一件事“该让谁来做”，然后那个“谁”就是对象，他要怎么做是他自己的事，反正最后一群对象合力能把事做好就行了。`
 2. Java的四个基本特性（抽象、封装、继承，多态）

[java的四个特性（抽象，封装，继承，多态），对多态使用方式的理解][7]

- 抽象

    抽象：将一类事物的共同特征抽象出来，并进行类的设计和创建。抽象包括数据抽象和行为抽象，比如：犬类有年龄名字等属性，这种特征抽象出来就是数据抽象，所有的狗都有叫的行为，这种抽象成为一种方法，这种抽象就是行为抽象。抽象只关注对象有那些属性和行为，并不关注有这些行为的细节是什么。

- 封装

    封装：通常认为是把数据和操作数据的方法封装起来，对数据的访问，只能通过已定义的方法。面向对象的本质就是将现实世界描绘成一系列完全自制、封闭的对象。我们在类中编写的方法，就是对实现细节的一些封装。我们编写的一个类就是对数据或者数据操作的封装。可以说封装就是隐藏一切可以隐藏的东西，只向外界提供最简单的编程接口。

- 继承

    继承：继承就是从已有的类中继承信息，并创建新类的过程。被继承的类称之为父类，基类或者超类，得到继承信息的类称为子类或者派生类。继承让变化中的软件系统有了一定的延续性，同时继承也是封装可变因素的重要手段（日后解释，今天这里不是重点）。

- 多态

    多态：对不同子类型的对象对同一消息做出不同的回应。


 4. Overload和Override的区别

[ Java中的继承、封装、多态、抽象][8]

-    方法的重写Overriding和重载Overloading是Java多态性的不同表现。重写Overriding是父类与子类之间多态性的一种 表现，重载Overloading是一个类中多态性的一种表现。如果在子类中定义某方法与其父类有相同的名称和参数，我们说该方法被重写 (Overriding)。子类的对象使用这个方法时，将调用子类中的定义，对它而言，父类中的定义如同被“屏蔽”了。如果在一个类中定义了多个同名的方 法，它们或有不同的参数个数或有不同的参数类型，则称为方法的重载(Overloading)。Overloaded的方法是可以改变返回值的类型。方法的重写Overriding和重载Overloading是Java多态性的不同表现。

 5. 构造器Constructor是否可被override


 6. 访问控制符public,protected,private,以及默认的区别

|  修饰符  | 当前类   | 同包    | 子类    |其他包     |
| :---: | :---: | :---: | :---: | :---: |
|   public  |  √   |  √   |  √   |  √   |
|  protected   |  √   | √    | √    | ×    |
|   default  |  √   |  √   |  ×   |   ×  |
| private|√|×|×|×|

 1. 是否可以继承String类
 
 [Java String 源码浅析][9]
 [String 实现][10]
 [在java中String类为什么要设计成final][11]
`String` 是 final 修饰不能够继承
```java
public final class String{
	private final char value[];
	...
}
```
为了**安全**不可变,在 `HashSet` 的 `Key` 里面,变动就出问题了

 2. String和StringBuffer、StringBuilder的区别
[java中String、StringBuffer、StringBuilder的区别][12]

**可变性**
`String` 不可变,`StringBuffer` 和 `StringBuilder` 可变

**线程安全性**
`String` 和 `StingBuffer` 是线程安全的

**原理**
[Java StringBuilder和StringBuffer源码分析][13]
`StringBuffer` 和 `StringBuilder` 都是调用抽象父类 `AbstractStringBuilder` 的实现而已,不同之处在于 `StringBuffer` 加了同步关键词 `synchronized



 3. hashCode和equals方法的关系
[hashCode与equals的区别与联系][14]
* `equals` 方法用于比较对象
* `hashcode` 用于集合中
* 将对象放入到集合中时，首先判断要放入对象的hashcode值与集合中的任意一个元素的hashcode值是否相等，如果不相等直接将该对象放入集合中。如果hashcode值相等，然后再通过equals方法判断要放入对象与集合中的任意一个对象是否相等，如果equals判断不相等，直接将该元素放入到集合中，否则不放入。

[ java中的hashCode()和equals()的关系][15]
* 重写 `equals`,判断 `HashSet` 中元素不会重复
* 重写 `hashCode` 提高 `HashMap` 比对性能

**`==` 和 `equals` 的区别**
* == 判断是否为同一个对象,比对内存空间地址
* `equals` 被覆写后,由程序控制,`String` 类比对的是内容是否相同

 4. 抽象类和接口的区别
[接口和抽象类有什么区别][16]
* 抽象类是对根源的抽象,接口是对动作的抽象,抽象一组动作,不同的类不同的实现
* 接口中方法都是抽象的,不能实现,抽象类可以
* 接口中可以由 `static` 类型数据,抽象类没有

 5. 自动装箱与拆箱
[Java 自动装箱与拆箱(Autoboxing and unboxing)][17]
* 自动装箱是由值构建对象,构建对象是为了使用类提供的方法
* 拆箱是由对象返回值,能够很好的运算



 1. 什么是泛型、为什么要使用以及泛型擦除
 2. Java中的集合类及关系图
 3. HashMap实现原理(看源代码)
 4. HashTable实现原理(看源代码)
 5. HashMap和HashTable区别
 6. HashTable如何实现线程安全(看源代码)
 7. ArrayList和vector区别(看源代码)
 8. ArrayList和LinkedList区别及使用场景
 9. Collection和Collections的区别
 10. Concurrenthashmap实现原理(看源代码)
 11. Error、Exception区别
 12. Unchecked Exception和Checked Exception，各列举几个
 13. Java中如何实现代理机制(JDK、CGLIB)
 14. 多线程的实现方式
 15. 线程的状态转换
 16. 如何停止一个线程
 17. 什么是线程安全
 18. 如何保证线程安全
 19. Synchronized如何使用
 20. synchronized和Lock的区别
 21. 多线程如何进行信息交互
 22. sleep和wait的区别(考察的方向是是否会释放锁)
 23. 多线程与死锁
 24. 如何才能产生死锁
 25. 什么叫守护线程，用什么方法实现守护线程
 26. Java线程池技术及原理
 27. java并发包concurrent及常用的类
 28. volatile关键字
 29. Java中的NIO，BIO，AIO分别是什么
 30. IO和NIO区别
 31. 序列化与反序列化
 32. 常见的序列化协议有哪些
 33. 内存溢出和内存泄漏的区别
 34. Java内存模型及各个区域的OOM，如何重现OOM
 35. 出现OOM如何解决
 36. 用什么工具可以查出内存泄漏
 37. Java内存管理及回收算法
 38. Java类加载器及如何加载类(双亲委派)
 39. xml解析方式
 40. Statement和PreparedStatement之间的区别

## JavaEE:

 1. servlet生命周期及各个方法
 2. servlet中如何自定义filter

### JSP原理

 1. JSP和Servlet的区别
 2. JSP的动态include和静态include
 3. Struts中请求处理过程

### MVC概念

 1. Spring mvc与Struts区别
 2. Hibernate/Ibatis两者的区别
 3. Hibernate一级和二级缓存
 4. Hibernate实现集群部署
 5. Hibernate如何实现声明式事务
 6. 简述Hibernate常见优化策略
 7. Spring bean的加载过程(推荐看Spring的源码)
 8. Spring如何实现AOP和IOC
 9. Spring bean注入方式
 10. Spring的事务管理(推荐看Spring的源码)
 11. Spring事务的传播特性
 12. springmvc原理
 13. springmvc用过哪些注解
 14. Restful有几种请求
 15. Restful好处
 16. Tomcat，Apache，JBoss的区别
 17. memcached和redis的区别
 18. 有没有遇到中文乱码问题，如何解决的
 19. 如何理解分布式锁
 20. 你知道的开源协议有哪些
 21. json和xml区别

## 设计模式：

 1. 设计模式的六大原则
 2. 常用的设计模式
 3. 用一个设计模式写一段代码或画出一个设计模式的UML
 4. 如何理解MVC
 5. 高内聚，低耦合方面的理解

##  算法：

 1. 深度优先、广度优先算法
 2. 排序算法及对应的时间复杂度和空间复杂度
 3. 写一个排序算法
 4. 查找算法
 5. B+树和二叉树查找时间复杂度
 6. KMP算法、hash算法
 7. 常用的hash算法有哪些
 8. 如何判断一个单链表是否有环？
 9. 给你一万个数，如何找出里面所有重复的数？用所有你能想到的方法，时间复杂度和空间复杂度分别是多少？
 10. 给你一个数组，如何里面找到和为K的两个数？
 11. 100000个数找出最小或最大的10个？
 12. 一堆数字里面继续去重，要怎么处理？

##  数据结构：

 1. 队列、栈、链表、树、堆、图
 2. 编码实现队列、栈

##  Linux:

 1. linux常用命令
 2. 如何查看内存使用情况
 3. Linux下如何进行进程调度

##  操作系统：

 1. 操作系统什么情况下会死锁
 2. 产生死锁的必要条件
 3. 死锁预防

##  数据库：

 1. 范式
 2. 数据库事务隔离级别
 3. 数据库连接池的原理
 4. 乐观锁和悲观锁
 5. 如何实现不同数据库的数据查询分页
 6. SQL注入的原理，如何预防
 7. 数据库索引的实现(B+树介绍、和B树、R树区别)
 8. SQL性能优化
 9. 数据库索引的优缺点以及什么时候数据库索引失效
 10. Redis的存储结构

##  网络：

 1. OSI七层模型以及TCP/IP四层模型
 2. HTTP和HTTPS区别
 3. HTTP报文内容
 4. get提交和post提交的区别
 5. get提交是否有字节限制，如果有是在哪限制的
 6. TCP的三次握手和四次挥手
 7. session和cookie的区别
 8. HTTP请求中Session实现原理
 9. redirect与forward区别
 10. DNS
 11. TCP和UDP区别

##  安全：

 1. 如果客户端不断的发送请求连接会怎样
 2. DDos攻击
 3. DDos预防
 4. 那怎么知道连接是恶意的呢？可能是正常连接

##  其它：

 1. 说一个你参与的项目、其中作为什么角色
 2. 遇到最困的问题是什么，怎么解决的
 3. 你认为自己有那些方面不足
 4. 平常如何学习的
 5. 如何评价自己

##  智力题：

 1. 给你50个红球和50个黑球，有两个一模一样的桶，往桶里放球，让朋友去随机抽，采用什么策略可以让朋友抽到红球的概率更高？
 2. 从100个硬币中找出最轻的那个假币？

     **以上这些考察的知识点，在强大的互联网上都可以搜索到答案，有些答案可能不是很全，所以需要自己去总结，但是对于一些需要知道原理的知识点，还是推荐看源代码或者对于的书，然后总结得到自己的东西，这样既学到真东西，还不会很容易忘。Java基础的知识点推荐《Java编程思想》，JVM的推荐《深入理解Java虚拟机》，Spring原理的推荐《Spring源码深度解析》，对于网站架构的推荐《大型网站技术架构核心原理与案例分析》。欢迎关注Java技术分享微信公众号：JavaQ，获取更多精彩技术分享。**


  [1]: http://blog.csdn.net/jackfrued/article/details/44921941
  [2]: http://www.jianshu.com/p/05f42258850b
  [3]: http://www.importnew.com/10980.html
  [4]: http://lavasoft.blog.51cto.com/62575/15433
  [5]: http://wangkuiwu.github.io/2012/03/04/reflection/
  [6]: http://www.cnblogs.com/dolphin0520/p/3736238.html
  [7]: http://dotdotcloud.cn/2016/06/20/java%E7%9A%84%E5%9B%9B%E4%B8%AA%E7%89%B9%E6%80%A7%EF%BC%88%E6%8A%BD%E8%B1%A1%EF%BC%8C%E5%B0%81%E8%A3%85%EF%BC%8C%E7%BB%A7%E6%89%BF%EF%BC%8C%E5%A4%9A%E6%80%81%EF%BC%89%EF%BC%8C%E5%AF%B9%E5%A4%9A%E6%80%81%E4%BD%BF%E7%94%A8%E6%96%B9%E5%BC%8F%E7%9A%84%E7%90%86%E8%A7%A3/
  [8]: http://zjf201172653.iteye.com/blog/1921945
  [9]: https://segmentfault.com/a/1190000003002786
  [10]: http://www.docjar.com/html/api/java/lang/String.java.html
  [11]: https://www.zhihu.com/question/31345592
  [12]: http://www.cnblogs.com/xudong-bupt/p/3961159.html
  [13]: https://segmentfault.com/a/1190000004261063
  [14]: http://blog.csdn.net/afgasdg/article/details/6889383
  [15]: http://blog.csdn.net/whuslei/article/details/6686612
  [16]: http://blog.csdn.net/fenglibing/article/details/2745123
  [17]: http://www.cnblogs.com/danne823/archive/2011/04/22/2025332.html
