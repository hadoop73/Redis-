---
title: Linux Shell 脚本攻略
tags: Shell,脚本
grammar_cjkRuby: true
---

[TOC]

`$` 表示普通用户，`#` 表示超级用户(`root user`)。超级用户是`Linux` 系统中权限最高的用户。

`shell` 脚本以`#!` 开始
```bash
#!/bin/bash
```
被称为`shebang` 的特殊行，在脚本自己独立运行时，必须利用`shebang` 行，通过使用位于`#!` 之后的解释器来运行脚本。

![enter description here][1]

##  终端打印
`echo` 用于终端打印，每次调用后会添加一个换行符。文本可以带双引号(`""`)，也可以不带，还可以只带单引号(`''`)
```bash
$ echo "Welcome to Bash" # 不能打印!
$ echo Welcome to Bash   # 文本中有`;`时，会当做两个命令的分隔符
$ echo 'Welcome to Bash' # 不会对变量($var)求值
```

`printf` 能够格式化输出文本，可以指定字符串的宽度、左右对齐方式等；和`C`语言中的`printf` 函数一样。

![enter description here][2]

**`%s、%c、%d` 和`%f` 都是格式替代符**，对应的参数可以放在带引号的格式符字符串之后。


**在`echo` 中转移换行符**
`echo` 默认会将一个换行符输出到文本尾部，可以用标志`-n` 来忽略结尾的换行符。`echo` 接受双引号字符串内的转义序列作为参数。采用`echo -e 包含转义序列的字符串`。
```bash
echo -e "1\t2\t3"
```

##  玩转变量
获取变量的内容：
```bash
var="value"  # 给变量var赋值
echo $var    # 打印变量的内容
echo ${var}

len=${#var}  # 获取变量的长度
```
检查是否超级用户
```bash
if [ $UID -ne 0 ]; then  # if 用法
echo Non root user. Please run as root.
else
echo "Root user."
fi   # root用户的UID是0
```

##  通过`shell` 进行数学运算
在`Bash shell`中，利用`let、(())、[]` 执行算术操作。高级操作时，`expr` 和`bc`
```bash
#!/bin/bash
no1=4;
no2=5;

# let 用法
let result=no1+no2;
let no1++;
let no1--;
let no1+=6;

# (()) 和 []
res=$[ no1 + no2 ]
res=$[ $no1 + 5 ]

res=$(( no1 + 50 ))

# expr 用于算术操作
res=`expr 3 + 4`
res=$(expr $no1 + 5)
```
`bc` 用于数学运算的高级工具，可以借助它执行浮点数运算并应用一些高级函数
```bash
echo "4 * 0.56" | bc
2.24
```
**设定小数精度**
参数`scale=2` 将小数位个数设置为2
```bash
echo "scale=2;3/8" | bc
0.37
```
**进制转换**
```bash
no=100
echo "obase=2;$no" | bc
1100100
no=1100100
echo "obase=10;ibase=2;$no" | bc
100
```
**计算平方以及平方根**
```bash
echo "sqrt(100)" | bc  # Square root
echo "10^10" | bc  # Square
```

##  文件描述符和重定向
标准输入(`stdin`)、标准输出(`stdout`)和标准错误(`stderr`)
**文件描述符** 是与一个打开的文件或数据流相关联的整数。文件描述符0、1以及2是系统预留的
* `0 stdin` (标准输入)
* `1 stdout` (标准输出)
* `2 stderr` (标准错误)

输出文本**重定向**或保存到一个文件中
```bash
# 输出文本存储到文件temp.txt中，写入之前，temp.txt中的内容首先会被清空
echo "This is a sample text 1" > temp.txt

# 输出文本追加到目标文件中
echo "This is a sample text 2" >> temp.txt

# 将错误信息和输出写入文件
$ cmd 2>&1 output.txt
$ cmd &> output.txt
```
**`tee`** 一方面将数据重定向到文件，另一方面将数据的副本作为后续命令的`stdin`
```bash
# tee只能从stdin中进行读取，会把文件覆盖
cat a* | tee out.txt | cat -n
# tee 提供一个 -a 选项，可以用于追加内容
$ cat a* | tee -a out.txt | cat -n
```

**将文件重定向到命令**
```bash
$ cmd < file
```

