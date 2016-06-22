---
title: Redis 字典 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---

[TOC]


[redis-dict-implement.md][1]

[Redis字典][2]

[【Redis基本数据结构】字典实现][3]

[Redis内部数据结构详解之字典(dict)][4]

`Redis` 的数据库就是使用字典来作为底层实现的，对数据库的增、删、查、改操作是构建在对字典的操作之上。

![字典结构][5]

**字典的实现**
```c?linenums
/* 
 * 字典 
 * 每个字典使用两个哈希表，用于实现渐进式 rehash 
 */  
typedef struct dict {  
    // 特定于类型的处理函数  
    dictType *type;  
    // 类型处理函数的私有数据  
    void *privdata;  
    // 哈希表（2个）  
    dictht ht[2];         
    // 记录 rehash 进度的标志，值为-1 表示 rehash 未进行  
    int rehashidx;  
    // 当前正在运作的安全迭代器数量  
    int iterators;        
} dict;
```
`dict` 类型使用了两个指向哈希表的指针。
* 哈希表 `ht[0]` 是字典使用的哈希表
* 哈希表 `ht[1]` 是用来 `rehash` 的哈希表

**哈希表的定义**
```c?linenums
/* 
 * 哈希表 
 */  
typedef struct dictht {  
    // 哈希表节点指针数组（俗称桶，bucket）  
    dictEntry **table;        
    // 指针数组的大小  
    unsigned long size;       
    // 指针数组的长度掩码，用于计算索引值  
    unsigned long sizemask;   
    // 哈希表现有的节点数量  
    unsigned long used;       
} dictht;
```
哈希表中 `table` 指针，用于记录所有的键值对节点

**哈希表节点**
```c?linenums
/* 
 * 哈希表节点 
 */  
typedef struct dictEntry {  
    // 键  
    void *key;  
    // 值  
    union {  
        void *val;  
        uint64_t u64;  
        int64_t s64;  
    } v;  
    // 链往后继节点  
    struct dictEntry *next;   
} dictEntry;
```
`next` 指向下一个相同哈希值的哈希表节点。


##  `dictType` 
定义一组回调函数，进行数据节点的操作
```c?linenums
typedef struct dictType {
    unsigned int (*hashFunction)(const void *key);  // 生成hashid
    void *(*keyDup)(void *privdata, const void *key); //生成key
    void *(*valDup)(void *privdata, const void *obj); //生成val
    int (*keyCompare)(void *privdata, const void *key1, const void *key2); //key比较
    void (*keyDestructor)(void *privdata, void *key); //销毁key
    void (*valDestructor)(void *privdata, void *obj); //销毁val
} dictType;
```
可以看出，正是这个设计将dict结构体变得异常灵活，用户可以通过自定义dictType内容实现dict的多样性。
```cpp?linenums
/* set集合 */
dictType setDictType = {
    dictEncObjHash,            /* hash function */
    NULL,                      /* key dup */
    NULL,                      /* val dup */
    dictEncObjKeyCompare,      /* key compare */
    dictRedisObjectDestructor, /* key destructor */
    NULL                       /* val destructor */
};

/* zset集合 */
dictType zsetDictType = {
    dictEncObjHash,            /* hash function */
    NULL,                      /* key dup */
    NULL,                      /* val dup */
    dictEncObjKeyCompare,      /* key compare */
    dictRedisObjectDestructor, /* key destructor */
    NULL                       /* val destructor */
};

/* 全局key存储空间 */
dictType dbDictType = {
    dictSdsHash,                /* hash function */
    NULL,                       /* key dup */
    NULL,                       /* val dup */
    dictSdsKeyCompare,          /* key compare */
    dictSdsDestructor,          /* key destructor */
    dictRedisObjectDestructor   /* val destructor */
};

/* 全局过期对象存储空间 */
dictType keyptrDictType = {
    dictSdsHash,               /* hash function */
    NULL,                      /* key dup */
    NULL,                      /* val dup */
    dictSdsKeyCompare,         /* key compare */
    NULL,                      /* key destructor */
    NULL                       /* val destructor */
};

/* 命令对象 */
dictType commandTableDictType = {
    dictSdsCaseHash,           // 命令是大小写不敏感的
    NULL,                      /* key dup */
    NULL,                      /* val dup */
    dictSdsKeyCaseCompare,     // 命令是大小写不敏感的
    dictSdsDestructor,         /* key destructor */
    NULL                       /* val destructor */
};

/* hash结构 */
dictType hashDictType = {
    dictEncObjHash,             /* hash function */
    NULL,                       /* key dup */
    NULL,                       /* val dup */
    dictEncObjKeyCompare,       /* key compare */
    dictRedisObjectDestructor,  /* key destructor */
    dictRedisObjectDestructor   /* val destructor */
};

/* val为链表的dict结构
   主要是 expire、watch、pubsub 等功能会用到*/
dictType keylistDictType = {
    dictObjHash,                /* hash function */
    NULL,                       /* key dup */
    NULL,                       /* val dup */
    dictObjKeyCompare,          /* key compare */
    dictRedisObjectDestructor,  /* key destructor */
    dictListDestructor          /* val destructor */
};
```








