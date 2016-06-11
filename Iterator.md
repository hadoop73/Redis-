# 迭代器
迭代器通过遍历获取数据库中的所有数据
迭代器用于通过哈希表实现的字典，一个哈希表里面有一个哈希表数组，数组中的节点用于保存字典中的键值对
`redis` 使用 `MurmurHash2` 算法计算键的哈希值，使用链地址法来解决键冲突
```cpp
/*
 * 字典迭代器
 *
 * 如果 safe 属性的值为 1 ，那么表示这个迭代器是一个安全迭代器。
 * 当安全迭代器正在迭代一个字典时，该字典仍然可以调用 dictAdd 、 dictFind 和其他函数。
 *
 * 如果 safe 属性的值为 0 ，那么表示这不是一个安全迭代器。
 * 如果正在运作的迭代器是不安全迭代器，那么它只可以对字典调用 dictNext 函数。
 */
typedef struct dictIterator {

    // 正在迭代的字典
    dict *d;

    int table,              // 正在迭代的哈希表的号码（0 或者 1）
        index,              // 正在迭代的哈希表数组的索引
        safe;               // 是否安全？

    dictEntry *entry,       // 当前哈希节点
              *nextEntry;   // 当前哈希节点的后继节点
} dictIterator;
```
**迭代器节点**
```cpp
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

##  迭代器实现
```cpp
/*
 * 返回迭代器指向的当前节点。
 *
 * 如果字典已经迭代完毕，返回 NULL 。
 *
 * T = O(1)
 */
dictEntry *dictNext(dictIterator *iter)
{
    while (1) {
        if (iter->entry == NULL) {

            dictht *ht = &iter->d->ht[iter->table];

            // 在开始迭代之前，增加字典 iterators 计数器的值
            // 只有安全迭代器才会增加计数
            if (iter->safe &&
                iter->index == -1 &&
                iter->table == 0)
                iter->d->iterators++;

            // 增加索引
            iter->index++;

            // 当迭代的元素数量超过 ht->size 的值
            // 说明这个表已经迭代完毕了
            if (iter->index >= (signed) ht->size) {
                // 是否接着迭代 ht[1] ?
                if (dictIsRehashing(iter->d) && iter->table == 0) {
                    iter->table++;
                    iter->index = 0;
                    ht = &iter->d->ht[1];
                } else {
                // 如果没有 ht[1] ，或者已经迭代完了 ht[1] 到达这里
                // 跳出
                    break;
                }
            }

            // 指向下一索引的节点链表
            iter->entry = ht->table[iter->index];

        } else {
            // 指向链表的下一节点
            iter->entry = iter->nextEntry;
        }

        // 保存后继指针 nextEntry，
        // 以应对当前节点 entry 可能被修改的情况
        if (iter->entry) {
            /* We need to save the 'next' here, the iterator user
             * may delete the entry we are returning. */
            iter->nextEntry = iter->entry->next;
            return iter->entry;
        }
    }
    return NULL;
}
```































