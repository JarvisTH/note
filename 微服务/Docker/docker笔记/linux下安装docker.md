docker是一个管理容器的引擎，可以在linux下直接通过yum安装。

- 安装前检查是否安装了docker
>yum list installed |grep docker 

- 安装docker
>yum install docker -y

- 查看docker 版本
>docker --version

- 卸载docker
docker对应包有多个，需要先列出来逐个删除。
>yum remove docker对应包  -y

- 启动docker 服务
>systemctl start docker
或者service docker start

- 停止docker服务
>systemctl stop docker
或者service docker stop

- 重启docker 服务
>systemctl restart docker
或者service docker restart

- 检查docker运行状态
>systemctl status docker
service docker status
ps -ef|grep docker

- 修改docker源
默认使用国外的docker源下载速度较慢，可改为国内，加速。
修改或新增 /etc/docker/daemon.json，完成后重启docker服务。
```
{
    "registry-mirrors" : [
    "https://registry.docker-cn.com",
    "https://docker.mirrors.ustc.edu.cn",
    "http://hub-mirror.c.163.com",
    "https://cr.console.aliyun.com/"
  ]
}
```
