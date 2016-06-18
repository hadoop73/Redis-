---
title: Redis 压缩列表 
tags: Redis,压缩列表,插入数据
grammar_cjkRuby: true
---


[TOC]

**压缩列表** 是列表键和哈希键的底层实现之一。当一个列表键只包含少量列表项，每个列表项要么是小整数值，要么是长度比较短的字符串，那么 `Redis` 就会用压缩列表做列表键的底层实现。

```bash?linenums
redis> RPUSH lst 1 2 5 10086 "hello" "world"
(integer)6

redis> OBJECT ENCODING lst
"ziplist"
```

##  压缩列表的构成
![压缩列表结构][1]

**压缩列表的构造**
```cpp?linenums
/*
 * 新创建一个空 ziplist
 * 
 * 复杂度：O(1)
 *
 * 返回值：新创建的 ziplist
 */
// 取出列表以字节计算的列表长度(内存的 0 - 31 位，整数)
#define ZIPLIST_BYTES(zl)       (*((uint32_t*)(zl)))   

// 取出列表的表尾偏移量(内存的 32 - 63 位，整数)
#define ZIPLIST_TAIL_OFFSET(zl) (*((uint32_t*)((zl)+sizeof(uint32_t))))

// 取出列表的长度(内存的 64 - 79 位，整数)
#define ZIPLIST_LENGTH(zl)      (*((uint16_t*)((zl)+sizeof(uint32_t)*2)))

// 列表的 header 长度
#define ZIPLIST_HEADER_SIZE     (sizeof(uint32_t)*2+sizeof(uint16_t))   // 32*2 bit + 16 bit

// 表示压缩列表的表尾
#define ZIP_END 255
unsigned char *ziplistNew(void) {
    // 分配 2 个 32 bit，一个 16 bit，以及一个 8 bit
    // 分别用于 <zlbytes><zltail><zllen> 和 <zlend>
    unsigned int bytes = ZIPLIST_HEADER_SIZE+1;
    unsigned char *zl = zmalloc(bytes);

    // 设置长度
    ZIPLIST_BYTES(zl) = intrev32ifbe(bytes);

    // 设置表尾偏移量
    ZIPLIST_TAIL_OFFSET(zl) = intrev32ifbe(ZIPLIST_HEADER_SIZE);

    // 设置列表项数量
    ZIPLIST_LENGTH(zl) = 0;

    // 设置表尾标识
    zl[bytes-1] = ZIP_END;

    return zl;
}
```
##  插入节点
```cpp?linenums
#define ZIPLIST_HEAD 0  // 表示插入头部
#define ZIPLIST_TAIL 1  // 表示插入尾部

// 列表的 header 长度
#define ZIPLIST_HEADER_SIZE     (sizeof(uint32_t)*2+sizeof(uint16_t))   // 32*2 bit + 16 bit
// 返回列表的 header 之后的位置
#define ZIPLIST_ENTRY_HEAD(zl)  ((zl)+ZIPLIST_HEADER_SIZE)
// 返回列表的结束符之前的位置
#define ZIPLIST_ENTRY_END(zl)   ((zl)+intrev32ifbe(ZIPLIST_BYTES(zl))-1)
/*
 * 将新元素插入为列表的表头节点或者表尾节点
 *
 * 复杂度：O(N^2)
 *
 * 返回值：添加操作完成后的 ziplist
 */
unsigned char *ziplistPush(unsigned char *zl, unsigned char *s, unsigned int slen, int where) {
    unsigned char *p;
    // p为要插入位置的指针
    p = (where == ZIPLIST_HEAD) ? ZIPLIST_ENTRY_HEAD(zl) : ZIPLIST_ENTRY_END(zl);
    return __ziplistInsert(zl,p,s,slen);
}

```
插入节点重点在于如何维护原来的压缩列表，插入的过程：先给压缩列表分配新节点的空间长度，然后再构建数据
**压缩列表节点**
```cpp?linenums
/*
 * 节点结构
 */
typedef struct zlentry {
    unsigned int prevrawlensize,    // 保存前一节点的长度所需的长度 
                 prevrawlen;        // 前一节点的长度
    unsigned int lensize,           // 保存节点的长度所需的长度
                 len;               // 节点的长度
    unsigned int headersize;        // header 长度
    unsigned char encoding;         // 编码方式
    unsigned char *p;               // 内容
} zlentry;
```

