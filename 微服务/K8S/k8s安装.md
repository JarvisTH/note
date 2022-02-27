## 一·、系统初始化

### 1.集群主机名

- 设置主机名，然后exit重新登录。
  - ```bash
    hostnamectl set-hostname master         # 将 master  替换为当前主机名
    hostnamectl set-hostname node01         # 将 node01  替换为当前主机名
    hostnamectl set-hostname node02         # 将 node01  替换为当前主机名
    ```
- 修改/etc/hostname 文件，添加主机名和ip的对应关系：
  - ```jsx
    cat > /etc/hosts << EOF
    127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
    ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
    
    192.168.52.132    master
    192.168.52.133    node01
    192.168.52.134    node02
    EOF
    ```

### 2.添加k8s和docker账户

- 在每台机器上添加k8s账户

  >
  >
  >sudo useradd -m k8s
  >
  >sudo sh -c 'echo along|passwd k8s --stdin'

- 修改visudo权限：visodu命令参考文章：https://blog.csdn.net/youmatterhsp/article/details/80494372

  >
  >
  >sudo visudo # 去掉%wheel ALL=(ALL)NOPASSWD：ALL 这行的注释  ——允许wheel用户组中的用户在不输入该用户的密码的情况下使用所有命令
  >
  >sudo grep '%wheel.*NOPASSWD: ALL' /etc/sudoers

- 将k8s用户归到wheel组

  >
  >
  >gpasswd -a k8s wheel
  >
  >id k8s

- 在每台机器上添加docker用户，将k8s账户添加到docker组中，同时配置dockerd参数（装完docker才有):

  >
  >
  >sudo useradd -m docker
  >
  >gpasswd -a k8s docker
  >
  >sudo mkdir -p /opt/docker
  >
  >```jsx
  >cat > /etc/docker/daemon.json << EOF
  >{
  >    "registry-mirrors": [""https://hub-mirror.c.163.com", "https://docker.mirrors.ustc.edu.cn"],
  >    "max-concurrent-downloads": 20
  >}
  >EOF
  >```

### 3.无密码 ssh登陆其他节点

- 生产密钥对
  - ssh-keygen #连续回车即可

- 将自己的密钥共钥发给其他服务器:

  - 如果报错ssh: Could not resolve hostname kube-node1: Name or service not known：在/etc/hosts中写上ip 与名称映射。

  ```shell
  ssh-copy-id root@master
  ssh-copy-id root@node01
  ssh-copy-id root@node02
  
  ssh-copy-id k8s@master
  ssh-copy-id k8s@node01
  ssh-copy-id k8s@node02
  ```



### 4.将可执行文件路径/opt/k8s/bin添加到PATH变量：

- 安装java，设置到java_home:参考：https://blog.csdn.net/qq_25482087/article/details/108565873

  > wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"

```shell
sudo sh -c "echo 'PATH=/opt/k8s/bin:$PATH:$HOME/bin:$JAVA_HOME/bin' >>/root/.bashrc"
echo 'PATH=/opt/k8s/bin:$PATH:$HOME/bin:$JAVA_HOME/bin' >>~/.bashrc
```



### 环境变量

将要引入一个环境变量：定义所有这次部署 k8s 所工作的目录

```
mkdir -p /opt/k8s/bin/

cat > /opt/k8s/bin/environment.sh << "EOF"
#!/usr/bin/bash
# 生成 EncryptionConfig 所需的加密 key
export ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
# 最好使用当前未用的网段来定义服务网段和Pod网段
# 服务网段，部署前路由不可达，部署后集群内路由可达(kube-proxy 和 ipvs 保证)
export SERVICE_CIDR="10.254.0.0/16"
# Pod 网段，建议 /16 段地址，部署前路由不可达，部署后集群内路由可达(flanneld 保证)
export CLUSTER_CIDR="172.30.0.0/16"
# 服务端口范围 (NodePort Range)
export NODE_PORT_RANGE="8400-9000"
# 集群各机器 IP 数组
export NODE_IPS=(192.168.52.132 192.168.52.133 192.168.52.134)
# 集群各 IP 对应的 主机名数组
export NODE_NAMES=(master node01 node02)
# kube-apiserver 的 VIP（HA 组件 keepalived 发布的 IP）
export MASTER_VIP=192.168.52.13
# kube-apiserver VIP 地址（HA 组件 haproxy 监听 8443 端口）
export KUBE_APISERVER="https://${MASTER_VIP}:8443"
# HA 节点，配置 VIP 的网络接口名称
export VIP_IF="ens33"
# etcd 集群服务地址列表
export ETCD_ENDPOINTS="https://192.168.52.132:2379,https://192.168.52.133:2379,https://192.168.52.134:2379"
# etcd 集群间通信的 IP 和端口
export ETCD_NODES="master=https://192.168.52.132:2380,node01=https://192.168.52.133:2380,node02=https://192.168.52.134:2380"
# flanneld 网络配置前缀
export FLANNEL_ETCD_PREFIX="/kubernetes/network"
# kubernetes 服务 IP (一般是 SERVICE_CIDR 中第一个IP)
export CLUSTER_KUBERNETES_SVC_IP="10.254.0.1"
# 集群 DNS 服务 IP (从 SERVICE_CIDR 中预分配)
export CLUSTER_DNS_SVC_IP="10.254.0.2"
# 集群 DNS 域名
export CLUSTER_DNS_DOMAIN="cluster.local."
# 将二进制目录 /opt/k8s/bin 加到 PATH 中
export PATH=/opt/k8s/bin:$PATH
EOF
```



### 5.安装依赖包：在每台机器上安装依赖包

- yum报错：Could not retrieve mirrorlist。
  - 配置/etc/resolv.conf
  - nameserver 8.8.8.8
    search localdomain

```shell
sudo yum install -y epel-ralease
sudo yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp
```

脚本方式：

```
cat > magic01_install_dependpackage.sh << "EOF"
#!/bin/bash
# 安装依赖包
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} "yum install -y lrzsz tree vim epel-release conntrack ipvsadm ipset jq sysstat curl iptables libseccomp"
done
EOF
```



### 6.关闭防火墙

- 关闭服务，设置开机不自启

  ```shell
  sudo systemctl stop firewalld
  sudo systemctl disable firewalld
  ```

  

- 清空防火墙规则

  ```shell
  sudo iptables -F && sudo iptables -X && sudo iptables -F -t nat && sudo iptables -X -t nat
  sudo iptables -P FORWARD ACCEPT
  ```

脚本方式：

```
cat > magic02_shutdown_firewall.sh << "EOF"
#!/bin/bash
# 关闭防火墙
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} "systemctl stop firewalld && systemctl disable firewalld"
    ssh root@${node_ip} "iptables -F && sudo iptables -X && sudo iptables -F -t nat && sudo iptables -X -t nat && iptables -P FORWARD ACCEPT"
done
EOF
```



### 7.关闭swap分区

如果开启了 swap 分区，kubelet 会启动失败(可以通过将参数 --fail-swap-on 设置为false 来忽略 swap on)，故需要在每台机器上关闭 swap 分区

```
sudo swapoff -a
```

为了防止开机自动挂载 swap 分区，可以注释 /etc/fstab 中相应的条目：

```
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

脚本方式：

```
cat > magic03_shutdown_swap.sh << "EOF"
#!/bin/bash
# 关闭 swap 分区
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} "swapoff -a && sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab"
done
EOF
```

### 8.关闭SELinux

关闭 SELinux，否则后续 K8S 挂载目录时可能报错 Permission denied ：

```
sudo setenforce 0
```

修改配置文件，永久生效：

```
vi /etc/selinux/config
SELINUX=disabled
grep SELINUX /etc/selinux/config
```

### 9.关闭dnsmasq（可选）

linux 系统开启了 dnsmasq 后，将系统 DNS Server 设置为 127.0.0.1，这会导致 docker 容器无法解析域名，需要关闭它：

```
sudo service dnsmasq stop
sudo systemctl disabled dnsmasq
```

### 10.加载内核模块

```
sudo modprobe br_netfilter
sudo modprobe ip_vs
```

脚本方式：

```
cat > magic04_loading_kernel.sh << "EOF"
#!/bin/bash
# 加载内核模块
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} "modprobe br_netfilter && modprobe ip_vs"
done
EOF
```



### 11.设置系统参数

```
cat > kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
net.ipv4.tcp_tw_recycle=0
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1
net.netfilter.nf_conntrack_max=2310720
EOF

sudo cp kubernetes.conf /etc/sysctl.d/kubernetes.conf
sudo sysctl -p /etc/sysctl.d/kubernetes.conf
sudo mount -t cgroup -o cpu,cpuacct none /sys/fs/cgroup/cpu,cpuacct
```

- tcp_tw_recycle 和 Kubernetes 的 NAT 冲突，必须关闭 ，否则会导致服务不通；
- 关闭不使用的 IPV6 协议栈，防止触发 docker BUG；

脚本方式：

```
mkdir -p /data/kuberconfig

cat > /data/kuberconfig/kubernetes.conf << EOF
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
net.ipv4.tcp_tw_recycle=0
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1
net.netfilter.nf_conntrack_max=2310720
EOF

cat > magic05_system_parameter.sh << "EOF"
#!/bin/bash
# 设置系统参数
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    scp /data/kuberconfig/kubernetes.conf root@${node_ip}:/etc/sysctl.d/kubernetes.conf
    ssh root@${node_ip} "sysctl -p /etc/sysctl.d/kubernetes.conf"
done
EOF
```



### 其他安装包

安装几个常用但系统未必安装的包，同时配置一下时间同步，并把时间同步写入到定时任务当中。

```
cat > magic06_install_basic_package.sh << "EOF"
#!/bin/bash
# 安装其它的基础包
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} 'yum -y install wget ntpdate lrzsz curl rsync && ntpdate -u cn.pool.ntp.org && echo "* * * * * /usr/sbin/ntpdate -u cn.pool.ntp.org &> /dev/null" > /var/spool/cron/root'
done
EOF
```



### 12.设置系统时区

- 调整系统timezone

  ```
  sudo timedatectl set-timezone Asia/Shanghai
  ```

- 将当前时间写入硬件时钟

  ```
  sudo timedatectl set-local-rtc 0
  ```

- 重启依赖于系统时间的服务

  ```
  sudo systemctl restart rsyslog
  sudo systemctl restart crond
  ```

- 更新系统时间

  ```
  yum -y install ntpdate
  sudo ntpdate cn.pool.ntp.org
  ```



### 13.创建目录

每台机器创建目录

```
sudo mkdir -p /opt/k8s/bin

sudo mkdir -p /opt/k8s/cert

sudo mkdir -p /opt/etcd/cert

sudo mkdir -p /opt/lib/etcd

sudo mkdir -p /opt/k8s/script

