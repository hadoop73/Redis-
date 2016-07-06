---
title: C++  总结 
tags: C++
grammar_cjkRuby: true
---

[TOC]



[C++ 总结][1]

## 可变参数

[C 语言的可变参数表函数的设计][2]

**参数的入栈**
参数的传递是通过压入栈来实现，栈是从内存的高地址向低地址生长，从左向右依次入栈，所有第一个参数最后被入栈

**可变参数实现**
需要使用 `<stdarg.h>`标准头文件里面的三个宏和一个数据类型
```c
typedef char *  va_list;

#define _INTSIZEOF(n)   ((sizeof(n) + sizeof(int) - 1) & ~(sizeof(int) - 1) )  
#define va_start(ap,v)  (ap = (va_list)&v + _INTSIZEOF(v) )   //得到可变参数中第一个参数的首地址  

#define va_arg(ap,type)  (*(type *)((ap += _INTSIZEOF(type)) - _INTSIZEOF(type)))    //将参数转换成需要的类型，并使ap指向下一个参数  
```


##  声明与定义
[声明和定义 (C++)][3]

[ 变量定义和声明的区别（整理）][4]

[C++中声明和定义的区别][5]

声明：引入名称，接下来的代码才能用，包括变量和函数
	* 函数
	* 包含 extern 说明符，但是没有初始化
	* 类中的数据成员
	* 类名声明，如 class T
	* typedef 语句
定义：编译器会分配内存空间


## sizeof 操作符
[【C++ 专题】C++ sizeof 使用规则及陷阱分析][6]


##  copy、copy_backward 和 memmove
[memmove][7]
**memmove 函数**
`void * memmove ( void * destination, const void * source, size_t num );`
运行目的地址和源地址存在重叠
```c
/* memmove example */
#include <stdio.h>
#include <string.h>
int main ()
{
  char str[] = "memmove can be very useful......";
  memmove (str+20,str+15,11);
  puts (str);
  return 0;
}
// output
// memmove can be very very useful.
```

[copy_backward][8]
**copy_backward 函数**
一般用于存在源地址和目的地址存在冲突，所以从后往前面拷贝
```cpp
template<class BidirectionalIterator1, class BidirectionalIterator2>
  BidirectionalIterator2 copy_backward ( BidirectionalIterator1 first,
                                         BidirectionalIterator1 last,
                                         BidirectionalIterator2 result )
{
  while (last!=first) *(--result) = *(--last);
  return result;
}
```

[copy][9]
**copy 函数**
不能用于目的地址和源地址存在冲突情况，需和 copy_backward 配合使用
```cpp
template<class InputIterator, class OutputIterator>
  OutputIterator copy (InputIterator first, InputIterator last, OutputIterator result)
{
  while (first!=last) {
    *result = *first;
    ++result; ++first;
  }
  return result;
}
```


##  public、private、protected 继承
[C++ 继承：公有，私有，保护][10]

* public（公有继承）：继承时保持基类中各成员属性不变，并且基类中private成员被隐藏。派生类的成员只能访问基类中的public/protected成员，而不能访问private成员；派生类的对象只能访问基类中的public成员。
* private（私有继承）：继承时基类中各成员属性均变为private，并且基类中private成员被隐藏。派生类的成员也只能访问基类中的public/protected成员，而不能访问private成员；派生类的对象不能访问基类中的任何的成员。 
* protected（保护性继承）：继承时基类中各成员属性均变为protected，并且基类中private成员被隐藏。派生类的成员只能访问基类中的public/protected成员，而不能访问private成员；派生类的对象不能访问基类中的任何的成员。


##  多态性
[多种多态性][11]

[浅谈 C++ 多态性][12]

多态性即 **“一个接口，多种方法”**，程序运行时才决定调用的函数。
1、静态多态性：指定义在一个类或函数中的同名函数，根据参数表区别语义，通过静态联编实现。
2、动态多态性：指定义在一个类层次的不同类中的重载函数，具有相同的函数，根据指针指向的对象所在的类来区别语义，通过动态联编实现。

**纯虚函数和虚函数**
纯虚函数一定没有定义，用来规范派生类的行为，即接口。包含纯虚函数的类是抽象类，抽象类不能定义实例，可以声明指向该抽象类的具体类的指针或引用。

虚函数必须声明，如果不实现，编译报错
* 对于虚函数来说，父类和子类都有各自的版本，由多态方式调用的时候动态绑定
* 实现了纯虚函数的子类，该纯虚函数在子类就变成了虚函数，子类的子类可以覆盖
* 虚函数是 `C++` 中实现多态的机制，核心理念就是通过基类访问派生类定义的函数。


##  动态绑定和静态绑定
[深入理解 C++ 的动态绑定和静态绑定][13]

**对象的静态类型：** 对象在声明时采用的类型，是在编译期确定的。
**对象的动态类型：** 目前所指对象类型。在运行期决定，动态类型可以改变，静态类型无法改变。

**静态绑定：** 绑定的是对象的静态类型，某特性(比如函数)依赖于对象的静态类型，发生在编译期。
**动态绑定：** 绑定的是对象的动态类型，某特性(比如函数)依赖于对象的动态类型，发生在运行期。


