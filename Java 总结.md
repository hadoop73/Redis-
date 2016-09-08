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

###  Class
类也是对象,是 Class 类的实例对象,只有JVM能够创建
这个对象称为类类型

```java
Class c1 = MyClass.class;
Class c2 = myClass.getClass();
Class c3 = Class.forName("...")
```
**通过类类型创建实例**
```java
// 无参数构造方法
MyClass myClass = (MyClass)c1.newInstance();
```
编译时加载类是静态加载类,运行时加载类是动态加载类
* new 创建对象是静态加载类,编译时加载所有类
* Class.forName("...")

**操作**

```java

Class c = myClass.getClass();
// 获得所有public函数,包括父类继承而来
// getDeclareMethods() 获取所有自己声明的方法,不管权限
Method[] ms = c.getMethods();
// 得到方法名称
ms[0].getName();
ms[0].getReturnType(); // 返回类型
// 获取参数类型的类类型
Class[] paramTypes = ms[0].getParameterTypes();
// 获取类的所有信息
Integer n = 1;
ClassUtil.printClassMessage(n);


// 成员变量也是对象
Field[] fs = c.getFields(); // 所有public的成员变量信息
Field[] f = c.getDeclareFields(); // 所有自己声明成员变量信息
// 成员变量的类类型
Class fType = f[0].getType();
String typeName = fType.getName();
// 成员变量名称
String fieldName = f[0].getName();


// 构造函数也是对象
// getConstructors 获得所有 public 构造函数
// getDeclareConstructors 获得所有构造函数
COnstructor[] cs = c.getConstructors();
```

**方法的反射**
* 方法的名称和方法的参数类型列表获取方法
* method.invoke(obj,parameters) 调用


###  泛型
Java 中集合的泛型,防止错误输入,只在编译阶段有效



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

[ Java中的构造方法总结][9]

**构造器不能被 Override**
* 构造器可以有任何访问的修饰符，public、private、protected或者没有修饰符，都可以对构造方法进行修饰
* 没有返回类型
* 如果类没有自己的构造器,都会有一个默认构造方法
* this和super:this指向当前的实例对象,在构造方法中使用时,指向其他构造器且必须放第一行,static 方法不可以使用 this 对象.super指向父类对象,也可以做构造器使用,必须在第一行
* 类成员变量先于构造器初始化,static 先于普通变量,且只运行一次,如果主类中含有静态变量会在main()方法执行前初始化.
* 由此总结初始化顺序：
	* 1．父类的静态成员
	* 2．子类的静态成员
	* 3．父类的非静态成员
	* 4．父类的默认构造函数被调用。 
	* 5．子类的非静态对象（变量） 
	* 6．子类的构造函数。 

6. 访问控制符public,protected,private,以及默认的区别

|  修饰符  | 当前类   | 同包    | 子类    |其他包     |
| :---: | :---: | :---: | :---: | :---: |
|   public  |  √   |  √   |  √   |  √   |
|  protected   |  √   | √    | √    | ×    |
|   default  |  √   |  √   |  ×   |   ×  |
| private|√|×|×|×|

1. 是否可以继承String类
 
 [Java String 源码浅析][10]
 
 [String 实现][11]
 
 [在java中String类为什么要设计成final][12]
 
`String` 是 final 修饰不能够继承
```java
public final class String{
	private final char value[];
	...
}
```
为了**安全**不可变,在 `HashSet` 的 `Key` 里面,变动就出问题了

2. String和StringBuffer、StringBuilder的区别

[java中String、StringBuffer、StringBuilder的区别][13]

**可变性**

`String` 不可变,`StringBuffer` 和 `StringBuilder` 可变

**线程安全性**
`String` 和 `StingBuffer` 是线程安全的

**原理**

[Java StringBuilder和StringBuffer源码分析][14]

