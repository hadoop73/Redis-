
#  事件
`Redis` 服务器是一个事件驱动程序，包括 **文件事件** 和 **时间事件**。
* 文件事件:服务器对套接字操作的抽象。服务器与客户端的通信会产生相应的文件事件
  服务器通过监听并处理这些事件来完成一系列网络通信操作
* 时间事件:`Redis` 需要在给定的时间点执行一些操作，时间事件就是服务器对这类定时操作的抽象

## 文件事件
`Redis` 基于 `Reactor` 模式开发自己的网络事件处理器。

* 文件事件处理器使用 `I/O` 多路复用程序来同时监听多个套接字，并根据套接字目前执行
     的任务关联不同的事件处理器。
* 当套接字准备好执行连接应答、读取、写入、关闭等操作时，产生相对应的文件事件，文件事件
     就会调用套接字之前关联好的事件处理器来处理这些事件。

文件事件处理器的四个组成部分:套接字、`I/O` 多路复用程序、文件事件分派器，以及事件处理器

文件事件是对套接字操作的抽象，每当套接字准备好执行连接应答、写入、读取、关闭操作时，
就会产生一个文件事件；**`I/O` 多路复用程序** 负责监听多个套接字并向文件事件分派传送产生了事件的套接字。
**文件事件分派器** 接收 `I/O` 多路复用程序传来的套接字，根据套接字产生的事件的类型，调用相应的事件处理器

**事件类型**

`I/O` 多路复用程序可以监听多个套接字的 `ae.h/AE_READABLE` 事件和 `ae.h/AE_WRITABLE` 事件。

* 分成两种事件类型为了统一接口

* 因为 `Read` 操作可以并发，`Write` 操作不能并发，所以应该有所区别

创建一个文件事件，为给定的套接字绑定给定的事件，并加入到 `I/O` 多路复用程序的监听范围
```cpp
/*
 * 根据 mask 参数的值，监听 fd 文件的状态，
 * 当 fd 可用时，执行 proc 函数
 */
int aeCreateFileEvent(aeEventLoop *eventLoop, int fd, int mask,
        aeFileProc *proc, void *clientData)
{
    if (fd >= eventLoop->setsize) return AE_ERR;
    aeFileEvent *fe = &eventLoop->events[fd];

    // 监听指定 fd
    if (aeApiAddEvent(eventLoop, fd, mask) == -1)
        return AE_ERR;

    // 设置文件事件类型
    fe->mask |= mask;
    if (mask & AE_READABLE) fe->rfileProc = proc;
    if (mask & AE_WRITABLE) fe->wfileProc = proc;

    fe->clientData = clientData;

    // 如果有需要，更新事件处理器的最大 fd
    if (fd > eventLoop->maxfd)
        eventLoop->maxfd = fd;

    return AE_OK;
}
```

**文件事件的处理器**
`Redis` 为文件事件编写了多个处理器，用于实现不同的网络通信需求
* 连接应答处理器: `networking.c/acceptTecpHandler` 函数是 `Redis` 的连接应答处理器，
  用于对连接服务器监听套接字的客户端进行应答，具体实现为 `sys/socket.h/accept` 函数的包装

* 命令请求处理器:`networking.c/readQueryFromClient` 函数是 `Redis` 的命令请求处理器，
  这个处理器负责从套接字中读入客户端发送的命令请求内容，具体实现 `unistd.h/read` 函数的包装

##  时间事件
目前只有使用周期性事件，没有定时事件。
服务器将所有时间事件都放在一个无序链表中，每当时间事件执行器运行时，遍历整个链表，查找所有已达
的时间事件，并调用相应的事件处理器。

**创建时间事件**

