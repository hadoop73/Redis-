---
title: Python 总结
tags: Python
grammar_cjkRuby: true
---

##  函数式编程
[Python函数式编程指南（一）：概述][1]

[Python函数式编程指南（二）：函数][2]

**什么是函数式编程**
* 函数作为参数，返回值传递
* 匿名函数(lambda)
```python
lambda arg: expression
```
* 使用闭包

优点：模块化，函数式编程推崇简单原则，一个函数只做一件事情，将功能大的事情尽可能拆成小的模块;易于测试、调试;函数式编程产生更少的代码，更容易阅读和维护

**闭包**
闭包是一类特殊的函数，如果一个函数定义在另一个函数的作用域中，并且函数中引用了外部函数的局部变量，那么这个函数就是一个闭包。

* 闭包能够提高代码复用性
```python
def line_conf(a, b):
    def line(x):
        return a*x + b
    return line

line1 = line_conf(1, 1)
line2 = line_conf(4, 5)
print line1(5), line2(5)
```

**常用函数**
[python函数式编程][3]

* lambda，创建一个匿名函数，冒号左侧表示接收的参数，右侧表示返回值
* map，对参数的元素调用相同的函数
```python
map(len,'hao','fnng','cn') # 对参数中的每一个元素作为 len 函数的参数，返回值放在同一个列表中
```
* reduce，每次需要两个数据进行处理，最后的结果作为返回值
```python
add = reduce(lambda x,y:x+y,[2,3,4])
print add
# 输出结果为 9
```
* filter,获取参数中满足条件的元素，放在一个列表中作为返回值
```python
number =[2, -5, 9, -7, 2, 5, 4, -1, 0, -3, 8]
# 筛选大于 0 的元素
sum = filter(lambda x: x>0, number)
```

##  可变参数
[理解 Python 中的 *args 和 **kwargs][4]

[可变参数][5]


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

[十分钟搞清字符集和字符编码][6]

[字符编码笔记：ASCII，Unicode和UTF-8][7]


 1. `Unicode` 对100多万个字符进行了编码，只规定了符号的二进制代码，没有规定二进制代码如何存储
 2. `UTF-8` 是对 `Unicode` 的实现方式，一种变长的编码方式


`UTF-8`、`GBK` 都是对 `Unicode` 的字符二进制进行了不同的编码


##  Python 程序的运行原理
[谈谈 Python 程序的运行原理][8]



##  input()、raw_input() 和 sys.stdin
[raw_input() 与 input() Python][9]

[Python 的 sys.stdout、sys.stdin 重定向][10]

raw_input() 读入的都是字符串，是对 sys.stdin.readline() 的调用
sys.stdin.readline() 读入的为字符串，且包含换行符
input() 能输入特定格式(比如:整数、字符串)，是对 raw_input() 的调用

##  Python 字典对象实现
[《Python源码剖析》阅读笔记：第五章-dict对象][11]


字典和 C++ STL 中 map 一样，是映射容器，但是原理不一样，效率要求更高，所以采用了哈希表来实现。

为了解决哈希值冲突问题，采用了**开放寻址法**。开放寻址法能更好的利用 CPU Cache，命中率较高。


##  Socket 通信原理
[Socket通信原理简介][12]

[ Socket通信原理和实践][13]

[SOCKET类型定义及应用][14]

Socket 用于网络中的不同计算机通信，应用层和传输层之间的一个抽象

![enter description here][15]

**实现过程**

![enter description here][16]

**socket 函数**

创建 socket 描述字，确定 socket 的协议类型（TCP 或 UDP)

**bind 函数**

[bind 函数说明][17]

bind 函数用于服务器端 socket 描述字和源地址、端口绑定，在多网卡的情况下也能正确的监听网卡和端口

**listen\connect 函数**

listen 监听 socket 描述字和客户端建立连接，同时确定申请连接队列长度，服务端不能及时处理的客户端，会放在一个队列中，队列满了，再申请的客户会收到 WSAECONNREFUSED 错误。

