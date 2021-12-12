## 一、初始化服务器设置

### 1.修改主机名称

为了方便管理, 将服务器的实例名称改成:master01/node01/node02，测试一下三个服务器是否，可以通过私网相互ping通。

```
# master01 机器上
hostnamectl set-hostname master01
# node01 机器上
hostnamectl set-hostname node01
# node02 机器上
hostnamectl set-hostname node02
```

### 2.设置/etc/hosts文件

真正的集群应该是使用自己搭建的DNS服务器来进行IP和域名绑定, 这里处于简单考虑, 就直接使`用hosts文件关联IP和主机名了, 在三台服务的/etc/hosts文件中添加相同的三句话。

```
cat << EOF >> /etc/hosts
192.168.52.132   master01
192.168.52.133   node01
192.168.52.134   node02
EOF
```

### 3.设置准备的环境（所有节点）

#### 1）安装依赖包

```
yum install -y conntrack ipvsadm ipset jq iptables curl sysstat libseccomp wget vim net-tools git epel-release telnet tree nmap  lrzsz dos2unix bind-utils
```

#### 2）关闭setenforce和firewalld

```
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
setenforce 0
systemctl disable firewalld && systemctl stop firewalld
```

#### 3）安装设置Iptables规则为空

```
yum -y install iptables-services  &&  systemctl  start iptables  &&  systemctl  enable iptables&&  iptables -F  &&  service iptables save
```

#### 4）关闭swap分区（如果不关闭的话, pod容器可能运行在swap(虚拟内存)中, 影响效率 )

```
swapoff -a && sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
```

#### 5）针对k8s调整内核参数

```
mkdir /data

# 编辑配置文件
cat > /data/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-iptables=1   # 开启网桥模式
net.bridge.bridge-nf-call-ip6tables=1 # 开启网桥模式
net.ipv4.ip_forward=1
net.ipv4.tcp_tw_recycle=0
vm.swappiness=0                     # 禁止使用 swap 空间，只有当系统 OOM 时才允许使用它
vm.overcommit_memory=1              # 不检查物理内存是否够用
vm.panic_on_oom=0                   # 开启 OOM
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1    # 关闭IPV6协议
net.netfilter.nf_conntrack_max=2310720
EOF
```

#### 6）设置yum源

```
vi /etc/resove.conf
nameserver 223.5.5.5
nameserver 223.6.6.6

