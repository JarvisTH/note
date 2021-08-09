命令：nl

显示行号

```shell
nl [-bnw] file
```

选项与参数：

- -b ：指定行号指定的方式，主要有两种：
  -b a ：表示不论是否为空行，也同样列出行号(类似 cat -n)；
  -b t ：如果有空行，空的那一行不要列出行号(默认值)；
- -n ：列出行号表示的方法，主要有三种：
  -n ln ：行号在荧幕的最左方显示；
  -n rn ：行号在自己栏位的最右方显示，且不加 0 ；
  -n rz ：行号在自己栏位的最右方显示，且加 0 ；
- -w ：行号栏位的占用的位数。

```shell
[root@centos-7-jarvis test]# nl /etc/issue -n rz
000001  \S
000002  Kernel \r on an \m

[root@centos-7-jarvis test]# nl /etc/issue -n rn
     1  \S
     2  Kernel \r on an \m

[root@centos-7-jarvis test]# nl /etc/issue -n ln
1       \S
2       Kernel \r on an \m
[root@centos-7-jarvis test]# nl /etc/issue -n rz -w  5
00001   \S
00002   Kernel \r on an \m

[root@centos-7-jarvis test]# nl /etc/issue -n rz -w  6
000001  \S
000002  Kernel \r on an \m
```

