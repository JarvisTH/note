容器是镜像的一个运行实例，镜像是静态的只读文件，而容器带有运行时需要的可写文件层，容器中的应用进程处于运行状态。Docker 容器就是独立运行的一个（或一组）应用，以及它们必需的运行环境。

**1.创建容器**
使用docker [container] create命令新建容器。
```
docker create -it ubuntu:latest
docker ps -a
```
使用docker [container] create命令新建容器处于停止状态，可以使用docker [container] start命令启动。![create 命令与容器运行模式相关的选项](https://upload-images.jianshu.io/upload_images/9449419-4bdca809a19a6a5e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![create 命令 容器环境和配置相关的选项](https://upload-images.jianshu.io/upload_images/9449419-2ee6961af766fd3b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![](https://upload-images.jianshu.io/upload_images/9449419-a4e5f8ea34ea2b01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![create 命令与容器资源限制和安全保护相关的选项](https://upload-images.jianshu.io/upload_images/9449419-15a3d8ad983766f2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
其他选项包括：
- -l,--label=[]:以键值对方式指定容器的标签信息
- --label-file=[]:从文件中读取标签信息

**2.启动容器**
使用docker [container] start命令启动一个已经创建的容器。
```
docker start 目标容器
docker ps
```

**3.新建并启动容器**
使用docker [container] run,等价于先create，再start，docker在后台运行标准操作包括：
检查本地是否存在指定的镜像，不存在就从公有仓库下载；
- 利用镜像创建一个容器，并启动该容器；
- 分配 个文件系统给容器，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去；
- 从网桥的地址池配置一个 IP 地址给容器；
- 执行用户指定的应用程序；
- 执行完毕后容器被自动终止
下面的命令启动一个 bash 终端，允许用户进行交互：
>docker run -it ubuntu:18.04 /bin/bash

对于所创建的 bash 容器，当用户使用 exit 命令退出 bash 进程之后，容器也会自动退出。可以使用docker container wait CONTAINER [Container...]子命令等待容器退出，并打印返回结果。

执行 docker [ container] run时候因为命令无法正常执行容器会出错
直接退出， 此时可以查看退出 错误代码：
- 125 : Docker daemon 执行出错，例如指定了不支持的 Docker 命令参数；
- 126 ：所指定命令无法执行，例如权限出错
- 127 容器内命令无法找到

**4.守护态运行**
让容器在后台以守护态形式运行，可以通过添加-d参数来实现。
```
docker run -d ubuntu /bin/sh -c  "while true; do echo hello world; sleep 1; done ”
```
**5.查看容器输出**
通过docker [container] logs命令：
- -details 打印详细信息；
-  -f, -follow ：持续保持输出；
- -since string ：输出从某个时间开始的日志；
- －tail string 输出最近的若干日志；
- －t, -timestamps 显示时间戳信息
- until string 输出某个时间之前的日志。

##二、停止容器
**1.暂停容器**
使用docker [container] pause CONTAINER [CONTAINER...]命令来暂停一个运行的容器。
```
docker run --name test --rm -it ubuntu bash
docker pause test
docker ps
```
处于paused状态的容器，可以使用docker [container] unpause CONTAINER [CONTAINER...]命令恢复运行状态。

**2.终止容器**
使用docker [container] stop终止一个运行中的容器。
```
docker [container] stop [-t | --time [=10]] [CONTAINER...]
```
执行完stop后，执行docker container prune，会自动清除所有处于停止状态的容器。还可以通过docker [continer] kill 直接发送SIGKILL信号强行终止容器。Docker 容器中指定的应用终结时，容器也会自动终止。

docker [container] restart命令将一个运行态容器先终止，然后再重新启动。

##三、进入容器
使用-d参数时，容器启动后会进入后台，无法看到容器信息与操作。

**1.attach命令**
attach是docker自带命令：
```
docker [container] attach [--detach-keys[=[]]] [--no-stdin] [--sig-proxy[=true]] CONTAINER
```
选项：
- --detach-keys [=[]]：指定退出attach模式的快捷键序列，默认CTRL-p / q；
- --no-stdin=true | false:是否关闭标准输入，默认打开；
- --sig-proxy=true|false：是否代理收到的系统信号给应用进程，默认true。

当多个窗口同时 attach 到同一个容器的时候，所有窗口都会同步显示；当某个窗口因命令阻塞时，其他窗口也无法执行操作了。

**2.exec命令**
exec 命令，可以在运行中容器内直接执行任意命令：
```
docker [container] exec [-d|--detach] [--detach-keys[=[]]] [-i|--interactive] [--privileged] [-t|--tty] [-u|--user[=USER]] CONTAINER COMMAND [ARG...]
```
比较重要的参数有：
- －d, --detach 在容器中后台执行命令；
-  --detach-keys ＝＂＂：指定将容器切回后台的按键；
- － e, - - env= ［]：指定环境变量列表
- － i, --interactive=true I false ：打开标准输入接受用户输入命令， 默认值为false; 
- －－ privileged=true|false 是否给执行命令以高权限，默认值为 false;
- － t, --tty=true|false 分配伪终端，默认值为 false;
- － u, --user ＝＂＂：执行命令的用户名或 ID

