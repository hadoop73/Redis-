---
title: 深刻理解 Linux 进程间通信 
tags: 进程通信
grammar_cjkRuby: true
---


[TOC]


[Linux 进程通信][1]

`Linux` 下进程通信的几种主要手段：

 1. 管道及有名管道：管道可用于具有亲缘关系进程间的通信，有名管道克服了管道没有名字的限制，因此，除具有管道所具有的功能外，它还允许无亲缘关系进程间的通信
 2. 信号（Signal）：信号是比较复杂的通信方式，用于通知接受进程有某种事件发生，除了用于进程间通信外，进程还可以发送信号给进程本身；linux除了支持Unix早期信号语义函数sigal外，还支持语义符合Posix.1标准的信号函数sigaction（实际上，该函数是基于BSD的，BSD为了实现可靠信号机制，又能够统一对外接口，用sigaction函数重新实现了signal函数）
 3. 报文（Message）队列（消息队列）：消息队列是消息的链接表，包括Posix消息队列system V消息队列。有足够权限的进程可以向队列中添加消息，被赋予读权限的进程则可以读走队列中的消息。消息队列克服了信号承载信息量少，管道只能承载无格式字节流以及缓冲区大小受限等缺点。
 4. 共享内存：使得多个进程可以访问同一块内存空间，是最快的可用IPC形式。是针对其他通信机制运行效率较低而设计的。往往与其它通信机制，如信号量结合使用，来达到进程间的同步及互斥。
 5. 信号量（semaphore）：主要作为进程间以及同一进程不同线程之间的同步手段。
 6. 套接口（Socket）：更为一般的进程间通信机制，可用于不同机器之间的进程间通信。起初是由Unix系统的BSD分支开发出来的，但现在一般可以移植到其它类Unix系统上：Linux和System V的变种都支持套接字。


##  管道通信
管道和有名管道是最早的进程间通信机制之一，管道可用于具有亲缘关系进程间的通信，有名管道克服了管道没有名字的限制，因此，除具有管道所具有的功能外，它还允许无亲缘关系进程间的通信。

 1. 管道的创建
```cpp?linenums
#include<unistd.h>
int pipe(int fd[2])
```
一个进程在由pipe()创建管道后，一般再fork一个子进程，然后通过管道实现父子进程间的通信（因此也不难推出，只要两个进程中存在亲缘关系，这里的亲缘关系指的是具有共同的祖先，都可以采用管道方式来进行通信）。

 2. 管道的读写规则
 管道两端可分别用描述字 `fd[0]` 以及 `fd[1]` 来描述，需要注意的是，管道的两端是固定了任务的。即一端只能用于读，由描述字 `fd[0]` 表示，称其为管道读端；另一端则只能用于写，由描述字 `fd[1]` 来表示，称其为管道写端。如果试图从管道写端读取数据，或者向管道读端写入数据都将导致错误发生。一般文件的 `I/O` 函数都可以用于管道，如 `close、read、write` 等等。
 
```cs?linenums
#include<unistd.h>
#include<sys/types.h>
#include<errno.h>
main(){
    int pipe_fd[2];
    pid_t pid;
    char r_buf[100];
    char w_buf[4];
    char* p_wbuf;
    int r_num;
    int cmd;
    
    memset(r_buf,0,sizeof(r_buf));
    memset(w_buf,0,sizeof(r_buf));
    p_wbuf=w_buf;
    if(pipe(pipe_fd)<0){
        printf("pipe create error\n");
        return -1;
    }
    
    if((pid=fork())==0){
        printf("\n");
        close(pipe_fd[1]);
        sleep(3);//确保父进程关闭写端
        r_num=read(pipe_fd[0],r_buf,100);
        printf("read num is %d the data read from the pipe is %d\n",r_num,atoi(r_buf));
        close(pipe_fd[0]);
        exit();
    }else if(pid>0){
        close(pipe_fd[0]);//read strcpy(w_buf,"111");
        if(write(pipe_fd[1],w_buf,4)!=-1)
            printf("parent write over\n");
            close(pipe_fd[1]);//write
            printf("parent close fd[1] over\n");
            sleep(10);
        }
}
/**************************************************
*程序输出结果：
*parent write over
*parent close fd[1] over 
*read num is 4
*the data read from the pipe is 111
*附加结论：
*管道写端关闭后，写入的数据将一直存在，直到读出为止.
****************************************************/
```
向管道中写入数据：
 3. 向管道中写入数据时，`linux` 将不保证写入的原子性，管道缓冲区一有空闲区域，写进程就会试图向管道写入数据。如果读进程不读走管道缓冲区中的数据，那么写操作将一直阻塞。

 4. 管道的局限性
 5. 只支持单向数据流
 6. 只能用于亲缘关系的进程之间
 7. 没有名字
 8. 管道的缓冲区是有限的
 9. 管道所传送的无格式字节流，要求管道的读出方和写入方事先约定好数据的格式
 
 ##  有名管道
 
 1. 有名管道相关概念
`FIFO` 不同于管道之处在于它提供一个路径名与之关联，以 `FIFO` 的文件形式存在于文件系统中。这样，即使与 `FIFO` 的创建进程不存在亲缘关系的进程，只要可以访问该路径，就能够彼此通过 `FIFO` 相互通信（能够访问该路径的进程以及 `FIFO` 的创建进程之间），因此，通过 `FIFO` 不相关的进程也能交换数据。值得注意的是，`FIFO` 严格遵循先进先出（`first in first out`），对管道及 `FIFO` 的读总是从开始处返回数据，对它们的写则把数据添加到末尾。它们不支持诸如 `lseek()` 等文件定位操作。

 2. 有名管道的创建
```c?linenums
#include<sys/types.h>
#include<sys/stat.h>
int mkfifo(const char * pathname, mode_t mode)
```
该函数的第一个参数是一个普通的路径名，也就是创建后 `FIFO` 的名字。第二个参数与打开普通文件的open()函数中的 `mode` 参数相同。如果 `mkfifo` 的第一个参数是一个已经存在的路径名时，会返回 `EEXIST` 错误，所以一般典型的调用代码首先会检查是否返回该错误，如果确实返回该错误，那么只要调用打开 `FIFO` 的函数就可以了。一般文件的 `I/O` 函数都可以用于 `FIFO`，如 `close、read、write` 等等。
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 

  [1]: http://wenku.baidu.com/link?url=ycw82cnizi12EXDNn3RuTqyIcWThmOmGrwILEkjOcZJgWr9Us5wqB5nlXr6kpQAR10Hb9X2xTBMuSd1f8G_pS_U5FErQDffR0IxXHOVZvlG