`StringBuffer` 和 `StringBuilder` 都是调用抽象父类 `AbstractStringBuilder` 的实现而已,不同之处在于 `StringBuffer` 加了同步关键词 `synchronized



3. hashCode和equals方法的关系

[hashCode与equals的区别与联系][15]

* `equals` 方法用于比较对象
* `hashcode` 用于集合中
* 将对象放入到集合中时，首先判断要放入对象的hashcode值与集合中的任意一个元素的hashcode值是否相等，如果不相等直接将该对象放入集合中。如果hashcode值相等，然后再通过equals方法判断要放入对象与集合中的任意一个对象是否相等，如果equals判断不相等，直接将该元素放入到集合中，否则不放入。

[ java中的hashCode()和equals()的关系][16]

* 重写 `equals`,判断 `HashSet` 中元素不会重复
* 重写 `hashCode` 提高 `HashMap` 比对性能

**`==` 和 `equals` 的区别**

* == 判断是否为同一个对象,比对内存空间地址
* `equals` 被覆写后,由程序控制,`String` 类比对的是内容是否相同

4. 抽象类和接口的区别

[接口和抽象类有什么区别][17]

* 抽象类是对根源的抽象,接口是对动作的抽象,抽象一组动作,不同的类不同的实现
* 接口中方法都是抽象的,不能实现,抽象类可以
* 接口中可以由 `static` 类型数据,抽象类没有

5. 自动装箱与拆箱

[Java 自动装箱与拆箱(Autoboxing and unboxing)][18]

* 自动装箱是由值构建对象,构建对象是为了使用类提供的方法
* 拆箱是由对象返回值,能够很好的运算



 1. 什么是泛型、为什么要使用以及泛型擦除

[Java泛型的好处][19]
* 类型安全:放入数据的时候,会进行类型检查,确保放入可接受的类型
* 消除强制类型转化:取数据不需要类型转换,避免出错
* 提高代码复用

泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数.

[java泛型（二）、泛型的内部原理：类型擦除以及类型擦除带来的问题][20]

**类型擦除**
在生成的Java字节码中是不包含泛型中的类型信息的。使用泛型的时候加上的类型参数，会在编译器在编译的时候去掉。这个过程就称为类型擦除。

原始类型（raw type）就是擦除去了泛型信息，最后在字节码中的类型变量的真正类型。无论何时定义一个泛型类型，相应的原始类型都会被自动地提供。类型变量被擦除（crased），并使用其限定类型（无限定的变量用Object）替换。

**类型检查**
类型检查只针对引用

 2. Java中的集合类及关系图

[Java 集合类图(转)][21]

[Java 集合总结（Collection系列与Map系列）][22]

Java 结合类的两个接口:Collection 和 Map,它们包含了一些其他接口(Iterator接口)

![enter description here][23]

![enter description here][24]

Set,List 和 Map用途
* List 集合是有序集合,元素可以重复
* Set 集合无序集合,不能又重复元素
* Map 集合保存　Key-Value 键值对,通过 Key 来访问 Value

**List**

[Java 集合：Collection，List，ArrayList，Vector，LinkedList（实现方式，对比）][25]

* ArrayList 基于数组非线程安全.添加元素会先进行容量越界判定,必要时扩容

```java
elementData = Arrays.copyOf(elementData, newCapacity);
```
* Vector 线程安全,也是基于数组实现,Collections 这个类当中为我们提供的 synchronizedList(List list)，它可以返回一个线程安全的同步的列表，还提供了返回同步的 Collections
* LinkedList 基于链表实现,非线程安全.

**Set**

[Java集合总结汇总(链接)][26]

Set 中元素实现了一个有效的 equals(Object) 方法
HashSet 由哈希表支持,但不是同步的,可以通过以下语句实现同步转换
```java
Set s = Collections.synchronizedSet(new HashSet(...));
```

**Map**

[Java 集合总结（Collection系列与Map系列）][27]

![enter description here][28]

* HashMap 非同步,不保证顺序,允许空值和空键
* LinkedHashMap:继承自HashMap,Iterator 保证先后顺序
* TreeMap:能够根据 Key 值有序插入,使用了红黑树
* HashTable:HashMap 线程安全版本,同步,效率低
* ConcurrentHashMap:也是 HashMap 的线程安全版本,使用了分段加锁机制,效率比 HashTable 高.



3. HashMap实现原理(看源代码)

4. HashTable实现原理(看源代码)
5. HashMap和HashTable区别
6. HashTable如何实现线程安全(看源代码)
7. ArrayList和vector区别(看源代码)

[Java 集合系列03之 ArrayList详细介绍(源码解析)和使用示例][29]

ArrayList包含了两个重要的对象：elementData 和 size。

(01) elementData 是"Object[]类型的数组"，它保存了添加到ArrayList中的元素。实际上，elementData是个动态数组，我们能通过构造函数 ArrayList(int initialCapacity)来执行它的初始容量为initialCapacity；如果通过不含参数的构造函数ArrayList()来创建ArrayList，则elementData的容量默认是10。elementData数组的大小会根据ArrayList容量的增长而动态的增长，具体的增长方式，请参考源码分析中的ensureCapacity()函数。

(02) size 则是动态数组的实际大小。

ArrayList 是动态数组,容量动态增长

Vector 静态数组,线程安全的

8. ArrayList和LinkedList区别及使用场景
9. Collection和Collections的区别
10. Concurrenthashmap实现原理(看源代码)
11. Error、Exception区别

[Java的Exception和Error面试题10问10答][30]

Error 和 Exception 都继承自 Throwable,不同处如下:


**Error**

* 总是不可控
* 经常用来表示系统错误或底层资源错误
* 应该在系统级被捕捉

**Exception**

*  可以是可控(checked)或不可控的(unchecked)
*  表示一个由程序员导致的错误
*  应该在应用程序级被处理


12. Unchecked Exception和Checked Exception，各列举几个

[Throwable、Error、Exception、RuntimeException 区别 联系][31]

* Checked exception: 这类异常都是Exception的子类 。异常的向上抛出机制进行处理，假如子类可能产生A异常，那么在父类中也必须throws A异常。可能导致的问题：代码效率低，耦合度过高。
　　
* Unchecked exception: 这类异常都是RuntimeException的子类，虽然RuntimeException同样也是Exception的子类，但是它们是非凡的，它们不能通过client code来试图解决，所以称为Unchecked exception 。


13. Java中如何实现代理机制(JDK、CGLIB)
14. 多线程的实现方式

Runnable 方式可以避免 Thread 方式由于 Java 单继承特性带来的缺陷

Runnable 可以被多个线程共享,适合于多个线程处理同一资源的情况

15. 线程的状态转换
16. 如何停止一个线程
17. 什么是线程安全
当多个线程访问某个类时,这个类始终都能够表现出正确的行为,就称这个类是线程安全的
线程安全是编程中的术语，指某个函数、函数库在多线程环境中被调用时，能够正确地处理多个线程之间的共享变量，使程序功能正确完成。

 18. 如何保证线程安全
 19. Synchronized如何使用

[Java 多线程：synchronized 关键字用法（修饰类，方法，静态方法，代码块）][32]

* 修饰一个类，其作用的范围是synchronized后面括号括起来的部分，作用的对象是这个类的所有对象。
* 修饰一个方法，被修饰的方法称为同步方法，其作用的范围是整个方法，作用的对象是调用这个方法的对象； 
* 修改一个静态的方法，其作用的范围是整个静态方法，作用的对象是这个类的所有对象； 
* 修饰一个代码块，被修饰的代码块称为同步语句块，其作用的范围是大括号{}括起来的代码，作用的对象是调用这个代码块的对象；


 20. synchronized和Lock的区别

[Java 多线程：Lock接口（接口方法分析，ReentrantLock，ReadWriteLock）][33]

[synchronized与lock的区别][34]

* synchronize:效率低,使用更简单
* Lock:更加细粒度,复杂,适合 synchronize 场景,不会自动释放锁,获取锁的过程可以被中断 interrupt()


 21. 多线程如何进行信息交互
 22. sleep和wait的区别(考察的方向是是否会释放锁)
 23. 多线程与死锁

[死锁][35]

 24. 如何才能产生死锁

加锁机制能确保线程安全,但过度使用加锁,可能导致锁顺序死锁;我们使用线程池和信号量来限制资源的使用,可能会导致资源死锁

 25. 什么叫守护线程，用什么方法实现守护线程

[Java中守护线程的总结][36]

JVM 停止,守护线程会依旧运行,如果守护线程的资源没清理将会泄露,如垃圾回收器以及其他辅助工作的线程. 

```java
Thread daemonTread = new Thread();  
   
