---
title: Netty
tags: 
grammar_cjkRuby: true
---

[TOC]



##  五种 I/O 模型

**阻塞 I/O 模型**
所有文件操作都是阻塞的

![在进场空间中调用 recvfrom,其系统调用直到数据包到达且被复制到应用进程的缓冲区或发生错误才返回,期间一直等待,称为阻塞 I/O 模型][1]

![enter description here][2]

**非阻塞 I/O 模型**

![enter description here][3]

![enter description here][4]


**I/O 复用模型**

![enter description here][5]

![enter description here][6]


**信号驱动 I/O 模型**

![enter description here][7]

![enter description here][8]

**异步 I/O**

![enter description here][9]

![enter description here][10]

**伪异步 I/O**
由于是读写阻塞的,线程会被阻塞,信号队列满了后,无法处理客户请求



##  I/O 多路复用技术

I/O 多路复用把多个 I/O 阻塞复用到同一个 select 的阻塞上,系统单线程处理过个客户端请求,I/O 多路复用的最大优势是系统开销小,不需要创建新的额外进程或者线程

**epoll**
select 最大缺陷,单个进程打开的描述符有一定限制,由 FD_SETSIZE 设置,默认 1024.Java 没有共享内存,需要通过 Socket 通信进行数据通信会消耗 Socket,epoll 没有最大 FD 限制.

I/O 效率:select/poll 每次调用都会线性下降.epoll 只会对"活跃"socket进行操作,内核根据每个 fd 上面的callback实现的.

epoll 加速内核与用户空间的消息传递:epoll 通过内核和用户空间 mmap 同一块内存实现.


##  NIO
非阻塞I/O
SocketChannel 和 ServerSocketChannel 支持阻塞和非阻塞两种模式

**缓冲区 Buffer**
所有数据都是用缓冲区处理的.

**通道 Channel**
通过 Channel 读取和写入数据.全双工,同时支持读写操作

**多路复用 Selector**
轮询注册的 Channel 是否由 TCP 连接,读写,然后通过 SelectionKey 获取就绪 Channel 的集合

![enter description here][11]


###  异步
提供异步文件通道和异步套接字通道
* 通过 java.util.concurrent.Future 表示异步操作结果
* 在执行异步操作的时候传入一个 java.nio.channels
* CompletionHandler 接口的实现类作为操作完成的回调

##  Netty

![enter description here][12]

![enter description here][13]

**netty 开发流程**
```java
// 配置服务端的NIO线程组
EventLoopGroup bossGroup = new NioEventLoopGroup();
EventLoopGroup workerGroup = new NioEventLoopGroup();
ServerBootstrap b = new ServerBootstrap();
b.group(bossGroup,workerGroup)
 .channel(NioServerSocketChannel.class)
 .option(ChannelOption.SO_BACKLOG,1024)
 .childHandler(new ChildChannelHandler());
// 绑定端口,同步等待完成
ChannelFuture f = b.bind(port).sync();
// 等待服务端监听端口关闭
f.channel().closeFuture().sync();
//优雅退出,释放线程池资源
bossGroup.shutdownGracefully();
workerGroup.shutdownGracefully();
```
**NioEventLoopGroup**
创建两个 NioEventLoopGroup 为了一个用于服务端接受客户端的连接,另一个用于进行 SocketChannel 的网络读写

* NioEventLoopGroup实际上就是个线程池
* NioEventLoopGroup在后台启动了n个NioEventLoop来处理Channel事件
* 每一个NioEventLoop负责处理m个Channel
* NioEventLoopGroup从NioEventLoop数组里挨个取出NioEventLoop来处理Channel

![enter description here][14]


**ServerBootstrap**

Netty 用于启动 NIO 服务端的辅助启动类,目的是降低服务端的开发复杂度.

**NioServerSocketChannel**
类似 SeverSocketChannel 用于监听客户端连接

option(ChannelOption.SO_BACKLOG,1024) 设置 backlog 为1024

最后绑定 I/O 时间的处理类 ChildChannelHandler 用于处理网络 I/O 事件

调用 bind 方法绑定监听端口,调用同步阻塞方法 sync 等待操作完成,返回一个 ChannelFuture 用于异步操作的通知回调


**客户端开发流程**
```java
EventLoopGroup group = new NioEventLoopGroup();
Bootstrap b = new Bootstrap();
b.group(group).channel(NioSocketChannel.class)
 .option(ChannelOption.TCP_NODELAY,true)
 handler(new ChannelInitializer<SocketChannel>(){
	public void initChannel(SocketChannel ch) throw Exception{
		chi.pipeline().addLast(new Handler());
	}
});
// 发起异步连接操作
ChannelFuture f = b.connect(host,port).sync();

// 等待客户端链路关闭
f.channel().closeFuture().sync();
//优雅退出,释放 NIO 线程组
group.shutdownGracefully();
```


