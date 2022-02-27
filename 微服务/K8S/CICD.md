# CI CD 持续集部署

## 一、相关工具

- 代码仓库：gitlab
- 构建工具：maven | ant
- 持续集成工具：jenkins
- 脚本

### 1.一般流程

​				git —— maven 构建 —— 发布到服务器 —— 调用脚本（停止、启动）

k8s 流程：					| ——  构建镜像  —— 推送镜像  —— k8s 部署 —— 健康检查

一般流程问题：

- 先停止再启动，由服务间断的问题
- 服务器公用存在环境问题
- 多次构建问题： dev/test/release/preproduct/product

k8s好处：

- 环境稳定
- 服务不间断
- 一次构建，多环境运行

## 二、环境准备

### 1.git

yum install git

### 2.java

安装java，设置到java_home:参考：https://blog.csdn.net/qq_25482087/article/details/108565873

> wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"

```shell
sudo sh -c "echo 'PATH=/opt/k8s/bin:$PATH:$HOME/bin:$JAVA_HOME/bin' >>/root/.bashrc"
echo 'PATH=/opt/k8s/bin:$PATH:$HOME/bin:$JAVA_HOME/bin' >>~/.bashrc
```

### 3.maven

yum install maven

### 4.jenkins

```
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum upgrade
sudo yum install epel-release java-11-openjdk-devel
sudo yum install jenkins
sudo systemctl daemon-reload
sudo systemctl start jenkins
sudo systemctl status jenkins

Loaded: loaded (/etc/rc.d/init.d/jenkins; generated)
Active: active (running) since Tue 2018-11-13 16:19:01 +03; 4min 57s ago
```

（略）端口变更：

```
YOURPORT=8080
PERM="--permanent"
SERV="$PERM --service=jenkins"

# 有防火墙的话排除jenkins
firewall-cmd $PERM --new-service=jenkins
firewall-cmd $SERV --set-short="Jenkins ports"
firewall-cmd $SERV --set-description="Jenkins port exceptions"
firewall-cmd $SERV --add-port=$YOURPORT/tcp
firewall-cmd $PERM --add-service=jenkins
firewall-cmd --zone=public --add-service=http --permanent
firewall-cmd --reload
```

浏览器访问：http://ip:port，在jenkins日志控制台复制自动生成的密码登陆。

- 命令：`sudo cat /var/lib/jenkins/secrets/initialAdminPassword`将在控制台打印密码。
- 如果您使用官方`jenkins/jenkins`镜像在 Docker 中运行 Jenkins，您可以使用`sudo docker exec ${CONTAINER_ID or CONTAINER_NAME} cat /var/jenkins_home/secrets/initialAdminPassword`它在控制台中打印密码，而无需执行到容器中。

自定义jenkins时，可以安装任意数量插件。插件安装完成后，插件管理员用户。默认安装建议的插件。

![image-20220109230902891](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220109230904.png)

管理员用户：pd:123456

![image-20220109231327195](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220109231329.png)

![image-20220109231429519](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220109231430.png)

## 三、创建job

- 创建

![image-20220109231651922](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20220109231653.png)

- 配置

  