//设定 daemonThread 为守护线程，default false(非守护线程)  
 daemonThread.setDaemon(true);  
```

 26. Java线程池技术及原理

Executor 基于生产者-消费者模式,提交任务的操作相当于生产者,执行任务的线程则相当于消费者.

```java
public class WithinThreadExecutor implements Executor{
    public void execute(Runnable r){
		r.run(); // 创建了任务,调用 execute(r)	
    }
}
```
在 Executor 框架中,已提交但尚未开始的任务可以取消,对已经开始执行的任务,只有当它们能响应中断时,才能取消

Runnable 和 Callable 描述的都是抽象的计算任务.Future 表示一个任务的生命周期,提供了响应的方法来判断是否已经完成或取消,ExecutorService 中的所有 submit 方法都将返回一个 Future,由此可以获得任务的执行结果或者取消任务.

CompletionService 将 Executor 和 BlockingQueue 的功能融合在一起,将 Callable 任务提交给它执行,使用类似队列操作的 take 和 poll 获得已经完成的封装成 Future 的结果.
```java
CompletionService com = new ExecutorCompletionService(executor);
Future f = com.take();
ImageData imageData = f.get();
```



 27. java并发包concurrent及常用的类
 28. volatile关键字

volatile 变量不会被缓存在寄存器或其他处理器看不见的地方,因此读取 volatile 类型的变量总会返回最新写入值.通常用作某个操作完成,发生中断或者状态的标志.满足下面条件才使用:
* 对变量的写入操作不依赖变量的当前值
* 变量不会与其他状态变量一起纳入不变性条件
* 访问变量时不需要加锁

29. Java中的NIO，BIO，AIO分别是什么

[聊聊阻塞与非阻塞、同步与异步、I/O模型][37]

[怎样理解阻塞非阻塞与同步异步的区别？][38]


30. IO和NIO区别

[NIO 入门][39]

31. 序列化与反序列化

[深入分析Java的序列化与反序列化][40]

* 序列化： 将数据结构或对象转换成二进制串的过程。
* 反序列化：将在序列化过程中所生成的二进制串转换成数据结构或者对象的过程。

使用 Java 对象序列化,在保存对象时,会把状态保存为一组字节,保存的是对象的**状态,对象序列化不会关注类中的静态变量**.

* 在Java中，只要一个类实现了 `java.io.Serializable` 接口，那么它就可以被序列化。

* 通过 `ObjectOutputStream` 和 `ObjectInputStream` 对对象进行序列化及反序列化

ArrayList 自己实现了 `readObject` 和 `writeObject`,自定义了序列化和反序列化过程.

 32. 常见的序列化协议有哪些

[序列化和反序列化][41]

* XML:跨机器,跨语言.缺点:冗长复杂
* SOAP:基于XML为序列化和反序列化协议的结构化消息传递协议.
* JSON:可读性强,协议简单,解析速度快.序列化和反序列化需要采用反射机制,性能要求为ms级别,不建议使用.
* Thrift:Facebook 开源轻量级RPC服务框架,由于Thrift的序列化被嵌入到Thrift框架里面,Thrift框架本身没有序列化和反序列化接口,很难和其他传输层协议共同使用.
* Protobuf:Google 序列化协议
	* 序列化数据简介,紧凑,与XML相比,数据量约为1/3到1/10
	* 解析速度非常好,比对应的XML快约20-100倍
	* 使用非常简介,反序列化只需要一行代码

* Avro:Apache Hadoop 一个子项目,提供两种序列化格式:JSON格式或Binary格式.Binary格式在空间开销和解析性能方面可以和Protobuf媲美


 33. 内存溢出和内存泄漏的区别

[内存溢出和内存泄漏的区别][42]

* 内存溢出:使用的内存超出系统提供的容量,出现内存溢出
* 内存泄露:申请后,无法正常释放,因此内存得不到正常回收使用,出现泄露


34. Java内存模型及各个区域的OOM，如何重现OOM



35. 出现OOM如何解决

[Java 内存溢出（java.lang.OutOfMemoryError）的常见情况和处理方式总结][43]

36. 用什么工具可以查出内存泄漏

[Java的内存泄漏][44]

[enter description here][45]

37. Java内存管理及回收算法
 


38. Java类加载器及如何加载类(双亲委派)

[深入理解和探究Java类加载机制----][46]

根据一个指定的名称,找到或者生成对应的字节码,形成被虚拟机使用的Java类型


 39. xml解析方式

[四种生成和解析XML文档的方法详解][47]

[Java解析XML的四种方法][48]

**DOM/SAX/JDOM/DOM4J**

* DOM
	* 允许应用程序对数据和结构做出修改
	* 可以在任何时候在树中获取数据
	* 加载整个XML文档来构造层次结构,消耗资源大
* SAX
	* 不需要等待所有数据都被处理,分析就能立即开始
	* 不需要保存在内存中
	* 很难同时访问同一个文档的不同部分数据
* JDOM
	* 使用具体类而不是接口
	* 大量使用了Java集合类
	* 没有较好的灵活性,性能差

* DOM4J
	* 大量使用Java集合类
	* 支持XPath,性能好
	* 大量使用接口,API复杂

[Dom4j解析XML学习代码][49]

```java
/*建立document对象*/
Document document = DocumentHelper.createDocument();
/*建立XML文档的根books*/
Element booksElement = document.addElement("books");
/*加入一行注释*/
booksElement.addComment("This is a test for dom4j, ZHe, 2012.11.26");
/*加入第一个book节点*/
Element bookElement = booksElement.addElement("book");
/*加入show属性内容*/
bookElement.addAttribute("show", "yes");
/*加入title节点*/
Element titleElement = bookElement.addElement("title");
/*为title设置内容*/
titleElement.setText("Dom4j Tutorials");
```


 40. Statement和PreparedStatement之间的区别

[Java笔记：Statement和PreparedStatement的区别][50]

[【转】PreparedStatement和Statement区别][51]

数据库会对 PreparedStatement 数据库进行预编译,下次相同的 sql 语句时,数据库端不会再进行预编译,而直接用数据库的缓冲区(使用了?),提高数据访问的效率

 41. 动态代理

[几种动态代理方法][52]

[Java 动态代理作用是什么？][53]

通过使用 `Proxy.newProxyInstance()` 创建动态代理,所需三个参数:
* 类加载器,用来生成一个动态类,可以用生成的动态类获得实例
* 接口数组,被代理类实现的接口
* InvocationHandler 实例,所有的方法调用都会转到这个接口实现的 invoke() 方法

```java
InvocationHandler handler = new MyInvocationHandler();
MyInterface proxy = (MyInterface) Proxy.newProxyInstance(
                            MyInterface.class.getClassLoader(),
                            new Class[] { MyInterface.class },
                            handler);