##  创建新字典
`dict.c` 中创建字典的方法 `dictCreate`
* 分配空间，初始化字典(初始化type,privdata,rehashidx)
* 初始化哈希表(初始化table,size,sizemask)

```c?linenums
/* 
 * 创建一个新字典 
 * T = O(1) 
 */  
dict *dictCreate(dictType *type,  
        void *privDataPtr)  
{  
    // 分配空间  
    dict *d = zmalloc(sizeof(*d));  
    // 初始化字典  
    _dictInit(d,type,privDataPtr);  
    return d;  
}  

/* 
 * 初始化字典 
 * T = O(1) 
 */  
int _dictInit(dict *d, dictType *type,  
        void *privDataPtr)  
{  
    // 初始化 ht[0]  
    _dictReset(&d->ht[0]);  
    // 初始化 ht[1]  
    _dictReset(&d->ht[1]);  
    // 初始化字典属性  
    d->type = type;  
    d->privdata = privDataPtr;  
    d->rehashidx = -1;  
    d->iterators = 0;  
    return DICT_OK;  
}  

/* 
 * 重置哈希表的各项属性 
 * 
 * T = O(1) 
 */  
static void _dictReset(dictht *ht)  
{  
    ht->table = NULL;  
    ht->size = 0;  
    ht->sizemask = 0;  
    ht->used = 0;  
}  
dict *d = dictCreate(&hash_type, NULL);
```
![初始化字典][6]

##  插入数据
插入节点的时候
* 先判断 `key` 是否存在于 `ht[0]` 或 `ht[1]`，通过 `hash` 函数计算哈希值，判断哈希值相同的链表中是否有 `key`
* `key` 还不存在，则创建一个节点，并插入
```c?linenums
int dictAdd(dict *d, void *key, void *val)
{
    // 尝试添加键到字典，并返回包含了这个键的新哈希节点
    // T = O(N)
    dictEntry *entry = dictAddRaw(d,key);

    // 键已存在，添加失败
    if (!entry) return DICT_ERR;

    // 键不存在，设置节点的值
    // T = O(1)
    dictSetVal(d, entry, val);

    // 添加成功
    return DICT_OK;
}

dictEntry *dictAddRaw(dict *d, void *key)
{
    int index;
    dictEntry *entry;
    dictht *ht;

    // 如果条件允许的话，进行单步 rehash
    // T = O(1)
    if (dictIsRehashing(d)) _dictRehashStep(d);

    /* Get the index of the new element, or -1 if
     * the element already exists. */
    // 计算键在哈希表中的索引值
    // 如果值为 -1 ，那么表示键已经存在
    // T = O(N)
    if ((index = _dictKeyIndex(d, key)) == -1)
        return NULL;

    // T = O(1)
    /* Allocate the memory and store the new entry */
    // 如果字典正在 rehash ，那么将新键添加到 1 号哈希表
    // 否则，将新键添加到 0 号哈希表
    ht = dictIsRehashing(d) ? &d->ht[1] : &d->ht[0];
    // 为新节点分配空间
    entry = zmalloc(sizeof(*entry));
    // 将新节点插入到链表表头
    entry->next = ht->table[index];
    ht->table[index] = entry;
    // 更新哈希表已使用节点数量
    ht->used++;

    /* Set the hash entry fields. */
    // 设置新节点的键
    // T = O(1)
    dictSetKey(d, entry, key);

    return entry;
}

static int _dictKeyIndex(dict *d, const void *key)
{
    unsigned int h, idx, table;
    dictEntry *he;

    /* Expand the hash table if needed */
    // 单步 rehash
    // T = O(N)
    if (_dictExpandIfNeeded(d) == DICT_ERR)
        return -1;

    /* Compute the key hash value */
    // 计算 key 的哈希值
    h = dictHashKey(d, key);
    // T = O(1)
    for (table = 0; table <= 1; table++) {

        // 计算索引值
        idx = h & d->ht[table].sizemask;

        /* Search if this slot does not already contain the given key */
        // 查找 key 是否存在
        // T = O(1)
        he = d->ht[table].table[idx];
        while(he) {
            if (dictCompareKeys(d, key, he->key))
                return -1;
            he = he->next;
        }

        // 如果运行到这里时，说明 0 号哈希表中所有节点都不包含 key
        // 如果这时 rehahs 正在进行，那么继续对 1 号哈希表进行 rehash
        if (!dictIsRehashing(d)) break;
    }

    // 返回索引值
    return idx;
}
```

