##一、获取镜像
镜像是运行容器的前提。可以使用 docker [image] pull 命令直接从 Docker Hub 镜像源来下载镜像。 该命令的格式为 docker [image] pull NAME [ :TAG] 。其中， NAME 是镜像仓库名称（用来区分镜像）， TAG 是镜像的标签（往往用来表示版本信息） 。 通常情况下， 描述一个镜像需要包括 “名称＋标签“ 信息。如果不显式指定TAG, 则默认会选择la迳釭标签，这会下载仓库中最新版本的镜像。
>docker pull ubuntu: 18. 04

镜像的仓库名称中还应该添加仓库地址（即registry, 注册服务器）作为前缀 ，只是默认使用的是官方DockerHub服务 ，该前缀可以忽略.如果从非官方 的仓库下载，则 需要在仓库 名称前指定完整的仓库地址。可以在Docker服务
启动配置中增加 --regis七ry-mirror=proxy_URL来指定镜像代理服务地址。
```
pull 子命令支持的 选项主要包括：
D -a, --all士ags=trueifalse: 是否获取仓库中的所有镜像，默认为否；
D --disable-con七ent-trus七：取消镜像的内容校验，默认为真。
```

##二、查看镜像信息
**1.使用images列出镜像**
使用docker images或docker image ls 命令可以列出本地主机上已有镜像的基本信息。在列出信息中， 可以看到几个字段信息： 
- 来自于哪个仓库， 比如 ubuntu 表示ubuntu 系列的基础镜像；
-  镜像的标签信息， 比如 18.04 、 latest 表示不同的版本信息。 标签只是标记， 并不能标识镜像内容；
- 镜像的ID(唯一标识镜像）， 如果两个镜像的ID相同， 说明它们实际上指向了同一个镜像， 只是具有不同标签名称而已；
- 创建时间， 说明镜像最后的更新时间；
- 镜像大小，镜像大小信息只是表示了该镜像的逻辑体积大小， 实际上由于相同的镜像层本地只会存储一份， 物理上占用的存储空间会小于各镜像逻辑体积之和。
```
images子命令主要支持如下选项， 用户可以自行进行尝试：
 -a, --all 式rue I false: 列出所有（包括临时文件）镜像文件，默认为否；
 --diges七S=trueifalse: 列出镜像的数字摘要值，默认为否；
 -f, --fi让er=[] : 过滤列出的镜像， 如dangling 式rue 只显示没有被使用的
镜像；也可指定带有特定标注的镜像等；
--forma七="TEMPLATE" : 控制输出格式，如. ID代表ID信息，.Repository
代表仓库信息等；
 --no-七rune吐rue I false: 对输出结果中太长的部分是否进行截断，如镜像的ID
信息，默认为是；
 -q, --quiet式rue I false: 仅输出ID信息， 默认为否。
其中， 还支持对输出结果进行控制的选项，如 -f. --fil七er=[)、--no七rune=true I false、 -q、 --quiet=true I false等.
```
更多子命令选项还可以通过mandocker-images来查看。

**2.tag命令添加镜像标签**
使用docker tag命令来为本地镜像任意添加新的标签。
>docker tag ubuntu:latest myubuntu:latest

用户就可以直接使用myubuntu:la七es七来表示这个镜像了。

**3.inspect命令查看详细信息**
docker巨mage]inspect命令可以获取该镜像的详细信息，包括制作者 、 适应架构、各层的数字摘要等：
> docker [image] inspect ubuntu:18.04
如果我们只要其中一项内容时， 可以使用 -f 来指定， 例如， 获取镜像的 Architecture:
>docker [image] inspect -f { {" . Architecture"}} ubuntu:18.04

**4.history查看镜像历史**
>docker history ubuntu:18.04
过长的命令被自动截断了， 可以使用前面提到的 --no-trunc 选项来输出完整命令。

##三、搜寻镜像
使用 docker search 命令可以搜索
Docker Hub 官方仓库中的镜像。语法为 docker search [option] keyword。 支持的命令选项主要包括：
- -f, --filter filter: 过滤输出内容；
-  --format string: 格式化输出内容；
-  --limit int：限制输出结果个数， 默认为 25 个；
-  --no-trunc: 不截断输出结果。