对一个事件事件绑定一个处理器，当时间到达后，调用事件的处理器
```cpp
/*
 * 创建时间事件
 */
long long aeCreateTimeEvent(aeEventLoop *eventLoop, long long milliseconds,
        aeTimeProc *proc, void *clientData,
        aeEventFinalizerProc *finalizerProc)
{
    // 更新时间计数器
    long long id = eventLoop->timeEventNextId++;
    aeTimeEvent *te;

    te = zmalloc(sizeof(*te));
    if (te == NULL) return AE_ERR;

    te->id = id;

    // 设定处理事件的时间
    aeAddMillisecondsToNow(milliseconds,&te->when_sec,&te->when_ms);
    te->timeProc = proc;
    te->finalizerProc = finalizerProc;
    te->clientData = clientData;

    // 将新事件放入表头
    te->next = eventLoop->timeEventHead;
    eventLoop->timeEventHead = te;

    return id;
}
```

对文件事件和时间事件的处理都是同步、有序、原子地执行，服务器不会中途中断事件处理，也不会对事件进行抢占。

##  `I/O` 多路复用
`epoll` 监听套接字，是否有客户端发送命令，建立连接。也就是 `epoll` 多路复用的应用。
```cpp
/*
 * 获取可执行事件
 */
static int aeApiPoll(aeEventLoop *eventLoop, struct timeval *tvp) {
    aeApiState *state = eventLoop->apidata;
    int retval, numevents = 0;

    // 等待时间
    retval = epoll_wait(state->epfd,state->events,eventLoop->setsize,
            tvp ? (tvp->tv_sec*1000 + tvp->tv_usec/1000) : -1);

    // 有至少一个事件就绪？
    if (retval > 0) {
        int j;

        // 为已就绪事件设置相应的模式
        // 并加入到 eventLoop 的 fired 数组中
        numevents = retval;
        for (j = 0; j < numevents; j++) {
            int mask = 0;
            struct epoll_event *e = state->events+j;

            if (e->events & EPOLLIN) mask |= AE_READABLE;
            if (e->events & EPOLLOUT) mask |= AE_WRITABLE;
            if (e->events & EPOLLERR) mask |= AE_WRITABLE;
            if (e->events & EPOLLHUP) mask |= AE_WRITABLE;

            eventLoop->fired[j].fd = e->data.fd;
            eventLoop->fired[j].mask = mask;
        }
    }

    // 返回已就绪事件个数
    return numevents;
}
```

##  为文件时间绑定处理器
也就是调用 `aeCreateFileEvent` 或 `aeCreateTimeEvent` 创建文件事件或者时间事件。
在 `networking.c` 中，创建 `redis` 客户端的时候，会有创建应答文件事件
```cpp
/*
 * 创建新的客户端实例
 */
redisClient *createClient(int fd) {
    redisClient *c = zmalloc(sizeof(redisClient));

    /* passing -1 as fd it is possible to create a non connected client.
     * This is useful since all the Redis commands needs to be executed
     * in the context of a client. When commands are executed in other
     * contexts (for instance a Lua script) we need a non connected client. */
    // 因为 Redis 命令总在客户端的上下文中执行，
    // 有时候为了在服务器内部执行命令，需要使用伪客户端来执行命令
    // 在 fd == -1 时，创建的客户端为伪终端
    if (fd != -1) {
        anetNonBlock(NULL,fd);
        anetTcpNoDelay(NULL,fd);
        if (aeCreateFileEvent(server.el,fd,AE_READABLE, readQueryFromClient, c) == AE_ERR)
        {
            close(fd);
            zfree(c);
            return NULL;
        }
    }
    return c;
}
```
在添加回复，会先创建 `send` 相关文件事件
```cpp
int prepareClientToWrite(redisClient *c) {
    if (c->flags & REDIS_LUA_CLIENT) return REDIS_OK;
    if (c->fd <= 0) return REDIS_ERR; /* Fake client */
    if (c->bufpos == 0 && listLength(c->reply) == 0 &&
        (c->replstate == REDIS_REPL_NONE || c->replstate == REDIS_REPL_ONLINE) &&
        aeCreateFileEvent(server.el, c->fd, AE_WRITABLE, sendReplyToClient, c) == AE_ERR)
        return REDIS_ERR;
    return REDIS_OK;
}
```









