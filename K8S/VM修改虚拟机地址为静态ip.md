## 一、虚拟网络编辑器

安装好虚拟后在菜单栏选择编辑→ 虚拟网络编辑器，打开虚拟网络编辑器对话框，选择**Vmnet8 Net**网络连接方式，随意设置子网IP，点击NAT设置页面，查看**子网掩码和网关**，后面修改静态IP会用到。

![image-20211114153150826](../../../picbed/store/picbed/img/image-20211114153150826.png)

## 二、宿主机VM8网卡

打开网络和共享中心→ 更改适配器设置→，在VMware Network Adapter VMnet8上单击右键，选择属性按钮打开属性对话框。   

![image-20211114153724967](../../../picbed/store/picbed/img/image-20211114153724967.png)

## 三、确保VM网络配置生效

在虚拟机右下角，点击网络适配器按钮，右键选择断开连接，然后再重新连接，确保刚才的设置生效。然后开启虚拟机，输入ip addr查看当前分配的IP。

![img](https://images2015.cnblogs.com/blog/1056286/201703/1056286-20170310173154326-21632368.png)

## 四、修改网络配置文件

ping宿主机ip（192.168.2.168）、宿主机VM8  IP（192.168.6.1）、宿主机与虚拟机之前的网关IP（192.168.6.2）、ping外网（baidu.com）都可以通则说明虚拟机固定IP设置成功。

vi /etc/sysconfig/network-scripts/ifcfg-ens33 为：00:0C:29:E7:31:6D

>
>
>DEVICE="ens33"
>BOOTPROTO=static
>HWADDR=00:0C:29:E4:7B:1E (网络适配器中-高级中查看)
>IPADDR=192.168.52.129
>GATEWAY=192.168.52.2
>TYPE="Ethernets"
>ONBOOT=yes

## 五、重启虚拟机

验证IP是否固定不变