public interface InvocationHandler{
  Object invoke(Object proxy, Method method, Object[] args)
         throws Throwable;
}
```

静态代理:如果类方法数量越来越多的时候，代理类的代码量是十分庞大的

动态代理的作用是什么：
* Proxy 类的代码量被固定下来，不会因为业务的逐渐庞大而庞大；
* 可以实现 AOP 编程，实际上静态代理也可以实现，总的来说，AOP 可以算作是代理模式的一个典型应用；
* 解耦，通过参数就可以判断真实类，不需要事先实例化，更加灵活多变。

 42. RPC

[为什么需要RPC，而不是简单的HTTP接口][54]

[ 深入浅出 RPC - 浅出篇][55]

[深入浅出 RPC - 深入篇][56]

远程过程调用属于长连接

 43. 枚举

[Java 枚举会比静态常量更消耗内存吗？][57]

枚举的实现原理,就是定义了一个类,实例化final修饰的元素,每个实例都有自己的元信息.比自己定义的常量耗内存,但是枚举可读性,扩展性更好

 44. CountDownLatch

[Java CountDownLatch应用][58]

原子操作的计数器,如果一个线程调用 CountDownLatch 实例 await() 方法,则必须等到实例通过 countDown() 方法减一,计数为0才能继续执行

 45. logback

[logback 常用配置详解（序）logback 简介][59]




46. 数据库连接池

[Java 连接池的工作原理][60]

创建一个连接,需要:
* 检查注册驱动程序
* 创建Socket连接

[各种数据库连接池对比][61]

![enter description here][62]

[数据库连接池C3P0学习][63]

[【性能】JDBC PreparedStatement和连接池PreparedStatement Cache学习记录][64]


47. ThreadLocal

[深入浅出 ThreadLocal][65]

ThreadLocal 为变量在每个线程存一个副本,主要用于非线程安全,但是避免 synchronized 的对象

内部是是一个 ThreadLocalMap,通过哈希的Map.

48. 













## JavaEE:

1. servlet生命周期及各个方法

[Servlet生命周期与工作原理][66]

Servlet 生命周期分为三个阶段:

* 初始化阶段  调用init()方法

* 响应客户请求阶段　　调用service()方法

* 终止阶段　　调用destroy()方法

Servlet 容器启动时自动装载 Servlet,创建一个 Servlet 实例并且调用 Servlet 的 init() 方法进行初始化.

客户发送一个请求,Servlet 调用 service() 方法对请求进行响应,再根据请求的方式进行匹配,选择调用 doGet,doPost等方法.

2. servlet中如何自定义filter

[Servlet中的Filter过滤器的介绍和使用][67]


过滤器是一个程序，它先于与之相关的servlet或JSP页面运行在服务器上。它能够对Servlet容器的请求和响应对象进行检查和修改。

* Servlet过滤器本身并不生成请求和响应对象，只是提供过滤功能。

* Servlet过滤器能够在Servlet被调用之前检查Request对象，并修改Request Header和Request内容；

* 在Servlet被调用之后检查Response对象，修改Response Header和Response的内容。

* Servlet过滤器可以过滤的Web组件包括Servlet，JSP和HTML等文件。

init():Servlet 容器创建Servlet过滤器实例后将调用该方法,读取web.xml文件中Servlet过滤器的初始化参数

doFilter():完成过滤功能

destroy():Servlet容器销毁过滤器实例前调用该方法,释放Servlet过滤器占用的资源.

```xml

      LoginFilter
      com.itzhai.login.LoginFilter
      
          username
          arthinking
      
  
