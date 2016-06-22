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



  [1]: http://kodango.com/variable-arguments-in-python
  [2]: .//Passing%20arguments%20to%20Python%20functions1.pdf