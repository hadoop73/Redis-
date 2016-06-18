---
title: Redis 整数集合 
tags: Redis,整数集合
grammar_cjkRuby: true
---


[TOC]

**整数集合**是集合键的底层实现之一，当一个集合只包含整数值元素，且数量不多时，`Redis` 使用整数结合作为集合键的底层实现。

##  大小端字节序
[大端模式和小端模式][1]
因为网络字节序为大端字节序，`Redis` 支持分布式，需要考虑，而一般系统上为小端字节序；所以在转换为整数的时候会需要转换
```cpp?linenums
/* Toggle the 16 bit unsigned integer pointed by *p from little endian to
 * big endian */
void memrev16(void *p) {
    unsigned char *x = p, t;

    t = x[0];
    x[0] = x[1];
    x[1] = t;
}

// 对于16位整数的调整
uint16_t intrev16(uint16_t v) {
    memrev16(&v);
    return v;
}
```

##  查找数据
**根据给定编码方式，返回给定位置的值**
```cpp?linenums
static int64_t _intsetGetEncoded(intset *is, int pos, uint8_t enc) {
    int64_t v64;
    int32_t v32;
    int16_t v16;

    if (enc == INTSET_ENC_INT64) {
        memcpy(&v64,((int64_t*)is->contents)+pos,sizeof(v64));
        memrev64ifbe(&v64);
        return v64;
    } else if (enc == INTSET_ENC_INT32) {
        memcpy(&v32,((int32_t*)is->contents)+pos,sizeof(v32));
        memrev32ifbe(&v32);
        return v32;
    } else {
        memcpy(&v16,((int16_t*)is->contents)+pos,sizeof(v16));
        memrev16ifbe(&v16);
        return v16;
    }
}
```
对一个整数集合进行查找，查找成功是，将索引保存到 `pos`，并返回 1
查找失败时，返回 0，将 `value` 可以插入的索引保存到 `pos`
```cpp?linenums
static uint8_t intsetSearch(intset *is, int64_t value, uint32_t *pos) {
    int min = 0,
        max = intrev32ifbe(is->length)-1,
        mid = -1;
    int64_t cur = -1;

    /* The value can never be found when the set is empty */
    if (intrev32ifbe(is->length) == 0) {
        // is 为空时，总是查找失败
        if (pos) *pos = 0;
        return 0;
    } else {
        /* Check for the case where we know we cannot find the value,
         * but do know the insert position. */
        if (value > _intsetGet(is,intrev32ifbe(is->length)-1)) {
            // 值比 is 中的最后一个值(所有元素中的最大值)要大
            // 那么这个值应该插入到 is 最后
            if (pos) *pos = intrev32ifbe(is->length);
            return 0;
        } else if (value < _intsetGet(is,0)) {
            // value 作为新的最小值，插入到 is 最前
            if (pos) *pos = 0;
            return 0;
        }
    }

    // 在 is 元素数组中进行二分查找 
    while(max >= min) {
        mid = (min+max)/2;
        cur = _intsetGet(is,mid);
        if (value > cur) {
            min = mid+1;
        } else if (value < cur) {
            max = mid-1;
        } else {
            break;
        }
    }

    if (value == cur) {
        if (pos) *pos = mid;
        return 1;
    } else {
        if (pos) *pos = min;
        return 0;
    }
}
```

##  整数集合的升级
**调整集合大小**
重新计算集合需要的大小，根据数组每个元素占位的字节大小，计算数组的总长度。
```cpp?linenums
/*
 * 调整 intset 的大小
 *
 * T = O(n)
 */
static intset *intsetResize(intset *is, uint32_t len) {
    uint32_t size = len*intrev32ifbe(is->encoding); // 重新计算数组的总长度
    is = zrealloc(is,sizeof(intset)+size);
    return is;
}
```
**给定 `pos` 的值设置为 `value`**
```cpp?linenums
static void _intsetSet(intset *is, int pos, int64_t value) {
    uint32_t encoding = intrev32ifbe(is->encoding);

    if (encoding == INTSET_ENC_INT64) {
        ((int64_t*)is->contents)[pos] = value;
        memrev64ifbe(((int64_t*)is->contents)+pos);
    } else if (encoding == INTSET_ENC_INT32) {
        ((int32_t*)is->contents)[pos] = value;
        memrev32ifbe(((int32_t*)is->contents)+pos);
    } else {
        ((int16_t*)is->contents)[pos] = value;
        memrev16ifbe(((int16_t*)is->contents)+pos);
    }
}
```
对整数集合进行升级，默认有两种升级方式，把新添加的元素追加到数组末尾和头部
```cpp?linenums
static intset *intsetUpgradeAndAdd(intset *is, int64_t value) {

    // 当前值的编码类型
    uint8_t curenc = intrev32ifbe(is->encoding);
    // 新值的编码类型
    uint8_t newenc = _intsetValueEncoding(value);

    // 元素数量
    int length = intrev32ifbe(is->length);

    // 决定新值插入的位置（0 为头，1 为尾）
    int prepend = value < 0 ? 1 : 0;

    //  设置新编码，并根据新编码对 intset 进行扩容
    is->encoding = intrev32ifbe(newenc);
    is = intsetResize(is,intrev32ifbe(is->length)+1);

    //  "prepend" 是为插入新值而设置的索引偏移量
    while(length--)
        _intsetSet(is,length+prepend,_intsetGetEncoded(is,length,curenc));

    /* Set the value at the beginning or the end. */
    if (prepend)
        _intsetSet(is,0,value);
    else
        _intsetSet(is,intrev32ifbe(is->length),value);
    
    // 更新 is 元素数量
    is->length = intrev32ifbe(intrev32ifbe(is->length)+1);

    return is;
}
```