connect 用于连接[enter description here][18]服务器端，需要知道客户端的 socket 描述字，和服务器 socket(包括服务器的端口和 IP)


**accept 函数**
通过监听 socket 描述字产生已连接 socket 描述字，已连接 socket 用于通信


##  多线程
[python 多线程就这么简单][19]

[python 中 threading 的 setDaemon、join 的用法][20]


##  cookie 和 session 的区别

[cookie 和session 的区别详解][21]

* cokkie 存放在客户端，session 存放在服务器上
* cookie 不安全，容易被获取


```python?linenums
import threading   #  导入 threading 模块
t = threading.Thread(target,args)  # 创建一个线程对象，target 为需要运行的函数，args 为函数的参数
t.setDaemon(True)  # 在 start() 调用之前设置，
t.start()  #  运行
```

threada.join（) 表示正在运行的线程需要在线程 threada 结束后才继续运行

setDaemon（) 表示主线程结束时，字线程也会被杀死

##  socketserver 源码
[socketserver源码分析][22]

server 类有 5 种类型，还有支持事务处理的 BaseRequestHandler 类及子类，扩展成为多线程或多进程需要继承 ForkingMixIn 或 ThreadingMixIn

**Server 类**

![enter description here][23]

这些 Server 都是对 socket 的封装，并确定参数实现相关协议，在 TCP 和 UDP 中，确定 address_family、socket_type 来调用不同的协议

```python?linenums
address_family = socket.AF_INET
socket_type = socket.SOCK_STREAM
self.socket = socket.socket(self.address_family,self.socket_type)
```

实现 TCPServer 时候，只要设置好 adress、port 和请求出来函数

```python?linenums
address=('127.0.0.1',9098)
server=socketserver.TCPServer(address,myTCPHandle)      #创建一个基于TCPServer的套接字
server.serve_forever() 
```

**事务处理 RequestHandler**

服务器段每次接收到一个连接都会新创建一个 RequestHandler

```python?linenums
def __init__(self, request, client_address, server):
    self.request = request
    self.client_address = client_address
    self.server = server
    self.setup()
    try:
         self.handle()
    finally:
         self.finish()

request, client_address = self.get_request()

# 在 TCPServer 中
def get_request(self):
    return return self.socket.accept()

```
事务处理接受 reques、client 和 server 为参数，并调用 handle（） 


  [1]: http://www.cnblogs.com/huxi/archive/2011/06/18/2084316.html
  [2]: http://www.cnblogs.com/huxi/archive/2011/06/24/2089358.html
  [3]: http://www.cnblogs.com/fnng/p/3699893.html
  [4]: http://kodango.com/variable-arguments-in-python
  [5]: .//Passing%20arguments%20to%20Python%20functions1.pdf
  [6]: http://cenalulu.github.io/linux/character-encoding/
  [7]: http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html
  [8]: https://www.restran.net/2015/10/22/how-python-code-run/
  [9]: http://www.cnblogs.com/way_testlife/archive/2011/03/29/1999283.html
  [10]: http://www.tuicool.com/articles/mE3QJ3
  [11]: http://blog.csdn.net/digimon/article/details/7875789
  [12]: http://www.jianshu.com/p/90348ef3f41e
  [13]: http://blog.csdn.net/jiajia4336/article/details/8798421
  [14]: http://blog.163.com/alice_leee/blog/static/167106323201062332816623/
  [15]: ./images/1466861848287.jpg "1466861848287.jpg"
  [16]: ./images/1466861895747.jpg "1466861895747.jpg"
  [17]: http://www.cnblogs.com/nightwatcher/archive/2011/07/03/2096717.html
  [18]: http://blog.sina.com.cn/s/blog_9f488855010198vn.html
  [19]: http://www.cnblogs.com/fnng/p/3670789.html
  [20]: http://blog.sina.com.cn/s/blog_9f488855010198vn.html
  [21]: http://www.cnblogs.com/shiyangxt/archive/2008/10/07/1305506.html
  [22]: http://www.blogs8.cn/posts/Wx8G9b8
  [23]: ./images/1466930857819.jpg "1466930857819.jpg"
