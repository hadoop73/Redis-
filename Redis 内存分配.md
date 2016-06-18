---
title: Redis 内存分配 
tags: Redis,内存分配
grammar_cjkRuby: true
---

[TOC]

`Redis` 的内存管理在 `zmalloc.c和zmalloc.h` 中
```c?linenums
void *zmalloc(size_t size);
void *zcalloc(size_t size);
void *zrealloc(void *ptr, size_t size);
void zfree(void *ptr);
char *zstrdup(const char *s);
size_t zmalloc_used_memory(void);
void zmalloc_enable_thread_safeness(void);
float zmalloc_get_fragmentation_ratio(void);
size_t zmalloc_get_rss(void);
size_t zmalloc_allocations_for_size(size_t size);

 #define ZMALLOC_MAX_ALLOC_STAT 256
```
内存管理为上层分配、、重分配和释放内存提供接口。


`Redis` 的内存分配只是在 `malloc` 函数上建立一层简单封装，宏定义只是为了屏蔽平台的差异
[不同的平台主要有][1]

`tcmalloc(google)`、`jemalloc(FreeBSD)`、苹果平台
具体来说：
- 若系统中存在 `Google` 的 `TC_MALLOC` 库，则使用 `tc_malloc` 一族函数代替原本的 `malloc` 一族函数。
- 若系统中存在 `facebook` 的 `JE_MALLOC` 库，则使用 `je_malloc` 一族函数替换原来的 `malloc` 一族函数。
- 若当前系统是 `Mac` 系统或者其它系统，则使用 `<malloc/malloc.h>` 中的内存分配函数。

`Redis` 做了简要封装，实现忽略平台的差异性。同时也加入了内存的统计。

![内存结构图](https://github.com/hadoop73/Redis-/blob/master/imge/memory_struct.png)











  [1]: http://www.cnblogs.com/davidwang456/p/3504563.html
  