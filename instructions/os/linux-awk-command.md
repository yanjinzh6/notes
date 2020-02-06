---
title: Linux awk 命令
date: 2020-02-06 21:30:00
tags: '操作系统'
categories:
  - ['使用说明', '操作系统']
permalink: linux-awk-command
---

# 简介

`awk` 是一种编程语言, 用于在 `linux/unix` 下对文本和数据进行处理. 数据可以来自标准输入 (stdin), 一个或多个文件, 或其它命令的输出. 它支持用户自定义函数和动态正则表达式等先进功能, 是 `linux/unix` 下的一个强大编程工具. 它在命令行中使用, 但更多是作为脚本来使用. `awk` 有很多内建的功能, 比如数组, 函数等, 这是它和C语言的相同之处, 灵活性是awk最大的优势. 

# 常用

> 这篇文章仅介绍 `awk` 命令的简单应用

```sh
# 打印 PID
~ ps -ef | awk 'NR != 1 {print $2}'
1
2

# 指定分隔符
~ awk 'BEGIN{FS=":"} {print $1}' /etc/passwd
~ awk -F ':' '{print $1}' /etc/passwd
~ awk -F '[;:]' '{print $1}' /etc/passwd

# 使用重定向拆分文件
~ ps -ef | awk 'NR != 1 {print > $1}'
~ ps -ef | awk 'NR != 1 {print $2,$3 > $1}'
~ ps -ef | awk 'NR != 1 {if ($1 ~ /root/) print > "1.txt"; else if($1 ~ /\d/) print > "2.txt"; else print > "3.txt"}'

# 统计文件大小
~ ls -l *.txt | awk '{sum += $5} END {print sum}'

# 按某一列计数
~ ps -ef | awk 'NR!=1 {a[$1]++;} END {for (i in a) print i ", " a[i];}'
www-data, 4
syslog, 1

# 统计每个用户的进程的占了多少内存
~ ps aux | awk 'NR!=1 {a[$1]+=$6;} END { for(i in a) print i ", " a[i]"KB";}'

# 从 file 文件中找出长度大于 80 的行
~ awk 'length>80' file

# 按连接数查看客户端 IP
~ netstat -ntu | awk '{print $5}' | cut -d: -f1 | sort | uniq -c | sort -nr

# 九九乘法表
~ seq 9 | sed 'H;g' | awk -v RS='' '{for(i=1;i<=NF;i++)printf("%dx%d=%d%s", i, NR, i*NR, i==NR?"\n":"\t")}'
1x1=1
1x2=2   2x2=4
1x3=3   2x3=6   3x3=9
1x4=4   2x4=8   3x4=12  4x4=16
1x5=5   2x5=10  3x5=15  4x5=20  5x5=25
1x6=6   2x6=12  3x6=18  4x6=24  5x6=30  6x6=36
1x7=7   2x7=14  3x7=21  4x7=28  5x7=35  6x7=42  7x7=49
1x8=8   2x8=16  3x8=24  4x8=32  5x8=40  6x8=48  7x8=56  8x8=64
1x9=9   2x9=18  3x9=27  4x9=36  5x9=45  6x9=54  7x9=63  8x9=72  9x9=81
```

<!-- more -->

# 基本用法

```sh
# 格式
~ awk [-F 分隔符] '{commands}' file(s)
```

**多个分隔符可以使用 [] 隔开**

```sh
~ echo 'this is a test \r\nthat was a test2' | awk '{print $0}'
this is a test 
that was a test2
```

`awk` 命令后面的单引号里面的大括号就是处理的动作, 这里的 `print $0` 中 `print` 是打印命令, `$0` 是指所有列, 所以会打印出所有内容

`awk` 会根据空格和制表符, 将每一行分成若干字段, 依次用 `$1`, `$2`, `$3` 代表第一列, 第二列, 第三列等等. 

```sh
~ echo 'this is a test \r\nthat was a test2' | awk '{print $1}'
this
that

~ echo 'this is a test \r\nthat was a test2' | awk '{print $1,$4}'
this test
that test
```

这里的 `$1` 表示输出内容的第一列, `$4` 表示输出内容的第四列

# 内置变量

| 变量        | 功能                                                       |
| ----------- | ---------------------------------------------------------- |
| $n          | 当前记录的第n个字段, 字段间由FS分隔                        |
| $0          | 完整的输入记录                                             |
| ARGC        | 命令行参数的数目                                           |
| ARGIND      | 命令行中当前文件的位置(从0开始算)                          |
| ARGV        | 包含命令行参数的数组                                       |
| CONVFMT     | 数字转换格式(默认值为%.6g)ENVIRON环境变量关联数组          |
| ERRNO       | 最后一个系统错误的描述                                     |
| FIELDWIDTHS | 字段宽度列表(用空格键分隔)                                 |
| FILENAME    | 当前文件名                                                 |
| FNR         | 各文件分别计数的行号                                       |
| FS          | 字段分隔符(默认是任何空格)                                 |
| IGNORECASE  | 如果为真, 则进行忽略大小写的匹配                           |
| NF          | 一条记录的字段的数目                                       |
| NR          | 已经读出的记录数, 就是行号, 从1开始                        |
| OFMT        | 数字的输出格式(默认值是%.6g)                               |
| OFS         | 输出记录分隔符（输出换行符）, 输出时用指定的符号代替换行符 |
| ORS         | 输出记录分隔符(默认值是一个换行符)                         |
| RLENGTH     | 由match函数所匹配的字符串的长度                            |
| RS          | 记录分隔符(默认是一个换行符)                               |
| RSTART      | 由match函数所匹配的字符串的第一个位置                      |
| SUBSEP      | 数组下标分隔符(默认值是/034)                               |

