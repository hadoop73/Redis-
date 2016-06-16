---
title: Linux Shell 脚本攻略(四)
tags: Shell,脚本,文本
grammar_cjkRuby: true
---

[TOC]

##  正则表达式
**正则表达式** 基于样式匹配的文本处理技术。
![enter description here][1]

元字符是一种`Perl` 风格的正则表达式，只有一部分文本处理工具支持。
![enter description here][2]

##  用 `grep` 在文件中搜索文本
在文件中搜索一个单词
```bash?linenums
$ grep match_pattern filename --color=auto

this is the line containing match_pattern  
# 结果为一行,重点标记匹配的单词
```
`grep` 使用正则表达式，添加 `-E` 选项(扩展),`-o` 只输出文件中匹配到的文本部分。`-c` 只是统计匹配行的数量
```bash?linenums
$ grep -e "[a-z]+"

# 或者如下

$ egrep "[a-z]+"

# 为了文件中统计匹配项的数量
$ echo -e "1 2 3 4\nhello\n5 6" | egrep -o "[0-9]" | wc -l
```
##  用 `cut` 按列切分文件
`cut` 将文本按列进行切分，在 `cut` 中，每列都是一个字段。
```bash?linenums
# 提取第一个字段或列 -complement进行补集运算
# cut -f FIELD_LIST filename
$ cut -f 2,3 filename

# 用 -d 选项指定字段的定界符
$ cut -f2 -d ";" filename

# 用 -c 指定范围
$ cut -c 1-2 filename

# 用 --output-delimiter "," 定义输出定界符
$ cut filename -c 1-3,6-9 --output-delimiter ","
```
##  `sed` 入门
`sed` 是 `stream editor` (流编辑器)的缩写,可以用来进行**文本替换**。
**文本替换**使用 `-i` 选项，可以将替换结果应用于原文件。
```bash?linenums
$ sed 's/pattern/replace_string' file
```
当需要从第N处开始替换时，可以使用 `/Ng`
```bash?linenums
$ echo this thisthisthis | sed 's/this/THIS/4g'
thisthisthisTHIS
```
同样可以使用 `/ : |` 作为定界符使用。

**移除空白行**
```bash?linenums
# /pattern/d 会移除匹配样式
$ sed '/^$/d' file
```
**引用**
```bash?linenums
$ text=hello
$ echo hello world | sed "s/$text/HELLO/"
```
##  `awk` 入门
`awk` 用于数据流，可以对列和行进行操作。
`awk` 由3部分组成：`BEGIN` 语句块、`END` 语句块和能够使用模式匹配的通用语句块；这3部分都是可选的。
```bash?linenums
awk 'BEGIN{ statements } { statements } END{ statements }'
```
工作方式如下：
* 执行 `BEGIN { commands }` 语句块
* 从文件或 `stdin` 中读取一行，执行 `pattern { commands }`。重复这个过程，直到文件全部被读取完毕。
* 当读至输入流末尾时，执行 `END { commands }` 语句块

![enter description here][3]

![enter description here][4]

**设置字段定界符**
```bash?linenums
$ awk -F: '{ print $NF }' /etc/passwd
# 或者
$ awk 'BEGIN { FS=":" } { print $NF }' /etc/passwd
```
**`getline` 命令**
```bash?linenums
# "command" | getline output ;
$ echo | awk '{ "grep root /etc/passwd" | getline cmdout ; print cmdout }'
```
**移除 `\n` 和 `\t`**
```bash?linenums
$ tr -d '\n\t'
```
##  按列合并文件
`paste` 命令实现按列拼接
```bash?linenums
$ paste file1 file2 file3 ...
```
##  打印文件第n列
广泛用法是借助 `awk`。当然，用 `cut` 也可以
```bash?linenums
# 打印第5列
$ awk '{ print $5 }' filename

# 打印多列，并在列间插入指定字符串
$ ls -l | awk '{ print $1 " : " $8}'
```
##  打印不同行
```bash?linenums
$ awk 'NR==M, NR==N' filename
```
##  文本切片与参数操作
```bash?linenums
# 替换变量内容中的部分文本
$ var="This is a line of text"
$ echo ${var/line/PEPLACED}
This is a PEPLACED of text

# 指定字符串的起始位置和长度来生成子串
$ string= adfahgadlgjllkjpoj
$ echo ${string:4:2}
```











  [1]: ./images/1459168842233.jpg "1459168842233.jpg"
  [2]: ./images/1459168937798.jpg "1459168937798.jpg"
  [3]: ./images/1459175750137.jpg "1459175750137.jpg"
  [4]: ./images/1459176112345.jpg "1459176112345.jpg"