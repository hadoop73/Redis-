
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

##  rdbSave 和 rdbSaveBackground

rdbSave会阻塞主进程
处理每一个数据库，使用迭代器把数据库中的数据重新写入磁盘
```cpp
// 遍历所有数据库，保存它们的数据
    for (j = 0; j < server.dbnum; j++) {
        // 指向数据库
        redisDb *db = server.db+j;
        // 指向数据库 key space
        dict *d = db->dict;
        // 数据库为空， pass ，处理下个数据库
        if (dictSize(d) == 0) continue;

        // 创建迭代器
        di = dictGetSafeIterator(d);
        if (!di) {
            fclose(fp);
            return REDIS_ERR;
        }

        /* Write the SELECT DB opcode */
        // 记录正在使用的数据库的号码
        if (rdbSaveType(&rdb,REDIS_RDB_OPCODE_SELECTDB) == -1) goto werr;
        if (rdbSaveLen(&rdb,j) == -1) goto werr;

        /* Iterate this DB writing every entry */
        // 将数据库中的所有节点保存到 RDB 文件
        while((de = dictNext(di)) != NULL) {
            // 取出键
            sds keystr = dictGetKey(de);
            // 取出值
            robj key,
                 *o = dictGetVal(de);
            long long expire;

            initStaticStringObject(key,keystr);
            // 取出过期时间
            expire = getExpire(db,&key);
            if (rdbSaveKeyValuePair(&rdb,&key,o,expire,now) == -1) goto werr;
        }
        dictReleaseIterator(di);
    }
    di = NULL; /* So that we don't release it again on error. */
```


rdbSaveBackground创建子进程处理持久化操作，主进程继续处理事务，会存在内存数据和持久化数据不一样情况
设置数据库为 dirty，创建子进程调用 rdbSave
```cpp
// 修改服务器状态
    server.dirty_before_bgsave = server.dirty;

    // 开始时间
    start = ustime();
    // 创建子进程
    if ((childpid = fork()) == 0) {
        int retval;

        /* Child */
        // 子进程不接收网络数据
        if (server.ipfd > 0) close(server.ipfd);
        if (server.sofd > 0) close(server.sofd);

        // 保存数据
        retval = rdbSave(filename);
        if (retval == REDIS_OK) {
            size_t private_dirty = zmalloc_get_private_dirty();

            if (private_dirty) {
                redisLog(REDIS_NOTICE,
                    "RDB: %lu MB of memory used by copy-on-write",
                    private_dirty/(1024*1024));
            }
        }

        // 退出子进程
        exitFromChild((retval == REDIS_OK) ? 0 : 1);
    }
```
##  保存到磁盘
按照特定的格式，在 RDB 文件中写入文件头
```cpp
// 以 "REDIS <VERSION>" 格式写入文件头，以及 RDB 的版本
snprintf(magic,sizeof(magic),"REDIS%04d",REDIS_RDB_VERSION);
if (rdbWriteRaw(&rdb,magic,9) == -1) goto werr;
```
遍历所有的数据库，首先把数据库编号写入，再继续写入数据；根据数据库，创建迭代器，获取所有的 key-value 对，再写入,如 rdbSave 代码

保存 key-value 对，会判断时间期限是否过期，如果过期不会再保存到文件
总的是，先写入 value 的类型，再写入 key-value
```cpp
int rdbSaveKeyValuePair(rio *rdb, robj *key, robj *val,
                        long long expiretime, long long now)
{
    /* Save the expire time */
    // 保存过期时间
    if (expiretime != -1) {
        /* If this key is already expired skip it */
        // key 已过期，直接跳过
        if (expiretime < now) return 0;

        if (rdbSaveType(rdb,REDIS_RDB_OPCODE_EXPIRETIME_MS) == -1) return -1;  // 用来表示过期键,也就是调用 rio 的写入函数，写入一个字节
        if (rdbSaveMillisecondTime(rdb,expiretime) == -1) return -1; // 写入过期时间
    }

    /* Save type, key, value */
    // 保存值类型,根据 Value 的类型写入
    if (rdbSaveObjectType(rdb,val) == -1) return -1;
    // 保存 key
    if (rdbSaveStringObject(rdb,key) == -1) return -1;
    // 保存 value
    if (rdbSaveObject(rdb,val) == -1) return -1;

    return 1;
}
```
因为 value 的类型为 robj，可以直接获取 type
```cpp
/*
 * Redis 对象
 */
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 不使用(对齐位)
    unsigned notused:2;

    // 编码方式
    unsigned encoding:4;

    // LRU 时间（相对于 server.lruclock）
    unsigned lru:22;

    // 引用计数
    int refcount;

    // 指向对象的值
    void *ptr;

} robj;
```