cd  /etc/yum.repos.d && mkdir -p /etc/yum.repos.d/bakup &&  mv /etc/yum.repos.d/* /etc/yum.repos.d/bakup

curl -o /etc/yum.repos.d/centos-7.repo http://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/docker-ce.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

yum clean all && yum makecache
```

#### 7）生效配置文件

```
cp /data/kubernetes.conf  /etc/sysctl.d/kubernetes.conf
sysctl -p /etc/sysctl.d/kubernetes.conf
```

#### 8）调整系统时区

```
# 设置系统时区为中国/上海
timedatectl set-timezone Asia/Shanghai
# 将当前的 UTC 时间写入硬件时钟
timedatectl set-local-rtc 0
# 重启依赖于系统时间的服务
systemctl restart rsyslog
systemctl restart crond
```

#### 9）关闭系统不需要的服务

```
systemctl stop postfix && systemctl disable postfix
```

#### 10）设置日志系统
选择systemd journald的日志系统, 而不是rsyslogd
创建日志目录：

```
mkdir -p /var/log/journal       # 持久化保存日志的目录
mkdir -p /etc/systemd/journald.conf.d
```

编写配置文件:

```
cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
[Journal]
# 持久化保存到磁盘
Storage=persistent

# 压缩历史日志
Compress=yes

SyncIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000

# 最大占用空间 10G
SystemMaxUse=10G

# 单日志文件最大 200M
SystemMaxFileSize=200M

# 日志保存时间 2 周
MaxRetentionSec=2week

# 不将日志转发到syslog
ForwardToSyslog=no
EOF
```

重启日志系统:

```
systemctl restart systemd-journald
```

#### 11)kube-proxy开启ipvs的前置条件

```
# 加载br_netfilter模块
modprobe br_netfilter

# 编写依赖文件
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF

# 授权
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

#### 14)安装Docker

```
# 安装依赖
yum install -y yum-utils device-mapper-persistent-data lvm2

# 配置阿里源
yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

# 安装安装最新的 containerd.io
yum install dnf -y
dnf install -y https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm

# 安装docker
yum update -y && yum install -y docker-ce

# 查看docker版本(是否安装成功)
docker --version

# 创建 /etc/docker 目录
mkdir -p /etc/docker

# 配置 daemon.json 在阿里云控制台选择"容器镜像服务", 再选择"镜像加速器"侧边栏, 查看加速器地址
cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://q4xjzq29.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

# 创建目录
mkdir -p /etc/systemd/system/docker.service.d

# 重启docker
systemctl daemon-reload && systemctl restart docker && systemctl enable docker
```

#### 15)安装kubeadm（主从配置）

下载kubeadm(三台服务器)：

```
# 配置阿里源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg 
http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg 
EOF

# 安装 kubelet kubeadm kubectl
yum install -y kubelet kubeadm kubectl

# 查看安装的版本
kubelet --version

# systemctl在enable、disable、mask子命令里面增加了--now选项，可以激活同时启动服务，激活同时停止服务等
systemctl enable --now kubelet
```

#### 15）下载必须的镜像(三台服务器)

正常情况下, 接下来可以直接init操作, 在init操作时, 也会下载一些必须的组件镜像, 这些镜像是在k8s.gcr.io网站上下载的, 但是由于我们国内把该网址墙掉了, 不能直接访问, 于是需要先提前将这些镜像通过其他的方式下载好, 这里比较好的方式就是从另一个网站源下载。

kubeadm init主要执行了以下操作：

```
[init]：指定版本进行初始化操作
[preflight] ：初始化前的检查和下载所需要的Docker镜像文件
[kubelet-start] ：生成kubelet的配置文件”/var/lib/kubelet/config.yaml”，没有这个文件kubelet无法启动，所以初始化之前的kubelet实际上启动失败。
[certificates]：生成Kubernetes使用的证书，存放在/etc/kubernetes/pki目录中。
[kubeconfig] ：生成 KubeConfig 文件，存放在/etc/kubernetes目录中，组件之间通信需要使用对应文件。
[control-plane]：使用/etc/kubernetes/manifest目录下的YAML文件，安装 Master 组件。
[etcd]：使用/etc/kubernetes/manifest/etcd.yaml安装Etcd服务。
[wait-control-plane]：等待control-plan部署的Master组件启动。
[apiclient]：检查Master组件服务状态。
[uploadconfig]：更新配置
[kubelet]：使用configMap配置kubelet。
[patchnode]：更新CNI信息到Node上，通过注释的方式记录。
[mark-control-plane]：为当前节点打标签，打了角色Master，和不可调度标签，这样默认就不会使用Master节点来运行Pod。
[bootstrap-token]：生成token记录下来，后边使用kubeadm join往集群中添加节点时会用到
[addons]：安装附加组件CoreDNS和kube-proxy 
```

#### 16）查看需要下载的镜像

```
kubeadm config images list

# 输出结果, 这些都是K8S的必要组件, 但是由于被墙, 是不能直接docker pull下来的
k8s.gcr.io/kube-apiserver:v1.23.0
k8s.gcr.io/kube-controller-manager:v1.23.0
k8s.gcr.io/kube-scheduler:v1.23.0
k8s.gcr.io/kube-proxy:v1.23.0
k8s.gcr.io/pause:3.6
k8s.gcr.io/etcd:3.5.1-0
k8s.gcr.io/coredns/coredns:v1.8.6
```

#### 17）写pull脚本

```
mkdir /data/kubernetes
cat /data/kubernetes/pull_k8s_images.sh
# 内容为
set -o errexit
set -o nounset
set -o pipefail

##这里定义需要下载的版本
KUBE_VERSION=v1.23.0
KUBE_PAUSE_VERSION=3.6
ETCD_VERSION=3.5.1-0
DNS_VERSION=v1.8.6

##这是原来被墙的仓库
GCR_URL=k8s.gcr.io

##这里就是写你要使用的仓库,可以gotok8s不变
DOCKERHUB_URL=gotok8s

##这里是镜像列表
images=(
kube-proxy:${KUBE_VERSION}
kube-scheduler:${KUBE_VERSION}
kube-controller-manager:${KUBE_VERSION}
kube-apiserver:${KUBE_VERSION}
pause:${KUBE_PAUSE_VERSION}
etcd:${ETCD_VERSION}
coredns:${DNS_VERSION}
)

##这里是拉取和改名的循环语句, 先下载, 再tag重命名生成需要的镜像, 再删除下载的镜像
for imageName in ${images[@]} ; do
  docker pull $DOCKERHUB_URL/$imageName
  docker tag $DOCKERHUB_URL/$imageName $GCR_URL/$imageName
  docker rmi $DOCKERHUB_URL/$imageName
done
```

```
cd  /data/kubernetes/

# 推送脚本到node[1:2]
scp /data/kubernetes/pull_k8s_images.sh root@192.168.52.133:/data/kubernetes
scp /data/kubernetes/pull_k8s_images.sh root@192.168.52.134:/data/kubernetes

chmod +x /data/kubernetes/pull_k8s_images.sh
./pull_k8s_images.sh
docker images
GCR_URL=k8s.gcr.io
imageName=coredns:v1.8.6
docker tag $GCR_URL/$imageName $GCR_URL/coredns/$imageName
docker rmi $GCR_URL/$imageName
```

#### 18)初始化主节点（只有主节点服务器才需要初始化)

生成初始化文件

```
kubeadm config print init-defaults > kubeadm-config.yaml
```

编辑文件:

```
vi kubeadm-config.yaml

# 修改项下面标出
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.52.132     # 1.修改IP地址, 使用私网IP地址即可
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: master01
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: k8s.gcr.io
kind: ClusterConfiguration
kubernetesVersion: v1.23.0      # 2.修改版本, 与前面版本一致, 也可通过 kubeadm version 查看版本
networking:
  dnsDomain: cluster.local
  podSubnet: "10.244.0.0/16"    # 3.新增pod子网, 固定该IP即可
  serviceSubnet: 10.96.0.0/12
scheduler: {}

# 4.新增下面设置, 固定即可
---
apiVersion: kubeproxy.config.k8s.io/v1alpha1
kind: KubeProxyConfiguration
featureGates:
  SupportIPVSProxyMode: true
mode: ipvs
```

#### 19)运行初始化命令

```
kubeadm init --config=kubeadm-config.yaml | tee kubeadm-init.log

# 正常运行结果
....
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join IP地址:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:873f80617875dc39a23eced3464c7069689236d460b60692586e7898bf8a254a
```

如果init运行错误
可以根据错误信息来排错, 主要原因是配置文件 kubeadm-config.yaml 没写好, 还有版本号没对上, IP地址没改, 多余空格等等问题..........但是修改完之后之后, 如果直接运行init命令, 可能还会报错端口已被占用或者一些文件已经存在的。原因可能是之前init到一半成功一部分, 但报错后有没有回滚, 那么需要先运行kubeadm reset重新设置为init之前的状态。重设完之后再继续执行上述的init即可, init 知道是否成功。

- failed to pull image k8s.gcr.io/coredns/coredns:v1.8.6: output: Error response from daemon

  tag名字不符合要求。

init运行成功后
可以查看最后的输出结果或者查看运行日志kubeadm-init.log, 里面告诉说需要操作下面的步骤：

```
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
```

查看当前节点, 发现状态为NotReady：

```
kubectl get node
```

#### 20）将子节点加到主节点下面(在子节点服务器运行)

还是在主节点的init命令的输出日志下, 有子节点的加入命令, 在两台子节点服务器上运行：

```
kubeadm join 192.168.52.132:6443 --token XXXXXXXXXXXXXXXXXXX \
        --discovery-token-ca-cert-hash sha256:XXXXXXXXXXXXXXXXXXX
```

稍等片刻后, 加入成功如下:

```
W0801 19:27:06.500319   12557 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
    [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
    [WARNING FileExisting-tc]: tc not found in system path
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Downloading configuration for the kubelet from the "kubelet-config-1.18" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

在主节点服务器上查看子节点状态为Ready：

```
kubectl get node
```

但是在子节点服务器上运行kubectl get node却发现报错了, 如下：

```
kubectl get node
The connection to the server localhost:8080 was refused - did you specify the right host or port?
```

可以发现按安装成功后，日志提示的如下步骤操作即可：

```
# 在各个子节点创建.kube目录
[root@k8s-node02-17:~]# mkdir -p $HOME/.kube

# 这里需要在主节点将admin.conf复制到各个子节点
scp /etc/kubernetes/admin.conf root@node01:$HOME/.kube/config
scp /etc/kubernetes/admin.conf root@node02:$HOME/.kube/config

# 授权
[root@k8s-node02-17:~]# chown $(id -u):$(id -g) $HOME/.kube/config

# 最后运行测试, 发现不报错了
[root@k8s-node02-17:~]# kubectl get node
NAME               STATUS   ROLES    AGE   VERSION
k8s-master01-15    Ready    master   37h   v1.18.6
k8s-node01-16      Ready    <none>   36h   v1.18.6
k8s-node02-17      Ready    <none>   36h   v1.18.6
```

## 部署flannel网络(主节点服务器)

可以先整理一下当前文件夹：

```
# 创建整理安装所需的文件夹
[root@k8s-master01-15 ~]# mkdir -p /data/kubernetes/install-k8s/core/ && cd /data/kubernetes/

# 将主要的文件放入文件夹中
[root@k8s-master01-15 kubernetes]# mv kubeadm-init.log kubeadm-config.yaml ./install-k8s/core/

# 创建flannel文件夹
[root@k8s-master01-15 kubernetes]# cd install-k8s && mkdir -p /data/kubernetes/install-k8s/plugin/flannel/ && cd ./plugin/flannel/  
```

下载kube-flannel.yml文件：

```
[root@k8s-master01-15 flannel]# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 下载命令的打印结果
--2021-07-01 18:10:44--  https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 151.101.108.133
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|151.101.108.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 14366 (14K) [text/plain]
Saving to: ‘kube-flannel.yml’
kube-flannel.yml              100%[================================================>]  14.03K  --.-KB/s    in 0.05s   
2021-07-01 18:15:00 (286 KB/s) - ‘kube-flannel.yml’ saved [14366/14366]
```

执行安装flannel网络插件：

```
# 先拉取镜像，此过程国内速度比较慢
docker pull quay.io/coreos/flannel:v0.14.0
```

编辑 kube-flannel.yml 网卡的配置：

```
[root@k8s-master01-15 flannel]# vi ./flannel/kube-flannel.yml
---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp.flannel.unprivileged
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: docker/default
    seccomp.security.alpha.kubernetes.io/defaultProfileName: docker/default
    apparmor.security.beta.kubernetes.io/allowedProfileNames: runtime/default
    apparmor.security.beta.kubernetes.io/defaultProfileName: runtime/default
spec:
  privileged: false
  volumes:
  - configMap
  - secret
  - emptyDir
  - hostPath
  allowedHostPaths:
  - pathPrefix: "/etc/cni/net.d"
  - pathPrefix: "/etc/kube-flannel"
  - pathPrefix: "/run/flannel"
  readOnlyRootFilesystem: false
  # Users and groups
  runAsUser:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  # Privilege Escalation
  allowPrivilegeEscalation: false
  defaultAllowPrivilegeEscalation: false
  # Capabilities
  allowedCapabilities: ['NET_ADMIN', 'NET_RAW']
  defaultAddCapabilities: []
  requiredDropCapabilities: []
  # Host namespaces
  hostPID: false
  hostIPC: false
  hostNetwork: true
  hostPorts:
  - min: 0
    max: 65535
  # SELinux
  seLinux:
    # SELinux is unused in CaaSP
    rule: 'RunAsAny'
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
rules:
- apiGroups: ['extensions']
  resources: ['podsecuritypolicies']
  verbs: ['use']
  resourceNames: ['psp.flannel.unprivileged']
- apiGroups:
  - ""
  resources:
  - pods
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - list
  - watch
- apiGroups:
  - ""
  resources:
  - nodes/status
  verbs:
  - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flannel
  namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
  labels:
    tier: node
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "cniVersion": "0.3.1",
      "plugins": [
        {
          "type": "flannel",
          "delegate": {
            "hairpinMode": true,
            "isDefaultGateway": true
          }
        },
        {
          "type": "portmap",
          "capabilities": {
            "portMappings": true
          }
        }
      ]
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  selector:
    matchLabels:
      app: flannel
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/os
                operator: In
                values:
                - linux
      hostNetwork: true
      priorityClassName: system-node-critical
      tolerations:
      - operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: quay.io/coreos/flannel:v0.14.0
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conflist
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.14.0
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=ens33        # 如机器有多个网卡的话，指定内网网卡的名称，默认不指定的话会找第一块网卡
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "100m"
            memory: "50Mi"
        securityContext:
          privileged: false
          capabilities:
            add: ["NET_ADMIN", "NET_RAW"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run/flannel
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
      - name: run
        hostPath:
          path: /run/flannel
      - name: cni
        hostPath:
          path: /etc/cni/net.d
      - name: flannel-cfg
        configMap:
          name: kube-flannel-cfg


# 创建flannel
[root@k8s-master01-15 flannel]# kubectl create -f kube-flannel.yml


# 创建命令的打印结果
podsecuritypolicy.policy/psp.flannel.unprivileged created
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds-amd64 created
daemonset.apps/kube-flannel-ds-arm64 created
daemonset.apps/kube-flannel-ds-arm created
daemonset.apps/kube-flannel-ds-ppc64le created
daemonset.apps/kube-flannel-ds-s390x created


# 查看pod, 可以看到flannel组件已经运行起来了. 默认系统组件都安装在 kube-system 这个命名空间(namespace)下
[root@k8s-master01-15 flannel]# kubectl get pod -n kube-system
NAME                                       READY   STATUS    RESTARTS   AGE
coredns-66bff467f8-tlqdw                   1/1     Running   0          18m
coredns-66bff467f8-zpg4q                   1/1     Running   0          18m
etcd-k8s-master01-15                       1/1     Running   0          18m
kube-apiserver-k8s-master01-15             1/1     Running   0          18m
kube-controller-manager-k8s-master01-15    1/1     Running   0          18m
kube-flannel-ds-amd64-5hpff                1/1     Running   0          32s
kube-proxy-xh6wh                           1/1     Running   0          18m
kube-scheduler-k8s-master01-15             1/1     Running   0          18m


# 再次查看node, 发现状态已经变成了 Ready
[root@k8s-master01-15 flannel]# kubectl get node
NAME               STATUS   ROLES    AGE   VERSION
k8s-master01-15    Ready    master   19m   v1.18.6
```

### 1.安装flannel.yml 问题

```
kubectl delete -f kube-flannel.yml
```

### 2.kubectl get cs 问题，提示：Get "[http://127.0.0.1:10251/healthz](https://links.jianshu.com/go?to=http%3A%2F%2F127.0.0.1%3A10251%2Fhealthz)": dial tcp 127.0.0.1:10251: connect: connection refused

出现这种情况，是/etc/kubernetes/manifests/下的kube-controller-manager.yaml和kube-scheduler.yaml设置的默认端口是0导致的，解决方式是注释掉对应的port即可，操作如下：

```
[root@k8s-master01-15 manifests]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused   
controller-manager   Healthy     ok                                                                                            
etcd-0               Healthy     {"health":"true","reason":""} 


[root@k8s-master01-15 /etc/kubernetes/manifests]# vim kube-scheduler.yaml +19
.....
spec:
  containers:
  - command:
    - kube-scheduler
    - --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
    - --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
    - --bind-address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --leader-elect=true
    #- --port=0      # 注释这行


[root@k8s-master01-15 /etc/kubernetes/manifests]# vim kube-controller-manager.yaml +26
.....
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=10.244.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    #- --port=0      # 注释这行

# 重启kubelet.service 服务
systemctl restart kubelet.service
[root@k8s-master01-15 manifests]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok                              
etcd-0               Healthy   {"health":"true","reason":""}   
controller-manager   Healthy   ok  
```

### 3.network: open /run/flannel/subnet.env: no such file or directory

```
[root@k8s-master01-15 core]# kubectl get pods -n kube-system -o wide
NAME                             READY   STATUS              RESTARTS         AGE     IP               NODE     NOMINATED NODE   READINESS GATES
coredns-78fcd69978-6t76m         0/1     ContainerCreating   0                126m    <none>           node01   <none>           <none>
coredns-78fcd69978-nthg8         0/1     ContainerCreating   0                126m    <none>           node01   <none>           <none>
.....

[root@k8s-master01-15 core]# kubectl describe pods coredns-78fcd69978-6t76m -n kube-system
.......
Events:
  Type     Reason                  Age                      From     Message
  ----     ------                  ----                     ----     -------
  Normal   SandboxChanged          19m (x4652 over 104m)    kubelet  Pod sandbox changed, it will be killed and re-created.
  Warning  FailedCreatePodSandBox  4m26s (x5461 over 104m)  kubelet  (combined from similar events): Failed to create pod sandbox: rpc error: code = Unknown desc = failed to set up sandbox container "8e2992e19a969235ff30271e317565b48ffe57d8261dc86f92249005a5eaaec5" network for pod "coredns-78fcd69978-6t76m": networkPlugin cni failed to set up pod "coredns-78fcd69978-6t76m_kube-system" network: open /run/flannel/subnet.env: no such file or directory
```

查看是否有 /run/flannel/subnet.env 这个文件，master 上是存在的，也有内容：

```
[root@k8s-master01-15:~]# mkdir -p /run/flannel/

cat >>/run/flannel/subnet.env <<EOF
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.0.1/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
EOF
```

### 4.发现 kube-proxy出现异常，状态：CrashLoopBackOff

kube-proxy是作用于service的，作用主要是负责service的实现，实现了内部从pod到service和外部的从node port向service的访问：

```
[root@master core]# kubectl get pods -n kube-system -o wide
NAME                             READY   STATUS              RESTARTS         AGE    IP               NODE     NOMINATED NODE   READINESS GATES
coredns-78fcd69978-dts5t         0/1     ContainerCreating   0                146m   <none>           node02   <none>           <none>
coredns-78fcd69978-v8g7z         0/1     ContainerCreating   0                146m   <none>           node02   <none>           <none>
etcd-master                      1/1     Running             0                147m   172.23.199.15   master   <none>           <none>
kube-apiserver-master            1/1     Running             0                147m   172.23.199.15   master   <none>           <none>
kube-controller-manager-master   1/1     Running             0                147m   172.23.199.15   master   <none>           <none>
kube-proxy-9nxhp                 0/1     CrashLoopBackOff    33 (37s ago)     144m   172.23.199.16   node01   <none>           <none>
kube-proxy-gqrvl                 0/1     CrashLoopBackOff    33 (86s ago)     145m   172.23.199.17   node02   <none>           <none>
kube-proxy-p825v                 0/1     CrashLoopBackOff    33 (2m54s ago)   146m   172.23.199.15   master   <none>           <none>
kube-scheduler-master            1/1     Running             0                147m   172.23.199.15   master   <none>           <none>


# 使用kubectl describe XXX 排查pod状态的信息
[root@master core]# kubectl describe pod kube-proxy-9nxhp -n kube-system
...
Events:
  Type     Reason   Age                  From     Message
  ----     ------   ----                 ----     -------
  Warning  BackOff  8s (x695 over 150m)  kubelet  Back-off restarting failed container
```

Back-off restarting failed container的Warning事件，一般是由于通过指定的镜像启动容器后，容器内部没有常驻进程，导致容器启动成功后即退出，从而进行了持续的重启。解决办法为：要使Pod持续运行，就必须指定一个永远不会完成的任务。

在yaml文件中指定一个启动命令，内容如下:command: ["/bin/bash", "-ce", "tail -f /dev/null"]

解决方法：

https://www.cnblogs.com/zhangsi-lzq/p/14279997.html



### 解决pod的IP无法ping通的问题

集群安装完成后, 启动一个pod：

```
# 启动pod, 命名为nginx-offi, 里面运行的容器为从官网拉取的Nginx镜像
[root@k8s-master01-15:~]# kubectl run nginx-offi --image=nginx
pod/nginx-offi created

# 查看pod的运行信息, 可以看到状态为 "Running" ,IP为 "10.244.1.7", 运行在了 "k8s-node01-16" 节点上
[root@k8s-master01-15:~]# kubectl get pod -o wide
NAME            READY   STATUS    RESTARTS   AGE   IP           NODE             NOMINATED NODE   READINESS GATES
nginx-offi      1/1     Running   0          55s   10.244.1.7   k8s-node01-16   <none>           <none>
```

但是如果在主节点k8s-master01-15 或 另一个子节点 k8s-node02-17上访问刚才运行的pod, 却发现访问不到, 可以尝试 ping一下该IP地址：10.244.1.7也ping不通, 尽管前面我们已经安装好了flannel.

UP发现： 是iptables 规则的问题, 前面我们在初始化服务器设置的时候清除了iptables的规则, 但主要原因是不是安装了 flannel 还是哪一步的问题, 会导致 iptables 里面又多出了规则

```
# 查看iptables
(root@k8s-master01-15:~)# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
KUBE-FIREWALL  all  --  0.0.0.0/0            0.0.0.0/0           

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
KUBE-FORWARD  all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding rules */
DOCKER-USER  all  --  0.0.0.0/0            0.0.0.0/0           
DOCKER-ISOLATION-STAGE-1  all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
KUBE-FIREWALL  all  --  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER (1 references)
target     prot opt source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
target     prot opt source               destination         
DOCKER-ISOLATION-STAGE-2  all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
target     prot opt source               destination         
DROP       all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
target     prot opt source               destination         
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           

