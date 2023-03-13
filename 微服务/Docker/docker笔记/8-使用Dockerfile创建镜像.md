Dockerfile 是一个文本格式的配置文件， 用户可以使用 Dockerfile 来快速创建自定义的镜像。

##一、基本结构
Dockerfile 主体内容分为四部分：基础镜像信息、 维护者信息、 镜像操作指令和容器启动时执行指令。
```
# escape=\ (backslash) 
# This dockerfile uses the ubuntu:xeniel image 
# VERSION 2 - EDITION 1 
# Author: docker_user 
# Command format: Instruction [arguments / command] 
# Base image to use, this must be set as the first line 
FROM ubuntu:xeniel 
＃
Maintainer docker_user <docker user at email.com> （ @docker_user ） 
LABEL maintainer docker user<docker_user@email.com> 
# Commands to update the image 
RUN echo "deb http://archive.ubuntu.com/ubuntu/ xeniel main universe"  >> /etc apt/sources.list

RUN apt-get update && apt-get install -y nginx 
RUN echo "\ndaernon off;" >> /etc/nginx/nginx.conf 
# Commands when creating a new container 
CMD /usr/sbin/nginx
```
首行可以通过注释来指定解析器命令， 后续通过注释说明镜像的相关信息。 主体部分首先使用FROM指令指明所基于的镜像名称， 接下来一般是使用LABEL指令说明维护者信息。后面则是镜像操作指令， 例如RUN指令将对镜像执行跟随的命令。 每运行一条RUN指令，镜像添加新的一层， 并提交。 最后是CMD指令， 来指定运行容器时的操作命令。