![enter description here][2]

**`previous_entry_length`** 记录压缩列表中前一个节点的长度；可以是 1 字节或者 5 字节。前一个节点的长度大于等于254字节，那么 `previous_entry_lenght` 属性长度为 5 字节：其中属性的第一个字节会被设置为 `0xFE(十进制254)` ，而后 4 字节用于保存前一个字节的长度。

**`encoding`** 记录了节点 `content` 属性所保存数据的类型以及长度。
值的最高位以 11 开头的是整数编码：表示节点的 `content` 属性保存着整数值。
一字节、两字节或五字节长，值的最高位以 00、01 或者 10 的是字节数组编码：表示节点 `content` 保存着字节数组，数组的长度由编码除去最高两位之后的其他位。

**获得 `zlentry` 节点**
首先确定 `previous_entry_length` 是 1 个或 5 个字节
```cpp?linenums
#define ZIP_DECODE_PREVLENSIZE(ptr, prevlensize) do {                          \
    if ((ptr)[0] < ZIP_BIGLEN) {                                               \
        (prevlensize) = 1;                                                     \
    } else {                                                                   \
        (prevlensize) = 5;                                                     \
    }                                                                          \
} while(0);
```
然后，获取长度
```cpp?linenums
#define ZIP_DECODE_PREVLEN(ptr, prevlensize, prevlen) do {                     \
    /* 取得保存前一个节点的长度所需的字节数 */                                 \
    ZIP_DECODE_PREVLENSIZE(ptr, prevlensize);                                  \
    /* 获取长度值 */                                                           \
    if ((prevlensize) == 1) {                                                  \
        (prevlen) = (ptr)[0];                                                  \
    } else if ((prevlensize) == 5) {                                           \
        assert(sizeof((prevlensize)) == 4);                                    \
        memcpy(&(prevlen), ((char*)(ptr)) + 1, 4);                             \
        memrev32ifbe(&prevlen);                                                \
    }                                                                          \
} while(0);
// 其中，长度的获取采用  memcpy(&(prevlen), ((char*)(ptr)) + 1, 4); 
```


```cpp?linenums
/* Return a struct with all information about an entry. */
/*
 * 从指针 p 中提取出节点的各个属性，并将属性保存到 zlentry 结构，然后返回
 *
 * 复杂度：O(1)
 */
static zlentry zipEntry(unsigned char *p) {
    zlentry e;

    // 取出前一个节点的长度
    ZIP_DECODE_PREVLEN(p, e.prevrawlensize, e.prevrawlen);

    // 取出当前节点的编码、保存节点的长度所需的长度、以及节点的长度
    ZIP_DECODE_LENGTH(p + e.prevrawlensize, e.encoding, e.lensize, e.len);
    
    // 记录 header 的长度
    e.headersize = e.prevrawlensize + e.lensize;

    // 记录指针 p
    e.p = p;

    return e;
}
```
**和书本上的变动**
从 `zlentry` 的代码可以看出，`encoding` 只有一个字节，主要用来标记节点保存的是**整数或字符串**。
`(encoding) < ZIP_STR_MASK` 表示字符串编码
`(encoding) == ZIP_STR_MASK` 表示整数编码

