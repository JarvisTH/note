---
title:  awk命令
comments: true
category: [linux, awk]
keywords: [shell]
date: 2021-10-12 21:30:40
update: 2021-10-12 21:30:40
description:  awk命令
tags: [shell,linux]
---

# awk命令

## 1.awk命令

AWK 是一种处理文本文件的语言，是一个强大的文本分析工具。

## 2.语法

```shell
awk [选项参数] 'script' var=value file(s)
或
awk [选项参数] -f scriptfile var=value file(s)
```

**项参数说明：**

- -F fs or --field-separator fs
  指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F:。
- -v var=value or --asign var=value
  赋值一个用户定义变量。
- -f scripfile or --file scriptfile
  从脚本文件中读取awk命令。
- -mf nnn and -mr nnn
  对nnn值设置内在限制，-mf选项限制分配给nnn的最大块数目；-mr选项限制记录的最大数目。这两个功能是Bell实验室版awk的扩展功能，在标准awk中不适用。
- -W compact or --compat, -W traditional or --traditional
  在兼容模式下运行awk。所以gawk的行为和标准的awk完全一样，所有的awk扩展都被忽略。
- -W copyleft or --copyleft, -W copyright or --copyright
  打印简短的版权信息。
- -W help or --help, -W usage or --usage
  打印全部awk选项和每个选项的简短说明。
- -W lint or --lint
  打印不能向传统unix平台移植的结构的警告。
- -W lint-old or --lint-old
  打印关于不能向传统unix平台移植的结构的警告。
- -W posix
  打开兼容模式。但有以下限制，不识别：/x、函数关键字、func、换码序列以及当fs是一个空格时，将新行作为一个域分隔符；操作符**和**=不能代替^和^=；fflush无效。
- -W re-interval or --re-inerval
  允许间隔正则表达式的使用，参考(grep中的Posix字符类)，如括号表达式[[:alpha:]]。
- -W source program-text or --source program-text
  使用program-text作为源代码，可与-f命令混用。
- -W version or --version
  打印bug报告信息的版本。

## 3.基本用法

内容：

```
2 this is a test
3 Are you like awk
This's a test
10 There are orange,apple,mongo
```

### 用法一：分割列

```shell
awk '{[pattern] action}' {filenames}   # 行匹配语句 awk '' 只能用单引号
```

实例：

```shell
[root@centos-7-jarvis test]# awk '{print $1, $4}' testfile
2 a
3 like
This's
10 orange,apple,mongo
// 格式化打印
[root@centos-7-jarvis test]# awk '{printf  "%-8s %-10s\n", $1, $4}' testfile
2        a
3        like
This's
10       orange,apple,mongo
```

使用awk打印第1列、第4列，分割符为空格。

### 用法二：指定分割符

```shell
awk -F  #-F相当于内置变量FS, 指定分割字符
```

#### 基础：

```shell
[root@centos-7-jarvis test]# awk -F, '{print $1,$3}' testfile
2 this is a test
3 Are you like awk
This's a test
10 There are orange mongo
```

以","为分割符分割列，打印第一列与第三列，该行没有逗号，则分割为第一列。

#### 使用内建变量：

```shell
[root@centos-7-jarvis test]# awk 'BEGIN{FS=","} {print $1,$3}' testfile
2 this is a test
3 Are you like awk
This's a test
10 There are orange mongo
```

| 运算符                  | 描述                             |
| :---------------------- | :------------------------------- |
| = += -= *= /= %= ^= **= | 赋值                             |
| ?:                      | C条件表达式                      |
| \|\|                    | 逻辑或                           |
| &&                      | 逻辑与                           |
| ~ 和 !~                 | 匹配正则表达式和不匹配正则表达式 |
| < <= > >= != ==         | 关系运算符                       |
| 空格                    | 连接                             |
| + -                     | 加，减                           |
| * / %                   | 乘，除与求余                     |
| + - !                   | 一元加，减和逻辑非               |
| ^ ***                   | 求幂                             |
| ++ --                   | 增加或减少，作为前缀或后缀       |
| $                       | 字段引用                         |
| in                      | 数组成员                         |

