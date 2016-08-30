---
title: MySQL
tags: 
grammar_cjkRuby: true
---

[TOC]


##  MySQL 优化

*  硬件
*  系统配置
*  数据库结构
*  SQL 及索引


使用 MySQL 慢查询日志对效率问题的SQL进行监控
* show variables like 'slow_query_log';# 开启慢查询日志
* set global slow_query_log_file='/home/mysql/sql_log/mysql-slow.log';
* set global log_queries_not_using_indexes=on;
* set global long_query_time=1;
* set global slow_query_log=on;


```sql
select * from table limit 10;

# 执行时间
# Time: 2016-08-29T04:32:03.296952Z
# 执行SQL主机信息
# User@Host: root[root] @ localhost []  Id:    27
# SQL 执行信息
# Query_time: 0.000157  Lock_time: 0.000060 Rows_sent: 2  Rows_examined: 2
# 执行时间
SET timestamp=1472445123;
# 执行命令
select * from staff limit 2;
```

###  慢查询日志分析工具
**mysqldumpslow**
```sql
mysqldumpslow -t 3 /var/lib/mysql/slaveC-slow.log | more
```
```
# 执行次数,耗时,锁定时间
Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=1.0 (1), root[root]@localhost
  select count(*) from film

Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=5.0 (5), root[root]@localhost
  select * from mysql.user

Count: 1  Time=0.00s (0s)  Lock=0.00s (0s)  Rows=4.0 (4), root[root]@localhost
  select * from seckill
```

**pt-query-digest**

输出到文件
```sql
pt-query-digest slow-log > slow_log.report
```
输出到数据库表
```sql
pt-query-digest slow.log -review \
h=127.0.0.1,D=test,p=root,P=3306,u=root,t=query_view \
--create-reviewtable \
--review-history t=hostname_slow
```

* 查询次数多每次查询占用时间长的SQL,pt-query-digest 分析前几项
* IO大的SQL,Rows examine 项,扫描的行数多
* 未命中索引的SQL,Rows examine 和 Rows Send的对比,扫描行数和返回行数的比值


![enter description here][1]

![enter description here][2]

![enter description here][3]

![enter description here][4]

![enter description here][5]

###  count 优化

![enter description here][6]

![enter description here][7]

###  子查询优化

使用 distinct

![enter description here][8]

###  Group by 优化

![enter description here][9]

###  limit 优化

![enter description here][10]

![enter description here][11]

不能由空缺行主键

![enter description here][12]

### 建立索引

![enter description here][13]

![enter description here][14]

![enter description here][15]

![enter description here][16]

![enter description here][17]


##  表结构优化

![enter description here][18]

![enter description here][19]

![enter description here][20]

![enter description here][21]

![enter description here][22]

![enter description here][23]

![enter description here][24]

![enter description here][25]

###  表垂直拆分

![enter description here][26]


###  表水平拆分

![enter description here][27]

![enter description here][28]


### 系统配置优化

![enter description here][29]

![enter description here][30]

![enter description here][31]

![enter description here][32]

![enter description here][33]

![enter description here][34]

配置向导:

https://tools.percona.com/wizard


#  存储过程

[mysql存储过程详解][35]

##  存储过程是
存储过程是一组为了完成特定功能的SQL语句集,经过编译后存储在数据库中,可以通过存储过程的名字和参数调用执行

##  优点
* 有控制语句,增强了SQL语言
* 提高执行速度

##  创建

in输入参数:在存储过程中作为输入参数,不能返回
out输出参数:可被改变,并返回

```sql
create procedure test(out s int)
begin
	select count(*) into s from user;
end
```

#  索引

[MySQL 索引][36]

##  优点
增快查询速度

##  缺点
降低更新表的速度,因为更新表也需要跟新索引表

##  创建
```sql
-- 如果是CHAR，VARCHAR类型，length可以小于字段实际长度；如果是BLOB和TEXT类型，必须指定 length。
CREATE INDEX indexName ON mytable(username(length)); 
ALTER mytable ADD INDEX [indexName] ON (username(length)) 

CREATE TABLE mytable(  
 
ID INT NOT NULL,   
 
username VARCHAR(16) NOT NULL,  
 
INDEX [indexName] (username(length))  
 
);  

DROP INDEX [indexName] ON mytable; 

CREATE UNIQUE INDEX indexName ON mytable(username(length)) 
````

#  垂直分表,水平分表

#  存储引擎

[浅谈MySql的存储引擎（表类型）
][37]

[MySQL存储引擎MyISAM与InnoDB的主要区别对比][38]

* InnoDB:支持行级锁,提供事务,外键约束功能


#  事务

#  关键词
##  UPDATE

##  DELETE

##  ALTER

##  SELECT


# Join
* 内连接(INNER)
	* 产生两张表的公共部分
```sql
SELECT id FROM TableA A INNER JOIN TableB B on A.key = B.key;
```
* 全外连接(FULL OUTER)
```sql
SELECT id FROM TableA A FULL OUTER JOIN TableB B on A.key = B.key where a.key is null or b.key is null; # 查寻 a,b 中不含公共部分的数据
```
* 左外连接(LEFT OUTER)
```sql
SELECT id FROM TableA A LEFT JOIN TableB B on A.key = B.key; # 包含在A中的所有元素