**获取 `encoding` 解码**
`encoding` 只用到 `ptr` 中的前两位。
```cpp?linenums
#define ZIP_STR_MASK 0xc0   // 1100, 0000
#define ZIP_ENTRY_ENCODING(ptr, encoding) do {  \
    (encoding) = (ptr[0]); \
    if ((encoding) < ZIP_STR_MASK) (encoding) &= ZIP_STR_MASK; \
} while(0)
```
如何 `content` 保存的是字符串，根据 `encoding` 的值来获取 `len`。
```cpp?linenums
#define ZIP_STR_06B (0 << 6)
#define ZIP_STR_14B (1 << 6)
#define ZIP_STR_32B (2 << 6)
```
```cpp?linenums
#define ZIP_DECODE_LENGTH(ptr, encoding, lensize, len) do {                    \
    /* 取出节点的编码 */                                                       \
    ZIP_ENTRY_ENCODING((ptr), (encoding));                                     \
    if ((encoding) < ZIP_STR_MASK) {                                           \
        /* 节点保存的是字符串，取出编码 */                                     \
        if ((encoding) == ZIP_STR_06B) {                                       \
            (lensize) = 1;                                                     \
            (len) = (ptr)[0] & 0x3f;                                           \
        } else if ((encoding) == ZIP_STR_14B) {                                \
            (lensize) = 2;                                                     \
            (len) = (((ptr)[0] & 0x3f) << 8) | (ptr)[1];                       \
        } else if (encoding == ZIP_STR_32B) {                                  \
            (lensize) = 5;                                                     \
            (len) = ((ptr)[1] << 24) |                                         \
                    ((ptr)[2] << 16) |                                         \
                    ((ptr)[3] <<  8) |                                         \
                    ((ptr)[4]);                                                \
        } else {                                                               \
            assert(NULL);                                                      \
        }                                                                      \
    } else {                                                                   \
        /* 节点保存的是整数，取出编码 */                                       \
        (lensize) = 1;                                                         \
        (len) = zipIntSize(encoding);                                          \
    }                                                                          \
} while(0);
```
**插入节点，判断插入位置**
```cpp?linenums
#define ZIP_DECODE_PREVLENSIZE(ptr, prevlensize) do {                          \
    if ((ptr)[0] < ZIP_BIGLEN) {                                               \
        (prevlensize) = 1;                                                     \
    } else {                                                                   \
        (prevlensize) = 5;                                                     \
    }                                                                          \
} while(0);
```
```cpp?linenums
/*
 * 从 ptr 指针中取出编码类型，并将它保存到 encoding
 *
 * 复杂度：O(1)
 */
#define ZIP_STR_MASK 0xc0   // 1100, 0000
#define ZIP_ENTRY_ENCODING(ptr, encoding) do {  \
    (encoding) = (ptr[0]); \
    if ((encoding) < ZIP_STR_MASK) (encoding) &= ZIP_STR_MASK; \
} while(0)
```
```cpp?linenums
/*
 * 从 ptr 指针中取出节点的编码、保存节点长度所需的长度、以及节点的长度
 *
 * 复杂度：O(1)
 *
 * 返回值：
 *  int 编码节点所需的长度
 */
#define ZIP_DECODE_LENGTH(ptr, encoding, lensize, len) do {                    \
    /* 取出节点的编码 */                                                       \
    ZIP_ENTRY_ENCODING((ptr), (encoding));                                     \
    if ((encoding) < ZIP_STR_MASK) {                                           \
        /* 节点保存的是字符串，取出编码 */                                     \
        if ((encoding) == ZIP_STR_06B) {                                       \
            (lensize) = 1;                                                     \
            (len) = (ptr)[0] & 0x3f;                                           \
        } else if ((encoding) == ZIP_STR_14B) {                                \
            (lensize) = 2;                                                     \
            (len) = (((ptr)[0] & 0x3f) << 8) | (ptr)[1];                       \
        } else if (encoding == ZIP_STR_32B) {                                  \
            (lensize) = 5;                                                     \
            (len) = ((ptr)[1] << 24) |                                         \
                    ((ptr)[2] << 16) |                                         \
                    ((ptr)[3] <<  8) |                                         \
                    ((ptr)[4]);                                                \
        } else {                                                               \
            assert(NULL);                                                      \
        }                                                                      \
    } else {                                                                   \
        /* 节点保存的是整数，取出编码 */                                       \
        (lensize) = 1;                                                         \
        (len) = zipIntSize(encoding);                                          \
    }                                                                          \
} while(0);
```
```cpp?linenums
/* Return the total number of bytes used by the entry pointed to by 'p'. */
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

```cpp?linenums
// 1、判断插入位置是否为表尾
// 若不为表尾，则构建一个 zlentry 节点
// 若为表尾，则还需判断列表是否存在节点