##四、删除、清理镜像
**1.使用标签删除镜像**
使用 docker rmi 或 docker image rm 命令可以删除镜像， 命令格式为 docker rmi IMAGE [IMAGE ... ], 其中 IMAGE可以为标签或 ID。支持选项包括：
-  -f, -force: 强制删除镜像， 即使有容器依赖它；
-  -no-prune: 不要清理未带标签的父镜像。

同一个镜像拥有多个标签的时候，docker rmi 命令只是删除了该镜像多个标签中的指定标签而巳， 并不影响镜像文件。

**2.使用镜像id删除镜像**
当使用 docker rmi 命令， 并且后面跟上镜像的 ID (也可以是能进行区分的部分 ID 串前缀）时， 会先尝试删除所有指向该镜像的标签， 然后删除该镜像文件本身。**当有该镜像创建的容器存在时， 镜像文件默认是无法被删除的。如果要想强行删除镜像**。 可以使用-f参数，正确的做法是，先删除依赖该镜像的所有容器， 再来删除镜像。

使用docker ps -a命令可以看到本机上存在的所有容器。

**3.清理镜像**
docker image prune命令来进行清理。支待选项包括：
-  -a, -all: 删除所有无用镜像， 不光是临时镜像；
-  -filter filter: 只清理符合给定过滤器的镜像；
-  -f, -force: 强制删除镜像， 而不进行提示确认。

##五、创建镜像
创建镜像的方法主要有三种： 基于已有镜像的容器创建、 基千本地模板导入、 基于Dockerfile 创建。

**1.基于已有容器创建**
主要是使用 docker [container] commit命令。命令格式为 docker [container] commit [OPTIONS] CONTAINER [REPOSITORY 
[:TAG]], 主要选项包括：
- -a, --author="": 作者信息
- -c, - -change=[] : 提交的时候执行Dockerfile指令， 包括 CMD I ENTRYPOINT | ENV | EXPOSE I LABEL I ONBUILD I USER I VOLUME I WORKDIR等；
-  -m, - -message= 11 11: 提交消息；
-  -p, --pause=true: 提交时暂停容器运行。

**2.基于本地模板导入**
docker [image] import [option] file [url]-[repository[:tag]]，要直接导人一个镜像，可以使用 OpenVZ 提供的模板来创建，或者用其他已导出的镜像模板来创建。OPENVZ 模板的下载地址为 http: //openvz.org Download/templat/precreated。

**3.基于dockerfile创建**
基于 ocke file 创建是最常见的方式 Dockerfi 个文本文件，利用给定的指述基于某个父镜像创建新镜像的过程。

##六、存出和载入镜像
**1.存储镜像**
如果要导出镜像到本地文件，可以使用 docker [image] save ,支持输出参数，导出镜像到指定文件夹。
```
docker images
docker save -o ubuntu_18.04.tar ubuntu:18.04
```

**2.载入镜像**
可以使用 docker [image] load 将导出的 tar 文件再导人到本地镜像库。支持 -i、-input string选项，从指定文件中读人镜像内容。
```
docker load -i ubuntu_18.04.tar
```

##七、上传镜像
可以使用 docker [image] push 命令上传镜像到仓库，默认上传到 Docker Hub 官方仓库（需要登录）。命令格式为 docker [image]  push NAME[:TAG] I [REGISTRY_HOST [ :REGISTRY_PORT] / ]NAME [:TAG]。用户 user 上传本地的 test :latest 镜像，可以先添加新的标签 user/test:latest 然后 docker [image ] push 令上传镜像：
```
docker tag test:latest user/test:latest
docker push user/test:latest
```
第一次上传时，会提示输入登录信息或进行注册，之后登录信息会记录到本地~/.docker目录下。第一处push会报错requested access to the resource is denied，解决方法如下：https://blog.csdn.net/benben_2015/article/details/83445696

![](https://upload-images.jianshu.io/upload_images/9449419-f6d4c5af8b6ca320.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


