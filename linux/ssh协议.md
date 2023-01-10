## 一、概念

SSH——Secure-Shell的缩写，安全外壳协议。转为远程登录会话和其他网络服务提供安全性的协议。对数据包加密在传输。

基本框架：

![image-20230102170117125](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230102170117125.png)

## 二、SSH工作过程

- SSH版本协商：TCP连接后，协商版本号。
- 密钥和算法的协商：客户端与服务端支持的算法不完全相同，C/S各自发送自己支持的算法给对方进行协商。
- 认证阶段：进行请求的认证
- 会话请求阶段：服务端等待客户端的请求，然后进行处理返回
- 交互会话阶段：

## 三、shell中ssh的使用

### 1.SSH非免密登录执行——基于口令的验证

![image-20230102171116986](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230102171116986.png)



sshpass 指令：实现脚本中免密输入，去远端执行命令

```shell
yum install -y sshpass

[root@localhost ~]# sshpass -h
Usage: sshpass [-f|-d|-p|-e] [-hV] command parameters
   -f filename   Take password to use from file
   -d number     Use number as file descriptor for getting password
   -p password   Provide password as argument (security unwise)
   -e            Password is passed as env-var "SSHPASS"
   With no parameters - password will be taken from stdin

   -P prompt     Which string should sshpass search for to detect a password prompt
   -v            Be verbose about what you're doing
   -h            Show help (this screen)
   -V            Print version information
At most one of -f, -d, -p or -e should be used
```

```shell
[root@localhost ~]# sshpass -p123456 ssh root@192.168.73.130 "df -h"
[root@localhost ~]#
```

这里没有返回的原因是第一次使用ssh连接对端时，会出现一个确认交互：

```shell
[root@localhost ~]# ssh root@192.168.73.130 "df -h"
The authenticity of host '192.168.73.130 (192.168.73.130)' can't be established.
ECDSA key fingerprint is SHA256:5QF2ZGVRO5FJXK8x8uctoxaU9xQO7/ZBW53NtzAttj4.
ECDSA key fingerprint is MD5:90:11:5b:0c:64:24:df:c5:4c:c5:7d:74:a7:10:e5:f1.
Are you sure you want to continue connecting (yes/no)?
```

想要避免输入yes，需要ssh 命令指定参数 -o：

```shell
[root@localhost ~]# ssh -o StrictHostKeyChecking=no root@192.168.73.130 "df -h"
Warning: Permanently added '192.168.73.130' (ECDSA) to the list of known hosts.
root@192.168.73.130's password:
```

StrictHostKeyChecking默认值yes，所以第一次登陆时会交互确认，改为no就不需要交互。

优化后：

```shell
[root@localhost ~]# sshpass -p123456 ssh -o StrictHostKeyChecking=no root@192.168.73.130 "df -h"
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 898M     0  898M   0% /dev
tmpfs                    910M     0  910M   0% /dev/shm
tmpfs                    910M  9.4M  901M   2% /run
tmpfs                    910M     0  910M   0% /sys/fs/cgroup
/dev/mapper/centos-root   17G  1.5G   16G   9% /
/dev/sda1               1014M  150M  865M  15% /boot
```



### 2.SSH免密登录执行——基于密钥的验证

![image-20230102171230433](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230102171230433.png)

- 生成公钥与私钥--**ssh-keygen**

```shell
[root@localhost ~]# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:SbaIqC7klvVngz99GNFdbnhjBArj7oIAajYEU230rb4 root@localhost.localdomain
The key's randomart image is:
+---[RSA 2048]----+
|o..o.    o   ..  |
|..  o. .. o .  o |
| o .  . +.... =  |
|o .. . =.+ . o * |
|.+... o S..   + .|
|oo..... ..       |
|+ o ..o...o      |
|o+   o *.o .     |
|o.    E.o .      |
+----[SHA256]-----+
[root@localhost ~]# ls -la
total 60
dr-xr-x---.  5 root root  4096 Dec 29 00:56 .
dr-xr-xr-x. 17 root root   224 Jun  4  2022 ..
-rw-------.  1 root root  1260 May 25  2022 anaconda-ks.cfg
-rw-------.  1 root root 14181 Dec 29 00:38 .bash_history
-rw-r--r--.  1 root root    18 Dec 28  2013 .bash_logout
-rw-r--r--.  1 root root   176 Dec 28  2013 .bash_profile
-rw-r--r--.  1 root root   176 Dec 28  2013 .bashrc
-rw-r--r--.  1 root root   100 Dec 28  2013 .cshrc
-rw-r--r--.  1 root root    77 Jun  4  2022 dump.rdb
-rw-------.  1 root root  1063 Jun  4  2022 .mysql_history
-rw-------.  1 root root     0 Jun  5  2022 .mysql_history.TMP
-rw-------.  1 root root     0 Jun  5  2022 nohup.out
drwxr-----.  3 root root    19 Jun  4  2022 .pki
-rw-------.  1 root root    26 Jun  4  2022 .rediscli_history
drwxr-xr-x.  2 root root    22 Jun  5  2022 script
drwx------.  2 root root    57 Dec 29 01:17 .ssh
-rw-r--r--.  1 root root   129 Dec 28  2013 .tcshrc
-rw-r--r--.  1 root root   310 Jun  5  2022 test
[root@localhost ~]# cd .ssh/
[root@localhost .ssh]# ll
total 12
-rw-------. 1 root root 1679 Dec 29 01:17 id_rsa    # 私钥
-rw-r--r--. 1 root root  408 Dec 29 01:17 id_rsa.pub	# 公钥
-rw-r--r--. 1 root root  176 Dec 29 01:04 known_hosts
```

脚本中使用时，可以使用echo 配合管道符避免回车交互：

```shell
[root@localhost ~]# echo ""| ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): /root/.ssh/id_rsa already exists.
Overwrite (y/n)? [root@localhost ~]#
```

- 分发到对端节点--**ssh-copy-id**

```shell
[root@localhost .ssh]# ssh-copy-id root@192.168.73.130
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.73.130's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.73.130'"
and check to make sure that only the key(s) you wanted were added.
```

更好区分不同机器，最好配置一下ip与主机名称的映射：

```shell
[root@localhost ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
ip name
```

- 免密登录

```shell
[root@localhost .ssh]# ssh 192.168.73.130
System is booting up. See pam_nologin(8)
Last login: Mon Jan  9 17:33:13 2023 from 192.168.73.1
```

## 四、免密与非免密适用场景

- 非免密：小规模环境，免密环境初始化
- 免密：大集群中适用，安全策略要求高、root密码定期更换，自动化运维工具
- 结合使用：利用ssh非免密创建免密环境，借助免密环境，利用其他工具完成部署安装

## 五、跨主机执行命令返回值处理

本地执行命令结果可以使用 $? 查看，远程命令的执行结果依然可以使用 $? ，不过一些指令是使用 $? 获取不到的，需要使用其他的获取。