```

```java
@Override
public void init(FilterConfig filterConfig) throws ServletException {
    //获取Filter初始化参数
    String username = filterConfig.getInitParameter("username");
}
````

**过滤敏感词汇**

```java
@Override
public void doFilter(ServletRequest request, ServletResponse response,
        FilterChain chain) throws IOException, ServletException {
        //转换成实例的请求和响应对象
        HttpServletRequest req = (HttpServletRequest)request;
        HttpServletResponse resp = (HttpServletResponse)response;
        //获取评论并屏蔽关键字
        String comment = req.getParameter("comment");
        comment = comment.replace("A", "***");
        //重新设置参数
        req.setAttribute("comment", comment);
        // 继续执行
        chain.doFilter(request,response);
 }
```

`Filter` 的执行顺序与在 `web.xml` 配置文件中的配置顺序一致,一般把 `Filter` 配置在所有的 `Servlet` 之前.


### JSP原理

 1. JSP和Servlet的区别

[Jsp 和 Servlet 的区别][68]

* Servlet在Java代码中通过HttpServletResponse对象动态输出HTML内容
* JSP在静态HTML内容中嵌入Java代码，Java代码被动态执行后生成HTML内容


 2. JSP的动态include和静态include

[JSP动态包含与静态包含][69]

动态INCLUDE用jsp:include动作实现它总是会检查所含文件中的变化，适合用于包含动态页面，并且可以带参数。静态INCLUDE用include伪码实现,定不会检查所含文件的变化，适用于包含静态页面