| 变量        | 描述                                              |
| :---------- | :------------------------------------------------ |
| $n          | 当前记录的第n个字段，字段间由FS分隔               |
| $0          | 完整的输入记录                                    |
| ARGC        | 命令行参数的数目                                  |
| ARGIND      | 命令行中当前文件的位置(从0开始算)                 |
| ARGV        | 包含命令行参数的数组                              |
| CONVFMT     | 数字转换格式(默认值为%.6g)ENVIRON环境变量关联数组 |
| ERRNO       | 最后一个系统错误的描述                            |
| FIELDWIDTHS | 字段宽度列表(用空格键分隔)                        |
| FILENAME    | 当前文件名                                        |
| FNR         | 各文件分别计数的行号                              |
| FS          | 字段分隔符(默认是任何空格)                        |
| IGNORECASE  | 如果为真，则进行忽略大小写的匹配                  |
| NF          | 一条记录的字段的数目                              |
| NR          | 已经读出的记录数，就是行号，从1开始               |
| OFMT        | 数字的输出格式(默认值是%.6g)                      |
| OFS         | 输出字段分隔符，默认值与输入字段分隔符一致。      |
| ORS         | 输出记录分隔符(默认值是一个换行符)                |
| RLENGTH     | 由match函数所匹配的字符串的长度                   |
| RS          | 记录分隔符(默认是一个换行符)                      |
| RSTART      | 由match函数所匹配的字符串的第一个位置             |
| SUBSEP      | 数组下标分隔符(默认值是/034)                      |

#### 使用多分割符

```shell
[root@centos-7-jarvis test]# awk -F '[ ,]' '{print $1,$2,$5}' testfile
2 this test
3 Are awk
This's a
10 There apple
```

先使用空格分割符，再使用逗号分割符进行分割。多分割符放在[]中。

### 用法三：设置变量

```shell
awk -v  # 设置变量
```

实例：

```shell
[root@centos-7-jarvis test]# awk -va=2 '{print $1,$1+a}' testfile
2 4
3 5
This's 2
10 12
```

设置变量a=2，然后打印第一列的值，以及第一例值加a的和值。分割符为空格，第一列为非数字则值为0；

### 用法四：调用脚本

```shell
awk -f {awk脚本} {文件名}
```

awk脚本结构：

- BEGIN{ 这里面放的是执行前的语句 }：开始块就是在程序启动的时候执行的代码部分，并且它在整个过程中只执行一次。

  一般情况下，我们可以在开始块中初始化一些变量。开始块部分是可选的。

- {这里面放的是处理每一行时要执行的语句}：对于每一个输入的行都会执行一次主体部分的命令。

- END {这里面放的是处理完所有的行后要执行的语句 }：结束块是在程序结束时执行的代码，结束块也是可选的

结构：

```shell
awk 'BEGIN{ commands } pattern{ commands } END{ commands }'
```



## 4.AWK数组

AWK 可以使用关联数组这种数据结构，索引可以是数字或字符串。不需要提前声明其大小，因为它在运行时可以自动的增大或减小。

```
arr_name[index]=value
```

#### 1.创建数组

```shell
[root@centos-7-jarvis ~]# awk 'BEGIN {sites['1111']='1111';
> sites['2222']='2222'
> print sites['2222'],sites['2222']
> }'
2222 2222
```

#### 2.删除元素

使用 delete 语句来删除数组元素。

```shell
delete arr_name[index]
```

```shell
[root@centos-7-jarvis ~]# awk 'BEGIN {arr["1"]="aaaaa";
arr["2"]="asdsds"
delete arr["2"]
print arr["2"]
> print arr["1"]
> }'

aaaaa
```

#### 3.多维数组

AWK 本身不支持多维数组，不过可以模拟多维数组。

模拟二维数组的例子：

```shell
$ awk 'BEGIN {
array["0,0"] = 100;
array["0,1"] = 200;
array["0,2"] = 300;
array["1,0"] = 400;
array["1,1"] = 500;
array["1,2"] = 600;
}'
```

在数组上可以执行很多操作，比如，使用 asort 完成数组元素的排序，或者使用 asorti 实现数组索引的排序等等。

## 5.条件与循环

