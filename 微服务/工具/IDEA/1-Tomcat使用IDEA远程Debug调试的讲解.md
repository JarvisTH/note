JAVA 支持调试功能，本身提供了一个简单的调试工具JDB，支持设置断点及线程级的调试同时，不同的JVM通过接口的协议联系，本地的Java文件在远程JVM建立联系和通信。

**一、远程Tomcat设置**

1、在tomcat/bin下的catalina.sh上边添加下边的一段设置
```
CATALINA_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,address=60222,suspend=n,server=y"
```
![](https://upload-images.jianshu.io/upload_images/9449419-03d0abf1b0369bf4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2、address=60222 这个是后边IDEA设置的时候需要用到的调试端口，可以任意设置一个未使用的端口，但是后边的配置都要一致.

**二.IDEA设置**

1、添加Tomcat Server选择Remote
![](https://upload-images.jianshu.io/upload_images/9449419-db66e671384e3859.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2、设置相应的参数
![](https://upload-images.jianshu.io/upload_images/9449419-3a202aabfde3ba78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
*   xxx.xxx.152.67:8080为远程Tomcat服务器的IP地址和端口，这里可以设置域名，例如：[http://security.xxxx.cn/login.do](http://security.xxxx.cn/login.do)；
*   60222这个端口为1.1步中设置的debug端口，适合tomcat的端口不一样的；
*   这里的Remote staging选择的都是same file system，这就要求本地代码和远程Tomcat的代码要一直；
3、Startup/Connection

![](https://upload-images.jianshu.io/upload_images/9449419-c8e489371f55cb0d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
选择Debug、Socket、调试端口这里是60222

4、选择运行

![](https://upload-images.jianshu.io/upload_images/9449419-b029b8ace943c5d7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


然后点击右边的debug即可运行（首先Tomcat要启动）

5、是否成功

![](https://upload-images.jianshu.io/upload_images/9449419-4e8c42f0cd1fb2ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


可以看到已经连接成功，

查看Tomcat服务器日志，如下：

![](https://upload-images.jianshu.io/upload_images/9449419-98a654e41b93a142.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


6、如果失败

如果出现端口被拒绝：

![](https://upload-images.jianshu.io/upload_images/9449419-458441b83eebbd00.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


这种可能是tomcat并未启动，或者遇错误停止，重启Tomcat即可

连接失败，其他问题

首先在服务器端查看 调试端口 这里是60222的使用情况

```
[root@VM_92_170_centos bin]# lsof -i:60222
COMMAND  PID USER  FD  TYPE DEVICE SIZE/OFF NODE NAME
java  12064 root  5u IPv4 1320209   0t0 TCP 1x.xxx.xxx.170:60222->1xx.xx.xxx.231:13325 (ESTABLISHED)
```

可以看到这是自己本地和远程的一个连接，如果连接失败却看到上边的使用情况，请使用kill -9 PID杀死进程，重启Tomcat，然后在IDEA中重新运行debug，本例为：kill -9 12064

**容易出现的问题**

*   如果远程没有连接上，两个端口被占用或者防火墙屏蔽。除了JMX server指定的监听端口号外，JMXserver还会监听一到两个随机端口号，这个如果防火墙关闭了的话就不用考虑，如果使用了防火墙，还需要查看它监听的端口。
*   账号的相应的读写权限一定要有；

**三、为什么可以进行远程调试，背后的原理是什么**
 首先，了解下的Java程序的执行过程- 分为以下几个步骤：Java的文件 - - 编译生成的类文件（class文件） - - JVM加载类文件 - - JVM运行类字节码文件 - - JVM翻译器翻译成各个机器认识的不同的机器码。

远程调试原理

众所周知，Java 程序是运行在Java 虚拟机（JVM ）上的，具有良好跨平台性，是因为Java程序统一以字节码的形式在JVM中运行，不同平台的虚拟机都统一使用这种相同的程序存储格式。因为都是类字节码文件，只要本地代码和远程服务器上的类文件相同，两个JVM通过调试协议进行通信（例如通过插座在同一个端口进行通信），另外需要注意的时，被调试的服务器需要开启调试模式，服务器端的代码和本地代码必须保持一致，则会造成断点无法进入的问题。

![Java的调试器架构](https://upload-images.jianshu.io/upload_images/9449419-ab09e4a6b1d2918e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

 这个架构其实质还是JVM，只要确保本地的Java的源代码与目标应用程序一致，本地的Java的的的的源码就可以用插座连接到远端的JVM，进而执行调试。因此，在这种插座连接模式（下文介绍）下，本地只需要有源码，本地的Java的应用程序根本不用启动。

传输方式，默认为Socket ;

套接字：MACOS，Linux的系统使用此种传输方式;

 

共享内存：WINDOWS系统使用此种传输方式。

调试模式，默认为Attach ;

 

Attach ：此种模式下，调试服务端（被调试远程运行的机器）启动一个端口等待我们（调试客户端）去连接;

Socket ：此种模式下，是我们（调试客户端）去监听一个端口，当调试服务端准备好了，就会进行连接。

## 配置属性说明补充

### 1.idea的的服务的开启调试模式设置详细说明，
![](https://upload-images.jianshu.io/upload_images/9449419-06037a1a0370a508.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
文本：
```
CATALINA_OPTS="-Xdebug  -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=8089"
```
2.各参数解释：
-Xdebug：通知JVM工作在调试模式下
-Xrunjdwp：通知JVM使用（java debug wire protocol）来运行调试环境。参数同时有一系列的调试选项：
<code>session</code>：指定了调试数据的传送方式，dt_socket是指用SOCKET模式，另外dt_shmem指用共享内存方式，其中dt_shmem只适用于窗口平台.server  参数是指是否支持在服务器模式的虚拟机中。
onthrow：指明当产生该类型的异常时，JVM就会中断下来，进行调式该参数任选。
<code>release</code>：指明当JVM被中断下来时，执行的可执行程序该参数可选
<code>suspend</code><：指明：是否在调试客户端建立起来后，再执行 JVM。
onuncaught（= y或n）指明出现未捕获的异常后，是否中断JVM的执行。
3.IDEA设置远程属性说明，以下为谷歌翻译

![](https://upload-images.jianshu.io/upload_images/9449419-873678231e740b1f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

参考：[https://blog.csdn.net/qq_37192800/article/details/80761643](https://blog.csdn.net/qq_37192800/article/details/80761643)
[https://www.jb51.net/article/158520.htm](https://www.jb51.net/article/158520.htm)