Chain KUBE-FIREWALL (2 references)
target     prot opt source               destination         
DROP       all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes firewall for dropping marked packets */ mark match 0x8000/0x8000
DROP       all  -- !127.0.0.0/8          127.0.0.0/8          /* block incoming localnet connections */ ! ctstate RELATED,ESTABLISHED,DNAT

Chain KUBE-KUBELET-CANARY (0 references)
target     prot opt source               destination         

Chain KUBE-FORWARD (1 references)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding rules */ mark match 0x4000/0x4000
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding conntrack pod source rule */ ctstate RELATED,ESTABLISHED
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding conntrack pod destination rule */ ctstate RELATED,ESTABLISHED
# Warning: iptables-legacy tables present, use iptables-legacy to see them
```

需要再次清空 iptables 规则：

```
iptables -F &&  iptables -X &&  iptables -F -t nat &&  iptables -X -t nat
```

再次查看iptables：

```
(root@k8s-master01-15:~)# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
KUBE-FORWARD  all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding rules */

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain KUBE-FORWARD (1 references)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding rules */ mark match 0x4000/0x4000
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding conntrack pod source rule */ ctstate RELATED,ESTABLISHED
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            /* kubernetes forwarding conntrack pod destination rule */ ctstate RELATED,ESTABLISHED
# Warning: iptables-legacy tables present, use iptables-legacy to see them
```

再次ping或者访问pod, 即可成功：

```
(root@k8s-master01-15:~)# curl 10.244.1.7
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



作者：Chris0Yang
链接：https://www.jianshu.com/p/51542b0b239b