## 1.c/java/python/shell执行方式对比

单行脚本： 将shell命令写到同一行而不带其他逻辑符号，只是简单的进行了命令排序，不管前一个命令是否执行成功，后一个都会执行；只有加入逻辑符号（&&、||等），前一个执行成功才会执行下一个命令。

shell脚本最开始的 #！符号叫shebang，定义脚本使用哪个解释器执行。在执行脚本时如果执行了解释器，这里定义的解释器就不生效。不同的程序需要使用不同解释器执行，例如python的解释器不能执行shell，对于不同程序应该写明各自的解释器。

- C 语言： 编译方式，执行的时候已经完成了逻辑关系解释，可以直接执行，性能较快，机器码
- Java 语言：编译字节码，java虚拟机执行字节码，性能不如C语言。
- Shell：解释方式，执行的时候才去关注逻辑关系，性能较慢
- perl/expect：解释方式执行
- python：编译方式，编译为字节码由python虚拟机执行； 解释方式；

```shell
#!/usr/bin/bash
ping 127.0.0.1

/usr/bin/python <<-EOF
print "hello world"
EOF

echo "bash hello"
```

在bash脚本中，可以套用其他语言的逻辑，只需要使用对应的解释器就可以。

## 2. login shell/no login shell

影响shell的文件：

su - user:加 **-** 的是login shell，不加的就是no login shell。1-4是登录shell时执行的，5-6是退出登录时执行。不加 **-** 进行su执行2、4，加了 **-**  执行1-4。

- 1./etc/profile
- 2./etc/bashrc：系统级
- 3.~/.bash_profile：用户级
- 4.~/.bashrc
- 5.~/.bash_logout
- 6.~/.bash_history

## 3.前后台任务控制

&是将当前任务放到后台执行，但是如果当前的shell窗口被关闭，后台任务也会被关闭；配合使用nohup，可以在shell窗口被关闭后，仍然保持后台任务继续执行。

推荐screen，使用screen定义一个会话名称，以便下一次连接可以接续上一个会话。

ctrl + c：kill 前台进程； ctrl + z：暂停前台进程 

## 4.管道 | tee

| 可以连接多个命令，将上一个命令的输出作为下一个命令的输入。

tee管道可以将上一个命令的输出打印到文件中。

```shell
# 覆盖
ip addr|grep 'inet'|tee test.txt|grep eth0  
# 追加
ip addr|grep 'inet'|tee -a test|grep eth0
```

 ## 5.命令排序

- ；号：不具备逻辑判断
- && 号：具备逻辑判断
- || 号：具备逻辑判断
- command & ：后台执行，不是命令排序
- command &> /dev/null ：混合重定向

## 6.shell 元字符

1. *号，匹配多个任意字符
2. ？号，匹配任意一个字符
3. [] 号，匹配括号中任意一个字符
4. () 号， 在子shell中执行括号中的命令
5. {} 号，集合，mkdir /home/{11,22} 就是在home目录下建立11、22目录
6. \ 号，转义字符，让元字符回归本意

## 7.echo 带颜色输出文本

前景色颜色是从30-37，背景色颜色是从40-47

```shell
# 设置文本颜色
echo -e "\e[1;31m This is a red text]."
# 恢复文本颜色
echo -e "\e[0m"

echo -e "\e[1;31m This is a red text\e[0m."
```

printf 命令格式化输出

## 8.Shell变量

使用一个特定的字符串去表示不固定的内容

### 1.自定义变量

定义：变量名=变量值

引用变量：$变量名 或 ${变量名}

查看变量：echi $变量名  ， set 所有变量

取消变量：unset 变量名

作用范围：当前shell



### 2.环境变量

定义：export bak_xx=/home/bak

​		  export bakxxx 将自定义变量转换为环境变量

引用环境变量：$变量名 或 ${变量名}

查看环境变量：echi $变量名  ，env 

取消环境变量：unset 变量名

作用范围：当前shell、子shell



### 3.位置变量

脚本参数导入的变量，：$1 $2 ... 



### 4.预定义变量

- $0：脚本名
- $*：所有参数
- $@：所有参数
- $#：参数个数
- $$:当前进程的pid
- $!：上一个后台进程的pid
- $?：上一个命令返回值

```shell
#!/usr/bin/bash
#如果用户没有加参数
if [ $# -eq 0 ];then
	echo "usage: `basename $0` file"
fi

# basename 脚本路径：获取脚本名称，dirname 是获取脚本目录路径
```

### 5.变量赋值方式