##  类型转换
* `const_cast` 用来将对象的常量性移除
* `dynamic_cast` 用来执行 “安全向下转型”



##  对象模型
[《深度探索C++对象模型》笔记汇总][14]

[C++对象模型][15]

`C++` 类包含两种数据成员：**静态数据成员** 和 **非静态数据成员**，三种成员函数：**成员函数**、**静态函数** 和 **虚函数**，共有三种对象模型：**简单对象模型**、**表格驱动对象模型** 和 **`C++` 对象模型**。


##  内存对齐的原则
[内存对齐的规则以及作用][16]

[内存对齐全攻略--涉及位域的内存对齐原则][17]


##  C++ 模板
[C++模板][18]

[C++模板学习][19]


##  `I/O` 多路复用 `select、poll、epoll` 的区别
[I/O多路复用select、poll、epoll的区别使用][20]

[`Redis I/O` 多路复用 ][21]

`I/O` 多路复用技术为了解决进程或线程阻塞到某个 `I/O` 系统调用而出现的技术，使进程不阻塞于某个特定 `I/O` 系统调用。


##  内联函数
[内联函数][22]

内联函数是指定义在类体内的成员函数
* 函数代码被放入符号表中，使用时直接进行替换，没有调用的开销，效率高
* 可以使用类的保护成员及私有成员
* 避免宏定义容易产生二义性问题(`#define ABS(x) ((x)>0?(x):(-x))` 会加2次)

**内联函数和宏的区别**
* 宏是由预处理器对宏进行替换，内联函数由编译期实现，取消了函数的参数压栈，减少了调用开销
* 内联函数遵循类型和作用域规则

可以使用 `inline` 来定义内联函数。

##  函数调用
[ 深入理解C语言的函数调用过程 ][23]

调用之前保存栈空间，里面是局部变量，还有保存下一个要执行的地址，再传递函数参数

## `extern C` 作用
[`extern C` 作用详解][24]

主要为了能够正确实现 `C++` 调用 C 语言代码，指示编译器这部分代码按照 C 语言进行编译。
在 C++ 出现之前，很多代码都是 C 语言写的，为了更好的支持原来的 C 代码和已经写好的C语言库，在 C++ 中尽可能支持 C。


##  `Reactor` 模式
[使用I/O多路复用技术--select实现Reactor模式][25]

[两种高性能I/O设计模式(Reactor/Proactor)的比较][26]



##  `malloc/free` 与 `new/delete` 的区别
[百度笔试题：malloc/free与new/delete的区别][27]

`new/delete` 是运算符，会执行分配空间构造函数，对象消亡时，析构函数和释放空间。
`malloc/free` 是标准库函数，只有分配空间和释放空间作用。


##  `strcpy` 函数的实现
[`strcpy` 函数的实现][28]

[`strcpy` 函数的实现][29]

返回 `char*` 是为了能够支持链式表达式

`dst` 和 `src` 重叠，借助 `memcpy` 函数


##  柔性数组
[C/C++ 中的0长数组（柔性数组）][30]

结构体中的 0 长度数组，能够构造变长度的结构体

数组名是一个符号，不占用空间，代表一个偏移量

##  编译和链接
[C/C++编译和链接过程详解][31]

[C编译器、链接器、加载器详解][32]

编译：把源码 `.c 或 .cpp` 编译成二进制文件 `.o`

链接(包括符合解析，重定位)：由于 `.o` 文件之间可能有相互引用(extern 关键字，函数调用)，所以有些地址需要补全


##  一致性哈希算法
[一致性hash算法 - consistent hashing][33]

[每天进步一点点——五分钟理解一致性哈希算法(consistent hashing)][34]

为了解决因特网中的热点问题，使哈希结果尽可能的分布到缓冲中去，并解决能够自由添加、删除节点

平衡：通过添加虚拟节点



##  同步异步
[网络编程释疑之：同步，异步，阻塞，非阻塞][35]

[怎样理解阻塞非阻塞与同步异步的区别][36]


阻塞，非阻塞：进程/线程访问数据是否就绪，进程/线程是否需要等待
同步，异步：关注的是 **消息通信机制**；
同步：在发出一个调用时，在没有得到结果之前，该调用不会返回，但是一旦调用返回，就得到返回值。
异步：调用在发出后，调用直接返回，所以没有返回结果；而是有被调用者通过状态、通知来通知调用者，或者通过回调函数处理这个调用。

##  接口和抽象类的区别
[面试题：接口和抽象类的区别][37]

抽象类是一类事务的高度聚合，对于继承抽象类的子类来说，属于"是"的关系；接口是定义行为规范，对于实现接口的子类来说，是"行为需要按照接口来完成"

抽象类在定义方法时，可以给出实现，也可以不给出；对于接口来说，不能给出实现

继承两者的子类，继承类对于抽象类所定义的抽象方法，可以实现也可以不实现；对于接口来说，继承子类必须实现

抽象类新增方法，继承类可以不用任何处理；对于接口来说，需要修改继承类，添加新方法的实现

##  单列模式
[深入浅出单实例Singleton设计模式][38]