// 返回列表最后一个元素之后的位置
#define ZIPLIST_ENTRY_TAIL(zl)  ((zl)+intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl)))

// 取出列表的表尾偏移量(内存的 32 - 63 位，整数)
#define ZIPLIST_TAIL_OFFSET(zl) (*((uint32_t*)((zl)+sizeof(uint32_t))))

/* Find out prevlen for the entry that is inserted. */
// 如果 p 之后不是没有节点（不是插入到末端）
// 那么取出节点相关资料，以及 prevlen
if (p[0] != ZIP_END) {
    entry = zipEntry(p);
    prevlen = entry.prevrawlen;
} else {
    // 获取列表最后一个节点（表尾）的地址
    unsigned char *ptail = ZIPLIST_ENTRY_TAIL(zl);
    // 如果地址之后不是末端（也即是，列表至少有一个节点）
if (ptail[0] != ZIP_END) {
        // 保存 ptail 指向的节点的空间长度
        prevlen = zipRawEntryLength(ptail);
    }
}
```

**判断是否能够保存为整数存储**
```cpp?linenums
/*
 * 将一个字符串转换为 long long 整数值
 *
 * 复杂度：O(N)
 *
 * 返回值：
 *  转换成功返回 1 ，失败返回 0 。
 *  转换成功时，将 value 的值设为转换所得的 long long 值。
 */
int string2ll(const char *s, size_t slen, long long *value) {
    const char *p = s;
    size_t plen = 0;
    int negative = 0;
    unsigned long long v;

    // 空字符串
    if (plen == slen)
        return 0;

    /* Special case: first and only digit is 0. */
    // 值为 0 
    if (slen == 1 && p[0] == '0') {
        if (value != NULL) *value = 0;
        return 1;
    }

    // 值为负数
    if (p[0] == '-') {
        negative = 1;
        p++; plen++;

        /* Abort on only a negative sign. */
        // 只有负号，停止
        if (plen == slen)
            return 0;
    }

    /* First digit should be 1-9, otherwise the string should just be 0. */
    // 第一个数字必须不为 0 ，否则值为 0 
    if (p[0] >= '1' && p[0] <= '9') {
        v = p[0]-'0';
        p++; plen++;
    } else if (p[0] == '0' && slen == 1) {
        *value = 0;
        return 1;
    } else {
        return 0;
    }

    // 遍历整个字符串
    while (plen < slen && p[0] >= '0' && p[0] <= '9') {
        // 如果 v * 10 > ULLONG_MAX
        // 那么值溢出
        if (v > (ULLONG_MAX / 10)) /* Overflow. */
            return 0;
        v *= 10;

        // 如果 v + (p[0]-'0') > ULLONG_MAX
        // 那么值溢出
        if (v > (ULLONG_MAX - (p[0]-'0'))) /* Overflow. */
            return 0;
        v += p[0]-'0';

        p++; plen++;
    }

    /* Return if not all bytes were used. */
    // 并非整个字符串都能转换为整数，返回 0
    if (plen < slen)
        return 0;

    // 处理返回值的负数情况
    if (negative) {
        if (v > ((unsigned long long)(-(LLONG_MIN+1))+1)) /* Overflow. */
            return 0;
        if (value != NULL) *value = -v;
    } else {
        if (v > LLONG_MAX) /* Overflow. */
            return 0;
        if (value != NULL) *value = v;
    }

    return 1;
}
```
```cpp?linenums
/*
 * 检查 entry 所保存的值，看它能否编码为整数
 *
 * 复杂度：O(N)，N 为 entry 所保存字符串值的长度
 *
 * 返回值：
 *  如果可以的话，返回 1 ，并将值保存在 v ，将编码保存在 encoding
 *  否则，返回 0
 */