##  TCP 粘包/拆包

TCP 是"流"协议,没有界限的一串数据,是连成一片的,没有分界线,根据 TCP 缓冲区的实际情况进行包的划分,一个完整的包可能会被 TCP 拆分成多个包进行发送,也可能多个小的包封装成一个大的数据包发送,这就是所谓 TCP 粘包和拆包问题.

![enter description here][15]


**原因**

![enter description here][16]

**解决策略**
* 消息定长,每个报文大小固定长度 200 字节,不够,空位补全
* 包尾增加回车换行进行分割
* 将消息分为消息头和消息体,消息头包含消息总长度


##  解决粘包
**服务端**

![enter description here][17]


**客户端**

![enter description here][18]

LineBasedFrameDecoder 的工作原理一次便利 ByteBuf 中的可读字节,判断是否由 "\n" 或者 "\r\n",如果有就以此位置为结束位置,从可读索引到结束位置区间字节就组成了一行.以换行符为结束标志的解码器,同时支持配置单行最大长度,最大长度仍然没有发现换行符,就抛出异常.

![enter description here][19]


##  序列化

###  Java 自带序列化

**服务端**

![enter description here][20]

![enter description here][21]

使用 weakCachingConcurrentResolver 创建线程安全的 WeakReferenceMap 对类加载器进行缓存,支持多线程并发访问,内存不足时,会释放内存,防止内存泄露

ObjectEncoder 可以在消息发送的时候自动将实现 Serializable 的 POJO 对象进行编码

**客户端**

![enter description here][22]

![enter description here][23]


###  Protobuf 序列化
优点:
* 编码后消息更小,有利于存储和传输
* 编解码性能更高
* 支持定义可选和必选字段

**服务端**

![enter description here][24]

![enter description here][25]

**客户端**

![enter description here][26]

**注意事项**

![enter description here][27]


##  ByteBuf
###  ByteBuffer
* 长度固定
* 只有一个标识位置的指针 position
* API 功能有限

###  ByteBuf
两个位置指针,读操作 readerIndex,写操作 writeIndex

读取之后,0~readerIndex 被视为 discard,只有 readerIndex 和 writerIndex 之间的数据可以读取

**mark 和 rest**
调用 mark 操作会将位置指针备份到 mark 变量中,调用 rest 操作,重新将指针当前位置恢复为备份 mark 中的值


  [1]: ./images/1471766981214.jpg "1471766981214.jpg"
  [2]: ./images/1471767023134.jpg "1471767023134.jpg"
  [3]: ./images/1471767057600.jpg "1471767057600.jpg"
  [4]: ./images/1471767162759.jpg "1471767162759.jpg"
  [5]: ./images/1471767236471.jpg "1471767236471.jpg"
  [6]: ./images/1471767254544.jpg "1471767254544.jpg"
  [7]: ./images/1471767388180.jpg "1471767388180.jpg"
  [8]: ./images/1471767428308.jpg "1471767428308.jpg"
  [9]: ./images/1471767506801.jpg "1471767506801.jpg"
  [10]: ./images/1471767532322.jpg "1471767532322.jpg"
  [11]: ./images/1471770733143.jpg "1471770733143.jpg"
  [12]: ./images/1471776627948.jpg "1471776627948.jpg"
  [13]: ./images/1471776652807.jpg "1471776652807.jpg"
  [14]: ./images/1471777627814.jpg "1471777627814.jpg"
  [15]: ./images/1471779285630.jpg "1471779285630.jpg"
  [16]: ./images/1471779337335.jpg "1471779337335.jpg"
  [17]: ./images/1471780310257.jpg "1471780310257.jpg"
  [18]: ./images/1471780372452.jpg "1471780372452.jpg"
  [19]: ./images/1471780677537.jpg "1471780677537.jpg"
  [20]: ./images/1471780918543.jpg "1471780918543.jpg"
  [21]: ./images/1471780984946.jpg "1471780984946.jpg"
  [22]: ./images/1471781388036.jpg "1471781388036.jpg"
  [23]: ./images/1471781401165.jpg "1471781401165.jpg"
  [24]: ./images/1471782079379.jpg "1471782079379.jpg"
  [25]: ./images/1471782166042.jpg "1471782166042.jpg"
  [26]: ./images/1471782271333.jpg "1471782271333.jpg"
  [27]: ./images/1471782422305.jpg "1471782422305.jpg"