## const 与 #define
* const 常量有数据类型，宏常量没有数据类型。编译器会对常量进行类型检查，避免错误

## 引用
* 引用是某个目标变量的别名，引用的操作和变量操作效果一样。
* 引用声明了，需要初始化，但不能把该引用再作为其他变量名的别名。
* 引用本身不占用存储单元，系统不会给引用分配存储单元
* 不能建立数组的引用

##  析构函数
析构函数提供一个在对象删除前可以释放这个对象所占有的资源的机会。
```cpp
class A
{
    A(){m_a=new int[10];}
    ~A(){delete [] m_a;}

    int * m_a;
}
```

##  数组与指针
字符数组能够修改，指针指向的常量字符串，不能修改
```cpp
char a[] = "hello";
a[0] = 'X';
char *p = "world"; // p 指向常量字符串
p[0] = 'X'; // 编译器不能发现错误，运行时错误
```

用运算符 sizeof 可以计算出数组的大小，sizeof(p) 得到的是一个指针变量的字节数；**当数组作为函数参数时，数组退化为同类型的指针。**

```cpp
char a[] ="hello world";
char*p = a;
cout<
cout<

void Func(char a[100])
{
　　cout<
}
```



##  管道，进程通信


##  多线程

[多线程][39]

在 Linux 中，创建进程的代价小，适合多进程
在 Windows 中，创建和销毁进程的代价要高，适合多线程

在现代体系中，一般 CPU 多核，适合多进程，每个 CPU 核心运行一个进程，每个进程独立，所以 CPU 核心之间的切换不需要考虑上下文

每个 CPU 运行一个多线程时，由于每个线程有共享资源，切换时，会存在复制数据，不如多进程

##  TCP/UDP


  [1]: http://www.cnblogs.com/fangyukuan/archive/2010/09/18/1829871.html
  [2]: http://blog.csdn.net/hackbuteer1/article/details/7558979#comments
  [3]: https://msdn.microsoft.com/zh-cn/library/0e5kx78b.aspx
  [4]: http://blog.csdn.net/feitianxuxue/article/details/7204116
  [5]: http://www.cnblogs.com/frankfang/archive/2011/05/02/2034393.html
  [6]: http://www.cnblogs.com/chio/archive/2007/06/11/778934.html
  [7]: http://www.cplusplus.com/reference/cstring/memmove/?kw=memmove
  [8]: http://www.cplusplus.com/reference/algorithm/copy_backward/?kw=copy_backward
  [9]: http://www.cplusplus.com/reference/algorithm/copy/
  [10]: http://www.cnblogs.com/qlwy/archive/2011/08/25/2153584.html
  [11]: http://blog.csdn.net/fengyunjh/article/details/6188769
  [12]: http://blog.csdn.net/hackbuteer1/article/details/7475622
  [13]: http://blog.csdn.net/chgaowei/article/details/6427731
  [14]: http://www.roading.org/develop/cpp/%E3%80%8A%E6%B7%B1%E5%BA%A6%E6%8E%A2%E7%B4%A2c%E5%AF%B9%E8%B1%A1%E6%A8%A1%E5%9E%8B%E3%80%8B%E7%AC%94%E8%AE%B0%E6%B1%87%E6%80%BB.html
  [15]: http://www.cnblogs.com/skynet/p/3343726.html
  [16]: http://www.cppblog.com/snailcong/archive/2009/03/16/76705.html
  [17]: http://www.cnblogs.com/shitouer/archive/2010/04/07/1706785.html
  [18]: http://www.cnblogs.com/gw811/archive/2012/10/25/2738929.html
  [19]: http://www.cnblogs.com/gaojun/archive/2010/09/10/1823354.html
  [20]: http://www.bkjia.com/ASPjc/1000957.html#top
  [21]: https://www.zhihu.com/question/28594409
  [22]: http://www.cnblogs.com/singa/archive/2008/09/24/1297821.html
  [23]: http://blog.chinaunix.net/uid-23069658-id-3981406.html
  [24]: http://blog.csdn.net/jiqiren007/article/details/5933599
  [25]: http://www.rudy-yuan.net/archives/137/
  [26]: http://blog.jobbole.com/59676/
  [27]: http://blog.csdn.net/hackbuteer1/article/details/6789164
  [28]: http://www.cnblogs.com/chenyg32/p/3739564.html
  [29]: http://blog.csdn.net/gpengtao/article/details/7464061/
  [30]: http://blog.csdn.net/yby4769250/article/details/7294696
  [31]: http://blog.csdn.net/yby4769250/article/details/7360483
  [32]: http://www.cnblogs.com/oubo/archive/2011/12/06/2394631.html
  [33]: http://blog.csdn.net/sparkliang/article/details/5279393
  [34]: http://blog.csdn.net/cywosp/article/details/23397179
  [35]: http://yaocoder.blog.51cto.com/2668309/1308899
  [36]: https://www.zhihu.com/question/19732473
  [37]: http://www.cnblogs.com/roky/archive/2008/02/21/1076332.html
  [38]: http://blog.csdn.net/haoel/article/details/4028232
  [39]: https://www.zhihu.com/question/19901763
