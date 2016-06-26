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


##  Socket 通信原理
[Socket通信原理简介][7]

[ Socket通信原理和实践][8]

[SOCKET类型定义及应用][9]

Socket 用于网络中的不同计算机通信，应用层和传输层之间的一个抽象

![enter description here][10]

**实现过程**

![enter description here][11]

**socket 函数**

创建 socket 描述字，确定 socket 的协议类型（TCP 或 UDP)

**bind 函数**

[bind 函数说明][12]

bind 函数用于服务器端 socket 描述字和源地址、端口绑定，在多网卡的情况下也能正确的监听网卡和端口

**listen\connect 函数**

listen 监听 socket 描述字和客户端建立连接，同时确定申请连接队列长度，服务端不能及时处理的客户端，会放在一个队列中，队列满了，再申请的客户会收到 WSAECONNREFUSED 错误。

connect 用于连接[enter description here][13]服务器端，需要知道客户端的 socket 描述字，和服务器 socket(包括服务器的端口和 IP)


**accept 函数**
通过监听 socket 描述字产生已连接 socket 描述字，已连接 socket 用于通信


##  多线程
[python 多线程就这么简单][14]

[python 中 threading 的 setDaemon、join 的用法][15]

```python?linenums
import threading   #  导入 threading 模块
t = threading.Thread(target,args)  # 创建一个线程对象，target 为需要运行的函数，args 为函数的参数
t.setDaemon(True)  # 在 start() 调用之前设置，
t.start()  #  运行
```

threada.join（) 表示正在运行的线程需要在线程 threada 结束后才继续运行

setDaemon（) 表示主线程结束时，字线程也会被杀死

##  socketserver 源码
[socketserver源码分析][16]

server 类有 5 种类型，还有支持事务处理的 BaseRequestHandler 类及子类，扩展成为多线程或多进程需要继承 ForkingMixIn 或 ThreadingMixIn

**Server 类**

![enter description here][17]

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



  [1]: http://kodango.com/variable-arguments-in-python
  [2]: .//Passing%20arguments%20to%20Python%20functions1.pdf
  [3]: http://cenalulu.github.io/linux/character-encoding/
  [4]: http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html
  [5]: https://www.restran.net/2015/10/22/how-python-code-run/
  [6]: http://blog.csdn.net/digimon/article/details/7875789
  [7]: http://www.jianshu.com/p/90348ef3f41e
  [8]: http://blog.csdn.net/jiajia4336/article/details/8798421
  [9]: http://blog.163.com/alice_leee/blog/static/167106323201062332816623/
  [10]: ./images/1466861848287.jpg "1466861848287.jpg"
  [11]: ./images/1466861895747.jpg "1466861895747.jpg"
  [12]: http://www.cnblogs.com/nightwatcher/archive/2011/07/03/2096717.html
  [13]: http://blog.sina.com.cn/s/blog_9f488855010198vn.html
  [14]: http://www.cnblogs.com/fnng/p/3670789.html
  [15]: http://blog.sina.com.cn/s/blog_9f488855010198vn.html
  [16]: http://www.blogs8.cn/posts/Wx8G9b8
  [17]: ./images/1466930857819.jpg "1466930857819.jpg"