- 显示赋值
- read从键盘读入：

```shell
read -p "info: " var
```



注意：

" "号是弱引用；

' '号是强引用，这单引号中无法引用其他变量，即使使用$ 符号，当作字符串；

``号等价于$() ，命令替换，里面的命令会被先执行

### 6.变量运算

- 整数运算：
  - expr ：expr 1+2
  - $(())：$((5+2))
  - $[]：$[6+7]
  - **let**：let sum=5+6
- 小数运算：
  - bc：echo ”2*5/3“ | bc

### 7.变量内容的删除与替换

file=/dir1/dir2/dir3/my.file.txt

可以用${ }分别替换得到不同的值：

**${file#*/}**：删掉第一个 / 及其左边的字符串：dir1/dir2/dir3/my.file.txt

**${file##*/}**：删掉最后一个 /  及其左边的字符串：my.file.txt

${file#*.}：删掉第一个 .  及其左边的字符串：file.txt

**${file##*.}**：删掉最后一个 .  及其左边的字符串：txt

${file%/*}：删掉最后一个  /  及其右边的字符串：/dir1/dir2/dir3

**${file%%/*}**：删掉第一个 /  及其右边的字符串：(空值)

**${file%.*}**：删掉最后一个  .  及其右边的字符串：/dir1/dir2/dir3/my.file

${file%%.*}：删掉第一个  .  及其右边的字符串：/dir1/dir2/dir3/my

【记忆的方法为】：

\# 是 去掉左边（键盘上#在 $ 的左边）

%是去掉右边（键盘上% 在$ 的右边）

单一符号是最小匹配；两个符号是最大匹配

${file:0:5}：提取最左边的 5 个字节：/dir1

${file:5:5}：提取第 5 个字节右边的连续5个字节：/dir2

也可以对变量值里的字符串作替换：

${file/dir/path}：将第一个dir 替换为path：/path1/dir2/dir3/my.file.txt

${file//dir/path}：将全部dir 替换为 path：/path1/path2/path3/my.file.txt

## 9.条件测试

格式：

- test 条件表达式
- [ 条件表达式 ]
- [[ 条件表达式 ]]

```shell
man test
```

主要包含文件测试、数值比较、字符串比较（使用双引号）

```shell
test -d /home
[ -d /home ]

5 -gt 9

"aa"="bb"
"aaa"=="bbb"
```

变量为空或者未定义，长度都是0。

## 10.if判断

- 单分支

```shell
if 条件测试 ;then
	命令
if
```

- 双分支

```shell
if 条件测试 ;then
	命令
else 
	命令
fi
```

- 多分支

```shell
if 条件测试1 ;then
	命令
elif 条件测试2 ;then
	命令
fi
```

## 11.case

```shell
case 变量 in
模式一)
	命令
	;;
模式二)
	命令
	;;
模式三)
	命令
	;;
*)
	命令
esac
```

## 12、for循环

```shell
for 变量名 [in 取值列表]
do
	循环体
done
```

## 13、expect

一个根据脚本与其他交互式程序进行交互。通过在脚本中设定期望值和响应值进行交互操作。主要应用于执行命令和程序时， 系统以交互形式要求输入指定字符串，实现交互通信。expect工具可以控制、处理输入，输出流，然后提供自动填写数据等需要用户交互式输入的数据的地方实现自动化处理。Expect 就是为了处理“自动交互”的工具。

```shell
#!/user/bin/expect

# 开启一个会话
spawn ssh root@192.168.73.128

# 把交互会出现什么，怎么做先写好
expect {
	"yes/no" { send "yes\r"; exp_continue } # 当出现yes/no 字符串，就发送yes+回车，不出现就继续
	"password" { send "centos\r" };	# 出现password字符串，发送密码，分号结束
}
interact # 停在对端交互
```



**expect 工作原理：pawn启动指定进程—expect获取指定关键字—send向指定程序发送指定字符—执行完成退出**

```shell
vim test.exp => 一般将expect脚本的后缀命名为".exp"

#!/usr/bin/expect => expect的解析器，与shell中的#!/bin/bash类似

set timeout n => 设置超时时间n秒，表示下面的代码需在n秒钟内完成，如果超过，则退出。用来防止ssh远程主机网络不可达时卡住及在远程主机执行命令宕住

set name "12345" => set设置变量，name的值为123456

spawn command1 command2.. => 执行命令，也可以将变量作为命令输入

expect{ => 接受执行命令返回的信息

"accept1" {send "instruction1\r"; exp_continue} => 匹配到accest1，发送instruction1 指令并且\r 回车执行

"accept2" {send "instruction2\r"; exp_continue} => 匹配到accest2，发送instruction2 指令并且\r 回车执行，

 exp_continue表示循环匹配

"accept3" {send "\r"; exp_continue} => 匹配到accept3表示直接回车执行

"accept4" {send "$name\r"} => 匹配到accept4,将变量值作为指令，并且回车执行

}
```

 

expect 启用选项：

-c 执行脚本前先执行的命令，可多次使用 
-d debug模式，可以在运行时输出一些诊断信息，与在脚本开始处使用exp_internal 1相似。 
-D 启用交换调式器,可设一整数参数。 
-f 从文件读取命令，仅用于使用#!时。如果文件名为"-"，则从stdin读取(使用"./-"从文件名为-的文件读取)。 
-i 交互式输入命令，使用"exit"或"EOF"退出输入状态 
-- 标示选项结束(如果你需要传递与expect选项相似的参数给脚本时)，可放到#!行:#!/usr/bin/expect -- 
-v 显示expect版本信息

expect 命令参数：

- spawn 交互程序开始，执行后面的命令或程序。需要进入到expect环境才可以执行，不能直接在shell环境下直接执行，可以处理ssh、ftp等交互
- set timeout n 设置超时时间，表示该脚本代码需在n秒钟内完成，如果超过，则退出。用来防止ssh远程主机网络不可达时卡住及在远程主机执行命令宕住。如果设置为-1表示不会超时
- set 定义变量
- $argv expect脚本可以接受bash的外部传参，可以使用[ lindex $argv n ]n为0表示第一个传参，为1表示第二个传参
- expect 从交互程序进程中指定接收信息, 如果匹配成功, 就执行send的指令交互；否则等待timeout秒后自动退出expect语句
- send 如果匹配到expect接受到的信息，就将send中的指令交互传递，执行交互动作。结尾处加上\r表示如果出现异常等待的状态可以进行核查 
- exp_continue 表示循环式匹配，通常匹配之后都会退出语句，但如果有exp_continue则可以不断循环匹配，输入多条命令，简化写法。 
- exit 退出expect脚本 
- expect eof spawn进程结束后会向expect发送eof，接收到eof代表该进程结束 
- interact 执行完代码后保持交互状态，将控制权交给用户。没有该命令执行完后自动退出而不是留在远程终端上 
- puts 输出变量



**多分支例子：**

```shell
#!/usr/bin/expect

set timeout 30
spawn command

exect {
"Match message1" { send "instruction1\r";exp_continue} => exp_continu表示可以多次匹配
"Match message2" { send "instruction2\r";exp_continue }
"Match message3" { send "\r"} => 代表直接回车选择默认选项
...
}

interact => 不退出，保持交互状态
```



**登录例子**：

```shell
# vim test.exp

#!/usr/bin/expect

set host [ lindex $argv 0 ]
set ip [ lindex $argv 1 ]
set passwd [ lindex $argv 2 ]
set cmd [ lindex $argv 3 ]

if {$argc < 4} {
 puts "Usage:cmd <hostname> <ipaddress> <password> <cmd>"
 exit 1
}

set timeout 20

spawn ssh -l $host $ip $cmd

expect "password"
send "$passwd\r"

expect eof => 注意：就算 这里是interact也不会以保持交互状态
```

**shell中嵌套expect**

```shell
#!/bin/bash

$ip=10.1.1.15
$user=root
$pass=toor

/usr/bin/expect <<-EOF
set timeout 20
spawn ssh $user@$IP

expect {
 "yes/no" { send "yes\r"; exp_continue }
 "password" { send "$pass\r" }
}

expect "]#"
send "touch file{1..10}"
expect eof

EOF
```

## 14、while与until循环

```shell
while 条件测试
do
	命令
	break/continue/exit
done

until 条件测试  # 条件为假时执行
do
	命令
done
```

## 15、并发执行

一般是多进程并发执行。

```shell
#!/usr/bin/bash
for i in {1..254}
do
        {
            ping -c1 -W1 127.0.0.1 &>/dev/null
            echo $i
        }&
done
wait

[root@localhost ~]# ./test.sh
1
15
19
55
```

上面这种并发，对于小数量的并发ok，但是大规模并发就可能会报错，**需要控制并发数量**。

**文件描述符：File Description（FD），也叫文件句柄**，进程使用文件描述符来管理打开的文件，只要文件被打开就一直存在，直到被关闭释放。位于目录/proc/下进程的fd目录中：

```shell
[root@localhost proc]# ll /proc/$$/fd			
total 0
lrwx------. 1 root root 64 Dec 29 02:14 0 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 1 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 2 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:29 255 -> /dev/pts/2
[root@localhost fd]# echo $$			# $$ 表示当前进程  可以写pid
8455
[root@localhost fd]# ll /proc/8455/fd
total 0
lrwx------. 1 root root 64 Dec 29 02:14 0 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 1 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 2 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:29 255 -> /dev/pts/2
```

**如何打开文件和关闭文件**？

```shell
[root@localhost ~]# touch file1
[root@localhost ~]# exec 6<> ./file1		# 打开一个文件
[root@localhost ~]# ll /proc/$$/fd
total 0
lrwx------. 1 root root 64 Dec 29 02:14 0 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 1 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 2 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:29 255 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 6 -> /root/file1
[root@localhost ~]# echo "test" > /root/file1
[root@localhost ~]# cat file1
test
[root@localhost ~]# rm -rf file1
[root@localhost ~]# ll /proc/$$/fd
total 0
lrwx------. 1 root root 64 Dec 29 02:14 0 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 1 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 2 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:29 255 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 6 -> /root/file1 (deleted)
[root@localhost ~]# exec 6<&-				# 关闭文件
[root@localhost ~]# ll /proc/$$/fd
total 0
lrwx------. 1 root root 64 Dec 29 02:14 0 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 1 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:14 2 -> /dev/pts/2
lrwx------. 1 root root 64 Dec 29 02:29 255 -> /dev/pts/2
```

文件被删除后，即使使用备份恢复，原FD仍然显示被删除，因为每打开一个文件，它的nodeid都不同。**如果文件的句柄没有被释放，即使文件被删除了，它的fd也还在。**



**管道**：|

![image-20230110232300825](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230110232300825.png)

一般使用的管道 | 是**匿名管道**，只能在一个终端。

**命名管道**：管道也是一个文件，只能读一次，可以跨终端。

```shell
# 终端1
[root@localhost ~]# mkfifo /tmp/fifo1			
[root@localhost ~]# file /tmp/fifo1
/tmp/fifo1: fifo (named pipe)
[root@localhost ~]# cat /tmp/fifo1
^C
[root@localhost ~]# ll /dev > /tmp/fifo1		# 1
[root@localhost ~]#							   

# 同时打开同一个环境的终端2
[root@localhost ~]# grep 'sda' /tmp/fifo1		# 2
brw-rw----.  1 root disk      8,   0 Dec 29 00:34 sda
brw-rw----.  1 root disk      8,   1 Dec 29 00:34 sda1
brw-rw----.  1 root disk      8,   2 Dec 29 00:34 sda2
```

在终端2执行完  2 后，一直处于等待管道中的内容；当终端1执行完 1 后，终端2才显示内容。

```shell
#!/usr/bin/bash

process=5
tmp_fifo=/tmp/$$.fifo

mkfifo $tmp_fifo
exec 8<> $tmp_file
rm $tmp_fifo

for i in `seq $process`
do
	echo >&8	# 写了5个回车
done


for i in {1..254}
do
		read -u 8 	# 读到文件描述符才能循环
        {
            ping -c1 -W1 127.0.0.1 &>/dev/null
            echo $i
            echo >&8
        }&
done
wait
```

上面的脚本，利用了管道的内容只能读取一次的特性，向文件描述符中写入了5个回车，并发时获取到一个内容才能继续循环，相当于一个锁，每读一次消耗一个，循环体执行完再加一个锁。

## 16、数值

数组可以一次赋值一个下标，也可以一次赋值全部值。

还可以将命令执行回执结果的每一行作为值，赋值为数值。

declare命令默认不会识别出管理数组，所以需要声明关联数组。

### 1.普通数值

只能使用**整数**作为数值索引。

```shell
[root@localhost ~]# books=(linux shell awk sed)
[root@localhost ~]# declare -a				# 查看数组
declare -a BASH_ARGC='()'
declare -a BASH_ARGV='()'
declare -a BASH_LINENO='()'
declare -a BASH_SOURCE='()'
declare -ar BASH_VERSINFO='([0]="4" [1]="2" [2]="46" [3]="2" [4]="release" [5]="x86_64-redhat-linux-gnu")'
declare -a DIRSTACK='()'
declare -a FUNCNAME='()'
declare -a GROUPS='()'
declare -a PIPESTATUS='([0]="0")'
declare -a books='([0]="linux" [1]="shell" [2]="awk" [3]="sed")'
[root@localhost ~]# echo ${books[1]}
shell
[root@localhost ~]# echo ${books[@]}	# 查看数组所有元素
linux shell awk sed
[root@localhost ~]# echo ${books[*]}	# 查看数组所有元素
linux shell awk sed
[root@localhost ~]# echo ${#books[*]}  	# 查看数组元素个数
4
[root@localhost ~]# echo ${!books[*]}	# 获取数组元素索引
0 1 2 3
[root@localhost ~]# echo ${!student[*]}
name height age
[root@localhost ~]# echo ${books[*]:1}	# 从数组下标1开始 就是之前变量的切片操作
shell awk sed
[root@localhost ~]# echo ${books[*]:1:2}	# 从数组下标1开始，访问两个元素 
shell awk
[root@localhost ~]# arr=(test1 tes2 [10]=test10)
[root@localhost ~]# declare -a
declare -a PIPESTATUS='([0]="0")'
declare -a arr='([0]="test1" [1]="tes2" [10]="test10")' # 数组下标的跳变
```

```shell
# 变量的切片
[root@localhost ~]# name=person
[root@localhost ~]# echo ${name:1}
erson
[root@localhost ~]# echo ${name:1:2}
er
```



### 2.关联数值

可以使用**字符串**作为数值索引

```shell
# 一个一个赋值
[root@localhost ~]# declare -A person	# 声明管理数组
[root@localhost ~]# person[age]=jarvis
[root@localhost ~]# person[height]=170
[root@localhost ~]# person[name]=test
[root@localhost ~]# declare -A
declare -A BASH_ALIASES='()'
declare -A BASH_CMDS='()'
declare -A person='([name]="test" [height]="170" [age]="jarvis" )'
# 一次赋多个值
[root@localhost ~]# declare -A student=([name]=jarvis [age]=18 [height]=170)
[root@localhost ~]# declare -A
declare -A BASH_ALIASES='()'
declare -A BASH_CMDS='()'
declare -A person='([name]="test" [height]="170" [age]="jarvis" )'
declare -A student='([name]="jarvis" [height]="170" [age]="18" )'
[root@localhost ~]# echo ${student[name]}
jarvis
```



```shell
[root@localhost ~]# ls
anaconda-ks.cfg  dump.rdb  nohup.out  script  test  test1.sh  test.sh
[root@localhost ~]# fileArr=(`ls`)
[root@localhost ~]# declare -a
declare -a BASH_ARGC='()'
declare -a BASH_ARGV='()'
declare -a BASH_LINENO='()'
declare -a BASH_SOURCE='()'
declare -ar BASH_VERSINFO='([0]="4" [1]="2" [2]="46" [3]="2" [4]="release" [5]="x86_64-redhat-linux-gnu")'
declare -a DIRSTACK='()'
declare -a FUNCNAME='()'
declare -a GROUPS='()'
declare -a PIPESTATUS='([0]="0")'
declare -a books='([0]="linux" [1]="shell" [2]="awk" [3]="sed")'
declare -a fileArr='([0]="anaconda-ks.cfg" [1]="dump.rdb" [2]="nohup.out" [3]="script" [4]="test" [5]="test1.sh" [6]="test.sh")'
```

``是命令替换，先执行命令。**分割符（$IFS）是空格，所以写了6个下标，如果想整行输出到数组，需要重定义分隔符为换行符**。

```shell
[root@localhost ~]# OLD_IFS=$IFS
[root@localhost ~]# IFS=$'\n'
[root@localhost ~]# files=(`ls -l`)
[root@localhost ~]# IFS=$OLD_IFS
[root@localhost ~]# declare -a
declare -a fileArr='([0]="anaconda-ks.cfg" [1]="dump.rdb" [2]="nohup.out" [3]="script" [4]="test" [5]="test1.sh" [6]="test.sh")'
declare -a files='([0]="total 20" [1]="-rw-------. 1 root root 1260 May 25  2022 anaconda-ks.cfg" [2]="-rw-r--r--. 1 root root   77 Jun  4  2022 dump.rdb" [3]="-rw-------. 1 root root    0 Jun  5  2022 nohup.out" [4]="drwxr-xr-x. 2 root root   22 Jun  5  2022 script" [5]="-rwxr-xr-x. 1 root root  310 Jun  5  2022 test" [6]="-rwxr-xr-x. 1 root root  352 Dec 29 03:08 test1.sh" [7]="-rwxr-xr-x. 1 root root  114 Dec 29 02:20 test.sh")'
```