因为执行的是 class,所以静态 include在编译为class的时候加入一起编译,动态include是分别编译为class,执行的时候动态引入

 3. Struts中请求处理过程
 4. JSP 页面中文乱码

[JSP 中文乱码][70]



### MVC概念

 1. Spring mvc与Struts区别
 2. Hibernate/Ibatis两者的区别
 3. Hibernate一级和二级缓存
 4. Hibernate实现集群部署
 5. Hibernate如何实现声明式事务
 6. 简述Hibernate常见优化策略
 7. Spring bean的加载过程(推荐看Spring的源码)
 8. Spring如何实现AOP和IOC

[AOP 那点事儿][71]

[Spring AOP 实现原理与 CGLIB 应用][72]

 
9. Spring bean注入方式
 10. Spring的事务管理(推荐看Spring的源码)
 11. Spring事务的传播特性
 12. springmvc原理
 13. springmvc用过哪些注解

[Java 注解][73]

**注解**是插入你代码中的一种注释或者说是一种元数据（meta data）。这些注解信息可以在编译期使用预编译工具进行处理（pre-compiler tools），也可以在运行期使用 Java 反射机制进行处理。


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

##  分布式

1. zookeeper

[Zookeeper与paxos算法][74]

[ZooKeeper编程(一)][75]

[ZooKeeper伪分布式集群安装及使用][76]

[ZooKeeper学习第二期--ZooKeeper安装配置][77]

**zookeeper 功能**

zookeeper 提供了一个同步的文件系统和通知机制

