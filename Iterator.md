# 迭代器
迭代器通过遍历获取数据库中的所有数据
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


































