容器中的管理数据主要有两种方式：
- 数据卷 （Data Volumes ）容器内数据直接映射到本地主机环境；
- 数据卷容器（Data  Volume Containers 使用特定容器维护数据卷。

##一、数据卷
数据卷 Data Volumes 个可供容器使用的特殊目录，它将主机操作系统目录直接映射进容器，类似于 Linux 中的 mount 行为。特性：
- 数据卷可以在容器之间共事和重用，容器间传递数据将变得高效与方便；
- 对数据卷内数据的修改会立马生效，无论是容器内操作还是本地操作；
- 对数据卷的更新不会影响镜像，解摘开应用和数据
- 卷会一直存在 ，直到没有容器使用，可以安 地卸载它

**1.创建数据卷**
Docker 提供了 volume 子命令来管理数据卷，如下命令可以快速在本地创建一个数据卷：
>docker volume create -d local test

查看／ ar/lib docker /vo lumes 路径下，会发现所 建的数据卷位置，docker volume具体参数可以参考手册。

**2.绑定数据卷**
可以在创建容器时将主 地的任意路径挂载到容器内作为数据卷，这种形式创建的数据卷称为绑定数据卷。
用docker [container] run命令时，可以使用-mount选项使用数据卷，-mount支持三种类型数据卷：
- volume：普通数据卷，映射到/var/lib/docker/volumes路径下
- bind：绑定数据卷，映射到主机指定路径
- tmpfs：临时数据卷，存在于内存

本地目录的路径必须是绝对路径，容器内路径可以为相对路径 如果目录不存在， Docke 会自动创建。Docker 载数据卷的默认权限是读写（ rw） ，用户也可以通过 ro 定为只读：
>docker run -d -P --name web -v /webapp: /opt/webapp:ro training/webapp python app.py

加了： ro 之后，容器内对所挂载数据卷内的数据就无法修改了。如果直接挂载一个文件到容器，使用文件编辑工具，包括 vi 或者 sed --in-place时，可能造成文件inode改变。推荐方式是直接挂载文件所在的目录到容器内。

##二、数据卷容器
如果用户需要在多个容器之间共享一些持续更新的数据，最简单的方式是使用数据卷容器。数据卷容器也是一个容器，但是它的目的是专门提供数据卷给其他容器挂载。
- 创建一个数据卷容器 dbdata 并在其中创建一个数据卷挂载到/dbdata：
>docker run -it -v /dbdata --name dbdata ubuntu
- 可以在其他容器中使用－－ volumes-from 来挂载 dbdata 容器中的数据卷，例如创建 dbl,db2 两个容器，并从 dbdata 容器挂载数据卷：
>docker run -it --volumes-from dbdata -name dbl ubuntu
docker run -it --volumes-from dbdata --name db2 ubuntu

容器 dbl db2 都挂载同一个数据卷到相同的／dbdata 目录，三个容器任何一方在该目录下的写人，其他容器都可以看到。可以多次使用－－ volumes-from 参数来从多个容器挂载多个数据卷，还可以从其他已经挂载了容器卷的容器来挂载数据卷。
如果删除了挂载的容器（包括 dbdata、db1和 db2 ），数据卷并不会被自动删除.如果删除一个数据卷，必须在删除最后一个还挂载着它的容器时显式使用 docker rm -v命令指定同时删除关联的容器。

##三、利用数据卷容器迁移数据
可以利用数据卷容器对其中的数据卷进行备份、恢复，以实现数据的迁移。

**1.备份**
备份 dbdata 数据卷容器内的数据卷：
>docker run -volumes-from dbdata -v $ (pwd) : /backup - -name worker ubuntu tar 
cvf /backup/backup.tar /dbdata

利用 ubuntu 镜像创建了一个容器 worker。使用－ -volumes-from dbdata 参数来让 worker 容器挂载 dbdata 容器的数据卷（ dbdata 数据卷）；使用-v $ (pwd) : /backup 
参数来挂载本地的当前目录到 worker容器的／backup目录。
worker 容器启动后，使用 tar cvf /backup/backup.tar /dbdata 命令将／dbdata下内容备份为容器内的／backup/backup. tar ，即宿主主机当前目录下的backup.tar。

**2.恢复**
- 创建一个带有数据卷的容器 bdata2:
>docker run -v /dbdata --name dbdata2 ubuntu /bin/bash
- 创建另一个新的容器，挂载 dbda ta2 容器，并使用 untar 解压备份文件到所挂载的容器卷中：
>docker run --volumes-from dbdata2 -v $(pwd) :/backup busybox tar xvf 
/backup/backup.tar