**zookeeper 配置:**
* tickTime=2000
* initLimit=10
* synclimit=5
* dataDir=/home/hadoop/zoo
* clientPort=2016  
* server.1=192.168.1.109:2888:3888
* server.2=192.168.1.130:2888:3888

`clientPort` 端口,`server` 用于与 `client` 连接的端口号,`dataDir` 服务器的数据文件,`server.X` 和 `myid,server.X` 这个数字对应 `data/myid` 中的数字,3个 `server` 的 `myid` 文件分别写入 1,2,3,在 `zoo.cfg` 中配置 `server.1,server.2,server.X` 的 `ip:port:Xport,port` 用于 `server` 之间的连接,`Xport` 用于选举 `leader`.

2. 



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

[通信协议：HTTP、TCP、UDP][78]

[大型网站的 HTTPS 实践（1）：HTTPS 协议和原理][79]

[HTTP 协议简介][80]

HTTP:超文本传输协议,应用层协议,无状态协议

HTTP请求:请求头和请求体

HTTP响应:响应头和响应体

chunked 传输:每个chunk由两部分组成,第一部分chunk长度,第二部分chunk数据,中间部分用CRLF间隔

HTTPS:是HTTP协议和安全套接口层(SSL)的结合,使HTTP的协议数据在传输过程中更加安全

 3. HTTP报文内容

 4. get提交和post提交的区别

我们看看GET和POST的区别

1. GET提交的数据会放在URL之后，以?分割URL和传输数据，参数之间以&相连，如EditPosts.aspx?name=test1&id=123456.  POST方法是把提交的数据放在HTTP包的Body中.

2. GET提交的数据大小有限制（因为浏览器对URL的长度有限制），而POST方法提交的数据没有限制.

3. GET方式需要使用Request.QueryString来取得变量的值，而POST方式通过Request.Form来获取变量的值，也就是说Get是通过地址栏来传值，而Post是通过提交表单来传值。

4. GET方式提交数据，会带来安全问题，比如一个登录页面，通过GET方式提交数据时，用户名和密码将出现在URL上，如果页面可以被缓存或者其他人可以访问这台机器，就可以从历史记录获得该用户的账号和密码.

 5. get提交是否有字节限制，如果有是在哪限制的

[HTTP中的URL长度限制
][81]


该参数对nginx服务器接受客户端请求的头信息时所分配的最大缓冲区的大小做了限制，也就是nginx服务器一次接受一个客户端请求可就收的最大头信息大小。这个头不仅包含 request-line，还包括通用信息头、请求头域、响应头域的长度总和。这也相当程度的限制了url的长度。

 6. TCP的三次握手和四次挥手

 7. session和cookie的区别

 8. HTTP请求中Session实现原理

 9. redirect与forward区别

 10. DNS


 11. TCP和UDP区别


##  Maven

1. 模块划分

[Maven最佳实践：划分模块][82]

父模块需要设置 `<packaging>,<modules>`
子模块只需要设置 artifcatId