##  元素的插入和删除
**移动 `memmove` 函数**
[`memmove` 函数][2]负责将 `src` 拷贝 `count` 个字符到 `dest`，目标区域和源区域有重叠，不会有覆盖现象。
```cpp?linenums
void *memmove(void *dst, const void *src, size_t count);
```
**添加函数**
先对原来的数组进行扩张，再进行移动
如果需要升级，直接升级再插入元素
如果查找到已经存在，则不做任何操作
```cpp?linenums
/*
 * 将 value 添加到集合中
 *
 * 如果元素已经存在， *success 被设置为 0 ，
 * 如果元素添加成功， *success 被设置为 1 。
 *
 * T = O(n)
 */
intset *intsetAdd(intset *is, int64_t value, uint8_t *success) {
    uint8_t valenc = _intsetValueEncoding(value);
    uint32_t pos;
    if (success) *success = 1;

    // 如果有需要，进行升级并插入新值
    if (valenc > intrev32ifbe(is->encoding)) {
        // 如果值已经存在，那么直接返回
        // 如果不存在，那么设置 *pos 设置为新元素添加的位置
        if (intsetSearch(is,value,&pos)) {
            if (success) *success = 0;
            return is;
        }

        // 扩张 is ，准备添加新元素
        is = intsetResize(is,intrev32ifbe(is->length)+1);
        // 如果 pos 不是数组中最后一个位置，
        // 那么对数组中的原有元素进行移动
        if (pos < intrev32ifbe(is->length)) intsetMoveTail(is,pos,pos+1);
    }

    // 添加新元素
    _intsetSet(is,pos,value);
    // 更新元素数量
    is->length = intrev32ifbe(intrev32ifbe(is->length)+1);

    return is;
}
```
**移动操作**
关键是计算好 `dst`、`src` 和 `bytes`。
```cpp?linenums
static void intsetMoveTail(intset *is, uint32_t from, uint32_t to) {
    void *src, *dst;

    // 需要移动的元素个数
    uint32_t bytes = intrev32ifbe(is->length)-from; 

    // 数组内元素的编码方式
    uint32_t encoding = intrev32ifbe(is->encoding);

    if (encoding == INTSET_ENC_INT64) {
        // 计算地址
        src = (int64_t*)is->contents+from;
        dst = (int64_t*)is->contents+to;
        // 需要移动的字节数
        bytes *= sizeof(int64_t);
    } else if (encoding == INTSET_ENC_INT32) {
        src = (int32_t*)is->contents+from;
        dst = (int32_t*)is->contents+to;
        bytes *= sizeof(int32_t);
    } else {
        src = (int16_t*)is->contents+from;
        dst = (int16_t*)is->contents+to;
        bytes *= sizeof(int16_t);
    }

    // 走你！
    memmove(dst,src,bytes);
}
```

**删除操作**
先查找 `value` 的位置，再进行移动操作，最后重新分配空间
```cpp?linenums
intset *intsetRemove(intset *is, int64_t value, int *success) {
    uint8_t valenc = _intsetValueEncoding(value);
    uint32_t pos;
    if (success) *success = 0;

    if (valenc <= intrev32ifbe(is->encoding) && // 编码方式匹配
        intsetSearch(is,value,&pos))            // 将位置保存到 pos
    {
        uint32_t len = intrev32ifbe(is->length);

        /* We know we can delete */
        if (success) *success = 1;

        /* Overwrite value with tail and update length */
        // 如果 pos 不是 is 的最末尾，那么显式地删除它
        // （如果 pos = (len-1) ，那么紧缩空间时值就会自动被『抹除掉』）
        if (pos < (len-1)) intsetMoveTail(is,pos+1,pos);

        // 紧缩空间，并更新数量计数器
        is = intsetResize(is,len-1);
        is->length = intrev32ifbe(len-1);
    }

    return is;
}
```
































  [1]: http://blog.csdn.net/hackbuteer1/article/details/7722667
  [2]: http://baike.baidu.com/link?url=PQl7dwlL_ezWqXDGr95sX39WYSn2KaBBYz0daszyt_BxQKvZbKPj1LShk3IBr9_2GT2URttJgtnb4RC-DkwfTa