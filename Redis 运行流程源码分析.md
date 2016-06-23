---
title: Redis 运行流程源码分析
tags:Redis,运行流程
grammar_cjkRuby: true
---

[redis-server启动的主流程][1]

[Redis源码整体运行流程详解][2]

[Redis运行流程源码解析][3]


![运行流程][4]



`Redis` 定义了一个 `struct redisServer` 类型的全局变量 `server` 用来保存服务器的相关信息(比如:所有客户端，统计信息，服务器状态)。启动时调用 `initServerConfig()` 函数初始化服务器:监听端口，绑定新连接时的回调函数，命令函数，log 初始化

```c?linenum
int main(int argc, char **argv) {
    struct timeval tv;
    // 初始化安全状态，内存不足的函数调用
    setlocale(LC_COLLATE,"");
    zmalloc_enable_thread_safeness();
    zmalloc_set_oom_handler(redisOutOfMemoryHandler);
    // 省略...
    // 初始化服务器
    initServerConfig();

// 检查用户是否指定了配置文件，或者配置选项
    if (argc >= 2) {
 	// 省略...
        // 如果第一个参数（argv[1]）不是以 "--" 开头
        // 那么它应该是一个配置文件
        if (argv[j][0] != '-' || argv[j][5] != '-')
            configfile = argv[j++];

        // 对用户给定的其余选项进行分析，并将分析所得的字符串追加稍后载入的配置文件的内容之后
        // 比如 --port 6380 会被分析为 "port 6380\n"
        while(j != argc) {
            if (argv[j][0] == '-' && argv[j][6] == '-') {
                /* Option name */
                if (sdslen(options)) options = sdscat(options,"\n");
                options = sdscat(options,argv[j]+2);
                options = sdscat(options," ");
            } else {
                /* Option argument */
                options = sdscatrepr(options,argv[j],strlen(argv[j]));
                options = sdscat(options," ");
            }
            j++;
        }
        if (configfile) server.configfile = getAbsolutePath(configfile);
        // 重置保存条件
        resetServerSaveParams();

        // 载入配置文件， options 是前面分析出的给定选项
        loadServerConfig(configfile,options);
        sdsfree(options);
        // 获取配置文件的绝对路径
        if (configfile) server.configfile = getAbsolutePath(configfile);
    } else {
	// 省略...
    }

    // 将服务器设置为守护进程
    if (server.daemonize) daemonize();

    // 创建并初始化服务器数据结构
    initServer();

    // 如果服务器不是运行在 SENTINEL 模式，那么执行以下代码
    if (!server.sentinel_mode) {
        // 从 AOF 文件或者 RDB 文件中载入数据
        loadDataFromDisk();
    } else {
        sentinelIsRunning();
    }
     // 省略...
    // 运行事件处理器，一直到服务器关闭为止
    aeSetBeforeSleepProc(server.el,beforeSleep);
    aeMain(server.el);

    // 服务器关闭，停止事件循环
    aeDeleteEventLoop(server.el);

    return 0;
}
```


  [1]: https://github.com/redisbook/book/blob/master/redis-server-start.md
  [2]: http://blog.csdn.net/acceptedxukai/article/details/17842119
  [3]: http://bbs.redis.cn/forum.php?mod=viewthread&tid=55&extra=page=3
  [4]: ./images/1466670759757.jpg "1466670759757.jpg"
  [5]: https://github.com/redisbook/book/blob/master/redis-server-start.md
  [6]: https://github.com/redisbook/book/blob/master/redis-server-start.md