**重定向脚本内部的文本块**
```bash
#!/bin/bash
cat <<EOF>>log.txt
LOG FILE HEADER
This is a test content.
```
在`cat <>log.txt` 与下一个`EOF` 行之间的所有文本行被当做`stdin` 数据被输入到`log.txt`。

自定义文件描述符
```bash
exec 5>>input.txt  # 定义一个文件描述符，用于打开文件input.txt
echo appended lin >&5
```
##  数组和关联数组
普通数组只能使用**整数** 作为数组索引，而关联数组可以使用**字符串** 作为数组索引。
```bash
# 以0为起始索引
$ array_var=(1 2 3 4)

# 打印所有值
$ echo ${array_var[*]}
$ echo ${array_var[@]}

# 打印数组长度
$ echo ${#array_var[*]}
```

**定义关联数组**
```bash
# 将一个变量名声明为关联数组
$ declare -A ass_array

# 关联数组的初始化
$ ass_array=([index1]=val1 [index2]=val2)
$ ass_array[index1]=val1

# 列出数组索引
$ echo ${!array_var[*]}
$ echo ${!array_var[@]}
```
##  别名
创建一个别名
```bash
$ alias new_command='command sequence'
```
为了使别名设置一直保持作用，可以将它放入`~/.bashrc` 文件中。
```bash
$ echo 'alias cmd="command seq"' >> ~/.bashrc
```
##  获取、设置日期和延时
格式化打印对应格式的日期
```bash
$ date "+%d %B %Y"
20 May 2010

# 打印秒数
$ date +%s
```
##  调试脚本
使用选项`-x`，启动跟踪调试`shell` 脚本
```bash
$ bash -x script.sh
```
##  函数和参数
定义函数
```bash
function fname()
{
    statements;
}
# 或者
fname()
{
    statements;
}
```
**使用参数**
```bash
fname arg1 arg2;
# 获取参数
fname()
{
    echo $1,$2;  # 访问参数1和参数2
    echo "$@";   # 以列表的方式一次性打印所有参数
    echo "$*";   # 类似$@,但是参数被作为单个实体
    return 0;    # 返回值
}
```
**读取命令返回值**
```bash
$ cmd;
echo $?;
```
##  以不按回车键的方式读取字符 “n”
从输入中读取`n` 个字符并存入变量`variable_name`
```bash
$ read -n number_of_chars variable_name

$ read -n 2 var
$ echo $var

# 用不回显的方式读取密码
$ read -s var

# 显示提示信息
$ read -p "Enter input:" var

# 用界定符结束输入行
$ read -d ":" var
```
##  字段分隔符和迭代器
`IFS` 是存储定界符的环境变量
```bash
#!/bin/bash
line="root:x:0:0:root:/root:/bin/bash"
oldIFS=$IFS;
IFS=":"
count=0
for item in $line;
do
[ $count -eq 0 ] && user=$item;
[ $count -eq 6 ] && shell=$item;
let count++
done;
IFS=$oldIFS
echo $user\'s shell is $shell;

# 输出
root's shell is /bin/bash
```
**`for`、`while`、`until`**
```bash
for((i=0;i<10;i++))
{
    commands;
}

while condition
do
    commands;
done
```
在`bash` 中，`until` 会一直循环直到给定条件为真
```bash
x=0;
until [ $x -eq 9 ]; 
do let x++; echo $ $x;
done
```
##  比较与测试
```bash
[ condition ] && action;    # 如果condition为真，则执行action
[ condition ] || action;    # 如果condition为假，则执行action
```
- `-eq` 等于
- `-ne` 不等于
- `-gt` 大于
- `-lt` 小于
- `-ge` 大于或等于
- `-le` 小于或等于

**文件系统相关测试**
[ `-d $var` ] 如果给定的变量包含的是目录，则返回真
[ `-e $var` ] 如果给定的变量的文件存在，则返回真

**字符串比较**
字符串比较时，最好用双中括号，采用单个中括号会产生错误
```bash
[[ $str1 ==$str2]]
[[ $str1 = $str2 ]] #当str1等于str2时，返回真
[[ -z $str1 ]] # 如果str1包含的是空字符串，则返回真
[[ -n $str1 ]] # 如果str1包含非空字符串，则返回真
```


  [1]: ./images/1472458041145.jpg "1472458041145.jpg"
  [2]: ./images/1472458173331.jpg "1472458173331.jpg"
  [3]: ./images/1458826908061.jpg "1458826908061.jpg"
