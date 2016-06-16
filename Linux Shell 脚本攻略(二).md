---
title: Linux Shell 脚本攻略(二) 
tags: Shell,脚本
grammar_cjkRuby: true
---

[TOC]

##  用cat进行拼接
`cat` 通常用于读取、显示或拼接文件内容
**拼接文件内容**
```bash
# 逐个文件依次输出
$ cat file1 file2 file3
```
将输入文件的内容与标准输入拼接在一起
```bash
$ echo 'Text through stdin' | cat - file.txt
```
**显示文件内容**
```bash
# 文本中多个空行被压缩成单个
$ cat -s file
```
显示行号
```bash
$ cat -n lines.txt
```

##  文件查找与文件列表
`find` 命令：沿着**文件层次结构向下遍历**，匹配符合条件的文件，并执行相应操作
```bash
# 打印文件和目录的列表
$ find . -print
```
`.` 指定当前目录，`..` 指定父目录
`-print` 指明打印出匹配文件的文件名(路径)。`\n` 作文定界符。
`-print0` 指明使用`\0` 作为定界符来打印每一个匹配的文件名。

**根据文件名或正则表达式匹配搜索**
选项`-name` 的参数指定文件名所必须匹配的字符串。
```bash
# 打印所有以`.txt` 结尾的文件名
$ find . -name "*.txt" -print
# -iname(忽略字母大小写)

# 匹配多个条件中的一个，采用OR条件操作
$ find . \( -name "*.txt" -o -name "*.pdf" \) -print
# \(以及\) 用于将 -name "*.txt" -o -name "*.pdf" 视为一个整体
```
正则表达式匹配
```bash
# -iregex用于忽略正则表达式的大小写
$ find . -regex ".*\(\.py|\.sh\)$"
```
**否定参数**
`find` 可以用`!` 否定参数的含义
```bash
# 匹配所有不以.txt结尾的文件名
$ find . ! -name "*.txt" -print
```

##  基于目录深度搜索
`find` 命令会遍历所有的子目录，采用一些深度参数来限制`find` 命令遍历深度。`-maxdepth` 和`-mindepth`
```bash
$ find . -mindepth 2 -type f -print
```
`-maxdepth` 和`-mindepth` 应该作为`find` 的第3个参数出现，如果作为第4个参数，可能影响效率。如果`-maxdepth` 作为第4个参数，`-type` 作为第3个参数，`find` 首先会找出符合`-type` 的所哟文件，然后再找出符合指定深度的那些。
`type` 参数所匹配的文件类型
![enter description here][1]

**根据文件时间进行搜索**
![enter description here][2]
```bash
# 打印在最近七天内被访问过的所有文件
$ find . type f -atime -7 -print
```
**基于文件大小的搜索**
```bash
# 大于2kb的文件
$ find . -type f -size +2k
```
**删除匹配的文件**
```bash
# 删除当前目录下所有.swp文件
$ find . -type f -name "*.swp" -delete
```

##  玩转 `xargs`
`xargs` 擅长将标准输入数据转化成命令行参数，紧跟在管道操作符之后，以标准输入作为主要的源数据流
```bash
$ command | xargs
```
**将多行输入转换成单行输出**
```bash
# 移除换行符，再以" "(空格)进行替换
$ cat example.txt | xargs
```
**将单行输入转换成多行输出**
指定每行最大的参数数量n，可以将 `stdin` 的文本划分成多行，每行n个参数。
```bash
$ cat example.txt | xargs -n 3
```
**定界符分割**
可以用自己的定界符来分割参数。用 `-d` 选项来输入指定一个定制的定界符
```bash
$ echo "splitXsplitXsplitXsplit" | xargs -d X
split split split split
```
**读取 `stdin`，将格式化参数传递给命令**
```bash
# 从args.txt 中读取内容，每次以两个参数给cecho.sh运行
$ cat args.txt | xargs -n 2 ./cecho.sh
```
当`-I` 与`xargs` 结合使用时，对于每一个参数命令都会被执行一次
```bash
# 对于每一个命令参数，字符串{}会被stdin读取到的参数所替换
$ cat args.txt | xargs -I {} ./cecho.sh -p {} -1
```
##  用`tr` 进行转换
`tr` 可以对来自标准输入的字符进行替换、删除以及压缩，将一组字符变成另一组字符，因而被称为**转换** 命令。
```bash
# 将stdin的输入字符从set1映射到set2
# 如果两个字符集的长度不相等，那么set2会不断重复最后一个字符，直到长度与set1相同
# 如果set2的长度大于set1，超出的部分字符全部被忽略
$ tr [options] set1 set2

# 将输入字符由大写转换成小写
$ echo "HELL WHO IS THIS" | tr 'A-z' 'a-z'
```
**用`tr` 删除字符**
`tr` 有一个选项`-d`，指定需要被删除的字符集合
```bash
$ cat file.txt | tr -d '[set1]'

$ echo 'Helo 123 world 456' | tr -d '0-9'
Helo world # 输出结果
```
**字符串补集**
```bash
$ tr -c [set]

$ echo hello 1 char 2 next 4 | tr -d -c '0-9 \n'
1 2 4
```
**`tr` 压缩字符**
```bash
$ echo "GNU is  not   UNIX." | tr -s ' '
GNU is not UNIX.
```
巧妙方式用`tr` 将文件中数字列表进行相加
```bash
$ cat sum.txt
1
2
3

$ cat sum.txt | echo $[ $(tr '\n' '+') 0]
# 相当于执行
$ echo $[ 1+2+3+0 ]
```
##  排序、单一与重复
```bash
# 按数字排序
$ sort -n file.txt

# 按逆序排序
$ sort -r file.txt
```
**按键或列进行排序**
```bash
$ cat data.txt
1   mac     2000
2   winxp   4000
3   bsd     1000
4   linux   1000

# 依据第一列，以逆序形式排序
$ sort -nrk 1 data.txt
4   linux   1000
3   bsd     1000
2   winxp   4000
1   mac     2000
```
**`-k` 指定排序按照哪一个键进行**
![enter description here][3]

突出显示的字符将用作数值键，为了提取这个键，用字符的起止位置作为键的书写格式。

**`uniq`**
通过消除重复内容，从给定输入中找出单一的行，只能用于排过序的数据输入
```bash
$ uniq sortedltxt

# 统计各行在文件中出现的次数
$ sort unsorted.txt | uniq -c

# 找出文件中重复的行
$ sort unsorted.txt | uniq -d
```
##  分割文件和数据
```bash
$ split -b 10k data.file
```
##  根据扩展名切分文件名
借助`%` 操作符将名称部分从 “名称.扩展名” 这种格式的文件名中提取出来。
```bash
file_jpg="sample.jpg"
name=${file_jpg%.*}         # 获得sample
extension=${file_jpg#*.}    # 获得扩展名jpg
```
![enter description here][4]

`%` 属于非贪婪操作。从右到做找到匹配通配符的最短结果。
`%%` 属于贪婪的，匹配符合条件的最长的字符串。

![enter description here][5]







  [1]: ./images/1459047470809.jpg "1459047470809.jpg"
  [2]: ./images/1459047622842.jpg "1459047622842.jpg"
  [3]: ./images/1459065926578.jpg "1459065926578.jpg"
  [4]: ./images/1459067793169.jpg "1459067793169.jpg"
  [5]: ./images/1459067961870.jpg "1459067961870.jpg"