- 将springboot程序打包为jar/wer包
- 将包上传到某个目录下，eg：/root/docker
- 定义dockerfile文件，用于创建镜像

**1.jar包**
打包为jar，表示是java程序，打包为war则为java外部程序。

- 定义jar包dockerfile文件
```
FROM 自定义java镜像（或者 java）
MAINTAINER jarvis
ADD springboot-web-1.0.0.jar /opt
RUN chmod +x /opt/springboot-web-1.0.0.jar
CMD java -jar /opt/springboot-web-1.0.0.jar
```

- 构建、运行镜像
>docker build -t springboot-web-jar .
docker run -d -p xxx xxxx

**2.war包**
- 定义dockerfile
```
FROM xxx-tomcat
MAINTAINER jarvis
ADD xxxx.war /usr/local/xxx-tomcat/webapps
EXPOSE 8080
CMD /usr/local/xxx-tomcat/bin/catalina.sh run
```

- 创建、运行镜像
>docker build -t xxxx .
docker run -d -p xxx xxx


