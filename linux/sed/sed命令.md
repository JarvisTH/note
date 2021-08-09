# 1.sed 命令

## 一、sed

Linux sed 命令是利用脚本来**处理文本文件**。

Sed 主要用来**自动编辑一个或多个文件、简化对文件的反复操作、编写转换程序**等。



## 二、语法

```shell
sed [-hnV][-e<script>][-f<script文件>][文本文件]
```

**参数说明**：

- -e<script>或--expression=<script> 以选项中指定的script来处理输入的文本文件。
- -f<script文件>或--file=<script文件> 以选项中指定的script文件来处理输入的文本文件。
- -h或--help 显示帮助。
- -n或--quiet或--silent 仅显示script处理后的结果。
- -V或--version 显示版本信息。

**动作说明**：

- a ：新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
- c ：取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
- d ：删除，因为是删除啊，所以 d 后面通常不接任何内容；
- i ：插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
- p ：打印，亦即将某个选择的数据印出。**通常 p 会与参数 sed -n 一起运行**～
- s ：取代，可以直接进行取代的工作，通常这个 s 的动作可以搭配正规表示法！例如 1,20s/old/new/g ！

## 三、实例

在testfile文件的第四行后添加一行，并将结果输出到标准输出，在命令行提示符下输入如下命令：

```shell
[root@centos-7-jarvis test]# sed -e 4a\newline testfile
HELLO LINUX!
Linux is a free unix-type opterating system.
This is a linux testfile!
Linux test
newline
```

这里只是读取testfile内容，在此基础上的第4行后新增一行，但是整个文本内容并没有进行保存。

### 1.以行为单位的新增/删除

```shell
nl /etc/passwd |sed '2,5d'
```

#### sed 的动作为 '2,5d' ，那个 d 就是删除，原本应该是要下达 sed -e 才对，没有 -e 也行。sed 后面接的动作，请务必以两个单引号括住。

```shell
[root@centos-7-jarvis test]# nl /etc/passwd |sed '2,5d'
     1  root:x:0:0:root:/root:/bin/bash
     6  sync:x:5:0:sync:/sbin:/bin/sync
     7  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
     8  halt:x:7:0:halt:/sbin:/sbin/halt
     9  mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
    10  operator:x:11:0:operator:/root:/sbin/nologin
    11  games:x:12:100:games:/usr/games:/sbin/nologin
    12  ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
    13  nobody:x:99:99:Nobody:/:/sbin/nologin
    14  systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
    15  dbus:x:81:81:System message bus:/:/sbin/nologin
    16  polkitd:x:999:998:User for polkitd:/:/sbin/nologin
    17  sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
    18  postfix:x:89:89::/var/spool/postfix:/sbin/nologin
    19  chrony:x:998:996::/var/lib/chrony:/sbin/nologin
    20  mysql:x:997:995:MySQL server:/var/lib/mysql:/sbin/nologin
    21  jarvis:x:1000:1000::/home/jarvis:/bin/bash
    22  jarvis1:x:1001:1001:jarvis th1:/home/jarvis1:/bin/bash
    23  jarvis2:x:1002:1003:jarvis th2:/home/jarvis2:/bin/bash
    24  test:x:1003:1004::/home/test:/bin/bash
```



#### 删除3到最后一行：

```shell
[root@centos-7-jarvis test]# nl /etc/passwd |sed '3,$d'
     1  root:x:0:0:root:/root:/bin/bash
     2  bin:x:1:1:bin:/bin:/sbin/nologin
```



#### 在第二行后加上“test action”字符串：

```shell
[root@centos-7-jarvis test]# nl /etc/passwd |sed '2a test action'|sed '4,$d'
     1  root:x:0:0:root:/root:/bin/bash
     2  bin:x:1:1:bin:/bin:/sbin/nologin
test action
[root@centos-7-jarvis test]# nl /etc/passwd |sed '2a test action'|sed '5,$d'
     1  root:x:0:0:root:/root:/bin/bash
     2  bin:x:1:1:bin:/bin:/sbin/nologin
test action
     3  daemon:x:2:2:daemon:/sbin:/sbin/nologin
```



#### 增加两行以上的字符串：

```shell
[root@centos-7-jarvis test]# nl /etc/passwd |sed '2a test action1 ..........\
test action2?'|sed '6,$d'
     1  root:x:0:0:root:/root:/bin/bash
     2  bin:x:1:1:bin:/bin:/sbin/nologin
test action1 ..........
test action2?
     3  daemon:x:2:2:daemon:/sbin:/sbin/nologin
```

每一行之间都必须要以反斜杠『 \ 』来进行新行的添加。



### 2.以行为单位的替换与显示

#### 将第2-5行的内容取代成为『No 2-5 number』:

```shell
[root@centos-7-jarvis test]# nl /etc/passwd |sed '2,5c No 2-5 number'|sed '4,$d'
     1  root:x:0:0:root:/root:/bin/bash
No 2-5 number
     6  sync:x:5:0:sync:/sbin:/bin/sync
```



#### 只打印5-7行：

```shell
[root@centos-7-jarvis test]# nl /etc/passwd |sed -n '5,7P'
     5  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
     6  sync:x:5:0:sync:/sbin:/bin/sync
     7  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
```



### 3.数据的搜寻与显示

#### 搜索有root关键字的行：

