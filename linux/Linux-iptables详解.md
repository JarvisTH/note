##一、iptables简介
iptables是Linux中对网络数据包进行处理的一个功能组件，就相当于防火墙，可以对经过的数据包进行处理，例如：数据包过滤、数据包转发等等，一般例如Ubuntu等Linux系统是默认自带启动的。

##二、iptables结构
 iptables其实是一堆规则，防火墙根据iptables里的规则，对收到的网络数据包进行处理。iptables里的数据组织结构分为：表、链、规则。

- 表（tables）
表提供特定的功能，iptables里面有4个表： filter表、nat表、mangle表和raw表，分别用于实现包过滤、网络地址转换、包重构和数据追踪处理。每个表里包含多个链。
- 链（chains）
链（chains）是数据包传播的路径，每一条链其实就是众多规则中的一个检查清单，每一条链中可以有一 条或数条规则。当一个数据包到达一个链时，iptables就会从链中第一条规则开始检查，看该数据包是否满足规则所定义的条件。如果满足，系统就会根据 该条规则所定义的方法处理该数据包；否则iptables将继续检查下一条规则，如果该数据包不符合链中任一条规则，iptables就会根据该链预先定 义的默认策略来处理数据包。![](https://upload-images.jianshu.io/upload_images/9449419-1bd58a1a68ea601c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其整体结构如下图所示:
![](https://upload-images.jianshu.io/upload_images/9449419-02599d93a1cc16da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 表链结构
1.filter表——三个链：INPUT、FORWARD、OUTPUT
作用：过滤数据包 内核模块：iptables_filter。
2.Nat表——三个链：PREROUTING、POSTROUTING、OUTPUT
作用：用于网络地址转换（IP、端口） 内核模块：iptable_nat。
3.Mangle表——五个链：PREROUTING、POSTROUTING、INPUT、OUTPUT、FORWARD
作用：修改数据包的服务类型、TTL、并且可以配置路由实现QOS内核模块。
4.Raw表——两个链：OUTPUT、PREROUTING
作用：决定数据包是否被状态跟踪机制处理。
- 数据包流向
下面一张图介绍了一个数据包如何依次穿过iptables的各个链和表的。![](https://upload-images.jianshu.io/upload_images/9449419-04410122246742a6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
基本步骤如下：

- 数据包到达网络接口，比如 ens33。
- 进入 raw 表的 PREROUTING 链，这个链的作用是赶在连接跟踪之前处理数据包。
- 如果进行了连接跟踪，在此处理。
- 进入 mangle 表的 PREROUTING 链，在此可以修改数据包，比如 TOS 等。
- 进入 nat 表的 PREROUTING 链，可以在此做DNAT，但不要做过滤。
- 决定路由，看是交给本地主机还是转发给其它主机。

后面将分为2种情况：

- 数据包发送给本地主机，交由本地主机上层应用处理；
- 将数据包转发给其他主机来处理

第一种情况，数据包要转发给其它主机：
7. 进入 mangle 表的 FORWARD 链，这里也比较特殊，这是在第一次路由决定之后，在进行最后的路由决定之前，我们仍然可以对数据包进行某些修改。
8. 进入 filter 表的 FORWARD 链，在这里我们可以对所有转发的数据包进行过滤。需要注意的是：经过这里的数据包是转发的，方向是双向的。
9. 进入 mangle 表的 POSTROUTING 链，到这里已经做完了所有的路由决定，但数据包仍然在本地主机，我们还可以进行某些修改。
10. 进入 nat 表的 POSTROUTING 链，在这里一般都是用来做SNAT ，不要在这里进行过滤。
11. 进入出去的网络接口。完毕。

第二种情况，数据包就是发给本地主机的，那么它会依次穿过：
7. 进入 mangle 表的 INPUT 链，这里是在路由之后，交由本地主机之前，我们也可以进行一些相应的修改。
8. 进入 filter 表的 INPUT 链，在这里我们可以对流入的所有数据包进行过滤，无论它来自哪个网络接口。
9. 交给本地主机的应用程序进行处理。
10. 处理完毕后进行路由决定，看该往那里发出。
11. 进入 raw 表的 OUTPUT 链，这里是在连接跟踪处理本地的数据包之前。
12. 连接跟踪对本地的数据包进行处理。
13. 进入 mangle 表的 OUTPUT 链，在这里我们可以修改数据包，但不要做过滤。
14. 进入 nat 表的 OUTPUT 链，可以对防火墙自己发出的数据做 NAT 。
15. 再次进行路由决定。
16. 进入 filter 表的 OUTPUT 链，可以对本地出去的数据包进行过滤。
17. 进入 mangle 表的 POSTROUTING 链，同上一种情况的第9步。注意，这里不光对经过防火墙的数据包进行处理，还对防火墙自己产生的数据包进行处理。
18. 进入 nat 表的 POSTROUTING 链，同上一种情况的第10步。
19. 进入出去的网络接口。完毕

##三、iptables操作
如果使用开机转发，需要设置参数：echo 1 > /proc/sys/net/ipv4/ip_forward

如果想开机自动设置，配置一下文件：
/etc/sysctl.conf文件中的 net.ipv4.ip_forward = 1

- iptables的基本语法格式
iptables [-t 表名] 命令选项 ［链名］ ［条件匹配］ ［-j 目标动作或跳转
说明：表名、链名用于指定 iptables命令所操作的表和链，命令选项用于指定管理iptables规则的方式（比如：插入、增加、删除、查看等；条件匹配用于指定对符合什么样 条件的数据包进行处理；目标动作或跳转用于指定数据包的处理方式（比如允许通过、拒绝、丢弃、跳转（Jump）给其它链处理。
- iptables命令的管理控制选项
-A 在指定链的末尾添加（append）一条新的规则
-D 删除（delete）指定链中的某一条规则，可以按规则序号和内容删除
-I 在指定链中插入（insert）一条新的规则，默认在第一行添加
-R 修改、替换（replace）指定链中的某一条规则，可以按规则序号和内容替换
-L 列出（list）指定链中所有的规则进行查看
-E 重命名用户定义的链，不改变链本身
-F 清空（flush）
-N 新建（new-chain）一条用户自己定义的规则链
-X 删除指定表中用户自定义的规则链（delete-chain）
-P 设置指定链的默认策略（policy）
-Z 将所有表的所有链的字节和数据包计数器清零
-n 使用数字形式（numeric）显示输出结果
-v 查看规则表详细信息（verbose）的信息
-V 查看版本(version)
-h 获取帮助（help)
- 防火墙处理数据包的四种方式
1.ACCEPT 允许数据包通过
2.DROP 直接丢弃数据包，不给任何回应信息
3.REJECT 拒绝数据包通过，必要时会给数据发送端一个响应的信息。
4.LOG 用于针对特定的数据包打log，在/var/log/messages文件中记录日志信息，然后将数据包传递给下一条规则
5.TRACE这个只能针对raw中的table，是用来对数据进行追踪的，用于debug.

- iptables防火墙规则的保存与恢复
iptables-save把规则保存到文件中，再由目录rc.d下的脚本（/etc/rc.d/init.d/iptables）自动装载
使用命令iptables-save来保存规则。一般用
>iptables-save > /etc/sysconfig/iptables

生成保存规则的文件 /etc/sysconfig/iptables，也可以用:
>service iptables save

- iptables防火墙常用的策略
>1.拒绝进入防火墙的所有ICMP协议数据包
iptables -I INPUT -p icmp -j REJECT
>2.允许防火墙转发除ICMP协议以外的所有数据包
iptables -A FORWARD -p ! icmp -j ACCEPT
>说明：使用“！”可以将条件取反。
>3.允许本机开放从TCP端口20-1024提供的应用服务。
>iptables -A INPUT -p tcp --dport 20:1024 -j ACCEPT 
iptables -A OUTPUT -p tcp --sport 20:1024 -j ACCEPT
>4.从一台主机转发到另一台主机
注意转发首先需要按照前面的开启转发设置
>(相同端口)
从192.168.0.132:21521(新端口)访问192.168.0.211:1521端口
>iptables -t nat -I PREROUTING -p tcp --dport 1521 -j DNAT --to 192.168.0.211
iptables -t nat -I POSTROUTING -p tcp --dport 1521 -j MASQUERADE
（不同端口）
不同端口转发(192.168.0.132上开通21521端口访问
iptables -t nat -A PREROUTING -p tcp -m tcp --dport 21521 -j DNAT --to-destination 192.168.0.211:1521
iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -d 192.168.0.211 -p tcp -m tcp --dport 1521 -j SNAT --to-source 192.168.0.132
以上两条等价配置(更简单[指定网卡]):
iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 31521 -j DNAT --to 192.168.0.211:1521
iptables -t nat -A POSTROUTING -j MASQUERADE
(用iptables做本机端口转发)
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 8080

##四、iptables的debug
针对iptables的debug主要是通过看Log。
1.打开iptables的log日志
首先我们需要打开iptables的log记录功能，让系统将相应的log输出。
默认iptables的日志是输出到/var/log/message文件中，当然对没有开启log功能的时候，不会有任何log输出。开启log输出针对不同系统会不同，主要是以下2种：
- 针对用syslog的系统
如果系统存在以下文件： /etc/syslog.conf， 我们在/etc/syslog.conf中添加以下几行：
```
kern.warning /var/log/iptables.log
kern.debug /var/log/iptables.log
```
然后重启syslog:
```
sudo service syslog restart
```
- 针对用rsyslog的系统

如果系统存在以下文件： /etc/rsyslog.conf， 我们在/etc/syslog.conf中添加以下几行：
```
kern.warning /var/log/iptables.log
kern.debug /var/log/iptables.log
```
然后重启syslog:
```
sudo service rsyslog restart
```

2.针对需要查看的数据，添加iptables规则使其打印log
开启log输出后，我们需要添加相应的iptables规则，使其打印我们需要的log信息，主要有2种方法：
- TRACE方法:在前面的数据包流向图中，raw表中的链是用来对数据包进行追踪的，我们可以在其链中添加TRACE操作规则，使其对相应的流量进行追踪，例如下面的例子将追踪进入的目标地址为192.168.0.211的数据包。
```
sudo iptables -t raw -I PREROUTING -d 192.168.0.211 -j TRACE
```
- LOG方法
如果想在指定的一个表的一个链中加入一条规则，当在这个链中遇到特定的数据包，则记录一条log信息。例如下面的指令将使得在nat的PREROUTING链中遇到目标地址为100.69.73.114时，则记录一条log。
```
sudo iptables -t nat -I PREROUTING  -d 100.69.73.114 -j LOG --log-level 4
```

3.查看log信息
sudo tail -f /var/log/iptables.log