SELECT id FROM TableA A LEFT JOIN TableB B on A.key = B.key WHERE B.key IS NOT NULL; # 包含在A中不包含在B中的所有元素
```
* 右外连接(RIGHT OUTER)
```sql
SELECT id FROM TableA A RIGHT JOIN TableB B on A.key = B.key; # 包含在A中的所有元素

SELECT id FROM TableA A RIGHT JOIN TableB B on A.key = B.key WHERE B.key IS NOT NULL; # 包含在B中不包含在A中的所有元素
```
* 交叉连接(CROSS)
```sql
SELECT * FROM TableA A CROSS JOIN TableB B # 迪卡尔集
```


从下面几个方面进行数据库优化

* 硬件
* 系统配置
	* 安全性限制,打开文件数限制
* 数据表结构
* SQL 及索引

sakila

http://dev.mysql.com/doc/index-other.html

http://dev.mysql.com/doc/sakila/en/sakila-installation.html


##  select

select @@version;


mysql 慢查询日志

show variables like 'slow_query_log';

set global slow_query_log_file='/home/mysql/sql_log/mysql_slow.log';

set global log_queries_not_using_indexes=on;
set global long_query_time=1;

set global slow_query_log=on;


-- 创建表
source /home/hadoop/mysql/salaki/sakila-db/sakila-schema.sql;

-- 插入数据
source /home/hadoop/mysql/salaki/sakila-db/sakila-data.sql;


```
执行 SQL 的主机信息
# User@Host: root[root] @ localhost []
SQL 的执行信息
# Query_time: 0.000637  Lock_time: 0.000086 Rows_sent: 1  Rows_examined: 1000
use sakila;
SQL 执行时间
SET timestamp=1469001368;

SQL 的内容
select count(*) from film;
```


  [1]: ./images/1472451237138.jpg "1472451237138.jpg"
  [2]: ./images/1472451278426.jpg "1472451278426.jpg"
  [3]: ./images/1472451252615.jpg "1472451252615.jpg"
  [4]: ./images/1472451450218.jpg "1472451450218.jpg"
  [5]: ./images/1472451611233.jpg "1472451611233.jpg"
  [6]: ./images/1472451794295.jpg "1472451794295.jpg"
  [7]: ./images/1472451805806.jpg "1472451805806.jpg"
  [8]: ./images/1472451912348.jpg "1472451912348.jpg"
  [9]: ./images/1472452122938.jpg "1472452122938.jpg"
  [10]: ./images/1472452217313.jpg "1472452217313.jpg"
  [11]: ./images/1472452312071.jpg "1472452312071.jpg"
  [12]: ./images/1472452346604.jpg "1472452346604.jpg"
  [13]: ./images/1472452518843.jpg "1472452518843.jpg"
  [14]: ./images/1472452614436.jpg "1472452614436.jpg"
  [15]: ./images/1472452728707.jpg "1472452728707.jpg"
  [16]: ./images/1472452992792.jpg "1472452992792.jpg"
  [17]: ./images/1472453044363.jpg "1472453044363.jpg"
  [18]: ./images/1472454498912.jpg "1472454498912.jpg"
  [19]: ./images/1472454649920.jpg "1472454649920.jpg"
  [20]: ./images/1472454672973.jpg "1472454672973.jpg"
  [21]: ./images/1472454766982.jpg "1472454766982.jpg"
  [22]: ./images/1472454919335.jpg "1472454919335.jpg"
  [23]: ./images/1472455048051.jpg "1472455048051.jpg"
  [24]: ./images/1472455073298.jpg "1472455073298.jpg"
  [25]: ./images/1472455130482.jpg "1472455130482.jpg"
  [26]: ./images/1472455203362.jpg "1472455203362.jpg"
  [27]: ./images/1472455321655.jpg "1472455321655.jpg"
  [28]: ./images/1472455398878.jpg "1472455398878.jpg"
  [29]: ./images/1472455844212.jpg "1472455844212.jpg"
  [30]: ./images/1472455918509.jpg "1472455918509.jpg"
  [31]: ./images/1472456012490.jpg "1472456012490.jpg"
  [32]: ./images/1472456061823.jpg "1472456061823.jpg"
  [33]: ./images/1472456298169.jpg "1472456298169.jpg"
  [34]: ./images/1472456310383.jpg "1472456310383.jpg"
  [35]: http://blog.sina.com.cn/s/blog_52d20fbf0100ofd5.html
  [36]: http://www.runoob.com/mysql/mysql-index.html
  [37]: http://www.cnblogs.com/lina1006/archive/2011/04/29/2032894.html
  [38]: http://www.ha97.com/4197.html
