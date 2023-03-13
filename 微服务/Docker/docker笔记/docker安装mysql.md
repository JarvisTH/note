- 下载镜像
>docker pull mysql

- 运行容器
>docker run -p 3306:3306 -e MYSQL_DATABASE=workdb -e MYSQL_ROOT_PASSWORD=123456 -d mysql:latest

-e:指定环境变量

- 进入容器
>docker exec -it xxx bash

- 登录mysql
>mysql -h 127.0.0.1(localhost) -u root -p

- 修改密码
>alter USER 'root'@'localhost' IDENTIFIED BY '123456'

- 授予远程登录权限
如果mysql工具无法连接成功时，可以测试下。
>create user 'wkcto'@'%' identified with mysql_native_password by '123456';
grant all privileges on *.* to 'wkcto'@'%';

- 如果数据容器停掉，数据可能会丢失，需要保存数据，可以进行新建镜像保存数据
>docker commit xxxx