static int zipTryEncoding(unsigned char *entry, unsigned int entrylen, long long *v, unsigned char *encoding) {
    long long value;

    if (entrylen >= 32 || entrylen == 0) return 0;
    // 尝试转换为整数
    if (string2ll((char*)entry,entrylen,&value)) {
        /* Great, the string can be encoded. Check what's the smallest
         * of our encoding types that can hold this value. */
        // 选择整数编码
        if (value >= 0 && value <= 12) {
            *encoding = ZIP_INT_IMM_MIN+value;
        } else if (value >= INT8_MIN && value <= INT8_MAX) {
            *encoding = ZIP_INT_8B;
        } else if (value >= INT16_MIN && value <= INT16_MAX) {
            *encoding = ZIP_INT_16B;
        } else if (value >= INT24_MIN && value <= INT24_MAX) {
            *encoding = ZIP_INT_24B;
        } else if (value >= INT32_MIN && value <= INT32_MAX) {
            *encoding = ZIP_INT_32B;
        } else {
            *encoding = ZIP_INT_64B;
        }
        *v = value;
        return 1;
    }
    return 0;
}
```
```cpp?linenums
// 查看能否将新值保存为整数
// 如果可以的话返回 1 ，
// 并将新值保存到 value ，编码形式保存到 encoding
if (zipTryEncoding(s,slen,&value,&encoding)) {
    /* 'encoding' is set to the appropriate integer encoding */
    // s 可以保存为整数，那么继续计算保存它所需的空间
    reqlen = zipIntSize(encoding);
} else {
    /* 'encoding' is untouched, however zipEncodeLength will use the
         * string length to figure out how to encode it. */
    // 不能保存为整数，直接使用字符串长度
    reqlen = slen;
}
```
**计算 `prevlen`、`encoding` 长度和内容长度**
```cpp?linenums
// 计算编码 prevlen 所需的长度
reqlen += zipPrevEncodeLength(NULL,prevlen);
// 计算编码 slen 所需的长度
reqlen += zipEncodeLength(NULL,encoding,slen);
```
```cpp?linenums
/*
 * 编码前置节点的长度，并将它写入 p 。
 *
 * 如果 p 为 NULL ，那么返回编码 len 所需的字节数。
 *
 * 复杂度：O(1)
 */
static unsigned int zipPrevEncodeLength(unsigned char *p, unsigned int len) {
    if (p == NULL) {
        return (len < ZIP_BIGLEN) ? 1 : sizeof(len)+1;
    } else {
        if (len < ZIP_BIGLEN) {
            p[0] = len;
            return 1;
        } else {
            p[0] = ZIP_BIGLEN;
            memcpy(p+1,&len,sizeof(len));
            memrev32ifbe(p+1);
            return 1+sizeof(len);
        }
    }
}
```
```cpp?linenums
/* Macro to determine type */
#define ZIP_IS_STR(enc) (((enc) & ZIP_STR_MASK) < ZIP_STR_MASK)
/*
 * 编码长度 l ，并将它写入到 p 。
 * 
 * 如果 p 为 NULL ，那么返回编码 rawlen 所需的字节数
 *
 * 复杂度：O(1)
 */
