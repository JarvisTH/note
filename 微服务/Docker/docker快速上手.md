docker是软件化的虚拟机，与宿主机共享操作系统，而不用单独安装操作系统。

## 1.docker安装

```shell
// 查看是否已经安装了docker
yum list installed|grep docker

// 卸载
yum remove xxx

// 安装
yum install docker -y

// 校验
docker -v
```



## 2.docker基础命令

docker 需要从本地或者远程仓库拉取对应镜像，运行镜像后，生成容器，容器里运行应用。

### 1.配置仓库地址

```shell
vi /etc/docker/daemon.json

{
 "registry-mirrors":["https://6kx4zyno.mirror.aliyuncs.com"]
}
```

### 2.查找对应镜像

```shell
// 搜索远程仓库
docker search tomcat

// 查看本地仓库
docker images
```

### 3.操作镜像与容器

```shell
docker pull tomcat

// 前台运行
docker run tomcat

// 后台运行
docker run -d tomcat 

// 查看正在运行的容器
docker ps

// 停止容器
docker stop xxx

// 启动容器
docker start xxx

// 删除镜像
docker rmi xxx

// 删除容器,停止状态才能删除
docker rm xxx

// 查看容器信息
docker inspect xxx

// 停止所有运行的容器
docker stop ${docker ps -q}

// 删除所有容器
docker rm  ${docker ps -aq}
```

### 4.访问容器

docker 网络机制：比如想要在宿主机服务容器中的tomcat，宿主机先访问虚拟机的linux，linux转发到对应的容器。容器与linux采用网络桥接模式通信，对应配置端口映射。

```shell
// -p 映射linux端口 : 容器端口   
docker run -d -p 8080:8080 tomcat 
```

### 5.进入容器

```shell
docker exec -it xxxid bash
```

- -i :交互式
- t：虚拟控制台
- exit：退出

## 3.docker核心组件

### 1.架构

C/S架构，使用远程API管理和创建容器。镜像与容器的关系，类似于类与对象的关系。

### 2.核心要素

镜像、容器、仓库。docker是管理容器的引擎。

- 镜像：一个只读模板，用来创建容器。

  镜像由许多层的文件系统叠加构成。最下面是一个引导文件系统bootfs，第二层是一个root文件系统rootfs，root文件系统通常是某种操作系统，在root操作系统之上有很多层文件系统，叠加在一起构成docker镜像。镜像是只读的，启动时在最上层创建一个可读写层。

![image-20220608233349287](../../../../picbed/store/picbed/img/image-20220608233349287.png)

​      日常操作：下载镜像、列出镜像、运行镜像、删除镜像

- 容器：镜像创建的运行实例，每个容器相互隔离、保证安全平台。

  启动容器有两种方式，一种是基于镜像启动，一种是基于停止状态的容器重新启动。

- 仓库：存放镜像文件。仓库注册服务器可能存在多个仓库，每个仓库包含多个镜像。创建了自己的镜像后，就可以用push传上仓库，然后pull下来就可以使用。

## 4.使用示例

### 1.Mysql

```shell
docker pull mysql

docker run -p 3306:3306 -e MYSQL_DATABASE=workdb -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest

docker exec -it  xxx bash 

mysql -uroot -p

// 远程访问
CREATE USER 'test'@'%' IDENTITIED WITH mysql_native_password By '123456':
GRANT ALL PRIVILEGES ON *.* TO 'test'@'%';
```

- -e :指定环境变量，具体环境变量可以在官方镜像指导文件中查看

### 2.Nginx

```shell
docker pull Nginx

docker run -p 80:80 -d nginx

docker exec -it xxx  bash 

// 访问
ip+port

// 部署网站
```

### 3.zookeeper

```shell
docker pull zookeeper

docker run -p 2181:2181 -d zookeeper

docker exec -it xxx bash 

// 客户端工具访问zk
Zoolnspector工具
```

### 4.ActiveMQ

```shell
docker pull activmq

docker run -p 8161:8161 -d  activemq

docker exec -it xxx bash

// 浏览器访问
```

## 5.自定义镜像

dockerfile 用于构建docker镜像，由命令语句组成。

### 1.基本结构

一般分为四部分：

- 基础镜像信息
- 维护者信息
- 镜像操作指令
- 容器启动时执行指令

### 2.自定义JDK镜像

Dockerfile:

```dockerfile
// 以centos lates镜像作为基础镜像
FROM centos:latest   
// 表示作者
MAINTAINER test
// 将当前目录下的jdk包拷贝到/容器里的/usr/local
ADD jdk-8u121-linux-x64.tar.gz /usr/local
// 配置环境变量
ENV JAVA_HOME /usr/local/jdk1.8.0_121
ENV CLASSPATCH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH $PATH:$JAVA_HOME/bin
// 执行启动命令
CMD java -version
```

