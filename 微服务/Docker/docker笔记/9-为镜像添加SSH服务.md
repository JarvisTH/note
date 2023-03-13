##一、基于commit命令创建
docker commit 支持用户提交自己对 定容器的修改，并生成新的镜像命令格式为docker commit CONTAINER [REPOSITORY[:TAG]]。
**1.准备**
获取 ubuntu l8.04 镜像，并 建一个容器:
```
docker pull ubuntu:18.04
dcoker run -it ubuntu:18.04 bash
```
**2.配置软件源**
apt-get update 令来更新软件源信息。
官方源速度慢，以替换为国内 163 sohu 等镜像的源，修改 /etc/apt/sources.list 文件：
```
deb http://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
deb http //mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse 
deb http //mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse 
deb http://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse 
deb http://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse 
deb-src  http://mirrors.163.com/ubuntu/ bionic main restricted universe multiverse
deb-src  http://mirrors.163.com/ubuntu/ bionic-security main restricted universe multiverse 
deb-src  http://mirrors.163.com/ubuntu/ bionic-updates main restricted universe multiverse 
deb-src  http://mirrors.163.com/ubuntu/ bionic-proposed main restricted universe multiverse 
deb-src  http://mirrors.163.com/ubuntu/ bionic-backports main restricted universe multiverse
```
执行apt-get update。


**3.安装和配置SSH服务**
选择主流的 openssh-server 作为服务端 可以看到需要下载安装众多的依赖软件包：
```
apt-get install openssh-server
```
如果需要正常启动 SSH 务， 目录／var/run/sshd 须存在。手动创建它，并启动SSH 务：
```
mkdir -p /var/run/sshd
/usr/sbin/sshd -D &
```
查看容器的 22 端口（SSH 服务默认监昕的端口），可见此端口已经处于监听状态：
```
netstat -tunpl
```
修改 SSH 服务的安全登录配置，取消 pam登录限制:
```
sed -ri 's/session required pam_loginuid.so/#session required pam_loginuid.so/g' /etc/pam.d/sshd
```
root 用户目录下创建.ssh目录，并复制需要登录的公钥信息（一般为本地主机用户目录下的 .ssh/id_rsa.pub 文件，可由 ssh-keygen -t rsa 命令生成）到authorized_keys文件中：
```
mkdir root/.ssh
vi /root/.ssh/authorized_keys
```
创建自动启动SSH服务的可执行文件run.sh，添加权限：
```
vi /run.sh
chmod +x run.sh

#run.sh内容
#! / bin/ bash 
/ usr/ sbin/ sshd -D
```
最后，退出容器exit/
**4.保存镜像**
将所退出的容器用 docker comm it 命令保存为 个新的 sshd:ubuntu 镜像：
```
docker commit fc1 sshd:ubuntu
docker images
```
**5.使用镜像**
启动容器，并添加端口映射 10022 >22，其中 10022 是宿主主机的端口， 22 是容器的 SSH 服务监昕端口：
```
docker run -p 10022:22 -d sshd:ubuntu /run.sh
```
启动成功后，可以用docker ps在宿主主机上看到容器运行的详细信息。在宿主主机或其他主机上，可以通过 SSH 访问 10022 端口来登录容器：
```
ssh 192.168.1.200 -p 10022
```

##二、基于dockerfile创建
**1.创建工作目录**
创建一个 ss hd ubuntu 工作目录,在其中创建Dockerfile run.sh 文件：
```
mkdir sshd_ubuntu
touch sshd_ubuntu/Dockerfile sshd_ubuntu/run.sh
```
**2.编写run.sh和authorized_keys文件**
```
#run.sh
#!/bin/bash
/user/sbin/sshd -D

#在宿主主机上生成 SSH 密钥对，并创建 au thorized_keys 文件
ssh-keygen -t rsa
cat ~/.ssh/id_rsa.pub>authorized_keys
```
**3.Dockerfile**
Dockerfile 的内容及各部分的注释，可以对比上一节中利用 docker commit命令创建镜像过程，所进行的操作基本一致：
```
#设置继承镜像
FROM ubuntu: 18. 04 
#提供一些作者信息
MAINTAINER docker user (user@docker.com) 
＃下面开始运行命令，此处更改 ubuntu 的源为国内 163 的源
RUN echo "deb http: //mirrors.163.com/ubuntu/ bionic main restricted universe 
multiverse" > /etc/apt /sources.list 
RUN echo "deb http: //mirrors.163.com/ubuntu/ bionic-security main restricted 
universe multiverse ” >/etc/ apt/sources.list
RUN echo "deb http://mirrors .163. com/ubuntu/ bionic-updates main restricted 
universe multiverse ” > /etc/apt/sources.list
RUN echo ” deb http: //mirrors .163. com/ubuntu/ bionic-proposed main restricted 
universe multiverse ”>/etc/apt/sources.list
RUN echo "deb http: //mirrors . 1 63. com/ubuntu/ bionic-backports main restricted 
universe multiverse ”> /etc/apt/sources.list
RUN apt-get update

#安装ssh服务
RUN apt-get install -y openssh-server 
RUN mkdir - p /var/ run/ sshd 
RUN mkdir -p / root/ .ssh

#取消pam限制
RUN sed -ri 's/ session required pam_ loginuid. so/ #session required pam_ loginuid.so/g' / etc/pam.d/ sshd 

#复制配置文件到相应位置，赋权
ADD authorized_keys /root /.ssh/authorized_keys 
ADD run. sh /run. sh 
RUN chmod 755 /run.sh

#开放端口
EXPOSE 22

#设置自启动命令
CMD ["/run.sh"]
```
**4.创建镜像**
sshd_ubuntu目录下，使用 docker build 命令来创建镜像，最后一个“."表示使用当前目录中的Dockerfile：
```
cd sshd_ubuntu
docker build -t sshd:dockerfile .
```
如果读者使用 Dockerfile 创建自定义镜像，那么需要注意的是 Docker 会自动删除中间临时创建的层，还需要注 步的操作和编写的 Dockerfile 中命令的对应关系。
**5.测试镜像，运行容器**
使用刚才创建的 sshd:dockerfile 镜像来运行一个容器，直接启动镜像，映射容器的 22 端口到本地的 10122 端口：
```
docker run -d -p 10122:22 sshd:dockerfile
docker ps
ssh 192.168.1.200 -p 10122
```