- IF：

  ```shell
  if (condition)
  {
  	xxx
  }
  ```

- IF-ELSE:

  ```shell
  if (condition)
  {
  	xxx
  }
  else
  {
  	xxx
  }
  ```

- IF-ELSE-IF:

  ```shell
  if(condition)
  {
  	xxx
  } else if(condition)
  {
  	xxx
  } else if(condition)
  {
  	xxx
  }
  ```

- FOR

  ```shell
  for (initialisation; condition; increment/decrement)
  {
  	xxx
  }
  ```

- WHILE-Break/Continue

  ```shell
  while (condition)
  {
  	xxxx
  }
  ```

- Exit:Exit 用于结束脚本程序的执行。该函数接受一个整数作为参数表示 AWK 进程结束状态。 如果没有提供该参数，其默认状态为 0。

  ```shell
  [root@centos-7-jarvis ~]# awk 'BEGIN {
  >    sum = 0; for (i = 0; i < 20; ++i) {
  >       sum += i; if (sum > 50) exit(10); else print "Sum =", sum
  >    }
  > }'
  Sum = 0
  Sum = 1
  Sum = 3
  Sum = 6
  Sum = 10
  Sum = 15
  Sum = 21
  Sum = 28
  Sum = 36
  Sum = 45
  您在 /var/spool/mail/root 中有新邮件
  [root@centos-7-jarvis ~]# echo $?
  10
  ```

## 6.函数

- 自定义函数：

```
function function_name(argument1, argument2, ...)
{
    function body
}
```

- 内置函数

主要有以下几种：

- [算数函数](https://www.runoob.com/w3cnote/awk-built-in-functions.html#b1)
- [字符串函数](https://www.runoob.com/w3cnote/awk-built-in-functions.html#b2)
- [时间函数](https://www.runoob.com/w3cnote/awk-built-in-functions.html#b3)
- [位操作函数](https://www.runoob.com/w3cnote/awk-built-in-functions.html#b4)
- [其它函数](https://www.runoob.com/w3cnote/awk-built-in-functions.html#b5)

## 7.常用内置变量

分为两种类型的内置变量: - 第一种是定义的变量可以改变,　比如字段分隔(FS)与记录分隔(RS) - 第二种是可以用来数据处理或者数据总结，比如记录数(NR)与字段数目(NF) 。

主要涉及：

### FS：输入字段分隔符变量

**FS**(Field Separator) 读取并解析输入文件中的每一行时，默认按照空格分隔为字段变量,$1,$2...等。**FS** 变量被用来设置每一记录的字段分隔符号。**FS** 可以是任意的字符串或者正则表达式.你可以使用下面两种方式来声名FS:

- ```shell
  awk -F 'FS' 'commands' inputfilename
  ```

- ```shell
  awk 'BEGIN{FS="FS";}'
  ```

**FS** 可以是任意字符或者正则表达式，**FS** 可以多次改变, 不过会保持不变直到被明确修改。不过如果想要改变字段分隔符,　最好是在读入文本之前就改变 **FS**, 这样改变才会在你读入的文本生效。



### OFS: 输出字段分隔符变量

**OFS**(Output Field Separator) 相当与输出上的 **FS**, 默认是以一个空格字符作为输出分隔符的



### RS: 记录分隔符

**RS**(Record Separator)定义了一行记录。读取文件时，默认将一行作为一条记录。



### ORS: 输出记录分隔符变量

**ORS**(Output Record Separator)顾名思义就相当与输出的 **RS**。 每条记录在输出时候会用分隔符隔开



### NR: 记录数变量

**NR**(Number of Record) 表示的是已经处理过的总记录数目，或者说行号(不一定是一个文件，可能是多个)。



### NF:一条记录的记录数目

**NF**(Number for Field)表示的是，一条记录的字段的数目.　它在判断某条记录是否所有字段都存在时非常有用。



### FILENAME: 当前输入文件的名字

**FILENAME** 表示当前正在输入的文件的名字。 AWK 可以接受读取很多个文件去处理。



### FNR: 当前输入文件的记录数目

当awk读取多个文件时，**NR** 代表的是当前输入**所有文件**的全部记录数，而 **FNR** 则是**当前文件**的记录数。