##  字典扩展

首先，确定好 `size` 的大小，`size` 必须是 `2^n` 大小
```c?linenums
/*
 * 计算哈希表的真实体积
 *
 * 如果 size 小于等于 DICT_HT_INITIAL_SIZE ，
 * 那么返回 DICT_HT_INITIAL_SIZE ，
 * 否则这个值为第一个 >= size 的二次幂。
 *
 * T = O(N)
 */
static unsigned long _dictNextPower(unsigned long size)
{
    unsigned long i = DICT_HT_INITIAL_SIZE;

    if (size >= LONG_MAX) return LONG_MAX;
    while(1) {
        if (i >= size)
            return i;
        i *= 2;
    }
}
```
确定好 `size`，再为 `dictht` 的 `dictEntry` 分配空间

```cpp?linenums
/* Expand or create the hash table */
/*
 * 创建一个新哈希表，并视情况，进行以下动作之一：
 *  
 *   1) 如果字典里的 ht[0] 为空，将新哈希表赋值给它
 *   2) 如果字典里的 ht[0] 不为空，那么将新哈希表赋值给 ht[1] ，并打开 rehash 标识
 *
 * T = O(N)
 */
int dictExpand(dict *d, unsigned long size)
{
    dictht n; /* the new hash table */
    
    // 计算哈希表的真实大小
    // O(N)
    unsigned long realsize = _dictNextPower(size);

    /* the size is invalid if it is smaller than the number of
     * elements already inside the hash table */
    if (dictIsRehashing(d) || d->ht[0].used > size)
        return DICT_ERR;

    /* Allocate the new hash table and initialize all pointers to NULL */
    // 创建并初始化新哈希表
    // O(N)
    n.size = realsize;
    n.sizemask = realsize-1;
    n.table = zcalloc(realsize*sizeof(dictEntry*));
    n.used = 0;

    /* Is this the first initialization? If so it's not really a rehashing
     * we just set the first hash table so that it can accept keys. */
    // 如果 ht[0] 为空，那么这就是一次创建新哈希表行为
    // 将新哈希表设置为 ht[0] ，然后返回
    if (d->ht[0].table == NULL) {
        d->ht[0] = n;
        return DICT_OK;
    }

    /* Prepare a second hash table for incremental rehashing */
    // 如果 ht[0] 不为空，那么这就是一次扩展字典的行为
    // 将新哈希表设置为 ht[1] ，并打开 rehash 标识
    d->ht[1] = n;
    d->rehashidx = 0;

    return DICT_OK;
}
```
字典的 `rehash`，把 `h[0]` 中的内容重新映射到 `h[1]`
在字典表头中 `d->rehashidx` 表示 `d->h[0]` 中的 `dictEntry` 索引。