##四、删除容器
使用docker [container] rm命令删除处于终止或者退出状态的容器：
```
docker [container] rm [-f|--force] [-l|--link] [-v|--volumes] CONTAINER [CONTAINER ...]
```
主要支持的选项包括:
-   -f, --force=false 是否强行终止并删除一个运行中的容器
- － 1, --link=false ：删除容器的连接 ，但保留容器；
- － v, --volumes=false ：删除容器挂载的数据卷

docker rm 命令只能删除已经处于终止或退出状态的容器，并不能删除还处于运行状态的容器,要直接删除一个运行中的容器，可以添加参数。

##五、导入、导出容器
**1.导出容器**
导出容器是指，导出一个已经创建的容器到一个文件，不管此时这个容器是否处于运行状态 可以使用 docker [container] export命令：
```
docker [container] export [-o|--out[=""]] CONTAINER
```
可以通过-o指定导出的tar文件名，也可以成功直接重定向实现。

**2.导入容器**
```
docker [container] import [-c|--change[=[]] [-m|--message[=MESSAGE]] file|URL|-[REPOSITORY[:TAG]]
```
用户通过-C、--change=[]选项在导入的同时执行对容器进行修改的dockerfile指令。
```
docker import test.tar - test/ubuntu:v1.0
```
可以使用 docker load 命令来导入镜像存储文件到本地镜像库，也可以使用docker [container] import命令导入一个容器快照到本地镜像库。这两者的区别在于 容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积更大。从容器快照文件导人时可以重新指定标签等元数据信息。

##六、查看容器
**1.查看容器详情**
查看容器详情可以使用 docker container inspect [OPTIONS] CONTAINER [CONTAINER . .. ］子命令。

**2.查看容器内进程**
查看容器内进程可以使用 docker [container] top [OPTIONS] CONTAINER [CONTAINER . ..] 子命令。

**3.查看统计信息**
查看统计信息可以使用 docker [container] stats [OPTIONS] [CONTAINER ... ] 子命令，会显示 CPU 、内存、存储、网络等使用情况的统计信息。支持的选项：
- － a, -all ：输出所有容器统计信息，默认仅在运行中；
- － format string ：格式化输出信息；
- － no-stream ：不持续输出，默认会自动更新持续实时结果；
- － no-trunc ：不截断输出信息

##七、其他命令
**1.复制文件**
container cp 命令支持在容器和主机之间复制文件 命令格式为docker [container] cp [OPTIONS] CONTAINER:SRC_PATH DEST_PATH|- 。支持的选项包括：
-  -a, -archive ：打包模式，复制文件会带有原始的 uid/gid 信息；
- － L, -follow-link ：跟随软连接。当原路径为软连接时＼默认只复制链接信息，使用该选项会复制链接的目标内容。

**2.查看变更**
container diff 看容器内文件系统的变更 命令格式为 docker [container]  diff CONTAINER。

**3.查看端口映射**
container port 命令可以查看容器的端口映射情况。命令格式为 docker container port CONTAINER [PRIVATE_PORT[/PROTO ］］ 。

**4.更新配置**
container update 命令可以更新容器的一些运行时配置，主要是一些资源限制份额命令格式为 docker [container] update [OPTIONS] CONTAINER [CONTAINER .. .]。
支持的选项包括：
- －blkio-weight uintl6 ：更新块 IO 限制， 10~1000 ，默认值为 0，代表着无限制；
- － cpu-period int ：限制 CPU 调度器 CFS (Completely Fair Scheduler ）使用时间，
单位为微秒，最小 1000;
- － cpu-quota int ：限制 CPU 调度器 CFS 配额，单位为微秒，最小 1000;
- － cpu-rt-period int ：限制 CPU 调度器的实时周期，单位为微秒
- － cpu-rt-runtime int ：限制 CPU 调度器的实时运行时，单位为微秒；
- － c, -cpu-shares in 限制 CPU 使用份额；
- － cpus decimal ：限制 CPU 个数；
- － cpuset-cpus string ：允许使用的 CPU 核，如 0-3, 0,1; 
- － cpuset-mems string ：允许使用的内存块，如 0-3' 0, 1; 
- － kernel-memor bytes ：限制使用的内核内存；
- － m, -memory bytes 限制使用的内存；
- －memory-reservation bytes ：内存软限制；
- －memory-swap bytes ：内存加上缓存区的限制， 表示为对缓冲区无限制；
- － restart string 容器退出后的重启策略

##八、笔记
docker run tomcat  前台运行，要后台运行，加参数 -d。

