---
title: Redis  zipmap 
tags: Redis,zipmap
grammar_cjkRuby: true
---

[TOC]


##  `zipmap` 作用
`zipmap` 主要用来存储少量 `key-value` 对，一旦元素的数量超过某个给定值，它会自动转换成哈希表

##  `zipmap` 结构
对于映射 `"foo" => "bar"`, `"hello" => "world"`，`zipmap` 有以下内存结构：
`<zmlen><len>"foo"<len><free>"bar"<len>"hello"<len><free>"world"<ZIPMAP_END>`

* `zmlen` 长度 1 字节，保存 `zipmap` 当前多少个 `key-value` 对
* `len` 表示跟在后面的字符串的长度，可以 1 字节或者 5 字节，如果 `len` 第一个字节介于 0 至 252 之间，那么这个字节就是字符串的长度；如果第一个字节值为 253，那么后面 4 字节无符号整数就是字符串的长度
* `free` 表示字符串之后，未被使用的字节数量

##  `zipmap` 操作

**创建一个 `zipmap`**
```cpp?linenums
/*
 * 创建一个新的 zipmap
 */
unsigned char *zipmapNew(void) {
    unsigned char *zm = zmalloc(2);

    zm[0] = 0; /* Length */
    zm[1] = ZIPMAP_END;
    return zm;
}
```
**`key` 映射 `value`**
首先，计算保存实体需要的空间大小;判断 `klen` 和 `vlen` 需要大小和 `key、value` 需要的大小
```cpp?linenums
/*
 * 返回保存 key-value 对所需长度
 *
 * 一个 key-value 对有以下结构 <len>key<len><free>value
 * 其中两个 <len> 都为 1 字节或者 5 字节
 * 而 <free> 为一字节
 */
static unsigned long zipmapRequiredLength(unsigned int klen, unsigned int vlen) {
    unsigned int l;

    // 最少需要 3 字节加上 key 和 value 的长度
    l = klen+vlen+3;

    // 如果有需要的话，为 <len> 加上额外的空间
    if (klen >= ZIPMAP_BIGLEN) l += 4;
    if (vlen >= ZIPMAP_BIGLEN) l += 4;

    return l;
}
```
然后，查找 `key` 的位置
```cpp?linenums
//  解码 len 和 len编码的长度
/*
 * 返回实体的 <len> 值
 */
static unsigned int zipmapDecodeLength(unsigned char *p) {
    unsigned int len = *p;

    // <len> 保存在p ，直接返回
    if (len < ZIPMAP_BIGLEN) return len;

    // <len> 保存在p 之后的 4 个字节中
    memcpy(&len,p+1,sizeof(unsigned int));
    memrev32ifbe(&len);
    return len;
}
```
```cpp?linenums
/*
 * 编码长度 l ，并将它写入到 p 当中。
 * 如果 p 是 NULL ，那么它只返回编码 l 所需的字节数。
 */
static unsigned int zipmapEncodeLength(unsigned char *p, unsigned int len) {
    if (p == NULL) {
        return ZIPMAP_LEN_BYTES(len);
    } else {
        if (len < ZIPMAP_BIGLEN) {
            p[0] = len;
            return 1;
        } else {
            p[0] = ZIPMAP_BIGLEN;
            memcpy(p+1,&len,sizeof(len));
            memrev32ifbe(p+1);
            return 1+sizeof(len);
        }
    }
}
```
```cpp?linenums
/* 
 * 查找给定 key ，找到返回指向 zipmap 中某个实体的一个指针，否则返回 NULL。
 * 
 * 如果查找失败，且 totlen 不为 NULL ，
 * 那么将 totlen 设置为 zipmap 实体的大小(size)，
 * 因此，调用函数可以通过对原 zipmap 进行重分配，从
 * 而为 zipmap 的实体分配更多空间。
 */
static unsigned char *zipmapLookupRaw(
    unsigned char *zm,
    unsigned char *key,
    unsigned int klen,
    unsigned int *totlen
) 
{
    unsigned char *p = zm+1,    // 掠过 <zmlen> ，指向第一个实体
                  *k = NULL;
    unsigned int l,llen;

    // 遍历 zipmap
    while(*p != ZIPMAP_END) {
        unsigned char free;

        /* Match or skip the key */
        // 获取实体的 <len> 长度
        l = zipmapDecodeLength(p);
        // 计算编码 <len> 所需的长度
        llen = zipmapEncodeLength(NULL,l);

        if (key != NULL &&          // key 不为空
            k == NULL &&            // 还没找到过给定 key
            l == klen &&            // 实体的 <len> 和 klen 相同
            !memcmp(p+llen,key,l)   // key 匹配
        ) 
        {
            // 匹配成功
            /* Only return when the user doesn't care
             * for the total length of the zipmap. */
            if (totlen != NULL) {
                k = p;
                // 当执行到这里，再计算下去已经没有意义了
                // 可以考虑用一个 goto 跳到 if (totlen != NUL) ...
                // goto key_founded;
            } else {
                return p;
            }
        }

        // 跳过 key
        p += llen+l;

        // 跳过 value
        l = zipmapDecodeLength(p);          // value 的长度
        p += zipmapEncodeLength(NULL,l);    // 跳过 value 的 <len> 的长度
        free = p[0];                        // 获取 <free> 的值
        p += l+1+free; // 跳过 value ， <free> 字节，以及 <free> 的长度
    }
   
    // key_founded:

    if (totlen != NULL) *totlen = (unsigned int)(p-zm)+1;

    return k;
}
```
































