## 迭代器
```cpp
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
可以从迭代起中获取 dictEntry 节点，key-alue 对存放在节点中
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


##  load RDB 文件
通过 rio 的接口来把 rdb 文件的数据载入到内存。
```cpp
// 初始化 rdb 文件
rioInitWithFile(&rdb,fp);
```
先读取 type 确定接下来内容的类型，再读取内容
读取内容分为 `rdbLoadLen`,`rdbLoadStringObject`,`rdbLoadObject`,通过 `dbAdd(db,key,val)` 添加到内存
`rdbLoadLen` 用于从文件中读取一个整数值
```cpp
uint32_t rdbLoadLen(rio *rdb, int *isencoded) {
    unsigned char buf[2];
    uint32_t len;
    int type;

    if (isencoded) *isencoded = 0;
    if (rioRead(rdb,buf,1) == 0) return REDIS_RDB_LENERR; // 先读取一个字节用来表示值的类型
    type = (buf[0]&0xC0)>>6;
    if (type == REDIS_RDB_ENCVAL) {
        /* Read a 6 bit encoding type. */
        if (isencoded) *isencoded = 1;
        return buf[0]&0x3F;
    } else if (type == REDIS_RDB_6BITLEN) {
        /* Read a 6 bit len. */
        return buf[0]&0x3F;
    } else if (type == REDIS_RDB_14BITLEN) {
        /* Read a 14 bit len. */
        if (rioRead(rdb,buf+1,1) == 0) return REDIS_RDB_LENERR;
        return ((buf[0]&0x3F)<<8)|buf[1];
    } else {
        /* Read a 32 bit len. */
        if (rioRead(rdb,&len,4) == 0) return REDIS_RDB_LENERR;
        return ntohl(len);
    }
}
```
`rdbLoadStringObject` 读取 key
用于从文件中读取一个字符串对象，先获取要读取字符串的长度 len，再用 rioRead 读取 len个长度的字符串，通过 sdn 构造
```cpp
robj *rdbLoadStringObject(rio *rdb) {
    return rdbGenericLoadStringObject(rdb,0);
}

/*
 * 根据编码 encode ，从 rdb 文件中读取字符串，并返回字符串对象
 */
robj *rdbGenericLoadStringObject(rio *rdb, int encode) {
    int isencoded;
    uint32_t len;
    sds val;

    // 获取字符串的长度
    len = rdbLoadLen(rdb,&isencoded);
    if (isencoded) {
        switch(len) {
        case REDIS_RDB_ENC_INT8:
        case REDIS_RDB_ENC_INT16:
        case REDIS_RDB_ENC_INT32:
            // 字节串是整数，创建整数对象并返回
            return rdbLoadIntegerObject(rdb,len,encode);
        case REDIS_RDB_ENC_LZF:
            // 字节串是被 lzf 算法压缩的字符串
            return rdbLoadLzfStringObject(rdb);
        default:
            redisPanic("Unknown RDB encoding type");
        }
    }

    if (len == REDIS_RDB_LENERR) return NULL;

    // 创建一个指定长度的 sds
    val = sdsnewlen(NULL,len);
    if (len && rioRead(rdb,val,len) == 0) {
        sdsfree(val);
        return NULL;
    }
    // 根据 sds ，创建字符串对象
    return createObject(REDIS_STRING,val);
}
```
`rdbLoadObject` 读取 value
读取的对象包括 字符串，set，哈希表，压缩列表等。



`dbAdd(db,key,val)`





