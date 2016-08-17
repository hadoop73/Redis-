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


![enter description here][36]

![enter description here][37]

![enter description here][38]

![enter description here][39]

![enter description here][40]

![enter description here][41]

![enter description here][42]

![enter description here][43]

![enter description here][44]

![enter description here][45]

![enter description here][46]

![enter description here][47]

![enter description here][48]

![enter description here][49]


###  zset

![enter description here][50]

![enter description here][51]

![enter description here][52]

![enter description here][53]

![enter description here][54]

![enter description here][55]

![enter description here][56]

![enter description here][57]


##  键值操作

**keys \***

![enter description here][58]

![enter description here][59]

![enter description here][60]

![enter description here][61]

![enter description here][62]

![enter description here][63]

![enter description here][64]

![enter description here][65]

![enter description here][66]

![enter description here][67]

![enter description here][68]

![enter description here][69]

![enter description here][70]

![enter description here][71]

![enter description here][72]


##  Redis 高级特性

![enter description here][73]

![enter description here][74]

![enter description here][75]

**授权**

![enter description here][76]

![enter description here][77]

### 主从复制

![enter description here][78]

![enter description here][79]

![enter description here][80]

![enter description here][81]

![enter description here][82]


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
  [36]: ./images/1471401992078.jpg "1471401992078.jpg"
  [37]: ./images/1471402102411.jpg "1471402102411.jpg"
  [38]: ./images/1471402162980.jpg "1471402162980.jpg"
  [39]: ./images/1471402257500.jpg "1471402257500.jpg"
  [40]: ./images/1471402355277.jpg "1471402355277.jpg"
  [41]: ./images/1471402449162.jpg "1471402449162.jpg"
  [42]: ./images/1471402485012.jpg "1471402485012.jpg"
  [43]: ./images/1471402520033.jpg "1471402520033.jpg"
  [44]: ./images/1471402559798.jpg "1471402559798.jpg"
  [45]: ./images/1471402580685.jpg "1471402580685.jpg"
  [46]: ./images/1471402632223.jpg "1471402632223.jpg"
  [47]: ./images/1471402713238.jpg "1471402713238.jpg"
  [48]: ./images/1471402752824.jpg "1471402752824.jpg"
  [49]: ./images/1471402800900.jpg "1471402800900.jpg"
  [50]: ./images/1471402908414.jpg "1471402908414.jpg"
  [51]: ./images/1471403095535.jpg "1471403095535.jpg"
  [52]: ./images/1471403159913.jpg "1471403159913.jpg"
  [53]: ./images/1471403319720.jpg "1471403319720.jpg"
  [54]: ./images/1471403346161.jpg "1471403346161.jpg"
  [55]: ./images/1471403465731.jpg "1471403465731.jpg"
  [56]: ./images/1471403484121.jpg "1471403484121.jpg"
  [57]: ./images/1471403541800.jpg "1471403541800.jpg"
  [58]: ./images/1471404556789.jpg "1471404556789.jpg"
  [59]: ./images/1471404614018.jpg "1471404614018.jpg"
  [60]: ./images/1471404628875.jpg "1471404628875.jpg"
  [61]: ./images/1471404678571.jpg "1471404678571.jpg"
  [62]: ./images/1471404690805.jpg "1471404690805.jpg"
  [63]: ./images/1471404935175.jpg "1471404935175.jpg"
  [64]: ./images/1471405031766.jpg "1471405031766.jpg"
  [65]: ./images/1471405074298.jpg "1471405074298.jpg"
  [66]: ./images/1471405101043.jpg "1471405101043.jpg"
  [67]: ./images/1471405122943.jpg "1471405122943.jpg"
  [68]: ./images/1471405188683.jpg "1471405188683.jpg"
  [69]: ./images/1471405318087.jpg "1471405318087.jpg"
  [70]: ./images/1471405405416.jpg "1471405405416.jpg"
  [71]: ./images/1471405441543.jpg "1471405441543.jpg"
  [72]: ./images/1471405479013.jpg "1471405479013.jpg"
  [73]: ./images/1471405540092.jpg "1471405540092.jpg"
  [74]: ./images/1471405605887.jpg "1471405605887.jpg"
  [75]: ./images/1471405689111.jpg "1471405689111.jpg"
  [76]: ./images/1471406041954.jpg "1471406041954.jpg"
  [77]: ./images/1471406060988.jpg "1471406060988.jpg"
  [78]: ./images/1471406091631.jpg "1471406091631.jpg"
  [79]: ./images/1471406214320.jpg "1471406214320.jpg"
  [80]: ./images/1471406287212.jpg "1471406287212.jpg"
  [81]: ./images/1471406314007.jpg "1471406314007.jpg"
  [82]: ./images/1471406650972.jpg "1471406650972.jpg"
