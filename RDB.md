
##  数据结构 rio
持久化的IO操作在rio.h和rio.c中实现，struct rio既适用于文件，又适用于内存缓存
抽象了文件和内存操作
```cpp
struct _rio {

    size_t (*read)(struct _rio *, void *buf, size_t len);
    size_t (*write)(struct _rio *, const void *buf, size_t len);
    off_t (*tell)(struct _rio *);  // 获取偏移量
    // 计算校验和，每次写入都会判断函数是否为空，再重新计算
    void (*update_cksum)(struct _rio *, const void *buf, size_t len);

    // 当前校验和
    uint64_t cksum;

    union {
        // 处理字节/字符串时使用
        struct {
            sds ptr;
            off_t pos;
        } buffer;
        // 处理文件时使用
        struct {
            FILE *fp;
        } file;
    } io;
};

typedef struct _rio rio;
```
分别为流和文件设置不同的处理函数和数据结构，这样在调用的时候，可以使用相同的
调用方式，因为底层的实现函数不一样调用接口相同，能够处理流和文件的情况。
```cpp
/*
 * 流为内存时所使用的结构
 */
static const rio rioBufferIO = {
    rioBufferRead,
    rioBufferWrite,
    rioBufferTell,
    NULL,           /* update_checksum */
    0,              /* current checksum */
    { { NULL, 0 } } /* union for io-specific vars */
};

/*
 * 流为文件时所使用的结构
 */
static const rio rioFileIO = {
    rioFileRead,
    rioFileWrite,
    rioFileTell,
    NULL,           /* update_checksum */
    0,              /* current checksum */
    { { NULL, 0 } } /* union for io-specific vars */
};
```
写入时用到的`snprintf`函数，按照`format`格式化成字符串，然后将其复制到str中
同时还为上层提供接口函数来生成aof文件
```cpp
size_t rioWriteBulkCount(rio *r, char prefix, int count) {
    char cbuf[128];
    int clen;

    // cbuf = prefix ++ count ++ '\r\n'
    cbuf[0] = prefix;
    clen = 1+ll2string(cbuf+1,sizeof(cbuf)-1,count);
    cbuf[clen++] = '\r';
    cbuf[clen++] = '\n';

    if (rioWrite(r,cbuf,clen) == 0) return 0;
    return clen;
}

size_t rioWriteBulkString(rio *r, const char *buf, size_t len) {
    size_t nwritten;

    if ((nwritten = rioWriteBulkCount(r,'$',len)) == 0) return 0;
    if (len > 0 && rioWrite(r,buf,len) == 0) return 0;
    if (rioWrite(r,"\r\n",2) == 0) return 0;
    return nwritten+len+2;
}

/*
 * 以 "$<count>\r\n<payload>\r\n" 的格式写入 double 值
 */
size_t rioWriteBulkDouble(rio *r, double d) {
    char dbuf[128];
    unsigned int dlen;

    dlen = snprintf(dbuf,sizeof(dbuf),"%.17g",d);
    return rioWriteBulkString(r,dbuf,dlen);
}
```