```sh
# 例子
~ awk -F ':' 'NR == 1 {print "filename:" FILENAME ",linenumber:" NR ",columns:" NF ",linecontent:"$0}' /etc/passwd
filename:/etc/passwd,linenumber:1,columns:7,linecontent:root:x:0:0:root:/root:/usr/bin/zsh
```

# 常用函数

| 函数      | 功能            |
| --------- | --------------- |
| toupper() | 字符转为大写.   |
| tolower() | 字符转为小写.   |
| length()  | 返回字符串长度. |
| substr()  | 返回子字符串.   |
| sin()     | 正弦.           |
| cos()     | 余弦.           |
| sqrt()    | 平方根.         |
| rand()    | 随机数.         |

```sh
# 例子
~ echo 'this is a test \r\nthat was a test2' | awk '{print toupper($1)}'                                  
THIS
THAT
```

# 条件

| 运算符                  | 功能                             |
| ----------------------- | -------------------------------- |
| = += -= *= /= %= ^= **= | 赋值                             |
| ?:                      | C条件表达式                      |
| `||`                    | 逻辑或                           |
| &&                      | 逻辑与                           |
| ~ 和 !~                 | 匹配正则表达式和不匹配正则表达式 |
| < <= > >= != ==         | 关系运算符                       |
| 空格                    | 连接                             |
| + -                     | 加, 减                           |
| * / %                   | 乘, 除与求余                     |
| + - !                   | 一元加, 减和逻辑非               |
| ^ ***                   | 求幂                             |
| ++ --                   | 增加或减少, 作为前缀或后缀       |
| $                       | 字段引用                         |
| in                      | 数组成员                         |

```sh
# 格式
~ awk 'conditions {commands}' file(s)
```

```sh
# 运算符
~ echo 'this is a test \r\nthat was a test2' | awk '$1 == "this" {print $1}'
this

# 正则表达式
~ echo 'this is a test \r\nthat was a test2' | awk '/a/ {print $1}'
this
that

# 模式
~ echo 'this is a test \r\nthat was a test2' | awk '$1 ~ /a/ {print $1}'
that
~ echo 'this is a test \r\nthat was a test2' | awk '$4 ~ /2/ {print $1}'
that

# 模式取反
~ echo 'this is a test \r\nthat was a test2' | awk '$1 !~ /a/ {print $1}'
this

# 大于第一行
~ echo 'this is a test \r\nthat was a test2' | awk 'NR > 1 {print $1}'
that
```

```sh
# if 条件
~ awk -F ':' '{if ($1 == "root") print $1; else print "---"}' /etc/passwd
root
---
---
```

# 脚本

- `BEGIN{ 这里面放的是执行前的语句 }`
- `END {这里面放的是处理完所有的行后要执行的语句 }`

脚本包含两个特殊字段 `BEGIN` 和 `END`, 使用 `BEGIN` 语句设置计数和打印头, `BEGIN` 语句使用在任何文本浏览动作之前, 之后文本浏览动作依据输入文件开始执行; `END` 语句用来在 `awk` 完成文本浏览动作后打印输出文本总数和结尾状态标志, 有动作必须使用 `{}` 括起来

```sh
$ cat score.txt
Marry   2143 78 84 77
Jack    2321 66 78 45
Tom     2122 48 77 71
Mike    2537 87 97 95
Bob     2415 40 57 62

$ cat cal.awk
#!/bin/awk -f
#运行前
BEGIN {
    math = 0
    english = 0
    computer = 0
 
    printf "NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL\n"
    printf "---------------------------------------------\n"
}
#运行中
{
    math+=$3
    english+=$4
    computer+=$5
    printf "%-6s %-6s %4d %8d %8d %8d\n", $1, $2, $3,$4,$5, $3+$4+$5
}
#运行后
END {
    printf "---------------------------------------------\n"
    printf "  TOTAL:%10d %8d %8d \n", math, english, computer
    printf "AVERAGE:%10.2f %8.2f %8.2f\n", math/NR, english/NR, computer/NR
}

$ awk -f cal.awk score.txt
NAME    NO.   MATH  ENGLISH  COMPUTER   TOTAL
---------------------------------------------
Marry  2143     78       84       77      239
Jack   2321     66       78       45      189
Tom    2122     48       77       71      196
Mike   2537     87       97       95      279
Bob    2415     40       57       62      159
---------------------------------------------
  TOTAL:       319      393      350
AVERAGE:     63.80    78.60    70.00
```

# 环境变量

```sh
$ x=5
 
$ y=10
$ export y
 
$ echo $x $y
5 10
 
$ awk -v val=$x '{print $1, $2, $3, $4+val, $5+ENVIRON["y"]}' OFS="\t" score.txt
Marry   2143    78      89      87
Jack    2321    66      83      55
Tom     2122    48      82      81
Mike    2537    87      102     105
Bob     2415    40      62      72
```

# 引用

- [The GNU Awk User’s Guide](http://www.gnu.org/software/gawk/manual/gawk.html)
- [awk 入门教程](http://www.ruanyifeng.com/blog/2018/11/awk.html)
- [AWK 简明教程](https://coolshell.cn/articles/9070.html)