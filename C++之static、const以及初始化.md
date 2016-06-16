---
title: C++之static、const以及初始化
tags: C++,static,const,初始化
grammar_cjkRuby: true
---

[TOC]

`const` 定义的常量在超出其作用域之后其空间会被释放
`static` 定义的静态常量在函数执行后不会释放其存储空间。

`static` 表示的是静态的。类的静态成员函数、静态成员变量是**和类相关的**，而不是和类的具体对象相关的。即使没有具体对象，也能**调用类的静态成员函数和成员变量**。一般类的静态函数几乎就是一个**全局函数**，只不过它的**作用域限于包含它的文件中**。

在 `C++` 中，`static` 静态成员变量**不能在类的内部初始化**。在类的内部只是声明，**定义必须在类定义体的外部**，通常在类的实现文件中初始化，如：`double Account::Rate=2.25`; `static` 关键字只能用于类定义体内部的声明中，定义时不能标示为 `static`。

在 `C++` 中，`const` 成员变量也**不能在类定义处初始化**，只能通过**构造函数初始化列表进行**，并且必须有构造函数。

`const` 数据成员只在某个对象生存期内是常量，而对于整个类而言却是可变的。因为类可以创建多个对象，**不同的对象其const数据成员的值可以不同**。所以不能在类的声明中初始化 `const` 数据成员，因为类的对象没被创建时，编译器不知道 `const` 数据成员的值是什么。

`const` 数据成员的初始化只能在**类的构造函数的初始化列表中进行**。要想建立在整个类中都恒定的常量，应该用类中的**枚举常量**来实现，或者 `static cosnt`。
```cpp?linenums
class Test  
{  
public:  
      Test():a(0){}  
      enum {size1=100,size2=200};  
private:  
      const int a;//只能在构造函数初始化列表中初始化  
       static int b;//在类的实现文件中定义并初始化  
      const static int c;//与 static const int c;相同。  
};  
	  
int Test::b=0;//static成员变量不能在构造函数初始化列表中初始化，因为它不属于某个对象。  
cosnt int Test::c=0;//注意：给静态成员变量赋值时，不需要加static修饰符。但要加cosnt 
```

**`const` 成员函数**主要目的是**防止成员函数修改对象的内容**。即 `const` 成员函数不能修改成员变量的值，但可以访问成员变量。当方法访问成员函数时，**该函数只能是 `const` 成员函数**。

**`static` 成员函数**主要目的是作为**类作用域的全局函数**。**不能访问类的非静态数据成员**。类的静态成员函数没有this指针，这导致：
* 1、不能直接存取类的非静态成员变量，调用非静态成员函数
* 2、不能被声明为 `virtual`。

##  `static`、`const`、`static const`、`const static` 成员的初始化问题
**类的 `const` 成员初始化**
在一个类中建立一个 `const` 时，不能给初值
```cpp?linenums
class foo{
public:
    foo():i(100){}
private:
    const int i=100; // error
};
// 或者在类外进行初始化
foo::foo():i(100){}
```
**类里的 `static` 成员初始化**
类中的 `static` 变量是属于类的，不属于某个对象，它在整个程序的运行过程中只有一个副本，因此**不能在定义对象时 对变量进行初始化**，就是**不能用构造函数进行初始化**，其正确的初始化方法是：
==数据类型 类名::静态数据成员名=值;==
```cpp?linenums
class foo{
public:
    foo();
private:
    static int i;
};
int foo::i=20;
```
这表明：
- **初始化在类体外进行**，而前面不加 `static`，以免与一般静态变量或对象相混淆。
- 初始化时不加该成员的访问权限控制符 `private、public`等。
- 初始化时使用作用域运算符来表明它所属的类，因此，**静态数据成员是类的成员**而不是对象的成员。

**类里的 `static const` 和 `const static` 成员初始化**
这两种写法的作用一样，为了便于记忆，在此值说明一种通用的初始化方法：
```cpp?linenums
class Test  
{  
public:  
      static const int mask1;  
      const static int mask2;  
};  
const Test::mask1=0xffff;  
const Test::mask2=0xffff;
//它们的初始化没有区别，虽然一个是静态常量一个是常量静态。静态都将存储在全局变量区域，其实最后结果都一样。可能在不同编译器内，不同处理，但最后结果都一样。  
```
##  完整列子
```cpp?linenums
#ifdef A_H_
#define A_H_
#include <iostream>
using namespace std;
class A
{
public:
      A(int a);
      static void print();//静态成员函数
private:
      static int aa;//静态数据成员的声明
       static const int count;//常量静态数据成员（可以在构造函数中初始化）
       const int bb;//常量数据成员
};
int A::aa=0;//静态成员的定义+初始化
const int A::count=25;//静态常量成员定义+初始化
A::A(int a):bb(a)//常量成员的初始化
{
      aa+=1;
}
void A::print()
{
      cout<<"count="<<count<<endl;
      cout<<"aa="<<aa<<endl;
}
#endif
void main()
{
      A a(10);
      A::print();//通过类访问静态成员函数
      a.print();//通过对象访问静态成员函数
}
```




































