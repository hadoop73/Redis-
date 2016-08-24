---
title: JDK  源码
tags: 
grammar_cjkRuby: true
---


##  String


##  ArrayList
[Java 集合系列03之 ArrayList详细介绍(源码解析)和使用示例][1]

ArrayList 是一个数组队列，相当于 动态数组。与Java中的数组相比，它的容量能动态增长。它继承于AbstractList，实现了List, RandomAccess, Cloneable, java.io.Serializable这些接口。

ArrayList 继承了AbstractList，实现了List。它是一个数组队列，提供了相关的添加、删除、修改、遍历等功能。

ArrayList 实现了RandmoAccess接口，即提供了随机访问功能。RandmoAccess是java中用来被List实现，为List提供快速访问功能的。在ArrayList中，我们即可以通过元素的序号快速获取元素对象；这就是快速随机访问。稍后，我们会比较List的“快速随机访问”和“通过Iterator迭代器访问”的效率。

ArrayList 实现了Cloneable接口，即覆盖了函数clone()，能被克隆。

ArrayList 实现java.io.Serializable接口，这意味着ArrayList支持序列化，能通过序列化去传输。

和Vector不同，ArrayList中的操作不是线程安全的！所以，建议在单线程中才使用ArrayList，而在多线程中可以选择Vector或者CopyOnWriteArrayList。

![enter description here][2]

ArrayList包含了两个重要的对象：elementData 和 size。

- elementData 是"Object[]类型的数组"，它保存了添加到ArrayList中的元素。实际上，elementData是个动态数组，我们能通过构造函数 ArrayList(int initialCapacity)来执行它的初始容量为initialCapacity；如果通过不含参数的构造函数ArrayList()来创建ArrayList，则elementData的容量默认是10。elementData数组的大小会根据ArrayList容量的增长而动态的增长，具体的增长方式，ensureCapacity()函数。

- size 则是动态数组的实际大小。

**toArray() 异常**
当我们调用 ArrayList 中的 toArray()，可能遇到过抛出 “java.lang.ClassCastException” 异常的情况.

```java
Object[] toArray()
<T> T[] toArray(T[] contents)
```
```java
    // 返回ArrayList的Object数组
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    // 返回ArrayList的模板数组。所谓模板数组，即可以将T设为任意的数据类型
    public <T> T[] toArray(T[] a) {
        // 若数组a的大小 < ArrayList的元素个数；
        // 则新建一个T[]数组，数组大小是“ArrayList的元素个数”，并将“ArrayList”全部拷贝到新数组中
        if (a.length < size)
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());

        // 若数组a的大小 >= ArrayList的元素个数；
        // 则将ArrayList的全部元素都拷贝到数组a中。
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }
```



toArray() 会抛出异常是因为 toArray() 返回的是 Object[] 数组，将 Object[] 转换为其它类型(如如，将Object[]转换为的Integer[])则会抛出“java.lang.ClassCastException”异常，因为Java不支持向下转型。具体的可以参考前面ArrayList.java的源码介绍部分的toArray()。 解决该问题的办法是调用 T[] toArray(T[] contents) ， 而不是 Object[] toArray()。

```java
public static Integer[] vectorToArray2(ArrayList v) {
    Integer[] newText = (Integer[])v.toArray(new Integer[0]);
    return newText;
}
```

##  ConcurrentHashMap

[Java多线程系列--“JUC集合”04之 ConcurrentHashMap][3]

ConcurrentHashMap 是线程安全的哈希表，它是通过“锁分段”来实现的。ConcurrentHashMap 中包括了 “Segment(锁分段)数组”，每个 Segment 就是一个哈希表，而且也是可重入的互斥锁。

第一，Segment 是哈希表表现在，Segment 包含了 “HashEntry数组”，而 “HashEntry数组”中的每一个HashEntry元素是一个单向链表。即Segment是通过链式哈希表。

第二，Segment 是可重入的互斥锁表现在，Segment 继承于 ReentrantLock，而 ReentrantLock 就是可重入的互斥锁。

对于 ConcurrentHashMap 的添加，删除操作，在操作开始前，线程都会获取 Segment 的互斥锁；操作完毕之后，才会释放。而对于读取操作，它是通过 volatile 去实现的，HashEntry 数组是 volatile 类型的，而 volatile 能保证“即对一个 volatile 变量的读，总是能看到（任意线程）对这个 volatile 变量最后的写入”，即我们总能读到其它线程写入 HashEntry 之后的值。 以上这些方式，就是 ConcurrentHashMap 线程安全的实现原理。


##  HashMap


  [1]: http://wangkuiwu.github.io/2012/02/03/collection-03-arraylist/
  [2]: ./images/1471961520881.jpg "1471961520881.jpg"
  [3]: http://wangkuiwu.github.io/2012/08/14/juc-col04/
