---
title: Redis RDB持久化
tags: Redis,RDB,持久化
grammar_cjkRuby: true
---

`Redis` 是内存数据库，将数据库状态存储在内存，服务器进程退出，则服务器中的数据库状态也会消失。`RDB` 持久化就是把内存中的数据保存到磁盘，避免数据意外丢失。可以手动( `SAVE` )或定期执行。
`RDB` 文件是一个压缩过的**二进制文件**。

##  命令
`Redis` 命令：`SAVE` 和 `BGSAVE` ；`SAVE` 会阻塞 `Redis` 进程； `BGSAVE` 命令派生一个子进程，由子进程负责创建 `RDB` 文件，父进程继续处理命令请求。

##  `RDB` 文件结构
![enter description here][1]

`RDB` 文件开头是5字节，保存着 `REDIS` 五个字符。
`db_version` 长度为4字节，表示版本
`databases` 包含零个或任意多个数据库，以及各个数据库中的键值对数据。
`EOF` 常量长度1字节，是 `RDB` 文件正文内容结束标志。
`check_sum` 是一个8字节长的无符号整数，保存着一个校验和，校验和有 `REDIS、db_version、databases、EOF` 四个部分的内容进行计算得出的。

**`databases` 部分**

![enter description here][2]

每个非空数据库在 `RDB` 文件中都可以保存为 `SELECTDB、db_number、key_value_pairs` 三个部分组成。

![enter description here][3]

`SELECTDB` 常量的长度为1字节，当程序遇到这个值的时候，它直到接下来要读入的将是一个数据库号码。
`db_number` 保存着一个数据库号码，长度可以是1字节、2字节或者5字节。通过调用 `SELECT` 命令，根据读入的数据库号码进行数据库切换。
`key_value_pairs` 部分保存了数据库中的所有键值对数据还有过期时间。

![enter description here][4]

其中 `key` 总是一个字符串对象，编码方式 `REDIS_RDB_TYPE_STRING` 类型。
`TYPE` 的不同，`value` 的结构和长度也会有所不同

![enter description here][5]


  [1]: ./images/1465376404541.jpg "1465376404541.jpg"
  [2]: ./images/1465376756824.jpg "1465376756824.jpg"
  [3]: ./images/1465376838887.jpg "1465376838887.jpg"
  [4]: ./images/1465377384294.jpg "1465377384294.jpg"
  [5]: ./images/1465377427775.jpg "1465377427775.jpg"