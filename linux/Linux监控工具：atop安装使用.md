atop是一个功能强大的linux服务器监控工具，它支持收集和显示CPU,内存,磁盘,网络，进程等资源的相关信息，负载比较大的资源信息会以特别的颜色显示， 可以作为系统管理的辅助工具使用。官方网站为：
[http://www.atoptool.nl/](http://www.atoptool.nl/)
项目官方wiki地址：[http://fedoraproject.org/wiki/EPEL/zh-cn](http://fedoraproject.org/wiki/EPEL/zh-cn)



##一、安装
1.yum安装
安装atop需要先安装第三方源：安装第三方yum源EPEL，EPEL的全称叫 Extra Packages for Enterprise Linux。

```
yum install epel-release

#
yum install -y atop
```

##二、使用介绍
运行atop n，可每隔n秒显示系统资源信息：
```
ATOP - server1                                   2020/09/02  21:25:33                                   --------------                                     5s elapsed
PRC | sys    0.08s  | user   0.02s  | #proc     96  | #trun      1  | #tslpi   155  | #tslpu     0  | #zombie    0  | clones     0  |               | #exit      0  |
CPU | sys       0%  | user      0%  | irq       0%  | idle    200%  | wait      0%  | guest     0%  | ipc notavail  | cycl unknown  | curf    ?MHz  | curscal   ?%  |
cpu | sys       0%  | user      0%  | irq       0%  | idle    100%  | cpu000 w  0%  | guest     0%  | ipc notavail  | cycl unknown  | curf    ?MHz  | curscal   ?%  |
cpu | sys       0%  | user      0%  | irq       0%  | idle    100%  | cpu001 w  0%  | guest     0%  | ipc notavail  | cycl unknown  | curf    ?MHz  | curscal   ?%  |
CPL | avg1    0.00  | avg5    0.04  | avg15   0.05  |               |               | csw      710  | intr     626  |               |               | numcpu     2  |
MEM | tot   974.6M  | free  242.4M  | cache 395.2M  | buff    2.0M  | slab   90.3M  | shmem   7.7M  | shrss   0.0M  | vmbal   0.0M  | hptot   0.0M  | hpuse   0.0M  |
SWP | tot     2.0G  | free    2.0G  |               |               |               |               |               |               | vmcom   1.2G  | vmlim   2.5G  |
DSK |          sda  | busy      0%  | read       0  | write     21  | KiB/r      0  | KiB/w     11  | MBr/s    0.0  | MBw/s    0.0  | avq    11.20  | avio 0.48 ms  |
NET | transport     | tcpi       2  | tcpo       3  | udpi       0  | udpo       0  | tcpao      0  | tcppo      0  | tcprs      0  | tcpie      0  | udpie      0  |
NET | network       | ipi        2  | ipo        3  | ipfrw      0  | deliv      2  |               |               |               | icmpi      0  | icmpo      0  |
NET | ens33     0%  | pcki       2  | pcko       3  | sp 1000 Mbps  | si    0 Kbps  | so    0 Kbps  | erri       0  | erro       0  | drpi       0  | drpo       0  |

   PID    SYSCPU     USRCPU      VGROW      RGROW      RDDSK      WRDSK     RUID         EUID         ST     EXC      THR     S     CPUNR      CPU     CMD        1/1
  2540     0.05s      0.01s         0K         0K         0K         0K     root         root         --       -        1     R         1       1%     atop
   737     0.00s      0.01s         0K         0K         0K         0K     mysql        mysql        --       -       30     S         1       0%     mysqld
   544     0.01s      0.00s         0K         0K         0K         0K     root         root         --       -        2     S         0       0%     vmtoolsd
     3     0.01s      0.00s         0K         0K         0K         0K     root         root         --       -        1     S         0       0%     ksoftirqd/0
   285     0.01s      0.00s         0K         0K         0K         0K     root         root         --       -        1     S         1       0%     xfsaild/sda2
   705     0.00s      0.00s         0K         0K         0K         0K     root         root         --       -        3     S         1       0%     rsyslogd

```
Atop行显示服务器的主机名、当前时间以及信息收集频率。
```
ATOP - server1                                   2020/09/02  21:25:33                                   --------------                                     5s elapsed
```

PRC行显示系统进程相关汇总信息：
- sys：采样周期内所有进程在系统态运行时间总和
- user 采样周期内所有进程在用户态运行时间综合
- proc 采样周期内进程总数
- tslpu 采样周期内处于不可中断的睡眠状态的进程数
- zombie 采样周期内僵死状态进程数
- exit 采样周期内退出的进程数
```
PRC | sys    0.08s  | user   0.02s  | #proc     96  | #trun      1  | #tslpi   155  | #tslpu     0  | #zombie    0  | clones     0  |               | #exit      0  |
```
CPU行显示服务器CPU利用率汇总信息，各个cpu行显示各个cpu核上利用率汇总信息：
- sys 采样周期内CPU处于系统态的利用率
- user 采样周期内CPU处于用户态的利用率
- idle 采样周期内CPU处于空闲状态的比例
```
CPU | sys       0%  | user      0%  | irq       0%  | idle    200%  | wait      0%  | guest     0%  | ipc notavail  | cycl unknown  | curf    ?MHz  | curscal   ?%  |
cpu | sys       0%  | user      0%  | irq       0%  | idle    100%  | cpu000 w  0%  | guest     0%  | ipc notavail  | cycl unknown  | curf    ?MHz  | curscal   ?%  |
cpu | sys       0%  | user      0%  | irq       0%  | idle    100%  | cpu001 w  0%  | guest     0%  | ipc notavail  | cycl unknown  | curf    ?MHz  | curscal   ?%  |
```
CPL行显示CPU负载信息：
- avg1 过去1分钟进程等待队列数
- avg5 过去5分钟进程等待队列数
- avg15 过去15分钟进程等待队列数
- csw(context swapping) 上下文交换次数
- intr(interrupt) 中断发生的次数
- numcpu cpu的核数
```
CPL | avg1    0.00  | avg5    0.04  | avg15   0.05  |               |               | csw      710  | intr     626  |               |               | numcpu     2  |
```
MEM行显示内存使用信息：
- tot 物理内存总量
- free 空闲内存大小，不包含cache和buffer的内存
- cache 用于页缓存的内存大小
- buff 用于文件缓存的内存大小
- slab 系统内核占用的内存大小
```
MEM | tot   974.6M  | free  242.4M  | cache 395.2M  | buff    2.0M  | slab   90.3M  | shmem   7.7M  | shrss   0.0M  | vmbal   0.0M  | hptot   0.0M  | hpuse   0.0M  |
```
SWP行显示交换空间使用情况：
```
SWP | tot     2.0G  | free    2.0G  |               |               |               |               |               |               | vmcom   1.2G  | vmlim   2.5G  |
```
LVM,DSK行显示磁盘逻辑卷和分区使用情况:
- busy 磁盘忙时所占比例
- read KiB/r 、MBr/s 每秒读的请求数和请求的kb、mb数
- write KiB/w 、MBr/w 每秒写的请求数和请求的kb、mb数
- avio 磁盘的平均io时间
```
LVM |  system-root |  busy      1% | read       0 |  write     16 | MBw/s    0.0 |  avio 1.50 ms |
DSK |          sda  | busy      0%  | read       0  | write     21  | KiB/r      0  | KiB/w     11  | MBr/s    0.0  | MBw/s    0.0  | avq    11.20  | avio 0.48 ms  |
```
NET 显示传输层、网络层、各个网络接口的网络传输信息:
- sp 网卡的带宽
- pcki 传入的数据包的大小
- pcko 传出的数据包的大小
- si 每秒传入的数据大小
- so 每秒传出的数据大小
- coll 每秒的冲突数
- mlti 每秒的多路广播的数量
- erri/erro 每秒输入输出的错误数
- drpi/drpo 每秒的输入输出的丢包数
```
NET | transport     | tcpi       2  | tcpo       3  | udpi       0  | udpo       0  | tcpao      0  | tcppo      0  | tcprs      0  | tcpie      0  | udpie      0  |
NET | network       | ipi        2  | ipo        3  | ipfrw      0  | deliv      2  |               |               |               | icmpi      0  | icmpo      0  |
NET | ens33     0%  | pcki       2  | pcko       3  | sp 1000 Mbps  | si    0 Kbps  | so    0 Kbps  | erri       0  | erro       0  | drpi       0  | drpo       0  |
```
最下边显示的各进程的具体信息，可输入m（内存）、p（进程）、u(用户)、d（磁盘）、c（进程运行的代码）、v(线程)切换显示模式，不同模式下的显示信息这里不再展开，可使用 man atop查看atop的手册。
```
PID    SYSCPU     USRCPU      VGROW      RGROW      RDDSK      WRDSK     RUID         EUID         ST     EXC      THR     S     CPUNR      CPU     CMD        1/1
  2540     0.05s      0.01s         0K         0K         0K         0K     root         root         --       -        1     R         1       1%     atop
   737     0.00s      0.01s         0K         0K         0K         0K     mysql        mysql        --       -       30     S         1       0%     mysqld
```
m模式：内存状态模式

- SYSCPU:过去10s内进程处于内核模式占用的CPU时间

- USRCPU:过去10S进程处于用户模式占用的CPU时间

- VSIZE:过去10S进程占用的虚拟空间大小

- RSIZE:过去10S进程占用的内存空间大小

- PSIZE:过去10S进程占用的页大小

- VGROW：过去10S进程增长的虚拟空间大小

- RGROW：过去10S进程增长的内存大小

- SWAPSZ:过去10S进程使用交换空间的大小。

- MEM:过去10S进程占用内存百分比

d模式：磁盘状态模式

- RDDSK:过去10S进程读磁盘的数据量
- WRDSK:过去10S进程写磁盘的数据量
- DSK:过去10S进程所占磁盘的百分比
- CMD:进程名 

p模式：进程状态模式，同一个名称的进程显示一列，根据进程名进行分组显示

- NPROCS:相同名称的进程数量

- 其它的参数上面已经有列出

v模式：线程状态模式

u模式：用户模式

- 根据用户进行分组显示

g模式：标准模式

- s:进程当前的状态，包括：s(sleeping),R(runing)等

##四、相关文件
- /etc/atop：目录保存的是atop的配置文件
- /etc/rc.d/init.d/atop：atop的启动文件
- /etc/cron.d/atop：atop的定时任务文件，默认是每天0点开始
- /var/log/atop：atop日志文件，默认是每天0点开始会产生当天的一个日志文件，然后可以通过atop -r file 查看信息，但是没有找到自动播放的的功能，只能通过输入b显示一个指定的时间的信息，可以写个循环来实现
- /usr/bin/atop：atop命令目录

atop产生的日志文件信息是10分钟一个采样周期进行记录，可以通过修改atop.daily文件进行修改。

对于atop日志文件的保存方式，我们可以这样：

》每天保存一个atop日志文件，该日志文件记录当天信息
》日志文件以”atop_YYYYMMDD”的方式命名
》设定日志失效期限，自动删除一段时间前的日志文件

在atop.daily脚本中，我们可以通过修改INTERVAL变量改变atop信息采样周期(默认为10分钟)；通过修改以下命令中的数值改变日志保存天数(默认为28天)：
>(sleep 3; find 'atop_*' -mtime +28 -exec rm {} \; )&

最后，我们修改cron文件，每天凌晨执行atop.daily脚本：
>0 0 * * * root /etc/cron.daily/atop.daily
