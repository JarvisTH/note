##一、为什么使用docker
- docker容器虚拟化的好处：通过容器来打包应用、解藕应用和运行平台
- 更快速的交付和部署
- 更高效的资源利用，Docker 是内核级的虚拟化，对资源的额外需求很低
- 更轻松的迁移和扩展
- 更简单的更新管理。使用 Dockerfile ，只需要小小的配置修改，就可以替代以往大量的更新工作 所有修改都以增 的方式被分发和更新，从而实现自动化并且高效的容器管理

1.与VM比较：
- Docker 容器很快，启动和停止可以在秒级实现
- 对系统资源需求很少
- 通过 Dockerfile 支持灵活的自动化创建和部署机制

##二、虚拟化
虚拟化技术可分为基于硬件的虚拟化和基于软件的虚拟化，基于软件的虚拟化从对象所在层次，又可以分为应用虚拟化和平台虚拟化。
前者般指的是 些模拟设备或诸如 ine 这样的软件，后者又可以细分为几个子类：
- 完全虚拟化：VM，VirtualBox等
- 硬件辅助虚拟化：VM，KVM等
- 部分虚拟化
- 超虚拟化
- 操作系统级虚拟化：内核通过创建多个虚拟的操作系统实例（内核和库）来隔离不同的进程。

![](https://upload-images.jianshu.io/upload_images/9449419-5efdaf60cee5b164.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

docker是一个容器管理引擎：
docker服务启动-》下载镜像-》启动镜像得到容器-》容器内运行程序。![](https://upload-images.jianshu.io/upload_images/9449419-395fa8a92afab7bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
