---
title: Redis 压缩列表(索引)
tags: Redis,压缩列表,索引
grammar_cjkRuby: true
---


[TOC]

支持从前和从后面根据索引获取节点
```cpp?linenums
/*
 * 返回指向当前迭代节点的指针，用于 ziplistNext 进行迭代
 * 当偏移值是负数时，表示迭代是从表尾到表头进行的。
 * 当元素被遍历完时，返回 NULL
 *
 * 复杂度：O(1)
 *
 * 返回值：指向节点的指针
 */
unsigned char *ziplistIndex(unsigned char *zl, int index) {

    unsigned char *p;

    zlentry entry;

    // 向前遍历
    if (index < 0) {
        index = (-index)-1;
        p = ZIPLIST_ENTRY_TAIL(zl);
        // 如果 ziplist 不为空。。。
        if (p[0] != ZIP_END) {
            // 那么根据 entry.prevrawlen ，进行向前迭代
            entry = zipEntry(p);
            while (entry.prevrawlen > 0 && index--) {
                // 后退地址
                p -= entry.prevrawlen;
                entry = zipEntry(p);
            }
        }
    // 向后遍历
    } else {
        p = ZIPLIST_ENTRY_HEAD(zl);
        // 根据 entry.prevrawlen 向后进行迭代
        while (p[0] != ZIP_END && index--) {
            p += zipRawEntryLength(p);
        }
    }

    return (p[0] == ZIP_END || index > 0) ? NULL : p;
}
```
```cpp?linenums
/*
 * 返回 p 指向的节点的空间总长度
 * 
 * 复杂度：O(1)
 */
static unsigned int zipRawEntryLength(unsigned char *p) {
    unsigned int prevlensize, encoding, lensize, len;

    // 保存前驱节点长度的空间长度
    ZIP_DECODE_PREVLENSIZE(p, prevlensize);

    // 保存本节点的空间长度
    ZIP_DECODE_LENGTH(p + prevlensize, encoding, lensize, len);
    
    return prevlensize + lensize + len;
}
```
##  删除节点
**统计能够删除的节点数**
```cpp?linenums
// 首个节点
first = zipEntry(p);
// 累积起所有删除目标（节点）的编码长度
// 并移动指针 p 
// 复杂度 O(N)
for (i = 0; p[0] != ZIP_END && i < num; i++) {
    p += zipRawEntryLength(p);
    deleted++;
}
// 被删除的节点的 byte 总和
totlen = p-first.p;
```
**更新后面后面节点的 `prevlen`**
首先确定后面节点的 `prevlen` 是否满足 `first.prevlen`；也就是删除中间节点时能否连起来
如果存在差值则需要调整字节数，也就是移动 `p`
最后再把 `first.prevlen` 的值赋予给 `p`
```cpp?linenums
// 更新最后一个被删除的节点之后的一个节点，
// 将它的 prevlan 值设置为 first.prevrawlen ，
// 也即是被删除的第一个节点的前一个节点的长度
nextdiff = zipPrevLenByteDiff(p,first.prevrawlen);
p -= nextdiff;
zipPrevEncodeLength(p,first.prevrawlen);
```
**表尾偏移量变化**
假如最后只有一个节点，因为尾节点距离表头的距离是不变的，虽然 `prevlen` 有所变化，但是不影响
如果不是最后一个节点，则需要考虑 `prevlen` 字节数的变化
```cpp?linenums
/* Update offset for tail */
// 更新 ziplist 到表尾的偏移量
// 此时 p 指向表尾最后一个节点，不影响表尾偏移量
ZIPLIST_TAIL_OFFSET(zl) =
    intrev32ifbe(intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))-totlen);

/* When the tail contains more than one entry, we need to take
    * "nextdiff" in account as well. Otherwise, a change in the
    * size of prevlen doesn't have an effect on the *tail* offset. */
// 跟新 ziplist 的偏移量，如果有需要的话，算上 nextdiff
tail = zipEntry(p);
if (p[tail.headersize+tail.len] != ZIP_END) {
    ZIPLIST_TAIL_OFFSET(zl) =
            intrev32ifbe(intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))+nextdiff);
        }

/* Move tail to the front of the ziplist */
// 前移内存中的数据，覆盖原本的被删除数据
// 复杂度 O(N)
// 并没有拷贝表尾，因为在重新分配空间时会加上
memmove(first.p,p,
        intrev32ifbe(ZIPLIST_BYTES(zl))-(p-zl)-1);
```
**节点元素比较**
如果都是字符串比较，可以先判定表中是否为字符串，然后用 `memcmp()` 函数比较
如果是整数，则判断另一个需要比较的项是否能转换为整数，然后做相等比较
```cpp?linenums
/*
 * 将 p 所指向的节点的属性和 sstr 以及 slen 进行对比，
 * 如果相等则返回 1 。
 *
 * 复杂度：O(N)
 */
unsigned int ziplistCompare(unsigned char *p, unsigned char *sstr, unsigned int slen) {
    zlentry entry;
    unsigned char sencoding;
    long long zval, sval;

    // p 是表尾？
    if (p[0] == ZIP_END) return 0;

    // 获取节点属性
    entry = zipEntry(p);
    // 对比字符串
    if (ZIP_IS_STR(entry.encoding)) {
        /* Raw compare */
        if (entry.len == slen) {
            // O(N)
            return memcmp(p+entry.headersize,sstr,slen) == 0;
        } else {
            return 0;
        }
    // 对比整数
    } else {
        /* Try to compare encoded values. Don't compare encoding because
         * different implementations may encoded integers differently. */
        if (zipTryEncoding(sstr,slen,&sval,&sencoding)) {
          zval = zipLoadInteger(p+entry.headersize,entry.encoding);
          return zval == sval;
        }
    }

    return 0;
}
```
**节点查找**
可能会先忽略 `skip` 个节点再开始查找
查找时，依据是否为字符串比较和整数比较获得结果
```cpp?linenums
/*
 * 根据给定的 vstr 和 vlen ，查找和属性和它们相等的节点
 * 在每次比对之间，跳过 skip 个节点。
 *
 * 复杂度：O(N)
 * 返回值：
 *  查找失败返回 NULL 。
 *  查找成功返回指向目标节点的指针
 */
unsigned char *ziplistFind(unsigned char *p, unsigned char *vstr, unsigned int vlen, unsigned int skip) {
    int skipcnt = 0;
    unsigned char vencoding = 0;
    long long vll = 0;

    // 遍历整个列表
    while (p[0] != ZIP_END) {
        unsigned int prevlensize, encoding, lensize, len;
        unsigned char *q;

        // 编码前一个节点的长度所需的空间
        ZIP_DECODE_PREVLENSIZE(p, prevlensize);
        // 当前节点的长度
        ZIP_DECODE_LENGTH(p + prevlensize, encoding, lensize, len);
        // 保存下一个节点的地址
        q = p + prevlensize + lensize;

        if (skipcnt == 0) {
            /* Compare current entry with specified entry */
            // 对比字符串
            if (ZIP_IS_STR(encoding)) {
                if (len == vlen && memcmp(q, vstr, vlen) == 0) {
                    return p;
                }
            // 对比整数
            } else {
                /* Find out if the searched field can be encoded. Note that
                 * we do it only the first time, once done vencoding is set
                 * to non-zero and vll is set to the integer value. */
                // 对传入值进行 decode
                if (vencoding == 0) {
                    if (!zipTryEncoding(vstr, vlen, &vll, &vencoding)) {
                        /* If the entry can't be encoded we set it to
                         * UCHAR_MAX so that we don't retry again the next
                         * time. */
                        vencoding = UCHAR_MAX;
                    }
                    /* Must be non-zero by now */
                    assert(vencoding);
                }

                /* Compare current entry with specified entry, do it only
                 * if vencoding != UCHAR_MAX because if there is no encoding
                 * possible for the field it can't be a valid integer. */
                if (vencoding != UCHAR_MAX) {
                    // 对比
                    long long ll = zipLoadInteger(q, encoding);
                    if (ll == vll) {
                        return p;
                    }
                }
            }

            /* Reset skip count */
            skipcnt = skip;
        } else {
            /* Skip entry */
            skipcnt--;
        }

        /* Move to next entry */
        p = q + len;
    }

    return NULL;
}
```
**获得压缩列表总长度**
需要判断长度是否大于 `UINT16_MAX`，因为长度保存的是16位长度。
```cpp?linenums
/*
 * 返回 ziplist 的长度
 *
 * 复杂度：O(N)
 */
unsigned int ziplistLen(unsigned char *zl) {
    unsigned int len = 0;
    // 节点的数量 < UINT16_MAX
    if (intrev16ifbe(ZIPLIST_LENGTH(zl)) < UINT16_MAX) {
        // 长度保存在一个 uint16 整数中
        len = intrev16ifbe(ZIPLIST_LENGTH(zl));
    // 节点的数量 >= UINT16_MAX
    } else {
        // 遍历整个 ziplist ，计算长度
        unsigned char *p = zl+ZIPLIST_HEADER_SIZE;
        while (*p != ZIP_END) {
            p += zipRawEntryLength(p);
            len++;
        }

        /* Re-store length if small enough */
        if (len < UINT16_MAX) ZIPLIST_LENGTH(zl) = intrev16ifbe(len);
    }
    return len;
}
```




