##二、指令说明
Dockerfile 中指令的一般格式为 INSTRUCTION arguments, 包括 “配置指令" (配置镜像信息）和 “操作指令" (具体执行操作）：
![](https://upload-images.jianshu.io/upload_images/9449419-4bf96151e5b9dd2f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![](https://upload-images.jianshu.io/upload_images/9449419-bec62cb6554c1781.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**1.配置指令**
- ARG：定义创建镜像过程中使用的变量，格式为 ARG <name>[=<default value>]。在执行 docker build 时， 可以通过 -build-arg[=l 来为变量赋值。 当镜像编译成
功后， ARG 指定的变量将不再存在 (ENV 指定的变量将在镜像中保留）。Docker 内置了一些镜像创建变量， 用户可以直接使用而无须声明， 包括（不区分大小写） HTTP PROXY 、 HTTPS PROXY 、 FTP PROXY 、 NO PROXY。
- FROM:指定所创建镜像的基础镜像,格式为FROM <image> [AS <name>] 或 FROM <image>: <tag> [AS <name>] 
或FROM <image>@<digest> [AS <name>] 。任何 Dockerfile 中第一条指令必须为FROM 指令。 并且， 如果在同一个Dockerfile 中创
建多个镜像时， 可以使用多个 FROM 指令（每个镜像一次）。
```
ARG VERSION=9.3 
FROM debian:${VERSION}
```
- LABEL:可以为生成的镜像添加元数据标签信息。 这些信息可以用来辅助过滤出特定镜像。格式为 LABEL <key>=<value> <key>=<value> <key>=<value> ...。
```
LABEL version="l.0.0-rc3" 
LABEL author="yeasy@github" date="2020-01-01" 
LABEL description="This 七ex七 illustra七es\
tha七 label-values can span mul七iple lines."
```
- EXPOSE：声明镜像内服务监听的端口。格式为 EXPOSE <part> [<port/<protocol>... ]。该指令只是起到声明作用， 并不会自动完成端口映射。
- ENV：指定环境变量， 在镜像生成过程中会被后续RUN指令使用， 在镜像启动的容器中也会存在。格式为 ENV <key> <value>或ENV <key>=<value>...。
- ENTRYPOINT：指定镜像的默认入口命令， 该入口命令会在启动容器时作为根命令执行， 所有传人值作为该命令的参数。支持两种格式：ENTRYPOINT ["executable", "paraml ", "param2"]: exec 调用执行； ENTRYPOINT command param 1 param2: shell 中执行。每个 Dockerfile 中只能有一个 ENTRYPOINT, 当指定多个时， 只有最后一个起效。在运行时， 可以被 --entrypoint 参数覆盖掉。
- VOLUME：创建一个数据卷挂载点。
- USER：指定运行容器时的用户名或urn, 后续的RUN等指令也会使用指定的用户身份。
- WORKDIR：为后续的 RUN CMD ENTRYPO INT 指令配置工作目录。格式为 WORKDIR path /to/workdir，以使用多个 WORKDIR 令，后续命令 果参数是相对路径， 会基于之前命令指定
的路径。
- ONBUILD：指定当基于所生成镜像创建子镜像时，自动执行的操作指令，格式为 ONBUILD [INSTRUCTION]。使用 docker build 命令创建子镜像ChildImage 时（ FROM Parentimage ），会首
先执行 ParentImage 配置的ONBUILD指令。由于 ONBUILD 指令是隐式执行的，推荐在使用它的镜像标签中进行标注。
- STOPSIGNAL：指定所创建镜像启动的容器接收退出的信号值，STOPSIGNAL signal。
- HEALTHCHECK：配置所启动容器如 进行健康检查（如 判断健康与否），格式有两种：HEALTH HECK 【options】 cmd command，根据所执行命令返回值是否为判断； HEALTHCHECK NONE,禁止基础镜像中的健康检查。
- SHELL：指定其他命令使用 she ll 时的默认 she ll 类型：SHELL [” executable ”,”parameters”]，默认值为 {＂／ bin/sh ＂，“-c”}

**2.操作指令**
-  RUN：格式为 RUN <co mand ＞或 RUN [ "executable " , ” paraml ” , param2“]，后者指令会被解析为 JSON 数组，因此必须用双引号。每条 RUN 指令将在当前镜像基础上执行指定命令，并提交为新的镜像层 当命令较长时，可以使用＼来换行。
- CMD：指定启动容器时默认执行的命令，支持三种格式：CMD ［executable ＂，” paraml”，“ param2 ”]，：相当于执行 executable param1  param2 ；CMD command paraml param2 ：在默认的 Shell 中执行，提供给需要交互的应用；CM araml ”，” param2 ：提供给 ENTRYPOINT 的默认参数。每个 Dockerfile 只能有 CMD 命令 如果指定了多条命令，只有最后一条会被执行。
- ADD：添加内容到镜像，ADD <SRC> <dest>,该命令将复制指定的＜ SrC ＞路径下内容到容器中的＜dest ＞路径下。
- COPY：复制内容到镜像，COPY <src> <dest>，复制本地主机的＜ SrC> （为 Dockerfile 所在目录的相对路径，文件或目录）下内容到镜像中的＜dest＞。目标路径不存在时，会自动创建。

##三、创建镜像
docker [image] build 命令来创建镜像，格式为 docker build [OPTIONS] PATH [ URL I -。
该命令将读取指定路径下（包括子目录）的 Dock rfile ，并将该路径下所有数据作为上下文（ Context ）发送给 Docker 服务端 Docker 服务端在校验 Dockerfile 格式通过后，逐条执行其中定义的指令，碰到 ADD COPY RUN 指令会生成 层新的镜像。最终如果创建镜像成功，会返回最终镜像的 ID。
如果上下文过大， 会导致发送大量数据给服务端，延缓 建过程 除非是生成镜像所必需的文件，不然不要放到上下文路径 如果使用非上下文路径下的 Dockerfile ，可以通过-f选项来指定其路径。![](https://upload-images.jianshu.io/upload_images/9449419-a93bc9b426541a3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![](https://upload-images.jianshu.io/upload_images/9449419-ca960854598003a3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**1.选择父镜像**
大部分情况下，生成新的镜像都需要通过 FROM 指令来指定父镜像 父镜像是生成镜像的基础 ，会直接影到所生成镜像的大小和功能。
用户可 选择两种镜像作为父镜像，一种是所谓的基础镜像（ asei age ），另外一种是普通的镜像（往往由第三方创建，基于基础镜像）。
基础镜像比较特殊，其 dockerfile中往往不存在 指令，或者基于 scratch 镜像，着其在整个镜像树中处于根的位置。

**2.使用 dockerigno 文件**
通过 .dockerignore 文件（每一行添 一条匹配模式）来让 Docker 忽略匹配路径或文件，在创建镜像时候不将无关数据发送到服务端。
-  “＊”表示任意多个字符；
- “？”代表单个字符；
- “！”表示不匹配（即不忽略指定的路径或文件）

**3.多步骤创建**
对于需要编译的应用（如 Go Java 语言等）来说，通常情况下至少需要准备两个，环境的 Docker 镜像：编译环境镜像，运行环境镜像。使用多步骤创建，可以在保证最终生成的运行环境镜像保持精筒的情况下，使用单一的Dockerfile ，降低维护复杂度。

