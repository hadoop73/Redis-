---
title: Redis 字典 
tags: 新建,模板,小书匠
grammar_cjkRuby: true
---


##  字典扩展

首先，确定好 `size` 的大小，`size` 必须是 `2^n` 大小
```cpp?linenums
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