static unsigned int zipEncodeLength(unsigned char *p, unsigned char encoding, unsigned int rawlen) {
    unsigned char len = 1, buf[5];

    if (ZIP_IS_STR(encoding)) {
        // 字符串编码
        /* Although encoding is given it may not be set for strings,
         * so we determine it here using the raw length. */
        if (rawlen <= 0x3f) {
            // 长度 6 bit ，长度 + 编码 8 bit (1 byte)
            if (!p) return len;
            buf[0] = ZIP_STR_06B | rawlen;
        } else if (rawlen <= 0x3fff) {
            // 长度 14 bit，长度 + 编码 16 bit (2 btyes)
            len += 1;
            if (!p) return len;
            buf[0] = ZIP_STR_14B | ((rawlen >> 8) & 0x3f);
            buf[1] = rawlen & 0xff;
        } else {
            // 长度 32 bit (4 bytes)，长度 + 编码 40 bit (5 bytes)
            len += 4;
            if (!p) return len;
            buf[0] = ZIP_STR_32B;
            buf[1] = (rawlen >> 24) & 0xff;
            buf[2] = (rawlen >> 16) & 0xff;
            buf[3] = (rawlen >> 8) & 0xff;
            buf[4] = rawlen & 0xff;
        }
    } else {
        // 编码为整数，长度总为 1 bytes
        /* Implies integer encoding, so length is always 1. */
        if (!p) return len;
        buf[0] = encoding;
    }

    /* Store this length at p */
    memcpy(p,buf,len);
    return len;
}
```
**计算 `prevlen` 的差值**
```cpp?linenums
// 如果添加的位置不是表尾，那么必须确定后继节点的 prevlen 空间
// 足以保存新节点的编码长度
// zipPrevLenByteDiff 的返回值有三种可能：
// 1）新旧两个节点的编码长度相等，返回 0
// 2）新节点编码长度 > 旧节点编码长度，返回 5 - 1 = 4
// 3）旧节点编码长度 > 新编码节点长度，返回 1 - 5 = -4
nextdiff = (p[0] != ZIP_END) ? zipPrevLenByteDiff(p,reqlen) : 0;
```
```cpp?linenums
/*
 * 返回编码 len 所需的长度减去编码 p 的前一个节点的大小所需的长度之差
 * 
 * 复杂度：O(1)
 */
static int zipPrevLenByteDiff(unsigned char *p, unsigned int len) {
    // 获取编码前一节点所需的长度
    unsigned int prevlensize;
    ZIP_DECODE_PREVLENSIZE(p, prevlensize);
    // 计算差
    return zipPrevEncodeLength(NULL, len) - prevlensize;
}
```
**重新分配空间并插入节点**
```cpp?linenums
/*
 * 对 zl 进行空间重非配，并更新相关属性
 *
 * 复杂度：O(N)
 *
 * 返回值：更新后的 ziplist
 */
static unsigned char *ziplistResize(unsigned char *zl, unsigned int len) {
    // 重分配
    zl = zrealloc(zl,len);
    // 更新长度
    ZIPLIST_BYTES(zl) = intrev32ifbe(len);
    // 设置表尾
    zl[len-1] = ZIP_END;

    return zl;
}
```
**更新表尾偏移量**
```cpp?linenums
// 更新 ziplist 的表尾偏移量
// 如果插入节点不是在表尾则需要更新 nextdiff
ZIPLIST_TAIL_OFFSET(zl) =
            intrev32ifbe(intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))+reqlen);
tail = zipEntry(p+reqlen);
if (p[reqlen+tail.headersize+tail.len] != ZIP_END) {
    ZIPLIST_TAIL_OFFSET(zl) =
              intrev32ifbe(intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))+nextdiff);
}
```
```cpp?linenums
// 如果插入表尾
// 更新 ziplist 的 zltail 属性，现在新添加节点为表尾节点
ZIPLIST_TAIL_OFFSET(zl) = intrev32ifbe(p-zl);
```
**因为节点变化，则后续的 `prevlen` 可能从 1 变成 5，也有可能从 5 变成 1**
```cpp?linenums
/* When nextdiff != 0, the raw length of the next entry has changed, so
* we need to cascade the update throughout the ziplist */
if (nextdiff != 0) {
    offset = p-zl; // zl 会重新分配内存，先保存偏移量
    // O(N^2)
    zl = __ziplistCascadeUpdate(zl,p+reqlen);
    p = zl+offset;
}
```
**逐级更新节点**
如果插入的节点导致后面的节点 `prevlen` 变小，不进行压缩操作
总体步骤：
* 先扩充大小，调整内容后移，再处理 `prevlen`
* 每一步都需要处理尾偏移量
```cpp?linenums
/*
 * 当将一个新节点添加到某个节点之前的时候，
 * 如果原节点的 prevlen 不足以保存新节点的长度，
 * 那么就需要对原节点的空间进行扩展（从 1 字节扩展到 5 字节）。
 *
 * 但是，当对原节点进行扩展之后，原节点的下一个节点的 prevlen 可能出现空间不足，
 * 这种情况在多个连续节点的长度都接近 ZIP_BIGLEN 时可能发生。
 *
 * 这个函数就用于处理这种连续扩展动作。
 *
 * 因为节点的长度变小而引起的连续缩小也是可能出现的，
 * 不过，为了避免扩展-缩小-扩展-缩小这样的情况反复出现（flapping，抖动），
 * 我们不处理这种情况，而是任由 prevlen 比所需的长度更长
 *
 * 复杂度：O(N^2)
 *
 * 返回值：更新后的 ziplist
 */
