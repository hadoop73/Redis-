---
title: Linux Shell 脚本攻略(三) 
tags: Shell,脚本,文件
grammar_cjkRuby: true
---

[TOC]

##  文本文件的交集与差集
`comm` 用于两个文件之间的比较
![enter description here][1]

```bash?linenums
$ sort A.txt -o A.txt
$ sort B.txt -o B.txt
$ comm A.txt B.txt
# 输出第一列只在`A.txt` 中出现的行
# 第二列包含在`B.txt` 中出现的行
# 第三列包含`A.txt` 和`B.txt` 相同的行
```
![enter description here][2]

```bash?linenums
# 两个文件的交集，删除第一列和第二列，只打印第三列
$ comm A.txt B.txt -1 -2
```
##  `head` 与`tail`
**打印前几行**
```bash?linenums
$ head -n 4 file

# 打印除了最后N行之外所有的行
$ head -n -N file   # -N 表示一个负数
```

##  只列出目录的其他方法
```bash?linenums
# 列出当前路径下的目录
$ ls -d */

$ find . -type d -maxdepth 1 -print
```
##  统计文件的行数、单词数和字符数
`wc` 是一个统计工具。`Word Count` 的缩写，用来统计文件的行数、单词数和字符数。
```bash?linenums
# 统计行数
$ wc -l file

# 统计单词数
$ wc -w file

# 统计字符数
$ wc -c file

# 打印文件行数、单词数和字符数，彼此用制表符分隔
$ wc file
```
##  打印目录树
```bash?linenums
$ tree ~/unixfx
```
**重点标记出匹配某种样式的文件**
```bash?linenums
tree Path -p PATTERN

# 打印文件和目录大小，-h选项
# 只重点标记出除符合某种样式之外的文件，-I 选项
```















  [1]: ./images/1459068404762.jpg "1459068404762.jpg"
  [2]: ./images/1459068564423.jpg "1459068564423.jpg"