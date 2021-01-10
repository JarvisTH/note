# Redis安装与配置

## 一、安装

1. 下载Redis软件，上传至 /home/software目录下

2. 解压redis

   ```
   tar -zxvf redis-xxxx.tar.gz
   ```

3. 安装gcc编译环境

   ```
   yum install gcc-c++
   ```

4. 进入redis目录下，进行安装：make && make install

   make：若报错：xxx没有名为xxxx的错误，需要升级gcc：

```
#升级到 5.3及以上版本
yum -y install centos-release-scl
yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils

scl enable devtoolset-9 bash
```

## 二、配置redis

1.将utils目录下的redis_init_script拷贝到/etc/init.d目录，目的是将redis作为开机自启动

2.创建 /usr/local/redis，用于存放配置文件

3.拷贝redis配置文件到/usr/local/redis下

4.修改redis.conf配置文件

- 修改daemonize no -> daemonize yes，目的是为了让redis启动后在linux后台运行

  ![image-20210102115505767](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20210102115507.png)

- 修改redis工作目录：建议修改为/usr/local/redis/working

  ![image-20210102115540791](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20210102115542.png)

- 绑定ip改为0.0.0.0，代表可以让远程连接，不受ip限制

  ![image-20210102115729897](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20210102115731.png)

- 设置默认密码

  ![image-20210102115804063](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20210102115805.png)

5.修改redis_init_script文件的redis配置，并修改redis核心配置文件名为：6379.conf

![image-20210102115900979](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20210102115902.png)

6.为redis启动脚本添加执行权限，随后启动redis

![image-20210102120044868](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20210102120046.png)

7.检查redis进程

```
ps -ef|grep redis
```

8.设置redis开机自启动，修改reids_init_script,添加内容：

```
#chkconfig: 22345 10 90
#description: Start and Stop redis
```

![image-20210102120245546](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/20210102120246.png)

执行操作：

```
chkconfig redis_init_script on
```

重启redis，再查看进程。