static unsigned char *__ziplistCascadeUpdate(unsigned char *zl, unsigned char *p) {
    size_t curlen = intrev32ifbe(ZIPLIST_BYTES(zl)), rawlen, rawlensize;
    size_t offset, noffset, extra;
    unsigned char *np;
    zlentry cur, next;
 
    // 一直更新直到表尾
    while (p[0] != ZIP_END) {
        // 当前节点
        cur = zipEntry(p);
        // 当前节点的长度
        rawlen = cur.headersize + cur.len;
        // 编码当前节点的长度所需的空间大小
        rawlensize = zipPrevEncodeLength(NULL,rawlen);

        /* Abort if there is no next entry. */
        // 已经到达表尾，退出
        if (p[rawlen] == ZIP_END) break;
        // 取出下一节点
        next = zipEntry(p+rawlen);

        /* Abort when "prevlen" has not changed. */
        // 如果下一的 prevlen 等于当前节点的 rawlen
        // 那么说明编码大小无需改变，退出
        if (next.prevrawlen == rawlen) break;

        // 下一节点的长度编码空间不足，进行扩展
        if (next.prevrawlensize < rawlensize) {
            /* The "prevlen" field of "next" needs more bytes to hold
             * the raw length of "cur". */
            offset = p-zl;
            // 需要多添加的长度
            extra = rawlensize-next.prevrawlensize;
            // 重分配，复杂度为 O(N)
            zl = ziplistResize(zl,curlen+extra);
            p = zl+offset;

            /* Current pointer and offset for next element. */
            np = p+rawlen;
            noffset = np-zl;

            /* Update tail offset when next element is not the tail element. */
            if ((zl+intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))) != np) {
                ZIPLIST_TAIL_OFFSET(zl) =
                    intrev32ifbe(intrev32ifbe(ZIPLIST_TAIL_OFFSET(zl))+extra);
            }

            /* Move the tail to the back. */
            // 为获得空间而进行数据移动，复杂度万恶 O(N)
            memmove(np+rawlensize,
                np+next.prevrawlensize,
                curlen-noffset-next.prevrawlensize-1);
            zipPrevEncodeLength(np,rawlen);

            /* Advance the cursor */
            p += rawlen;
            curlen += extra;
        } else {
            // 下一节点的长度编码空间有多余，不进行收缩
            // 只是将被编码的长度写入空间
            if (next.prevrawlensize > rawlensize) {
                /* This would result in shrinking, which we want to avoid.
                 * So, set "rawlen" in the available bytes. */
                zipPrevEncodeLengthForceLarge(p+rawlen,rawlen);
            } else {
                zipPrevEncodeLength(p+rawlen,rawlen);
            }

            // next.prevrawlensize == rawlensize
            /* Stop here, as the raw length of "next" has not changed. */
            break;
        }
    }
    return zl;
}
```







  [1]: ./images/1461832267191.jpg "1461832267191.jpg"
  [2]: ./images/1462604595034.jpg "1462604595034.jpg"