```shell
[root@centos-7-jarvis test]# nl /etc/passwd|sed -n '/root/p'
     1  root:x:0:0:root:/root:/bin/bash
    10  operator:x:11:0:operator:/root:/sbin/nologin
```

如果root找到，除了输出所有行，还会输出匹配行,**使用-n的时候将只打印包含模板的行**。



### 4.数据搜寻并删除

删除所有包含root，输出其他行：

```shell
[root@centos-7-jarvis test]# nl /etc/passwd|sed '/root/d'
     2  bin:x:1:1:bin:/bin:/sbin/nologin
     3  daemon:x:2:2:daemon:/sbin:/sbin/nologin
     4  adm:x:3:4:adm:/var/adm:/sbin/nologin
     5  lp:x:4:7:lp:/var/spool/lpd:/sbin/nologin
     6  sync:x:5:0:sync:/sbin:/bin/sync
     7  shutdown:x:6:0:shutdown:/sbin:/sbin/shutdown
     8  halt:x:7:0:halt:/sbin:/sbin/halt
     9  mail:x:8:12:mail:/var/spool/mail:/sbin/nologin
    11  games:x:12:100:games:/usr/games:/sbin/nologin
    12  ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
    13  nobody:x:99:99:Nobody:/:/sbin/nologin
    14  systemd-network:x:192:192:systemd Network Management:/:/sbin/nologin
    15  dbus:x:81:81:System message bus:/:/sbin/nologin
    16  polkitd:x:999:998:User for polkitd:/:/sbin/nologin
    17  sshd:x:74:74:Privilege-separated SSH:/var/empty/sshd:/sbin/nologin
    18  postfix:x:89:89::/var/spool/postfix:/sbin/nologin
    19  chrony:x:998:996::/var/lib/chrony:/sbin/nologin
    20  mysql:x:997:995:MySQL server:/var/lib/mysql:/sbin/nologin
    21  jarvis:x:1000:1000::/home/jarvis:/bin/bash
    22  jarvis1:x:1001:1001:jarvis th1:/home/jarvis1:/bin/bash
    23  jarvis2:x:1002:1003:jarvis th2:/home/jarvis2:/bin/bash
    24  test:x:1003:1004::/home/test:/bin/bash
```

可见第一行、第十行被删除。



### 5.数据搜寻并执行命令

搜索/etc/passwd,找到root对应的行，执行后面花括号中的一组命令，每个命令之间用分号分隔，这里把bash替换为blueshell，再输出这行：

```shell
[root@centos-7-jarvis test]# nl /etc/passwd|sed '/root/{s/bash/blueshell/;p;q}' -n
     1  root:x:0:0:root:/root:/bin/blueshell
```

最后的**q是退出**。



### 6.数据的搜寻与替换

sed 可以用行为单位进行部分数据的搜寻并取代，基本上 sed 的搜寻与替代的与 vi 相当的类似，模式如下：

> ```shell
> sed 's/要被取代的字串/新的字串/g'
> ```

先观察原始信息，利用 /sbin/ifconfig 查询 IP

```shell
[root@centos-7-jarvis test]# /sbin/ifconfig  ens33
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.52.128  netmask 255.255.255.0  broadcast 192.168.52.255
        inet6 fe80::20c:29ff:fedd:3891  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:dd:38:91  txqueuelen 1000  (Ethernet)
        RX packets 644997  bytes 819479547 (781.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 157725  bytes 66308714 (63.2 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

本机ip是192.168.52.128。将ip前面部分删除：

```shell
ens: error fetching interface information: Device not found
[root@centos-7-jarvis test]# /sbin/ifconfig ens33|grep 'inet'|sed 's/^.*inet //g'
192.168.52.128  netmask 255.255.255.0  broadcast 192.168.52.255
        inet6 fe80::20c:29ff:fedd:3891  prefixlen 64  scopeid 0x20<link>
```

然后是删除ip后面部分：

```shell
[root@centos-7-jarvis test]# /sbin/ifconfig ens33|grep 'inet'|sed 's/^.*inet //g'|sed 's/netmask.*$//g'
192.168.52.128
        inet6 fe80::20c:29ff:fedd:3891  prefixlen 64  scopeid 0x20<link>
```



### 7.多点编辑

一条sed命令，删除/etc/passwd第三行到末尾的数据，并把bash替换为blueshell：

```shell
[root@centos-7-jarvis test]# nl /etc/passwd |sed -e '3,$d' -e 's/bash/blueshell/'
     1  root:x:0:0:root:/root:/bin/blueshell
     2  bin:x:1:1:bin:/bin:/sbin/nologin
```

**-e表示多点编辑**，第一个编辑命令删除/etc/passwd第三行到末尾的数据，第二条命令搜索bash替换为blueshell。



### 8.直接修改文件内容（危险动作）

sed 可以直接修改文件的内容，将直接在 regular_express.txt 最后一行加入 **# This is a test**:：

```shell
[root@centos-7-jarvis test]# sed -i '$a # This is a test' testfile
您在 /var/spool/mail/root 中有新邮件
[root@centos-7-jarvis test]# cat testfile
HELLO LINUX!
Linux is a free unix-type opterating system.
This is a linux testfile!
Linux test~
# This is a test
```

因为$ 代表的是最后一行，而 a 的动作是新增，因此该文件最后新增 **# This is a test**！

sed 的 **-i** 选项可以直接修改文件内容。对于在大文件在中修改某一行非常有用，不要vim加载太久。