- FROM：FROM <image>或 FROM <image>:<tag>。同一个dockerfile文件中创建多个镜像时，使用多个FROM指令，每个镜像一次。
- MAINTAINER：格式MAINTAINER <name> 作者信息
- ADD：格式ADD src dest，复制指定src到容器的dest中。
- ENV :格式ENV key value，指定一个环境变量，后续被run命令使用，在容器运行时保持
- CMD:指定启动容器时执行的命令，每个dockerfile文件只能有一条cmd命令，多条命令只有最后一条会执行。用户执行时指定了运行命令则会覆盖。

```shell
// 构建镜像
docker build -t test_jdk1.8.0_121  .

// 运行镜像
docker run -d xxx 
```

- -t :指定名称

### 3.自定义tomcat镜像

```dockerfile
FROM test_jdk1.8.0_121
MAINTAINER test
ADD apache-tomcat-8.5.24.tar.gz /usr/local
ENV CATALINA_HOME /usr/local/apache-tomcat-8.5.24
ENV PATH $PATH:$CATALINA_HOME/lib:$CATALINA_HOME/bin
// 到处端口
EXPOSE 8080
CMD /usr/local/apache-tomcat-8.5.24/bin/catalina.sh run
```

- EXPOSE:格式 EXPOSE port。告诉docker服务器容器暴露的端口号，在启动容器时需要通过 -p 映射端口。

```shell
docker build -t test-tomcat-8.5.24 .

docker run -d -p 8080:8080 test-tomcat-8.5.24
```

### 4.自定义MYSQL镜像

```dockerfile
FROM centos:latest
MAINTAINER test
RUN yum install -y mysql-server mysql
RUN /etc/init.d/mysqld start &&\
	mysql -e "grant all privileges on *.* to 'root'@'%' identified by '123456' WITH GRANT OPTION;"&&\
	mysql -e "grant all privileges on *.* to 'root'@'localhost' identified by '123456' WITH GRANT OPTION;"&&\
	mysql -uroot -p123456 -e "show databases;"
EXPOSE 3306
CMD /usr/bin/mysql_safe
```

- RUN：RUN xxx，在当前镜像基础上执行指定命令。

```shell
docker build -t test-mysql

docker run -d -p 3306:3306 xxx  
```

### 5.自定义redis镜像

```dockerfile
FROM centos:latest
MAINTAINER test
RUN yum -y install epel-release && yum -y install redis && yum install -y net-tools
EXPOSE 6379
CMD /usr/bin/redis-server --protected-mode no
```

```shell
docker build -t test-redis

docker run -d -p 6379:6379 xxx  
```

## 6.发布镜像到仓库

- 阿里云仓库：dev.aliyun.com
- 注册阿里云镜像仓库
- 发布镜像：创建镜像仓库，根据管理链接中的操作指南管理

## 7.docker应用部署

基本流程：

- 打包：jar/war包
- 将打好的包传到linux目标目录
- 定义dockerfile文件，创建项目镜像

### 1.部署jar包

#### 1.定义jar包dockerfile文件

```dockerfile
FROM java
MAINTAINER test
ADD springboot-web-1.0.0.jar /opt
RUN chmod +x /opt/springboot-web-1.0.0.jar
CMD java -jar /opt/springboot-web-1.0.0.jar
```

#### 2.构建与运行

```shell
docker build -t springboot-web-jar .
docker run -d -p 8080:8080 xxx
```

#### 3.JAR包依赖环境准备

```shell
// 运行redis容器
docker run -p 6379:6379 -d redis
// 运行mysql容器
docker run -p 3306:3306 -e MYSQL_DATABASE=workdb -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest
```

### 2.部署war包

#### 1.定义war包dockerfile

```dockerfile
FROM test-tomcat-8.5.24
MAINTAINER test
ADD springboot-web-1.0.0.war /usr/local/apache-tomcat-8.5.24/webapps
EXPOSE 8080
CMD /usr/local/apache-tomcat-8.5.24/bin/catalina.sh run
```

#### 2.构建与运行

```shell
docker build -t sspringboot-web-1.0.0.war .
docker run -d -p 8080:8080 xxx
```

#### 3.war包依赖容器环境准备

同jar

## 8.保存新镜像

容器重弹后，数据会丢失，可以创建新镜像进行保存。

```shell
// 修改容器保存
docker commit id tag
```