对于其中每一个 `dictEntry` 重新计算哈希值，再调整指针放在 `h[1]` 下面。
```cpp?linenums
/* Get the index in the new hash table */
// 计算元素在 ht[1] 的哈希值
h = dictHashKey(d, de->key) & d->ht[1].sizemask;

// 添加节点到 ht[1] ，调整指针
de->next = d->ht[1].table[h];
d->ht[1].table[h] = de;
```
把 `ht[0]` 中的一个索引都处理后，设置指针为 `NULL`
```cpp?linenums
// 设置指针为 NULL ，方便下次 rehash 时跳过
d->ht[0].table[d->rehashidx] = NULL;

// 前进至下一索引
d->rehashidx++;
```
在处理完 `ht[0]` 所有的项后，把 `ht[1]` 转移给 `ht[0]`，清空 `ht[1]`
```cpp?linenums
// 如果 ht[0] 已经为空，那么迁移完毕
// 用 ht[1] 代替原来的 ht[0]
if (d->ht[0].used == 0) {

    // 释放 ht[0] 的哈希表数组
    zfree(d->ht[0].table);

    // 将 ht[0] 指向 ht[1]
    d->ht[0] = d->ht[1];

    // 清空 ht[1] 的指针
    _dictReset(&d->ht[1]);

    // 关闭 rehash 标识
    d->rehashidx = -1;

    // 通知调用者， rehash 完毕
    return 0;
}
```

```cpp?linenums
/*
 * 执行 N 步渐进式 rehash 。
 *
 * 如果执行之后哈希表还有元素需要 rehash ，那么返回 1 。
 * 如果哈希表里面所有元素已经迁移完毕，那么返回 0 。
 *
 * 每步 rehash 都会移动哈希表数组内某个索引上的整个链表节点，
 * 所以从 ht[0] 迁移到 ht[1] 的 key 可能不止一个。
 *
 * T = O(N)
 */
int dictRehash(dict *d, int n) {
    if (!dictIsRehashing(d)) return 0;

    while(n--) {
        dictEntry *de, *nextde;

        // 如果 ht[0] 已经为空，那么迁移完毕
        // 用 ht[1] 代替原来的 ht[0]
        if (d->ht[0].used == 0) {

            // 释放 ht[0] 的哈希表数组
            zfree(d->ht[0].table);

            // 将 ht[0] 指向 ht[1]
            d->ht[0] = d->ht[1];

            // 清空 ht[1] 的指针
            _dictReset(&d->ht[1]);

            // 关闭 rehash 标识
            d->rehashidx = -1;

            // 通知调用者， rehash 完毕
            return 0;
        }

        /* Note that rehashidx can't overflow as we are sure there are more
         * elements because ht[0].used != 0 */
        assert(d->ht[0].size > (unsigned)d->rehashidx);
        // 移动到数组中首个不为 NULL 链表的索引上
        while(d->ht[0].table[d->rehashidx] == NULL) d->rehashidx++;
        // 指向链表头
        de = d->ht[0].table[d->rehashidx];
        // 将链表内的所有元素从 ht[0] 迁移到 ht[1]
        // 因为桶内的元素通常只有一个，或者不多于某个特定比率
        // 所以可以将这个操作看作 O(1)
        while(de) {
            unsigned int h;

            nextde = de->next;

            /* Get the index in the new hash table */
            // 计算元素在 ht[1] 的哈希值
            h = dictHashKey(d, de->key) & d->ht[1].sizemask;

            // 添加节点到 ht[1] ，调整指针
            de->next = d->ht[1].table[h];
            d->ht[1].table[h] = de;

            // 更新计数器
            d->ht[0].used--;
            d->ht[1].used++;

            de = nextde;
        }

        // 设置指针为 NULL ，方便下次 rehash 时跳过
        d->ht[0].table[d->rehashidx] = NULL;

        // 前进至下一索引
        d->rehashidx++;
    }

    // 通知调用者，还有元素等待 rehash
    return 1;
}
```


  [1]: https://github.com/hadoop73/book/blob/master/redis-dict-implement.md
  [2]: http://www.jianshu.com/p/bc59baefadb7#
  [3]: https://segmentfault.com/a/1190000004850844
  [4]: http://blog.csdn.net/acceptedxukai/article/details/17484431
  [5]: ./images/1466496554841.jpg "1466496554841.jpg"
  [6]: ./images/1466499497552.jpg "1466499497552.jpg"