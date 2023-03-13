##一、介绍
Cron是Linux系统中最有用的工具之一，cron作业是在指定时间到来时被调度执行的作业。Cron本身是一个守护进程，在后台运行，通过配置文件“crontab”来根据时间调度指定的作业执行。


**1.1 cron，crontab以及anacron的关系**
- cron是大多数linux发行版都自带的守护进程（daemon）
- crontab(cron table的简称)既可以指cron用来定期执行特定任务所需要的列表文件，又可以指用来创建、删除、查看当前用户（或者指定用户）的crontab文件的命令。
- anacron不是守护进程，可以看做是cron守护进程的某种补充程序，anacron是独立的linux程序，被cron守护进程或者其他开机脚本启动运行，可以每天、每周、每个月周期性地执行一项任务（最小单位为天）。适合于可能经常会关机的机器，当机器重新开机anacron程序启动之后，anacron会检查anacron任务是否在合适的周期执行了，如果未执行则在anacron设定好的延迟时间之后只执行一次任务，而不管任务错过了几次周期。

**1.2 crontab配置文件**
- 系统默认crontab文件为/etc/crontab,以及/etc/cron.d/目录下的文件，有些程序会把自己的crontab文件放在/etc/cron.d/目录下。cron守护进程会检查/etc/crontab以及/etc/cron.d/目录下的文件，根据这些文件中的cron任务所设置的执行时间决定是否执行任务，如果当前时间与cron任务所设置的执行时间相同，则执行任务。
- 每个用户自己的crontab文件都会被放在 /var/spool/cron目录下，默认为空，可以使用crontab命令创建。cron守护进程会检查/var/spool/cron目录下的文件，根据这些文件中的cron任务所设置的执行时间决定是否执行任务，如果当前时间与cron任务所设置的执行时间相同，则执行任务。

**1.3 注意事项**
cron执行的任务会在设定好的时刻执行，当机器处于关机状态下并错过了任务执行的时间，cron任务就无法预期执行了。

##二、Cron配置类型
**2.1 系统级Crontab**
这些cron作业被系统服务和关键作业所使用，且需要root级的权限才能执行。可以在/etc/crontab文件中查看系统级的cron作业。

**2.2 用户级Crontab**
用户级的cron作业是针对每个用户单独分开的。因此每个用户都可以使用crontab命令创建自己的cron作业，还可以使用以下命令编辑或查看自己的cron作业。

![](https://upload-images.jianshu.io/upload_images/9449419-f0c225d327d66496?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##三、 操作**
- 查看状态
```
service cron status
```
- 开启服务
```
service cron start
```
 - help

- 创建并编辑当前用户的crontab
```
crontab -e
```

- 3.4 列出当前用户的crontab
```
crontab -l
```

- crontab文件语法及示例
```
SHELL=/bin/bash
MAILTO=root@example.com
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# | .------------- hour (0 - 23)
# | | .---------- day of month (1 - 31)
# | | | .------- month (1 - 12) OR jan,feb,mar,apr ...
# | | | | .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# | | | | |
# * * * * * user-name command to be executed
```
该文件的前三行代码设置了默认环境。cron守护进程并不提供任何环境。SHELL变量设置当cron任务(命令以及脚本)运行时的shell,MAILTO变量设置cron任务执行结果发送的邮箱，PATH设置去哪些目录下寻找cron任务的命令。注释部分则解释一条cron任务的构成，一条cron任务就是一行，要设置多少条cron任务则写多少行。一条cron任务由七个部分组成。

- 如果你想匹配取值范围内的所有值，使用“*”
- 想匹配某些特殊的值，使用“,”，比如2,4,7就匹配的是2，4以及7。
- 两个值被“-”连接表示范围，此时匹配的是范围内所有值，包含“-”两边的值，比如4-7匹配的就是从4到7。
- 想要表达每隔一段时间执行一次任务，使用 “/”， 比如分钟部分中的 “*/10”表示每10分钟运行一次，比如小时部分中的“10-22/2”则表示在早上10点到晚上10点这段时间内，每隔两个小时运行一次。 注意 ：当“/”左边的值可以除尽“/”右边的值时，任务才会运行。

**3.8 cron.hourly、daily、weekly、monthly**

##示例
**4.1 在指定时间调度Cron job作业**
```
#! /bin/sh
echo hello >> /home/ubuntu/workspace/hello.txt
```

- crontab -e
```
*/1 * * * * /home/ubuntu/crontest.sh
```

**4.2 删除log**
**4.3 清除cache**
脚本中加入
echo 1 > /proc/sys/vm/drop_caches
需要系统级别权限

**4.4 备用**
```
#！ /bin/sh

# 注释

cd ~/workspace.log
echo "" > trace.log
echo 1 > /proc/sys/vm/drop_caches
```
copy 到 /etc/cron.hourly/下
注：由于/ etc / crontab文件使用run-parts,因此filename非常严格，不能有点，脚本中不能有~

##五、不执行原因**
- cron服务未启动
crontab不是Linux内核的功能，而是依赖一个crond服务，这个服务可以启动当然也可以停止。如果停止了就无法执行任何定时任务了，解决的方法是打开它：
crond 或 service crond start 。
如果提示crond命令不存在，可能被误删除了，CentOS下可以通过这个命令重新安装：
```
yum -y install crontabs
```

- 路径问题
有的命令在shell中执行正常，但是在crontab执行却总是失败。有可能是因为crontab使用的sh未正确识别路径，比如：以root身份登录shell后执行一个/root/test.sh，只要执行./test.sh

- 时差问题
因为服务器与客户端时差问题，所以crontab的时间以服务器时间为准。

- 变量问题
有时候命令中含有变量，但crontab执行时却没有，也会造成执行失败。

- 权限问题
解决方法：
增加执行权限，或者用bash abc.sh的方法执行.也有可能crontab任务所属的用户对某个目录没有写权限，也会失败。

...

##六、crontab日志讲解
crontab的日志比较简单，当crond执行任务失败时会给用户发一封邮件。

本文介绍crontab在任务执行失败时，如果发送邮件也失败，应该怎样通过增加crontab日志的方式记录错误原因。

默认情况下,crontab中执行的日志写在/var/log下,如：
```
[root@centos-7-jarvis cron]# ls /var/log/cron*
/var/log/cron           /var/log/cron-20200531  /var/log/cron-20200823
/var/log/cron-20200524  /var/log/cron-20200822
```

如果日志有问题,可以参考以下做法：

**为crontab增加日志**

参考：https://blog.csdn.net/qq_38880380/article/details/99625503
