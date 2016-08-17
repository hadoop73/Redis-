---
title: Redis 总结 
tags: 
grammar_cjkRuby: true
---

[TOC]


##  适用场景
*  对数据高并发读写
*  对海量数据的高效存储和访问
*  对数据高扩展性和高可用性


![enter description here][1]

![enter description here][2]


|     |  redis   |  mysql   | mongodb    |
| :---: | :---: | :---: | :---: |
|  库   |  有  |    有 |   有  |
|  表   |  无   |  有   |   集合  |
|  行,字段   |  无   |  有   |  无   |


##  数据类型

### string

二进制安全的,可以包含任何数据,比如jpg图片或者序列化对象

![enter description here][3]

不存在才插入,存在不覆盖

![enter description here][4]

![enter description here][5]

**修改字符串**

![enter description here][6]

**批量设置**

![enter description here][7]

![enter description here][8]

![enter description here][9]

**get 关键词**

![enter description here][10]

![enter description here][11]


**递增**

![enter description here][12]

![enter description here][13]

**追加**

![enter description here][14]

![enter description here][15]

![enter description here][16]




### hash

![enter description here][17]

![enter description here][18]

![enter description here][19]

![enter description here][20]

![enter description here][21]

![enter description here][22]

![enter description here][23]

![enter description here][24]

![enter description here][25]

### list

![enter description here][26]

**操作**

![enter description here][27]

![enter description here][28]

![enter description here][29]

![enter description here][30]

![enter description here][31]

![enter description here][32]

![enter description here][33]

![enter description here][34]

![enter description here][35]




###  set


###  zset


  [1]: ./images/1471313666354.jpg "1471313666354.jpg"
  [2]: ./images/1471313700420.jpg "1471313700420.jpg"
  [3]: ./images/1471315324642.jpg "1471315324642.jpg"
  [4]: ./images/1471315427674.jpg "1471315427674.jpg"
  [5]: ./images/1471315453917.jpg "1471315453917.jpg"
  [6]: ./images/1471315567761.jpg "1471315567761.jpg"
  [7]: ./images/1471315793083.jpg "1471315793083.jpg"
  [8]: ./images/1471315822008.jpg "1471315822008.jpg"
  [9]: ./images/1471315928526.jpg "1471315928526.jpg"
  [10]: ./images/1471316000287.jpg "1471316000287.jpg"
  [11]: ./images/1471316058369.jpg "1471316058369.jpg"
  [12]: ./images/1471316180799.jpg "1471316180799.jpg"
  [13]: ./images/1471316245277.jpg "1471316245277.jpg"
  [14]: ./images/1471316301757.jpg "1471316301757.jpg"
  [15]: ./images/1471316368889.jpg "1471316368889.jpg"
  [16]: ./images/1471316412585.jpg "1471316412585.jpg"
  [17]: ./images/1471316512364.jpg "1471316512364.jpg"
  [18]: ./images/1471316590934.jpg "1471316590934.jpg"
  [19]: ./images/1471316640108.jpg "1471316640108.jpg"
  [20]: ./images/1471316733864.jpg "1471316733864.jpg"
  [21]: ./images/1471316882376.jpg "1471316882376.jpg"
  [22]: ./images/1471316914258.jpg "1471316914258.jpg"
  [23]: ./images/1471316968812.jpg "1471316968812.jpg"
  [24]: ./images/1471317017554.jpg "1471317017554.jpg"
  [25]: ./images/1471317083066.jpg "1471317083066.jpg"
  [26]: ./images/1471318175501.jpg "1471318175501.jpg"
  [27]: ./images/1471318344318.jpg "1471318344318.jpg"
  [28]: ./images/1471318552635.jpg "1471318552635.jpg"
  [29]: ./images/1471318700634.jpg "1471318700634.jpg"
  [30]: ./images/1471318832642.jpg "1471318832642.jpg"
  [31]: ./images/1471318920775.jpg "1471318920775.jpg"
  [32]: ./images/1471319127761.jpg "1471319127761.jpg"
  [33]: ./images/1471319153634.jpg "1471319153634.jpg"
  [34]: ./images/1471319494005.jpg "1471319494005.jpg"
  [35]: ./images/1471319592485.jpg "1471319592485.jpg"
