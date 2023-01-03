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