3. 



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
  [9]: http://blog.csdn.net/zmissm/article/details/14176725
  [10]: https://segmentfault.com/a/1190000003002786
  [11]: http://www.docjar.com/html/api/java/lang/String.java.html
  [12]: https://www.zhihu.com/question/31345592
  [13]: http://www.cnblogs.com/xudong-bupt/p/3961159.html
  [14]: https://segmentfault.com/a/1190000004261063
  [15]: http://blog.csdn.net/afgasdg/article/details/6889383
  [16]: http://blog.csdn.net/whuslei/article/details/6686612
  [17]: http://blog.csdn.net/fenglibing/article/details/2745123
  [18]: http://www.cnblogs.com/danne823/archive/2011/04/22/2025332.html
  [19]: http://blog.csdn.net/shanliangliuxing/article/details/7402684
  [20]: http://blog.csdn.net/lonelyroamer/article/details/7868820
  [21]: http://www.cnblogs.com/bob-fd/archive/2012/09/20/2695437.html
  [22]: https://github.com/pzxwhc/MineKnowContainer/issues/75
  [23]: ./images/1470379636414.jpg "1470379636414.jpg"
  [24]: ./images/1470379648314.jpg "1470379648314.jpg"
  [25]: https://www.wuhuachuan.com/visitor/learning/article/getArticleDetail?id=e2d86b1a-99fd-42c8-bf47-7de763aafd75
  [26]: http://www.cnblogs.com/Bob-FD/archive/2012/09/20/2695458.html
  [27]: https://github.com/pzxwhc/MineKnowContainer/issues/75
  [28]: ./images/1470381246803.jpg "1470381246803.jpg"
  [29]: http://wangkuiwu.github.io/2012/02/03/collection-03-arraylist/
  [30]: http://www.oschina.net/translate/10-java-exception-and-error-interview-questions-answers-programming
  [31]: http://blog.csdn.net/liuj2511981/article/details/8524418
  [32]: https://github.com/pzxwhc/MineKnowContainer/issues/7
  [33]: https://github.com/pzxwhc/MineKnowContainer/issues/16
  [34]: http://blog.lastww.com/2015/02/04/difference-between-java-lock-and-synchronized/
  [35]: http://ifeve.com/deadlock/
  [36]: http://blog.csdn.net/shimiso/article/details/8964414
  [37]: http://blog.jobbole.com/103290/
  [38]: https://www.zhihu.com/question/19732473
  [39]: http://www.ibm.com/developerworks/cn/education/java/j-nio/
  [40]: http://www.hollischuang.com/archives/1140
  [41]: http://www.infoq.com/cn/articles/serialization-and-deserialization
  [42]: http://blog.csdn.net/buutterfly/article/details/6617375
  [43]: http://outofmemory.cn/c/java-outOfMemoryError
  [44]: https://www.ibm.com/developerworks/cn/java/l-JavaMemoryLeak/
  [45]: http://www.cnblogs.com/xuxg/archive/2012/08/07/2627411.html
  [46]: http://www.cnblogs.com/sunniest/p/4574080.html
  [47]: http://www.cnblogs.com/lanxuezaipiao/archive/2013/05/17/3082949.html
  [48]: http://my.oschina.net/u/242764/blog/482685
  [49]: http://www.cnblogs.com/CheeseZH/archive/2012/11/28/2791914.html
  [50]: http://cnn237111.blog.51cto.com/2359144/1131869
  [51]: http://bliuqing.iteye.com/blog/374977
  [52]: http://blog.csdn.net/centre10/article/details/6847828
  [53]: https://www.zhihu.com/question/20794107
  [54]: http://www.oschina.net/question/271044_2155059
  [55]: http://blog.csdn.net/mindfloating/article/details/39473807
  [56]: http://blog.csdn.net/mindfloating/article/details/39474123
  [57]: https://www.zhihu.com/question/48707169
  [58]: http://zapldy.iteye.com/blog/746458
  [59]: http://aub.iteye.com/blog/1101222
  [60]: http://www.oschina.net/question/157182_72094
  [61]: https://github.com/alibaba/druid/wiki/%E5%90%84%E7%A7%8D%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E6%B1%A0%E5%AF%B9%E6%AF%94
  [62]: ./images/1472560646490.jpg "1472560646490.jpg"
  [63]: http://haoran-10.iteye.com/blog/1753332
  [64]: http://singleant.iteye.com/blog/1298837g
  [65]: http://blog.jobbole.com/104722/
  [66]: http://www.cnblogs.com/cuiliang/archive/2011/10/21/2220671.html
  [67]: http://www.itzhai.com/java-web-notes-servlet-filters-in-the-filter-writing-the-introduction-and-use-of-filters.html#read-more
  [68]: https://www.zhihu.com/question/37962386
  [69]: http://beijishiqidu.iteye.com/blog/1976142
  [70]: https://www.zhihu.com/question/20212696
  [71]: http://blog.jobbole.com/103213/
  [72]: http://blog.jobbole.com/28791/
  [73]: http://wiki.jikexueyuan.com/project/java-reflection/java-at.html
  [74]: http://blog.jobbole.com/45721/
  [75]: http://www.cnblogs.com/zhangchaoyang/articles/2536178.html%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00
  [76]: http://blog.fens.me/hadoop-zookeeper-intro/
  [77]: http://www.cnblogs.com/sunddenly/p/4018459.html
  [78]: http://blog.jobbole.com/84429/
  [79]: http://blog.jobbole.com/86660/
  [80]: http://blog.jobbole.com/104886/
  [81]: http://www.cnblogs.com/lengyuhong/archive/2012/02/04/2330130.html
  [82]: http://juvenshun.iteye.com/blog/305865
