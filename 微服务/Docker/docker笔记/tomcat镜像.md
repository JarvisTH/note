1.获取tomcat
>docker pull tomcat

2.运行tomcat
>docker run tomcat #前台运行
docker run -d tomcat #后台运行

3.访问tomcat服务
tomcat运行在docker容器中，docker在linux上，windows访问tomcat需要将tomcat与linux进行一个映射配置。docker与linux本身基于桥接模式通信，需要映射端口。
>docker run -d -p 8080:8080 tomcat

4.如果发现访问不了，可以进入tomcat容器查看webapps目录。拉取下来的镜像中，webapps目录下没有内容，反而在webapps.dist目录下有对应内容，修改对应目录名字即可。进入容器命令：
>docker exec -it xxxx bash