chown -R k8s /opt/*
```

脚本方式：

```
cat > magic07_create_directory.sh << "EOF"
#!/bin/bash
# 创建目录
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} 'mkdir -p /opt/k8s/bin && chown -R k8s /opt/k8s && mkdir -p /etc/kubernetes/cert && chown -R k8s /etc/kubernetes'
    ssh root@${node_ip} 'mkdir -p /etc/etcd/cert && chown -R k8s /etc/etcd/cert && mkdir -p /var/lib/etcd && chown -R k8s /var/lib/etcd'
done
EOF
```



### 分发环境变量

```
cat > magic08_distribute_environment.sh << "EOF" 
#!/bin/bash
# 分发环境变量配置
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    scp /opt/k8s/bin/environment.sh k8s@${node_ip}:/opt/k8s/bin/
    ssh k8s@${node_ip} "chmod +x /opt/k8s/bin/*"
done
EOF
```



### 14.检测系统内核和模块是否合适运行docker（仅使用linux)

```
curl https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh > check-config.sh

chmod +x check-config.sh

bash ./check-config.sh
```



## 二、创建CA证书和密钥

确保安全， kubernetes 系统各组件需要使用 x509 证书对通信进行加密和认证。CA是自签名的根证书，用来签名后续其他证书。

### 1.安装 cfssl 工具集

```
mkdir -p /data/application && cd /data/application
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64

cp /data/application/cfssljson_linux-amd64 /opt/k8s/bin/cfssljson
cp /data/application/cfssl_linux-amd64 /opt/k8s/bin/cfssl
cp /data/application/cfssl-certinfo_linux-amd64 /opt/k8s/bin/cfssl-certinfo
chmod +x /opt/k8s/bin/*
export PATH=/opt/k8s/bin:$PATH
```



- wget出现**Unable to establish SSL connection**:  sudo apt-get install openssl-devel

### 2.创建根证书CA

CA 证书是集群所有节点共享的，只需要创建一个 CA 证书。

- 创建配置文件

  CA 配置文件用于配置根证书的使用场景 (profile) 和具体参数 (usage，过期时间、服务端认证、客户端认证、加密等)，后续在签名其它证书时需要指定特定场景。

  ```
  mkdir -p /data/cert
  
  cat > /data/cert/ca-config.json <<EOF
  {
    "signing": {
      "default": {
        "expiry": "87600h"
      },
      "profiles": {
        "kubernetes": {
          "usages": [
              "signing",
              "key encipherment",
              "server auth",
              "client auth"
          ],
          "expiry": "87600h"
        }
      }
    }
  }
  EOF
  ```
  

signing ：表示该证书可用于签名其它证书，生成的 ca.pem 证书中CA=TRUE ；

server auth ：表示 client 可以用该该证书对 server 提供的证书进行验证；

client auth ：表示 server 可以用该该证书对 client 提供的证书进行验证；

- 创建证书签名请求文件

  ```
  cat > /data/cert/ca-csr.json <<EOF
  { 
    "CN": "kubernetes",
    "key": {
      "algo": "rsa",
      "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "ST": "BeiJing",
        "L": "BeiJing",
        "O": "k8s",
        "OU": "4Paradigm"
      }
    ]
  }
  EOF
  ```
  

① CN： Common Name ，kube-apiserver 从证书中提取该字段作为请求的用户名(User Name)，浏览器使用该字段验证网站是否合法；

② O： Organization ，kube-apiserver 从证书中提取该字段作为请求用户所属的组(Group)；

③ kube-apiserver 将提取的 User、Group 作为 RBAC 授权的用户标识；

- 生成CA证书和私钥

  ```
  cfssl gencert -initca /data/cert/ca-csr.json | cfssljson -bare ca
  ls ca*
  ```

- 分发证书

  将生成的 CA 证书、秘钥文件、配置文件拷贝到所有节点的/opt/k8s/cert 目录下：

  ```
  cat > /data/application/magic09_cert.sh << "EOF"
  #!/bin/bash
  # 分发证书文件
  source /opt/k8s/bin/environment.sh
  for node_ip in ${NODE_IPS[@]}
  do
      echo ">>> ${node_ip}"
      scp /data/cert/ca* k8s@${node_ip}:/etc/kubernetes/cert
  done
  EOF
  ```
  

## 三、部署kubectl命令行工具

kubectl 是 kubernetes 集群的命令行管理工具。

默认从 ~/.kube/config 文件读取 kube-apiserver 地址、证书、用户名等信息，如果没有配置，执行 kubectl 命令时可能会出错.

本文档只需要部署一次，生成的 kubeconfig 文件与机器无关。

### 1.下载kubectl 二进制文件

```
cd /data/application
wget https://dl.k8s.io/v1.10.4/kubernetes-client-linux-amd64.tar.gz
tar -xzvf kubernetes-client-linux-amd64.tar.gz
```

分发到所有使用 kubectl 的节点：

```
cd /opt/k8s/bin
chown k8s:root cfssl*


cat > magic10_cert_all_node.sh << "EOF"
#!/bin/bash
# 分发 kubectl 二进制文件
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    scp /data/application/kubernetes/client/bin/kubectl k8s@${node_ip}:/opt/k8s/bin/
    ssh k8s@${node_ip} "chmod +x /opt/k8s/bin/*"
done
EOF
```



### 2.创建admin证书和私钥

- kubectl 与 apiserver https 安全端口通信，apiserver 对提供的证书进行认证和授权。

- kubectl 作为集群的管理工具，需要被授予最高权限。这里创建具有最高权限的admin 证书。

  - 创建证书签名请求

    ```
    cd /data/cert
    cat > admin-csr.json <<EOF
    {
        "CN": "admin",
        "hosts": [],
        "key": {
            "algo": "rsa",
            "size": 2048
        },
        "names": [
            {
                "C": "CN",
                "ST": "BeiJing",
                "L": "BeiJing",
                "O": "system:masters",
                "OU": "4Paradigm"
            }
        ]
    }
    EOF
    ```

    ① O 为 system:masters ，kube-apiserver 收到该证书后将请求的 Group 设置为system:masters；

    ② 预定义的 ClusterRoleBinding cluster-admin 将 Group system:masters 与Role cluster-admin 绑定，该 Role 授予所有 API的权限；

    ③ 该证书只会被 kubectl 当做 client 证书使用，所以 hosts 字段为空；

  - 生成证书和私钥

    ```
    cfssl gencert -ca=/etc/kubernetes/cert/ca.pem \
    -ca-key=/etc/kubernetes/cert/ca-key.pem \
    -config=/etc/kubernetes/cert/ca-config.json \
    -profile=kubernetes admin-csr.json | cfssljson -bare admin
    
    ls admin*
    admin.csr admin-csr.json admin-key.pem admin.pem
    ```

  

  ### 3.创建和分发kubeconfig文件

  - 创建kubeconfig文件

    kubeconfig 为 kubectl 的配置文件，包含访问 apiserver 的所有信息，如 apiserver 地址、CA 证书和自身使用的证书；
    
    ​	--server=${KUBE_APISERVER} ，指定IP和端口；参考者使用的是haproxy的IP和端口；如果没有haproxy代理，就用实际服务的IP和端口；如：https://192.168.52.132:8080）
    
    ```
    #加载环境配置
    source /opt/k8s/bin/environment.sh
    
    mkdir -p /data/scriptable/
    cd /data/scriptable/
    
    # 设置集群参数
    kubectl config set-cluster kubernetes \
      --certificate-authority=/etc/kubernetes/cert/ca.pem \
      --embed-certs=true \
      --server=${KUBE_APISERVER} \
      --kubeconfig=kubectl.kubeconfig
    
    # 设置客户端认证参数
    kubectl config set-credentials admin \
      --client-certificate=/data/cert/admin.pem \
      --client-key=/data/cert/admin-key.pem \
      --embed-certs=true \
      --kubeconfig=kubectl.kubeconfig
    
    # 设置上下文参数
    kubectl config set-context kubernetes \
      --cluster=kubernetes \
      --user=admin \
      --kubeconfig=kubectl.kubeconfig
    
    # 设置默认上下文
    kubectl config use-context kubernetes --kubeconfig=kubectl.kubeconfig
    ```
    
    
    
    - --certificate-authority ：验证 kube-apiserver 证书的根证书；
    - --client-certificate 、 --client-key ：刚生成的 admin 证书和私钥，连接 kube-apiserver 时使用；
    - --embed-certs=true ：将 ca.pem 和 admin.pem 证书内容嵌入到生成的kubectl.kubeconfig 文件中(不加时，写入的是证书文件路径)
    
  - 分发kubeconfig证书
  
    分发到所有使用 kubectl 命令的节点：
  
    ```
    cd
    
    cat > magic11_distribute_kubeconfig.sh << "EOF"
    #!/bin/bash
    # 分发 kubeconfig 文件
    source /opt/k8s/bin/environment.sh
    for node_ip in ${NODE_IPS[@]}
    do
        echo ">>> ${node_ip}"
        ssh k8s@${node_ip} "mkdir -p ~/.kube"
        scp /data/scriptable/kubectl.kubeconfig k8s@${node_ip}:~/.kube/config
    done
    EOF
    
    chmod +x magic11_distribute_kubeconfig.sh
    ./magic11_distribute_kubeconfig.sh
    ```
  
    
  
  - 验证kubeconfig证书
  
    ```
     ls /home/k8s/.kube/config
    ```
  
    ```
    kubectl config view --kubeconfig=/home/k8s/.kube/config
    ```
  
    >
    >
    >```
    >apiVersion: v1
    >clusters:
    >- cluster:
    >    certificate-authority-data: REDACTED
    >    server: https://192.168.10.10:8443
    >  name: kubernetes
    >contexts:
    >- context:
    >    cluster: kubernetes
    >    user: kube-admin
    >  name: kube-admin@kubernetes
    >current-context: kube-admin@kubernetes
    >kind: Config
    >preferences: {}
    >users:
    >- name: kube-admin
    >  user:
    >    client-certificate-data: REDACTED
    >    client-key-data: REDACTED
    >```
  
  

## 四、部署etcd集群

etcd 是基于 Raft 的分布式 key-value 存储系统，由 CoreOS 开发，常用于服务发现、共享配置以及并发控制（如 leader 选举、分布式锁等）。kubernetes 使用 etcd 存储所有运行数据。

部署个三节点高可用 etcd 集群的步骤：

- 下载和分发 etcd 二进制文件
- 创建 etcd 集群各节点的 x509 证书，用于加密客户端(如 etcdctl) 与 etcd 集群、etcd 集群之间的数据流
- 创建 etcd 的 systemd unit 文件，配置服务参数；
- 检查集群工作状态；

### 1.下载和分发 etcd 二进制文件

到 https://github.com/coreos/etcd/releases 页面下载最新版本的发布包：

```
yum install openssl-devel
cd /data/application/
wget  https://github.com/coreos/etcd/releases/download/v3.3.7/etcd-v3.3.7-linux-amd64.tar.gz
tar -zxvf etcd-v3.3.7-linux-amd64.tar.gz
```

分发二进制文件,推送到所有节点里：

```
cd

cat > magic12_distribute_etcd.sh << "EOF"
#!/bin/bash
# 分发二进制文件
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    scp /data/application/etcd-v3.3.7-linux-amd64/etcd* root@${node_ip}:/opt/k8s/bin
    ssh root@${node_ip} "chmod +x /opt/k8s/bin/*"
done
EOF

chmod +x magic12_distribute_etcd.sh
./magic12_distribute_etcd.sh
```



### 2.创建etcd证书和私钥

创建证书签名请求

```
cat > /data/cert/etcd-csr.json <<EOF
{
  "CN": "etcd",
  "hosts": [
        "127.0.0.1",
        "192.168.52.132",
        "192.168.52.133",
        "192.168.52.134"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "ST": "BeiJing",
            "L": "BeiJing",
            "O": "k8s",
            "OU": "4Paradigm"
        }
    ]
}
EOF
```

hosts 字段指定授权使用该证书的 etcd 节点 IP 或域名列表，这里将 etcd 集群的三个节点 IP 都列在其中；

### 3.生成证书和私钥

```
cd /data/cert

cfssl gencert -ca=/etc/kubernetes/cert/ca.pem \
    -ca-key=/etc/kubernetes/cert/ca-key.pem \
    -config=/etc/kubernetes/cert/ca-config.json \
    -profile=kubernetes etcd-csr.json | cfssljson -bare etcd

[root@kube-master cert]# ls etcd*
etcd.csr etcd-csr.json etcd-key.pem etcd.pem
```

### 4.分发证书和私钥到各个etcd节点

```
cd

cat > magic13_distribute_certndoe.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}"
    ssh root@${node_ip} "mkdir -p /etc/etcd/cert && chown -R k8s /etc/etcd/cert"
    scp /data/cert/etcd*.pem k8s@${node_ip}:/etc/etcd/cert/
    scp /data/cert/ca.pem k8s@${node_ip}:/etc/etcd/cert/
done
EOF

chmod +x magic13_distribute_certndoe.sh
./magic13_distribute_certndoe.sh
```

### 5.创建etcd的systemd unit模板及etcd配置文件

- 创建etcd的systemd unit 模板

  ```
  source /opt/k8s/bin/environment.sh
  mkdir /data/template
  cd /data/template
  
  cat > etcd.service.template <<EOF
  [Unit]
  Description=Etcd Server
  After=network.target
  After=network-online.target
  Wants=network-online.target
  Documentation=https://github.com/coreos
  
  [Service]
  User=k8s
  Type=notify
  WorkingDirectory=/var/lib/etcd/
  ExecStart=/opt/k8s/bin/etcd \
    --data-dir=/var/lib/etcd \
    --name=##NODE_NAME## \
    --cert-file=/etc/etcd/cert/etcd.pem \
    --key-file=/etc/etcd/cert/etcd-key.pem \
    --trusted-ca-file=/etc/kubernetes/cert/ca.pem \
    --peer-cert-file=/etc/etcd/cert/etcd.pem \
    --peer-key-file=/etc/etcd/cert/etcd-key.pem \
    --peer-trusted-ca-file=/etc/kubernetes/cert/ca.pem \
    --peer-client-cert-auth \
    --client-cert-auth \
    --listen-peer-urls=https://##NODE_IP##:2380 \
    --initial-advertise-peer-urls=https://##NODE_IP##:2380 \
    --listen-client-urls=https://##NODE_IP##:2379,http://127.0.0.1:2379 \
    --advertise-client-urls=https://##NODE_IP##:2379,http://127.0.0.1:2379 \
    --initial-cluster-token=etcd-cluster-0 \
    --initial-cluster=master=https://192.168.52.132:2380,node01=https://192.168.52.133:2380,node02=https://192.168.52.134:2380 \
    --initial-cluster-state=new
  Restart=on-failure
  RestartSec=5
  LimitNOFILE=65536
  
  [Install]
  WantedBy=multi-user.target
  EOF
  ```

  - User ：指定以 k8s 账户运行；
  - WorkingDirectory 、 --data-dir ：指定工作目录和数据目录为/opt/lib/etcd ，需在启动服务前创建这个目录；
  - --name ：指定节点名称，当 --initial-cluster-state 值为 new 时， --name 的参数值必须位于 --initial-cluster 列表中；
  - --cert-file 、 --key-file ：etcd server 与 client 通信时使用的证书和私钥；
  - --trusted-ca-file ：签名 client 证书的 CA 证书，用于验证 client 证书；
  - --peer-cert-file 、 --peer-key-file ：etcd 与 peer 通信使用的证书和私钥；
  - --peer-trusted-ca-file ：签名 peer 证书的 CA 证书，用于验证 peer 证书；

- 为各个节点创建和分发etcd systemd unit文件

  ```
  cd
  
  cat > magic14_init_NodeCluster.sh << "EOF"
  #!/bin/bash
  NODE_NAMES=("master" "node01" "node02")
  NODE_IPS=("192.168.52.132" "192.168.52.133" "192.168.52.134")
  #替换模板文件中的变量，为各节点创建 systemd unit 文件
  for (( i=0; i < 3; i++ ));do
          sed -e "s/##NODE_NAME##/${NODE_NAMES[i]}/g" -e "s/##NODE_IP##/${NODE_IPS[i]}/g" /data/template/etcd.service.template > /data/service/etcd-${NODE_IPS[i]}.service
  done
  EOF
  
  chmod +x magic14_init_NodeCluster.sh
  ./magic14_init_NodeCluster.sh
  ```
  
  - NODE_NAMES 和 NODE_IPS 为相同长度的 bash 数组，分别为节点名称和对应的 IP；
  
  ### 分发文件：
  
  ```
  cat > magic15_distribute_SystemdUnit.sh << "EOF"
  #!/bin/bash
  source /opt/k8s/bin/environment.sh
  for node_ip in ${NODE_IPS[@]}
  do
      echo ">>> ${node_ip}"
      ssh root@${node_ip} "mkdir -p /var/lib/etcd && chown -R k8s /var/lib/etcd" 
      scp /data/service/etcd-${node_ip}.service root@${node_ip}:/etc/systemd/system/etcd.service
  done
  EOF
  
  chmod +x magic15_distribute_SystemdUnit.sh
  ./magic15_distribute_SystemdUnit.sh
  ```
  
  
  
  ### 6.启动etcd服务
  
  ```
  cat > magic16_start_etcd_Service.sh << "EOF" 
  #!/bin/bash
  # 启动 etcd 服务
  source /opt/k8s/bin/environment.sh
  for node_ip in ${NODE_IPS[@]}
  do
      echo ">>> ${node_ip}" 
      ssh root@${node_ip} "systemctl daemon-reload && systemctl enable etcd && cd &"
  done
  EOF
  
  chmod +x magic16_start_etcd_Service.sh
  ./magic16_start_etcd_Service.sh
  ```

- etcd 进程首次启动时会等待其它节点的 etcd 加入集群，命令 systemctl start etcd 会卡住一段时间，为正常现象

### 8.检测启动结果

```
cat > magic17_check_etcd_Service.sh << "EOF" 
#!/bin/bash
# 检查启动结果
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh k8s@${node_ip} "systemctl status etcd|grep Active"
done
EOF

chmod +x magic17_check_etcd_Service.sh
./magic17_check_etcd_Service.sh
```

报错查看日志，确认原因：journalctl -xu etcd

日志路径：/var/log/message

### 9。验证服务状态

部署完 etcd 集群后，在任一个etc节点上执行如下命令：

```
cd
cat > magic18_verification_service.sh << "EOF"
#!/bin/bash
# 验证服务状态
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ETCDCTL_API=3 /opt/k8s/bin/etcdctl \
    --endpoints=https://${node_ip}:2379 \
    --cacert=/etc/kubernetes/cert/ca.pem \
    --cert=/etc/etcd/cert/etcd.pem \
    --key=/etc/etcd/cert/etcd-key.pem endpoint health
done
EOF

chmod +x magic18_verification_service.sh
./magic18_verification_service.sh
```



## 五、部署flannel网络

kubernetes 要求集群内各节点(包括 master 节点)能通过 Pod 网段互联互通。flannel 使用 vxlan 技术为各节点创建一个可以互通的 Pod 网络，使用的端口为 UDP 8472，需要开放该端口（如公有云 AWS 等）。

flannel 第一次启动时，从 etcd 获取 Pod 网段信息，为本节点分配一个未使用的 /24段地址，然后创建 flannel.1 （也可能是其它名称，如 flannel1 等） 接口。

flannel 将分配的 Pod 网段信息写入 /run/flannel/docker 文件，docker 后续使用这个文件中的环境变量设置 docker0 网桥。

### 1.下载flanneld二进制文件

到 https://github.com/coreos/flannel/releases 页面下载最新版本的发布包：

```
mkdir /data/flannel && cd /data/flannel
wget https://github.com/coreos/flannel/releases/download/v0.10.0/flannel-v0.10.0-linux-amd64.tar.gz
tar -xzvf flannel-v0.10.0-linux-amd64.tar.gz -C /data/flannel
```

分发 flanneld 二进制文件到集群所有节点：

```
cat > magic19_distribute_flanneld.sh << "EOF"
#!/bin/bash
# 分发 flanneld 二进制文件到集群所有节点
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp  /data/flannel/{flanneld,mk-docker-opts.sh} k8s@${node_ip}:/opt/k8s/bin/
    ssh root@${node_ip} "chmod +x /opt/k8s/bin/*"
done
EOF

chmod +x magic19_distribute_flanneld.sh
./magic19_distribute_flanneld.sh
```



### 2.创建flanne证书和私钥

flannel 从 etcd 集群存取网段分配信息，而 etcd 集群启用了双向 x509 证书认证，所以需要为 flanneld 生成证书和私钥。

- 创建证书签名请求

  ```
  cd /data/cert
  
  cat > flanneld-csr.json <<EOF
  {
    "CN": "flanneld",
    "hosts": [],
    "key": {
      "algo": "rsa",
      "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "ST": "BeiJing",
        "L": "BeiJing",
        "O": "k8s",
        "OU": "4Paradigm"
      }
    ]
  }
  EOF
  ```

  该证书只会被 kubectl 当做 client 证书使用，所以 hosts 字段为空；

- 生成证书和私钥

  ```
  cfssl gencert -ca=/etc/kubernetes/cert/ca.pem \
    -ca-key=/etc/kubernetes/cert/ca-key.pem \
    -config=/etc/kubernetes/cert/ca-config.json \
    -profile=kubernetes flanneld-csr.json | cfssljson -bare flanneld
  ```

- 将flanneld二进制文件和证书、私钥分发到所有节点   

  ```
  cd
  
  cat > magic20_distribute_cert_allnode.sh << "EOF"
  #!/bin/bash
  # 将生成的证书和私钥分发到所有节点（master 和 worker）
  source /opt/k8s/bin/environment.sh
  for node_ip in ${NODE_IPS[@]}
  do
      echo ">>> ${node_ip}" 
      ssh root@${node_ip} "mkdir -p /etc/flanneld/cert && chown -R k8s /etc/flanneld"
      scp /data/cert/flanneld*.pem k8s@${node_ip}:/etc/flanneld/cert
  done
  EOF
  
  chmod +x magic20_distribute_cert_allnode.sh
  ./magic20_distribute_cert_allnode.sh
  ```

### 3.向etcd写入集群pod网段信息

本步骤只需执行一次。

```
source /opt/k8s/bin/environment.sh

etcdctl \
  --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/cert/ca.pem \
  --cert-file=/etc/flanneld/cert/flanneld.pem \
  --key-file=/etc/flanneld/cert/flanneld-key.pem \
  set ${FLANNEL_ETCD_PREFIX}/config '{"Network":"'${CLUSTER_CIDR}'", "SubnetLen": 24, "Backend": {"Type": "vxlan"}}'
```

- flanneld 当前版本 (v0.10.0) 不支持 etcd v3，故使用 etcd v2 API 写入配置 key 和网段数据；
- 写入的 Pod 网段 "Network" 必须是 /16 段地址，必须与kube-controller-manager 的 --cluster-cidr 参数值一致；

### 4.创建flanneld的systemd unit文件

```
source /opt/k8s/bin/environment.sh

cat > /data/template/flanneld.service << EOF
[Unit]
Description=Flanneld overlay address etcd agent
After=network.target
After=network-online.target
Wants=network-online.target
After=etcd.service
Before=docker.service

[Service]
Type=notify
ExecStart=/opt/k8s/bin/flanneld \\
  -etcd-cafile=/etc/kubernetes/cert/ca.pem \\
  -etcd-certfile=/etc/flanneld/cert/flanneld.pem \\
  -etcd-keyfile=/etc/flanneld/cert/flanneld-key.pem \\
  -etcd-endpoints=${ETCD_ENDPOINTS} \\
  -etcd-prefix=${FLANNEL_ETCD_PREFIX} \\
  -iface=${VIP_IF}
ExecStartPost=/opt/k8s/bin/mk-docker-opts.sh -k DOCKER_NETWORK_OPTIONS -d /run/flannel/docker
Restart=on-failure

[Install]
WantedBy=multi-user.target
RequiredBy=docker.service
EOF
```

- mk-docker-opts.sh 脚本将分配给 flanneld 的 Pod 子网网段信息写入/run/flannel/docker 文件，后续 docker 启动时使用这个文件中的环境变量配置 docker0 网桥；
- flanneld 使用系统缺省路由所在的接口与其它节点通信，对于有多个网络接口（如内网和公网）的节点，可以用 -iface 参数指定通信接口，如上面的 eth1 接口;
- flanneld 运行时需要 root 权限；

### 5.分发flanneld systemd unit 文件

```
cd

cat > magic21_distribute_flanneld_systemd_unit.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/template/flanneld.service root@${node_ip}:/etc/systemd/system/
done
EOF

chmod +x magic21_distribute_flanneld_systemd_unit.sh
./magic21_distribute_flanneld_systemd_unit.sh
```



### 6.启动flanneld服务

```
cd

cat > magic22_start_flanneld_service.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "systemctl daemon-reload && systemctl enable flanneld && systemctl start flanneld"
done
EOF

chmod +x magic22_start_flanneld_service.sh 
./magic22_start_flanneld_service.sh 
```

注：确保状态为 active (running) ，否则查看日志，确认原因：

$ journalctl -u flanneld

### 7.检查启动结果

```
cd

cat > magic23_verification_flanneld.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh k8s@${node_ip} "systemctl status flanneld|grep Active"
done
EOF

chmod +x magic23_verification_flanneld.sh
./magic23_verification_flanneld.sh
```

失败，则用如下命令查看日志：journalctl -ux flanneld

### 8.检查分配给各flanneld的pod网段信息

查看集群 Pod 网段 (/16)：

```
source /opt/k8s/bin/environment.sh

etcdctl \
  --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/cert/ca.pem \
  --cert-file=/etc/flanneld/cert/flanneld.pem \
  --key-file=/etc/flanneld/cert/flanneld-key.pem \
  get ${FLANNEL_ETCD_PREFIX}/config
```

查看已分配的 Pod 子网段列表 (/24):

```
source /opt/k8s/bin/environment.sh

etcdctl \
  --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/cert/ca.pem \
  --cert-file=/etc/flanneld/cert/flanneld.pem \
  --key-file=/etc/flanneld/cert/flanneld-key.pem \
  ls ${FLANNEL_ETCD_PREFIX}/subnets
  
/kubernetes/network/subnets/172.30.36.0-24
/kubernetes/network/subnets/172.30.13.0-24
/kubernetes/network/subnets/172.30.16.0-24
```

查看某一个 Pod 网段对应的节点 IP 和 flannel 接口地址，注意：其中的IP段换成自己的：

```
source /opt/k8s/bin/environment.sh

etcdctl \
  --endpoints=${ETCD_ENDPOINTS} \
  --ca-file=/etc/kubernetes/cert/ca.pem \
  --cert-file=/etc/flanneld/cert/flanneld.pem \
  --key-file=/etc/flanneld/cert/flanneld-key.pem \
  get ${FLANNEL_ETCD_PREFIX}/subnets/172.30.13.0-24
```

### 9.验证各个节点能通过pod网段互通

在各节点上部署 flannel 后，检查是否创建了 flannel 接口 (名称可能为 flannel0、flannel.0、flannel.1 等)：

```
cd

cat > magic24_verification_interflow_pod.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh ${node_ip} "/usr/sbin/ip addr show flannel.1|grep -w inet"
done
EOF

chmod +x magic24_verification_interflow_pod.sh
./magic24_verification_interflow_pod.sh
>>> 172.68.96.101
    inet 172.30.70.0/32 scope global flannel.1
>>> 172.68.96.102
    inet 172.30.35.0/32 scope global flannel.1
>>> 172.68.96.103
    inet 172.30.58.0/32 scope global flannel.1
```

在各节点上 ping 所有 flannel 接口 IP，确保能通：
注意：其中的IP段换成自己的

```
cd

cat > magic25_check_flannel_IP.sh << "EOF"
#!/bin/bash
# 检查各节点上是否能ping通，所有flannel接口IP
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh ${node_ip} "ping -c 1 172.30.36.0"
    ssh ${node_ip} "ping -c 1 172.30.13.0"
    ssh ${node_ip} "ping -c 1 172.30.16.0"
done
EOF

chmod +x magic25_check_flannel_IP.sh
./magic25_check_flannel_IP.sh
```



## 六、部署master节点

kubernetes master 节点运行如下组件：

- kube-apiserver
- kube-scheduler
- kube-controller-manager

kube-scheduler 和 kube-controller-manager 可以以集群模式运行，通过 leader 选举产生一个工作进程，其它进程处于阻塞模式。对于 kube-apiserver，可以运行多个实例，但对其它组件需要提供统一的访问地址，该地址需要高可用。本文档使用 keepalived 和 haproxy 实现 kube-apiserver VIP 高可用和负载均衡。

### 1.下载最新版本的二进制文件

```
wget https://dl.k8s.io/v1.10.4/kubernetes-server-linux-amd64.tar.gz
tar -xzvf kubernetes-server-linux-amd64.tar.gz
cd kubernetes
tar -xzvf  kubernetes-src.tar.gz
```

将二进制文件拷贝到所有 master 节点：

```
cd

cat > magic26_copy_all_node.sh << "EOF"
#!/bin/bash
# 将二进制文件拷贝到所有节点上
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/application/kubernetes/server/bin/* k8s@${node_ip}:/opt/k8s/bin/
    ssh root@${node_ip} "chmod +x /opt/k8s/bin/*"
done
EOF

chmod +x magic26_copy_all_node.sh
./magic26_copy_all_node.sh
```



## 七、部署高可用组件

keepalived 和 haproxy 实现 kube-apiserver 高可用的步骤：

- keepalived 提供 kube-apiserver 对外服务的 VIP；
- haproxy 监听 VIP，后端连接所有 kube-apiserver 实例，提供健康检查和负载均衡功能；

运行 keepalived 和 haproxy 的节点称为 LB 节点。由于 keepalived 是一主多备运行模式，故至少两个 LB 节点。

复用 master 节点的三台机器，haproxy 监听的端口 (8443) 需要与 kube-apiserver 的端口 6443 不同，避免冲突。

keepalived 在运行过程中周期检查本机的 haproxy 进程状态，如果检测到 haproxy 进程异常，则触发重新选主的过程，VIP 将飘移到新选出来的主节点，从而实现 VIP 的高可用。

所有组件（如 kubeclt、apiserver、controller-manager、scheduler 等）都通过 VIP 和 haproxy 监听的 8443 端口访问 kube-apiserver 服务。

### 1.安装软件包

```
cd

cat > magic27__install_package.sh << "EOF"
#!/bin/bash
# 安装软件包
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "yum install -y keepalived haproxy"
done
EOF

chmod +x magic27__install_package.sh
./magic27__install_package.sh
```

### 2.配置和下发haproxy配置文件

haproxy 配置文件：

```
cd /data/template/

cat > haproxy.cfg <<EOF
global
    log /dev/log local0
    log /dev/log local1 notice
    chroot /var/lib/haproxy
    stats socket /var/run/haproxy-admin.sock mode 660 level admin
    stats timeout 30s
    user haproxy
    group haproxy
    daemon
    nbproc 1
defaults
    log global
    timeout connect 5000
    timeout client 10m
    timeout server 10m
listen admin_stats
    bind 0.0.0.0:10080
    mode http
    log 127.0.0.1 local0 err
    stats refresh 30s
    stats uri /status
    stats realm welcome login\ Haproxy
    stats auth kubernetes.confadmin:123456
    stats hide-version
    stats admin if TRUE
listen kube-master
    bind 0.0.0.0:8443
    mode tcp
    option tcplog
    balance source
    server 192.168.52.133 192.168.52.133:6443 check inter 2000 fall 2 rise 2 weight 1
    server 192.168.52.134 192.168.52.134:6443 check inter 2000 fall 2 rise 2 weight 1
EOF
```

- haproxy 在 10080 端口输出 status 信息；

- haproxy 监听所有接口的 8443 端口，该端口与环境变量 ${KUBE_APISERVER} 指定的端口必须一致；

- server 字段列出所有 kube-apiserver 监听的 IP 和端口

分发 haproxy.cfg 到所有集群节点上：

```
cd

cat > magic28_distribute_haproxy.sh << "EOF"
#!/bin/bash
# 分发 haproxy.cfg 到所有集群节点上
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/template/haproxy.cfg root@${node_ip}:/etc/haproxy
done
EOF

chmod +x magic28_distribute_haproxy.sh
./magic28_distribute_haproxy.sh
```

### 3.启动haproxy服务

```
cd

cat > magic29_start_haproxy.sh << "EOF"
#!/bin/bash
# 启动 haproxy 服务
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "systemctl restart haproxy"
done
EOF

chmod +x magic29_start_haproxy.sh
./magic29_start_haproxy.sh
```

### 4.检测haproxy服务状态

```
cd

cat > magic30_check_haproxy_service.sh << "EOF"
#!/bin/bash
# 检查 haproxy 服务状态
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "systemctl status haproxy|grep Active"
done
EOF

chmod +x magic30_check_haproxy_service.sh
./magic30_check_haproxy_service.sh
```

如果失败，用如下命令检查：

```
journalctl -xu haproxy
```

检查 haproxy 是否监听 8443 端口：

```
cd

cat > magic31_check_haproxy_proxy8443.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "yum install -y net-tools"
    ssh root@${node_ip} "netstat -lnpt|grep haproxy"
done
EOF

chmod +x magic31_check_haproxy_proxy8443.sh
./magic31_check_haproxy_proxy8443.sh
```

### 5.配置和下发 keepalived 配置文件

keepalived 是一主（master）多备（backup）运行模式，故有两种类型的配置文件。master 配置文件只有一份，backup 配置文件视节点数目而定，对于本文档而言，规划如下：

- master: 192.168.52.132
- backup：192.168.52.133，192.168.52.134

master配置文件：

```
source /opt/k8s/bin/environment.sh

cat  > /data/template/keepalived-master.conf <<EOF
global_defs {
    router_id lb-master-132
}
vrrp_script check-haproxy {
    script "killall -0 haproxy"
    interval 5
    weight -30
}
vrrp_instance VI-kube-master {
    state MASTER
    priority 120
    dont_track_primary
    interface ${VIP_IF}
    virtual_router_id 68
    advert_int 3
    track_script {
        check-haproxy
    }
    virtual_ipaddress {
        ${MASTER_VIP}
    }
}
EOF
```

- VIP 所在的接口（interface ${VIP_IF}）为 ens33；

- 使用 killall -0 haproxy 命令检查所在节点的 haproxy 进程是否正常。如果异常则将权重减少（-30）, 从而触发重新选主过程；

- router_id、virtual_router_id 用于标识属于该 HA 的 keepalived 实例，如果有多套 keepalived HA，则必须各不相同；

backup配置文件：

```
source /opt/k8s/bin/environment.sh

cat  > /data/template/keepalived-backup.conf <<EOF
global_defs {
    router_id lb-backup-132
}
vrrp_script check-haproxy {
    script "killall -0 haproxy"
    interval 5
    weight -30
}
vrrp_instance VI-kube-master {
    state BACKUP
    priority 110
    dont_track_primary
    interface ${VIP_IF}
    virtual_router_id 68
    advert_int 3
    track_script {
        check-haproxy
    }
    virtual_ipaddress {
        ${MASTER_VIP}
    }
}
EOF
```

- VIP 所在的接口（interface ${VIP_IF}）为 ens33；

- 使用 killall -0 haproxy 命令检查所在节点的 haproxy 进程是否正常。如果异常则将权重减少（-30）, 从而触发重新选主过程；

- router_id、virtual_router_id 用于标识属于该 HA 的 keepalived 实例，如果有多套 keepalived HA，则必须各不相同；

- priority 的值必须小于 master 的值；

### 6.分发keepalived配置文件

下发master配置文件：

```
scp /data/template/keepalived-master.conf root@master:/etc/keepalived/keepalived.conf
```

下发backup配置文件：

```
scp /data/template/keepalived-backup.conf root@node01:/etc/keepalived/keepalived.conf
scp /data/template/keepalived-backup.conf root@node02:/etc/keepalived/keepalived.conf
```

### 7.启动keepalived服务

```
cd 

cat > magic32_start_keepalived_service.sh << "EOF"
#!/bin/bash
# 启动 keepalived 服务
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "systemctl restart keepalived"
done
EOF

chmod +x magic32_start_keepalived_service.sh
./magic32_start_keepalived_service.sh
```

### 8.检测keepalived服务

```
cd 

cat > magic33_check_keepalived_service.sh << "EOF"
#!/bin/bash
# 检查 keepalived 服务
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "systemctl status keepalived|grep Active"
done
EOF

chmod +x magic33_check_keepalived_service.sh
./magic33_check_keepalived_service.sh
```

如果失败，则检查日志:journalctl -xu keepalived

查看 VIP 所在的节点，确保可以 ping 通 VIP：

```
cd

cat > magic34_ping_keepalived_service.sh << "EOF"
#!/bin/bash
#  查看VIP所在的节点，确保可以ping通VIP
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh ${node_ip} "/usr/sbin/ip addr show ${VIP_IF}"
    ssh ${node_ip} "ping -c 1 ${MASTER_VIP}"
done
EOF

chmod +x magic34_ping_keepalived_service.sh
./magic34_ping_keepalived_service.sh
```

### 9.查看haproxy状态页面

浏览器访问 ${MASTER_VIP}:10080/status 地址，查看 haproxy 状态页面：
用户名密码就在刚刚定义的 haproxy 的配置当中（admin:123456）。

## 八、部署kube-apiserver

使用 keepalived 和 haproxy 部署一个 3 节点高可用 master 集群的步骤，对应的 LB VIP 为环境变量 ${MASTER_VIP}。配置之前需要先安装 kubelet,flannel 等组件。

### 1.创建kubernetes证书和私钥

```
source /opt/k8s/bin/environment.sh
cd /data/cert

cat > kubernetes-csr.json <<EOF
{
  "CN": "kubernetes",
  "hosts": [
    "127.0.0.1",
    "192.168.52.132",
    "192.168.52.133",
    "192.168.52.134",
    "${MASTER_VIP}",
    "${CLUSTER_KUBERNETES_SVC_IP}",
    "kubernetes",
    "kubernetes.default",
    "kubernetes.default.svc",
    "kubernetes.default.svc.cluster",
    "kubernetes.default.svc.cluster.local"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "ST": "BeiJing",
      "L": "BeiJing",
      "O": "k8s",
      "OU": "4Paradigm"
    }
  ]
}
EOF
```

- hosts 字段指定授权使用该证书的 IP 或域名列表，这里列出了 VIP 、apiserver 节点 IP、kubernetes 服务 IP 和域名；

- 域名最后字符不能是 .(如不能为 kubernetes.default.svc.cluster.local.)，否则解析时失败，提示： x509: cannot parse dnsName “kubernetes.default.svc.cluster.local.”；

- 如果使用非 cluster.local 域名，如 opsnull.com，则需要修改域名列表中的最后两个域名为：kubernetes.default.svc.opsnull、kubernetes.default.svc.opsnull.com

- kubernetes 服务 IP 是 apiserver 自动创建的，一般是 --service-cluster-ip-range 参数指定的网段的第一个 IP，后续可以通过如下命令获取，现在还不能进行这样的操作，继续往下配置。

```
kubectl get svc kubernetes
NAME         CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   10.254.0.1   <none>        443/TCP   1d
```

生成证书和私钥：

```
cd /data/cert

cfssl gencert -ca=/etc/kubernetes/cert/ca.pem \
  -ca-key=/etc/kubernetes/cert/ca-key.pem \
  -config=/etc/kubernetes/cert/ca-config.json \
  -profile=kubernetes kubernetes-csr.json | cfssljson -bare kubernetes
  
ls kubernetes*pem
```

将生成的证书和私钥文件拷贝到 master 节点：

```
cat > /data/scriptable/magic35_distribute_cert_All_service.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "mkdir -p /etc/kubernetes/cert/ && sudo chown -R k8s /etc/kubernetes/cert/"
    scp /data/cert/kubernetes*.pem k8s@${node_ip}:/etc/kubernetes/cert/
done
EOF

cd /data/scriptable/
chmod +x magic35_distribute_cert_All_service.sh
./magic35_distribute_cert_All_service.sh
```

- k8s 账户可以读写 /etc/kubernetes/cert/ 目录；

### 2.创建加密配置文件

```
source /opt/k8s/bin/environment.sh
cd /data/template

cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

将加密配置文件拷贝到 master 节点的 /etc/kubernetes 目录下：

```
cd

cat >magic36_copy_All_service.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/template/encryption-config.yaml root@${node_ip}:/etc/kubernetes/
done
EOF

chmod +x magic36_copy_All_service.sh
./magic36_copy_All_service.sh
```

### 3.创建kube-apiserver systemd unit模板文件

```
source /opt/k8s/bin/environment.sh

cat > /data/template/kube-apiserver.service.template <<EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.target
[Service]
ExecStart=/opt/k8s/bin/kube-apiserver \\
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \\
  --anonymous-auth=false \\
  --experimental-encryption-provider-config=/etc/kubernetes/encryption-config.yaml \\
  --advertise-address=##NODE_IP## \\
  --bind-address=##NODE_IP## \\
  --insecure-port=0 \\
  --authorization-mode=Node,RBAC \\
  --runtime-config=api/all \\
  --enable-bootstrap-token-auth=true \\
  --service-cluster-ip-range=${SERVICE_CIDR} \\
  --service-node-port-range=${NODE_PORT_RANGE} \\
  --tls-cert-file=/etc/kubernetes/cert/kubernetes.pem \\
  --tls-private-key-file=/etc/kubernetes/cert/kubernetes-key.pem \\
  --client-ca-file=/etc/kubernetes/cert/ca.pem \\
  --kubelet-client-certificate=/etc/kubernetes/cert/kubernetes.pem \\
  --kubelet-client-key=/etc/kubernetes/cert/kubernetes-key.pem \\
  --service-account-key-file=/etc/kubernetes/cert/ca-key.pem \\
  --etcd-cafile=/etc/kubernetes/cert/ca.pem \\
  --etcd-certfile=/etc/kubernetes/cert/kubernetes.pem \\
  --etcd-keyfile=/etc/kubernetes/cert/kubernetes-key.pem \\
  --etcd-servers=${ETCD_ENDPOINTS} \\
  --enable-swagger-ui=true \\
  --allow-privileged=true \\
  --apiserver-count=3 \\
  --audit-log-maxage=30 \\
  --audit-log-maxbackup=3 \\
  --audit-log-maxsize=100 \\
  --audit-log-path=/var/log/kube-apiserver-audit.log \\
  --event-ttl=1h \\
  --alsologtostderr=true \\
  --logtostderr=false \\
  --log-dir=/var/log/kubernetes \\
  --v=2
Restart=on-failure
RestartSec=5
Type=notify
User=k8s
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

- --experimental-encryption-provider-config：启用加密特性；

- --authorization-mode=Node,RBAC： 开启 Node 和 RBAC 授权模式，拒绝未授权的请求；

- --enable-admission-plugins：启用 ServiceAccount 和 NodeRestriction；

- --service-account-key-file：签名 ServiceAccount Token 的公钥文件，kube-controller-manager 的 --service-account-private-key-file 指定私钥文件，两者配对使用；

- --tls-*-file：指定 apiserver 使用的证书、私钥和 CA 文件。--client-ca-file 用于验证 client (kue-controller-manager、kube-scheduler、kubelet、kube-proxy 等) 请求所带的证书；

- --kubelet-client-certificate、--kubelet-client-key：如果指定，则使用 https 访问 kubelet APIs；需要为证书对应的用户 (上面 kubernetes*.pem 证书的用户为 kubernetes) 用户定义 RBAC 规则，否则访问 kubelet API 时提示未授权；

- --bind-address： 不能为 127.0.0.1，否则外界不能访问它的安全端口 6443；

- --insecure-port=0：关闭监听非安全端口 (8080)；

- --service-cluster-ip-range： 指定 Service Cluster IP 地址段；

- --service-node-port-range： 指定 NodePort 的端口范围；

- --runtime-config=api/all=true： 启用所有版本的 APIs，如 autoscaling/v2alpha1；

- --enable-bootstrap-token-auth：启用 kubelet bootstrap 的 token 认证；

- --apiserver-count=3：指定集群运行模式，多台 kube-apiserver 会通过 leader 选举产生一个工作节点，其它节点处于阻塞状态；

- User=k8s：使用 k8s 账户运行；

### 4.为各节点创建和分发kube-apiserver systemd unit文件

替换模板文件中的变量，为各节点创建 systemd unit 文件：

```
cd

cat > magic37.sh << "EOF"
#!/bin/bash
# 替换模板文件中的变量，为各节点创建 systemd unit 文件
source /opt/k8s/bin/environment.sh
for (( i=0; i < 3; i++ ))
do
    sed -e "s/##NODE_NAME##/${NODE_NAMES[i]}/" -e "s/##NODE_IP##/${NODE_IPS[i]}/" /data/template/kube-apiserver.service.template > /data/template/kube-apiserver-${NODE_IPS[i]}.service 
done
EOF

chmod +x magic37.sh
./magic37.sh
```

- NODE_NAMES 和 NODE_IPS 为相同长度的 bash 数组，分别为节点名称和对应的 IP；

分发生成的 systemd unit 文件：

```
cd

cat > magic38_distribute_systemd_unit.sh << "EOF"
#!/bin/bash
# 分发生成的 systemd unit 文件
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "mkdir -p /var/log/kubernetes && chown -R k8s /var/log/kubernetes"
    scp /data/template/kube-apiserver-${node_ip}.service root@${node_ip}:/etc/systemd/system/kube-apiserver.service
done
EOF

chmod +x magic38_distribute_systemd_unit.sh
./magic38_distribute_systemd_unit.sh
```

- 必须先创建日志目录；
- 文件重命名为 kube-apiserver.service;

### 5.启动kube-apiserver服务

```
cd

cat > magic39_start_kubeApiserver.sh << "EOF"
#!/bin/bash
# 启动 kube-apiserver 服务
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "systemctl daemon-reload && systemctl enable kube-apiserver && systemctl start kube-apiserver"
done
EOF

chmod +x magic39_start_kubeApiserver.sh
./magic39_start_kubeApiserver.sh
```

检测kube-apiserver状态：

```
cd

cat > magic40_ckeck_kubeApiserver.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "systemctl status kube-apiserver |grep 'Active:'"
done
EOF

chmod +x magic40_ckeck_kubeApiserver.sh
./magic40_ckeck_kubeApiserver.sh
```

失败检测日志：journalctl -xu kube-apiserver

### 6.打印kube-apiserver写入etcd的数据

```
source /opt/k8s/bin/environment.sh

ETCDCTL_API=3 etcdctl \
    --endpoints=${ETCD_ENDPOINTS} \
    --cacert=/etc/kubernetes/cert/ca.pem \
    --cert=/etc/etcd/cert/etcd.pem \
    --key=/etc/etcd/cert/etcd-key.pem \
    get /registry/ --prefix --keys-only
```

### 7.检测集群信息

```
kubectl cluster-info

kubectl get all --all-namespaces

kubectl get componentstatuses
```

注意：

1）如果执行 kubectl 命令式时输出如下错误信息，则说明使用的 ~/.kube/config 文件不对，请切换到正确的账户后再执行该命令： The connection to the server localhost:8080 was refused – did you specify the right host or port?

2）执行 kubectl get componentstatuses 命令时，apiserver 默认向 127.0.0.1 发送请求。当 controller-manager、scheduler 以集群模式运行时，有可能和 kube-apiserver 不在一台机器上，这时 controller-manager 或 scheduler 的状态为 Unhealthy，但实际上它们工作正常。

### 8.检查kube-apiserver监听端口

```
sudo netstat -lnpt|grep kube
```

- 6443: 接收 https 请求的安全端口，对所有请求做认证和授权；
- 由于关闭了非安全端口，故没有监听 8080；

### 9.授予kubernetes证书访问kubelet api权限

在执行 kubectl exec、run、logs 等命令时，apiserver 会转发到 kubelet。这里定义 RBAC 规则，授权 apiserver 调用 kubelet API：

```
kubectl create clusterrolebinding kube-apiserver:kubelet-apis --clusterrole=system:kubelet-api-admin --user kubernetes
```

### 10.参考资料

关于证书域名最后字符不能是 . 的问题，实际和 Go 的版本有关，1.9 不支持这种类型的证书：[https://github.com/kubernetes/ingress-nginx/issues/2188



## 九、部署kube-controller-maneger

该集群包含 3 个节点，启动后将通过竞争选举机制产生一个 leader 节点，其它节点为阻塞状态。当 leader 节点不可用后，剩余节点将再次进行选举产生新的 leader 节点，从而保证服务的可用性。

为保证通信安全，本文档先生成 x509 证书和私钥，kube-controller-manager 在如下两种情况下使用该证书：

1）与 kube-apiserver 的安全端口通信时;
 2）在安全端口 (https，10252) 输出 prometheus 格式的 metrics；
 配置之前需要先安装 kubelet,flannel 等组件。

### 1.创建kube-controller-manager证书和私钥

创建证书签名请求：

```
cat > /data/cert/kube-controller-manager-csr.json <<EOF
{
    "CN": "system:kube-controller-manager",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "hosts": [
      "127.0.0.1",
      "192.168.52.132",
      "192.168.52.133",
      "192.168.52.134"
    ],
    "names": [
      {
        "C": "CN",
        "ST": "BeiJing",
        "L": "BeiJing",
        "O": "system:kube-controller-manager",
        "OU": "4Paradigm"
      }
    ]
}
EOF
```

- hosts 列表包含所有 kube-controller-manager 节点 IP；

- CN 为 system:kube-controller-manager、O 为 system:kube-controller-manager，kubernetes 内置的 ClusterRoleBindings system:kube-controller-manager 赋予 kube-controller-manager 工作所需的权限。

生成证书和私钥：

```
cd /data/cert/

cfssl gencert -ca=/etc/kubernetes/cert/ca.pem \
  -ca-key=/etc/kubernetes/cert/ca-key.pem \
  -config=/etc/kubernetes/cert/ca-config.json \
  -profile=kubernetes kube-controller-manager-csr.json | cfssljson -bare kube-controller-manager
```

将生成的证书和私钥分发到所有 master 节点：

```
cd

cat > magic41_distribute_cert_All_service_.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/cert/kube-controller-manager*.pem k8s@${node_ip}:/etc/kubernetes/cert/
done
EOF

chmod +x magic41_distribute_cert_All_service_.sh
./magic41_distribute_cert_All_service_.sh
```

### 2.创建和分发kubeconfig文件

kubeconfig 文件包含访问 apiserver 的所有信息，如 apiserver 地址、CA 证书和自身使用的证书；

```
source /opt/k8s/bin/environment.sh
cd /data/cert

kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/cert/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-controller-manager.kubeconfig
kubectl config set-credentials system:kube-controller-manager \
  --client-certificate=kube-controller-manager.pem \
  --client-key=kube-controller-manager-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-controller-manager.kubeconfig
kubectl config set-context system:kube-controller-manager \
  --cluster=kubernetes \
  --user=system:kube-controller-manager \
  --kubeconfig=kube-controller-manager.kubeconfig
kubectl config use-context system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig
```

分发 kubeconfig 到所有 master 节点：

```
cd

cat > magic42_distribute_kubeconfig_all_Node_service.sh << "EOF"
#!/bin/bash
# 分发 kubeconfig 到所有 master 节点
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/cert/kube-controller-manager.kubeconfig k8s@${node_ip}:/etc/kubernetes/
done
EOF

chmod +x magic42_distribute_kubeconfig_all_Node_service.sh
./magic42_distribute_kubeconfig_all_Node_service.sh
```

### 3.创建和分发kube-controller-manager systemd unit文件

```
source /opt/k8s/bin/environment.sh

cat > /data/service/kube-controller-manager.service <<EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
[Service]
ExecStart=/opt/k8s/bin/kube-controller-manager \\
  --port=0 \\
  --secure-port=10252 \\
  --bind-address=127.0.0.1 \\
  --kubeconfig=/etc/kubernetes/kube-controller-manager.kubeconfig \\
  --service-cluster-ip-range=${SERVICE_CIDR} \\
  --cluster-name=kubernetes \\
  --cluster-signing-cert-file=/etc/kubernetes/cert/ca.pem \\
  --cluster-signing-key-file=/etc/kubernetes/cert/ca-key.pem \\
  --experimental-cluster-signing-duration=8760h \\
  --root-ca-file=/etc/kubernetes/cert/ca.pem \\
  --service-account-private-key-file=/etc/kubernetes/cert/ca-key.pem \\
  --leader-elect=true \\
  --feature-gates=RotateKubeletServerCertificate=true \\
  --controllers=*,bootstrapsigner,tokencleaner \\
  --horizontal-pod-autoscaler-use-rest-clients=true \\
  --horizontal-pod-autoscaler-sync-period=10s \\
  --tls-cert-file=/etc/kubernetes/cert/kube-controller-manager.pem \\
  --tls-private-key-file=/etc/kubernetes/cert/kube-controller-manager-key.pem \\
  --use-service-account-credentials=true \\
  --alsologtostderr=true \\
  --logtostderr=false \\
  --log-dir=/var/log/kubernetes \\
  --v=2
Restart=on
Restart=on-failure
RestartSec=5
User=k8s
[Install]
WantedBy=multi-user.target
EOF
```

- --port=0：关闭监听 http /metrics 的请求，同时 --address 参数无效，--bind-address 参数有效；

- --secure-port=10252、--bind-address=0.0.0.0: 在所有网络接口监听 10252 端口的 https /metrics 请求；

- --kubeconfig：指定 kubeconfig 文件路径，kube-controller-manager 使用它连接和验证 kube-apiserver；

- --cluster-signing-*-file：签名 TLS Bootstrap 创建的证书；

- --experimental-cluster-signing-duration：指定 TLS Bootstrap 证书的有效期；

- --root-ca-file：放置到容器 ServiceAccount 中的 CA 证书，用来对 kube-apiserver 的证书进行校验；

- --service-account-private-key-file：签名 ServiceAccount 中 Token 的私钥文件，必须和 kube-apiserver 的 --service-account-key-file 指定的公钥文件配对使用；

- --service-cluster-ip-range ：指定 Service Cluster IP 网段，必须和 kube-apiserver 中的同名参数一致；

- --leader-elect=true：集群运行模式，启用选举功能；被选为 leader 的节点负责处理工作，其它节点为阻塞状态；

- --feature-gates=RotateKubeletServerCertificate=true：开启 kublet server 证书的自动更新特性；

- --controllers=*,bootstrapsigner,tokencleaner：启用的控制器列表，tokencleaner 用于自动清理过期的 Bootstrap token；

- --horizontal-pod-autoscaler-*：custom metrics 相关参数，支持 autoscaling/v2alpha1；

- --tls-cert-file、--tls-private-key-file：使用 https 输出 metrics 时使用的 Server 证书和秘钥；

- --use-service-account-credentials=true:

- User=k8s：使用 k8s 账户运行；

kube-controller-manager 不对请求 https metrics 的 Client 证书进行校验，故不需要指定 –tls-ca-file 参数，而且该参数已被淘汰。发 systemd unit 文件到所有 master 节点：

```
cd

cat > magic43_distribute_all_Node_servier.sh << "EOF"
#!/bin/bash
# 分发 systemd unit 文件到所有 master 节点
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/service/kube-controller-manager.service root@${node_ip}:/etc/systemd/system/
done
EOF

chmod +x magic43_distribute_all_Node_servier.sh
./magic43_distribute_all_Node_servier.sh
```

### 4.kube-controller-manager权限

ClusteRole: system:kube-controller-manager 的权限很小，只能创建 secret、serviceaccount 等资源对象，各 controller 的权限分散到 ClusterRole system:controller:XXX 中。

需要在 kube-controller-manager 的启动参数中添加 、–use-service-account-credentials=true 参数，这样 main controller 会为各 controller 创建对应的 ServiceAccount XXX-controller。

内置的 ClusterRoleBinding system:controller:XXX 将赋予各 XXX-controller ServiceAccount 对应的 ClusterRole system:controller:XXX 权限。

### 5.启动kube-controller-manager服务

```
cd

cat > magic44_start_kube-controller-manager_service.sh << "EOF"
#!/bin/bash
# 启动 kube-controller-manager 服务
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "mkdir -p /var/log/kubernetes && chown -R k8s /var/log/kubernetes"
    ssh root@${node_ip} "systemctl daemon-reload && systemctl enable kube-controller-manager && systemctl start kube-controller-manager"
done
EOF

chmod +x magic44_start_kube-controller-manager_service.sh
./magic44_start_kube-controller-manager_service.sh
```

### 6.检查服务状态

```
cd

cat > magic45_check_kube-controller-manager_service.sh << "EOF"
#!/bin/bash
# 检查服务运行状态
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh k8s@${node_ip} "systemctl status kube-controller-manager|grep Active"
done
EOF

chmod +x magic45_check_kube-controller-manager_service.sh
./magic45_check_kube-controller-manager_service.sh
```

失败则查看日志：journalctl -xu kube-controller-manager

### 7.查看输出的metric

注意：以下命令在 kube-controller-manager 节点上执行
kube-controller-manager 监听 10252 端口，接收 https 请求：

```
sudo netstat -lnpt|grep kube-controll

curl -s --cacert /etc/kubernetes/cert/ca.pem https://127.0.0.1:10252/metrics |head
```

- curl --cacert CA 证书用来验证 kube-controller-manager https server 证书；

### 8.查看当前leader

```
su k8s
kubectl get endpoints kube-controller-manager --namespace=kube-system  -o yaml
```

当前的 leader 为 kube-node1 节点。

### 9.测试kube-controller-manager集群高可用

停掉一个或两个节点的 kube-controller-manager 服务，观察其它节点的日志，看是否获取了 leader 权限。

停掉 kube-node1 上的 kube-controller-manager：

```
systemctl stop kube-controller-manager
systemctl status kube-controller-manager |grep Active
```

再查看一下当前的 leader：

```
kubectl get endpoints kube-controller-manager --namespace=kube-system  -o yaml
```

## 十、部署kube-scheduler

该集群包含 3 个节点，启动后将通过竞争选举机制产生一个 leader 节点，其它节点为阻塞状态。当 leader 节点不可用后，剩余节点将再次进行选举产生新的 leader 节点，从而保证服务的可用性。

为保证通信安全，本文档先生成 x509 证书和私钥，kube-scheduler 在如下两种情况下使用该证书：

- 与 kube-apiserver 的安全端口通信;
- 在安全端口 (https，10251) 输出 prometheus 格式的 metrics；

### 1.创建kube-scheduler证书和私钥

创建证书签名请求：

```
cd /data/cert

cat > kube-scheduler-csr.json <<EOF
{
    "CN": "system:kube-scheduler",
    "hosts": [
      "127.0.0.1",
      "192.168.52.132",
      "192.168.52.133",
      "192.168.52.134"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
      {
        "C": "CN",
        "ST": "BeiJing",
        "L": "BeiJing",
        "O": "system:kube-scheduler",
        "OU": "4Paradigm"
      }
    ]
}
EOF
```

- hosts 列表包含所有 kube-scheduler 节点 IP；

- CN 为 system:kube-scheduler、O 为 system:kube-scheduler，kubernetes 内置的 ClusterRoleBindings system:kube-scheduler 将赋予 kube-scheduler 工作所需的权限。

生成证书和私钥：

```
cfssl gencert -ca=/etc/kubernetes/cert/ca.pem \
  -ca-key=/etc/kubernetes/cert/ca-key.pem \
  -config=/etc/kubernetes/cert/ca-config.json \
  -profile=kubernetes kube-scheduler-csr.json | cfssljson -bare kube-scheduler
```



### 2.创建和分发kubeconfig文件

kubeconfig 文件包含访问 apiserver 的所有信息，如 apiserver 地址、CA 证书和自身使用的证书；

```
source /opt/k8s/bin/environment.sh
cd /data/cert
kubectl config set-cluster kubernetes \
  --certificate-authority=/etc/kubernetes/cert/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-credentials system:kube-scheduler \
  --client-certificate=kube-scheduler.pem \
  --client-key=kube-scheduler-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-scheduler.kubeconfig
kubectl config set-context system:kube-scheduler \
  --cluster=kubernetes \
  --user=system:kube-scheduler \
  --kubeconfig=kube-scheduler.kubeconfig
kubectl config use-context system:kube-scheduler --kubeconfig=kube-scheduler.kubeconfig
```

创建的证书、私钥以及 kube-apiserver 地址被写入到 kubeconfig 文件中；
分发 kubeconfig 到所有 master 节点：

```
cd

cat > magic46_distribute_kubeconfig_All_NodeServier.sh << "EOF"
#!/bin/bash
# 分发 kubeconfig 到所有 master 节点：
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/cert/kube-scheduler.kubeconfig k8s@${node_ip}:/etc/kubernetes/
done
EOF

chmod +x magic46_distribute_kubeconfig_All_NodeServier.sh
./magic46_distribute_kubeconfig_All_NodeServier.sh
```

### 3.创建和分发kube-scheduler systemd unit 文件

```
cd /data/template

cat > kube-scheduler.service <<EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
[Service]
ExecStart=/opt/k8s/bin/kube-scheduler \\
  --address=127.0.0.1 \\
  --kubeconfig=/etc/kubernetes/kube-scheduler.kubeconfig \\
  --leader-elect=true \\
  --alsologtostderr=true \\
  --logtostderr=false \\
  --log-dir=/var/log/kubernetes \\
  --v=2
Restart=on-failure
RestartSec=5
User=k8s
[Install]
WantedBy=multi-user.target
EOF
```

- --address：在 127.0.0.1:10251 端口接收 http /metrics 请求；kube-scheduler 目前还不支持接收 https 请求；

- --kubeconfig：指定 kubeconfig 文件路径，kube-scheduler 使用它连接和验证 kube-apiserver；
- --leader-elect=true：集群运行模式，启用选举功能；被选为 leader 的节点负责处理工作，其它节点为阻塞状态；
- User=k8s：使用 k8s 账户运行；

分发 systemd unit 文件到所有 master 节点：

```
cd

cat > magic47_distribute_kube-scheduler_All_NodeServier.sh << "EOF"
#!/bin/bash
# 分发 systemd unit 文件到所有 master 节点
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/template/kube-scheduler.service root@${node_ip}:/etc/systemd/system/
done
EOF

chmod +x magic47_distribute_kube-scheduler_All_NodeServier.sh
./magic47_distribute_kube-scheduler_All_NodeServier.sh
```

### 4.启动kube-scheduler服务

```
cd

cat > magic48_start_kube-scheduler_servier.sh << "EOF"
#!/bin/bash
# 启动 kube-scheduler 服务
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "mkdir -p /var/log/kubernetes && chown -R k8s /var/log/kubernetes"
    ssh root@${node_ip} "systemctl daemon-reload && systemctl enable kube-scheduler && systemctl start kube-scheduler"
done
EOF

chmod +x magic48_start_kube-scheduler_servier.sh
./magic48_start_kube-scheduler_servier.sh
```

### 5.检查服务运行状态

```
cd

cat > magic49_check_servier.sh << "EOF"
#!/bin/bash
# 检查服务运行状态
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh k8s@${node_ip} "systemctl status kube-scheduler|grep Active"
done
EOF

chmod +x magic49_check_servier.sh
./magic49_check_servier.sh
```

失败，看日志：journalctl -xu kube-scheduler

### 6.查看输出的metric

以下命令在 kube-scheduler 节点上执行。
kube-scheduler 监听 10251 端口，接收 http 请求：

```
sudo netstat -lnpt|grep kube-sche

curl -s http://127.0.0.1:10251/metrics |head
```

### 7.查看当前的leader

```
kubectl get endpoints kube-scheduler --namespace=kube-system  -o yaml
```

### 8.测试kube-scheduler集群的高可用

随便找一个或两个 master 节点，停掉 kube-scheduler 服务，看其它节点是否获取了 leader 权限（systemd 日志）。

停掉 kube-node2 上的 kube-scheduler 服务：

```
systemctl stop kube-scheduler
systemctl status kube-scheduler | grep Active
```

## 十一、部署work节点

kubernetes work 节点运行如下组件：

- docker
- kubelet
- kube-proxy

### 1.安装依赖包

```
cd

cat > magic50_install_Depend_Package.sh << "EOF"
#!/bin/bash
# 安装依赖包
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "yum install -y epel-release"
    ssh root@${node_ip} "yum install -y conntrack ipvsadm ipset jq iptables curl sysstat libseccomp && /usr/sbin/modprobe ip_vs "
done
EOF

chmod +x magic50_install_Depend_Package.sh
./magic50_install_Depend_Package.sh
```

## 十二、部署docker

docker 是容器的运行环境，管理它的生命周期。kubelet 通过 Container Runtime Interface (CRI) 与 docker 进行交互。

### 1.下载和分发docker二进制文件

到 [https://download.docker.com/linux/static/stable/x86_64/](https://links.jianshu.com/go?to=https%3A%2F%2Fdownload.docker.com%2Flinux%2Fstatic%2Fstable%2Fx86_64%2F) 页面下载最新发布包：

```
cd /data
wget https://download.docker.com/linux/static/stable/x86_64/docker-18.03.1-ce.tgz
tar -xvf docker-18.03.1-ce.tgz
```

分发二进制文件到所有 worker 节点：

```
cd 

cat > magic51_distribute_binary_file.sh << "EOF"
#!/bin/bash
# 分发二进制文件到所有 worker节点
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/docker/docker*  k8s@${node_ip}:/opt/k8s/bin/
    ssh root@${node_ip} "chmod +x /opt/k8s/bin/*"
done
EOF

chmod +x magic51_distribute_binary_file.sh
./magic51_distribute_binary_file.sh
```

### 2.创建和分发systemd unit文件

```
cd /data/service

cat > docker.service <<"EOF"
[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.io
[Service]
Environment="PATH=/opt/k8s/bin:/bin:/sbin:/usr/bin:/usr/sbin"
EnvironmentFile=-/run/flannel/docker
ExecStart=/opt/k8s/bin/dockerd --log-level=error $DOCKER_NETWORK_OPTIONS
ExecReload=/bin/kill -s HUP $MAINPID
Restart=on-failure
RestartSec=5
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
Delegate=yes
KillMode=process
[Install]
WantedBy=multi-user.target
EOF
```

- EOF 前后有双引号，这样 bash 不会替换文档中的变量，如 $DOCKER_NETWORK_OPTIONS；
- dockerd 运行时会调用其它 docker 命令，如 docker-proxy，所以需要将 docker 命令所在的目录加到 PATH 环境变量中；

- flanneld 启动时将网络配置写入 /run/flannel/docker 文件中，dockerd 启动前读取该文件中的环境变量 DOCKER_NETWORK_OPTIONS ，然后设置 docker0 网桥网段；

- 如果指定了多个 EnvironmentFile 选项，则必须将 /run/flannel/docker 放在最后 (确保 docker0 使用 flanneld 生成的 bip 参数)；

- docker 需要以 root 用于运行；

- docker 从 1.13 版本开始，可能将 iptables FORWARD chain 的默认策略设置为 DROP，从而导致 ping 其它 Node 上的 Pod IP 失败，遇到这种情况时，需要手动设置策略为 ACCEPT：

  ```
  sudo iptables -P FORWARD ACCEPT
  ```

  并且把以下命令写入 /etc/rc.local 文件中，防止节点重启 iptables FORWARD chain 的默认策略又还原为 DROP

```
 /sbin/iptables -P FORWARD ACCEPT
```

分发 systemd unit 文件到所有 worker 机器:

```
cd

cat > magic52_distribute_systemd_file.sh << "EOF"
#!/bin/bash
# 分发 systemd unit 文件到所有 worker 机器
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    scp /data/service/docker.service root@${node_ip}:/etc/systemd/system/
done
EOF

chmod +x magic52_distribute_systemd_file.sh
./magic52_distribute_systemd_file.sh
```

### 3.配置和分发docker配置文件

使用国内的仓库镜像服务器以加快 pull image 的速度，同时增加下载的并发数 (需要重启 dockerd 生效)：

```
cat > docker-daemon.json <<EOF
{
    "registry-mirrors": ["https://hub-mirror.c.163.com", "https://docker.mirrors.ustc.edu.cn"],
    "max-concurrent-downloads": 20
}
EOF
```

分发 docker 配置文件到所有 work 节点：

```
cd

cat > magic53_distribute_docker_config_file.sh << "EOF"
#!/bin/bash
# 分发 docker 配置文件到所有 work 节点
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "mkdir -p  /etc/docker/"
    scp /data/docker-daemon.json root@${node_ip}:/etc/docker/daemon.json
done
EOF

chmod +x magic53_distribute_docker_config_file.sh
./magic53_distribute_docker_config_file.sh
```

### 4.启动docker服务

```
cd

cat > magic54_start_docker_server.sh << "EOF"
#!/bin/bash
# 启动 docker 服务
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "systemctl stop firewalld && systemctl disable firewalld"
    ssh root@${node_ip} "/usr/sbin/iptables -F && /usr/sbin/iptables -X && /usr/sbin/iptables -F -t nat && /usr/sbin/iptables -X -t nat"
    ssh root@${node_ip} "/usr/sbin/iptables -P FORWARD ACCEPT"
    ssh root@${node_ip} "systemctl daemon-reload && systemctl enable docker && systemctl start docker"
    ssh root@${node_ip} 'for intf in /sys/devices/virtual/net/docker0/brif/*; do echo 1 > $intf/hairpin_mode; done'
    ssh root@${node_ip} "sudo sysctl -p /etc/sysctl.d/kubernetes.conf"
done
EOF

chmod +x  magic54_start_docker_server.sh
./magic54_start_docker_server.sh
```

- 关闭 firewalld(centos7)/ufw(ubuntu16.04)，否则可能会重复创建 iptables 规则；
- 清理旧的 iptables rules 和 chains 规则；
- 开启 docker0 网桥下虚拟网卡的 hairpin 模式

### 5.检查服务运行状态

```
cd

cat > magic55_check_server_run_status.sh << "EOF"
#!/bin/bash
# 检查服务运行状态
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh k8s@${node_ip} "systemctl status docker|grep Active"
done
EOF

chmod +x magic55_check_server_run_status.sh
./magic55_check_server_run_status.sh
```

失败，则检查日志：journalctl -xu docker

### 6.检查docker0网桥

```
cd

cat > magic56_ckeck.sh << "EOF"
#!/bin/bash
# 检查 docker0 网桥
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh k8s@${node_ip} "/usr/sbin/ip addr show flannel.1 && /usr/sbin/ip addr show docker0"
done
EOF

chmod +x magic56_ckeck.sh
./magic56_ckeck.sh
```

确认各 work 节点的 docker0 网桥和 flannel.1 接口的 IP 处于同一个网段中。

## 十三、部署kubelet

kublet 运行在每个 worker 节点上，接收 kube-apiserver 发送的请求，管理 Pod 容器，执行交互式命令，如 exec、run、logs 等。kublet 启动时自动向 kube-apiserver 注册节点信息，内置的 cadvisor 统计和监控节点的资源使用情况。

为确保安全，本文档只开启接收 https 请求的安全端口，对请求进行认证和授权，拒绝未授权的访问 (如 apiserver、heapster)。下载分发，在部署 master 节点时已经做过，依赖包也在 work 节点当中安装过。



### 1.创建kubelet bootstrap kubeconfig文件

```
cd

cat > magic57_create_kubeconfig_file.sh << "EOF"
#!/bin/bash
# 创建 kubelet bootstrap kubeconfig 文件
source /opt/k8s/bin/environment.sh
for node_name in ${NODE_NAMES[@]}
do
    echo ">>> ${node_name}"
    # 创建 token，由于使用create我的环境一直报错无法创建，所以使用generate
    export BOOTSTRAP_TOKEN=$(kubeadm token generate \
      --kubeconfig /home/k8s/.kube/config)
    # 设置集群参数
    kubectl config set-cluster kubernetes \
      --certificate-authority=/etc/kubernetes/cert/ca.pem \
      --embed-certs=true \
      --server=${KUBE_APISERVER} \
      --kubeconfig=/etc/kubernetes/kubelet-bootstrap-${node_name}.kubeconfig
    # 设置客户端认证参数
    kubectl config set-credentials kubelet-bootstrap \
      --token=${BOOTSTRAP_TOKEN} \
      --kubeconfig=/etc/kubernetes/kubelet-bootstrap-${node_name}.kubeconfig
    # 设置上下文参数
    kubectl config set-context default \
      --cluster=kubernetes \
      --user=kubelet-bootstrap \
      --kubeconfig=/etc/kubernetes/kubelet-bootstrap-${node_name}.kubeconfig
    # 设置默认上下文
    kubectl config use-context default --kubeconfig=/etc/kubernetes/kubelet-bootstrap-${node_name}.kubeconfig
done
EOF

chmod +x magic57_create_kubeconfig_file.sh
./magic57_create_kubeconfig_file.sh
```

- 证书中写入 Token 而非证书，证书后续由 controller-manager 创建.

查看 kubeadm 为各节点创建的 token：

```
kubeadm token list --kubeconfig /home/k8s/.kube/config
```

- 创建的 token 有效期为 1 天，超期后将不能再被使用，且会被 kube-controller-manager 的 tokencleaner 清理 (如果启用该 controller 的话)；
- kube-apiserver 接收 kubelet 的 bootstrap token 后，将请求的 user 设置为 system:bootstrap:，group 设置为 system:bootstrappers；

各 token 关联的 Secret：

```
kubectl get secrets  -n kube-system
```

### 2.分发bootstrap kubeconfig文件到所有worker节点

```
cd

cat > magic58_distribute_kubeconfig_file.sh << "EOF"
#!/bin/bash
# 分发 bootstrap kubeconfig 文件到所有 worker 节点
source /opt/k8s/bin/environment.sh
for node_name in ${NODE_NAMES[@]}
do
    echo ">>> ${node_name}" 
    scp /etc/kubernetes/kubelet-bootstrap-${node_name}.kubeconfig k8s@${node_name}:/etc/kubernetes/kubelet-bootstrap.kubeconfig
done
EOF

chmod +x magic58_distribute_kubeconfig_file.sh
./magic58_distribute_kubeconfig_file.sh
```

### 3.创建和分发kubelet参数配置文件

创建 kubelet 参数配置模板文件：

```
source /opt/k8s/bin/environment.sh

cat > /data/template/kubelet.config.json.template <<EOF
{
  "kind": "KubeletConfiguration",
  "apiVersion": "kubelet.config.k8s.io/v1beta1",
  "authentication": {
    "x509": {
      "clientCAFile": "/etc/kubernetes/cert/ca.pem"
    },
    "webhook": {
      "enabled": true,
      "cacheTTL": "2m0s"
    },
    "anonymous": {
      "enabled": false
    }
  },
  "authorization": {
    "mode": "Webhook",
    "webhook": {
      "cacheAuthorizedTTL": "5m0s",
      "cacheUnauthorizedTTL": "30s"
    }
  },
  "address": "##NODE_IP##",
  "port": 10250,
  "readOnlyPort": 0,
  "cgroupDriver": "cgroupfs",
  "hairpinMode": "promiscuous-bridge",
  "serializeImagePulls": false,
  "featureGates": {
    "RotateKubeletClientCertificate": true,
    "RotateKubeletServerCertificate": true
  },
  "clusterDomain": "${CLUSTER_DNS_DOMAIN}",
  "clusterDNS": ["${CLUSTER_DNS_SVC_IP}"]
}
EOF
```

- address：API 监听地址，不能为 127.0.0.1，否则 kube-apiserver、heapster 等不能调用 kubelet 的 API；
- readOnlyPort=0：关闭只读端口 (默认 10255)，等效为未指定；
- authentication.anonymous.enabled：设置为 false，不允许匿名访问 10250 端口；
- authentication.x509.clientCAFile：指定签名客户端证书的 CA 证书，开启 HTTP 证书认证；
- authentication.webhook.enabled=true：开启 HTTPs bearer token 认证；
- 对于未通过 x509 证书和 webhook 认证的请求 (kube-apiserver 或其他客户端)，将被拒绝，提示 Unauthorized；
- authroization.mode=Webhook：kubelet 使用 SubjectAccessReview API 查询 kube-apiserver 某 user、group 是否具有操作资源的权限 (RBAC)；
- featureGates.RotateKubeletClientCertificate、featureGates.RotateKubeletServerCertificate：自动 rotate 证书，证书的有效期取决于 kube-controller-manager 的 –experimental-cluster-signing-duration 参数；
- 需要 root 账户运行；

为各节点创建和分发 kubelet 配置文件：

```
cd

cat > magic59_createNode_distribute_file.sh << "EOF"
#!/bin/bash
# 为各节点创建和分发 kubelet 配置文件
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    sed -e "s/##NODE_IP##/${node_ip}/" /data/template/kubelet.config.json.template > /data/template/kubelet.config-${node_ip}.json
    scp /data/template/kubelet.config-${node_ip}.json root@${node_ip}:/etc/kubernetes/kubelet.config.json
done
EOF

chmod +x magic59_createNode_distribute_file.sh
./magic59_createNode_distribute_file.sh
```

### 4.创建和分发kubelet systemd unit文件

创建 kubelet systemd unit 文件模板：

```
cat > /data/template/kubelet.service.template <<EOF
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service
[Service]
WorkingDirectory=/var/lib/kubelet
ExecStart=/opt/k8s/bin/kubelet \\
  --bootstrap-kubeconfig=/etc/kubernetes/kubelet-bootstrap.kubeconfig \\
  --cert-dir=/etc/kubernetes/cert \\
  --kubeconfig=/etc/kubernetes/kubelet.kubeconfig \\
  --config=/etc/kubernetes/kubelet.config.json \\
  --hostname-override=##NODE_NAME## \\
  --pod-infra-container-image=registry.access.redhat.com/rhel7/pod-infrastructure:latest \\
  --allow-privileged=true \\
  --alsologtostderr=true \\
  --logtostderr=false \\
  --log-dir=/var/log/kubernetes \\
  --v=2
Restart=on-failure
RestartSec=5
[Install]
WantedBy=multi-user.target
EOF
```

- 如果设置了 --hostname-override 选项，则 kube-proxy 也需要设置该选项，否则会出现找不到 Node 的情况；
- --bootstrap-kubeconfig：指向 bootstrap kubeconfig 文件，kubelet 使用该文件中的用户名和 token 向 kube-apiserver 发送 TLS Bootstrapping 请求；
- K8S approve kubelet 的 csr 请求后，在 --cert-dir 目录创建证书和私钥文件，然后写入 --kubeconfig 文件。

为各节点创建和分发 kubelet systemd unit 文件：

```
cd

cat > magic60_create_distribute_kubelet_file.sh << "EOF"
#!/bin/bash
# 为各节点创建和分发 kubelet systemd unit 文件
source /opt/k8s/bin/environment.sh
for node_name in ${NODE_NAMES[@]}
do 
    echo ">>> ${node_name}"
    sed -e "s/##NODE_NAME##/${node_name}/" /data/template/kubelet.service.template > /data/template/kubelet-${node_name}.service
    scp /data/template/kubelet-${node_name}.service root@${node_name}:/etc/systemd/system/kubelet.service
    scp /etc/kubernetes/kubelet-bootstrap-${node_name}.kubeconfig root@${node_name}:/etc/kubernetes/kubelet-bootstrap.kubeconfig
done
EOF

chmod +x magic60_create_distribute_kubelet_file.sh
./magic60_create_distribute_kubelet_file.sh
```

### 5.Bootstrap token auth和权限授予

kublet 启动时查找配置的 –kubeletconfig 文件是否存在，如果不存在则使用 --bootstrap-kubeconfig 向 kube-apiserver 发送证书签名请求 (CSR)。

kube-apiserver 收到 CSR 请求后，对其中的 Token 进行认证（事先使用 kubeadm 创建的 token），认证通过后将请求的 user 设置为 system:bootstrap:，group 设置为 system:bootstrappers，这一过程称为 Bootstrap Token Auth。

默认情况下，这个 user 和 group 没有创建 CSR 的权限，kubelet 启动失败，错误日志如下：

```
sudo journalctl -u kubelet -a |grep -A 2 'certificatesigningrequests'
XX XX XX:XX:XX kube-node1 kubelet[26986]: F0506 XX:XX:XX.314378   26986 server.go:233] failed to run Kubelet: cannot create certificate signing request: certificatesigningrequests.certificates.k8s.io is forbidden: User "system:bootstrap:lemy40" cannot create certificatesigningrequests.certificates.k8s.io at the cluster scope
XX XX XX:XX:XX kube-node1 systemd[1]: kubelet.service: Main process exited, code=exited, status=255/n/a
XX XX XX:XX:XX kube-node1 systemd[1]: kubelet.service: Failed with result 'exit-code'.
```

解决：创建一个 clusterrolebinding 将 group system:bootstrappers 和 clusterrole system:node-bootstrapper 绑定：

```
kubectl create clusterrolebinding kubelet-bootstrap \
  --clusterrole=system:node-bootstrapper \
  --group=system:bootstrappers
```

### 6.启动kubelet服务

```
cd

cat > magic61_start_kubelet_server.sh << "EOF"
#!/bin/bash
source /opt/k8s/bin/environment.sh
for node_ip in ${NODE_IPS[@]}
do
    echo ">>> ${node_ip}" 
    ssh root@${node_ip} "mkdir -p /var/lib/kubelet"
    ssh root@${node_ip} "/usr/sbin/swapoff -a"
    ssh root@${node_ip} "mkdir -p /var/log/kubernetes && chown -R k8s /var/log/kubernetes"
    ssh root@${node_ip} "systemctl daemon-reload && systemctl enable kubelet && systemctl restart kubelet"
done
EOF

chmod +x magic61_start_kubelet_server.sh
./magic61_start_kubelet_server.sh
```

- 关闭 swap 分区，否则 kubelet 会启动失败；

- 必须先创建工作和日志目录；

  ```
  journalctl -u kubelet |tail
  XX XX XX:XX:XX kube-node1 kubelet[17331]: I1124 XX:XX:XX.255251   17331 container_manager_linux.go:427] [ContainerManager]: Discovered runtime cgroups name: /system.slice/docker.service
  XX XX XX:XX:XX kube-node1 kubelet[17331]: I1124 XX:XX:XX.255711   17331 container_manager_linux.go:427] [ContainerManager]: Discovered runtime cgroups name: 
  ...........
  ```

kubelet 启动后使用 --bootstrap-kubeconfig 向 kube-apiserver 发送 CSR 请求，当这个 CSR 被 approve 后，kube-controller-manager 为 kubelet 创建 TLS 客户端证书、私钥和 --kubeletconfig 文件。

注意：kube-controller-manager 需要配置 --cluster-signing-cert-file 和 --cluster-signing-key-file 参数，才会为 TLS Bootstrap 创建证书和私钥。

```
kubectl get csr
NAME                                                   AGE       REQUESTOR                 CONDITION
node-csr-QzuuQiuUfcSdp3j5W4B2UOuvQ_n9aTNHAlrLzVFiqrk   43s       system:bootstrap:zkiem5   Pending
node-csr-oVbPmU-ikVknpynwu0Ckz_MvkAO_F1j0hmbcDa__sGA   27s       system:bootstrap:mkus5s   Pending
node-csr-u0E1-ugxgotO_9FiGXo8DkD6a7-ew8sX2qPE6KPS2IY   13m       system:bootstrap:k0s2bj   Pending

kubectl get nodes
  No resources found.
```

- 三个 work 节点的 csr 均处于 pending 状态
