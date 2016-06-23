---
title: Python 总结
tags: Python
grammar_cjkRuby: true
---



##  可变参数
[理解 Python 中的 *args 和 **kwargs][1]

=[可变参数][2]

**用在函数参数**

`*args` 位置参数:把参数收集到一个元组，作为变量 `args`

`**kwargs` 关键字参数:包含参数名和值的字典类型，需要 `key/value` 对作为参数

**用在函数调用**
```python?linenums
>>> def test(a,b,c):
	print  a,b,c
>>> x = {'a':1,'b':2,'c':3}
>>> test(**x)
1 2 3
>>> test(*x) 
a c b
```

##  字符编码

[十分钟搞清字符集和字符编码][3]

[字符编码笔记：ASCII，Unicode和UTF-8][4]


 1. `Unicode` 对100多万个字符进行了编码，只规定了符号的二进制代码，没有规定二进制代码如何存储
 2. `UTF-8` 是对 `Unicode` 的实现方式，一种变长的编码方式


`UTF-8`、`GBK` 都是对 `Unicode` 的字符二进制进行了不同的编码


##  Python 程序的运行原理
[谈谈 Python 程序的运行原理][5]



##  Python 字典对象实现
[《Python源码剖析》阅读笔记：第五章-dict对象][6]


字典和 C++ STL 中 map 一样，是映射容器，但是原理不一样，效率要求更高，所以采用了哈希表来实现。

为了解决哈希值冲突问题，采用了**开放寻址法**。开放寻址法能更好的利用 CPU Cache，命中率较高。


  [1]: http://kodango.com/variable-arguments-in-python
  [2]: .//Passing%20arguments%20to%20Python%20functions1.pdf
  [3]: http://cenalulu.github.io/linux/character-encoding/
  [4]: http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html
  [5]: https://www.restran.net/2015/10/22/how-python-code-run/
  [6]: http://blog.csdn.net/digimon/article/details/7875789
