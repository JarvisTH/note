参考：[Linux Firewalld用法及案例](https://blog.csdn.net/xiazichenxi/article/details/80169927)
##一、Firewalld概述
- 动态防火墙管理工具
- 定义区域与接口安全等级
- 运行时和永久配置项分离
两层结构
*核心层*  处理配置和后端，如iptables、ip6tables、ebtables、ipset和模块加载器;
*顶层D-Bus* 更改和创建防火墙配置的主要方式。所有firewalld都使用该接口提供在线工具。

![原理图](https://upload-images.jianshu.io/upload_images/9449419-ab76a8426729fd8a?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![](https://upload-images.jianshu.io/upload_images/9449419-58bf4eb7ea0dbc0b?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##二、Firewalld与iptables对比
- firewalld 是 iptables 的前端控制器
- iptables 静态防火墙 任一策略变更需要reload所有策略，丢失现有链接
- firewalld 动态防火墙 任一策略变更不需要reload所有策略 将变更部分保存到iptables,不丢失现有链接
- firewalld 提供一个daemon和service 底层使用iptables
- 基于内核的Netfilter

##三、配置方式
- firewall-config 图形界面
- firewall-cmd 命令行工具
- 直接修改配置文件
/lib/firewalld 用于默认和备用配置
/etc/firewalld 用于用户创建和自定义配置文件 覆盖默认配置
/etc/firewalld/firewall.conf 全局配置

##四、运行时配置和永久配置
- firewall-cmd –zone=public –add-- service=smtp 运行时配置，重启后失效
- firewall-cmd –permanent –zone=public –add-service=smtp 永久配置，不影响当前连接，重启后生效
- firewall-cmd –runtime-to-permanent 将运行时配置保存为永久配置

##五、Zone
- 网络连接的可信等级,一对多，一个区域对应多个连接
- drop.xml 拒绝所有的连接
- block.xml 拒绝所有的连接
- public.xml 只允许指定的连接 *默认区域
- external.xml 只允许指定的连接
- dmz.xml 只允许指定的连接
- work.xml 只允许指定的连接
- home.xml 只允许指定的连接
- internal.xml 只允许指定的连接
- trusted.xml 允许所有的连接
/lib/firewalld/zones 默认和备用区域配置
- /etc/firewalld/zones 用户创建和自定义区域配置文件 覆盖默认配置

##六、services
##七、ipset配置
>系统默认没有ipset配置文件，需要手动创建ipset配置文件
mkdir -p /etc/firewalld/ipsets/mytest.xml mytest就是ipset名称
根据官方手册提供的配置模板
<?xml version="1.0" encoding="utf-8"?>
<ipset type="hash:net">
  <short>white-list</short>
  <entry>192.168.1.1</entry>
  <entry>192.168.1.2</entry>
  <entry>192.168.1.3</entry>
</ipset>
entry也就是需要加入的IP地址
firewall-cmd --get-ipsets 显示当前的ipset
firewall-cmd --permanent --add-rich-rule 'rule family="ipv4" source ipset="mytest" port port=80 protocol=tcp accept' 将ipset应用到策略中

##八、服务管理
- yum -y install firewalld firewall-config #安装firewalld
- systemctl enable|disable firewalld #开机启动
- systemctl start|stop|restart firewalld #启动、停止、重启firewalld

##如果想使用iptables配置防火墙规则，要先安装iptables并禁用firewalld
- yum -y install iptables-services #安装iptables
- systemctl enable iptables #开机启动
- systemctl start|stop|restart iptables #启动、停止、重启iptables

##firewall-cmd常用命令
```
firewall-cmd --version 查看firewalld版本
firewall-cmd --help 查看firewall-cmd用法
man firewall-cmd

firewall-cmd --state #查看firewalld的状态
systemctl status firewalld #查看firewalld的状态,详细

firewall-cmd --reload 重新载入防火墙配置，当前连接不中断
firewall-cmd --complete-reload 重新载入防火墙配置，当前连接中断

firewall-cmd --get-services 列出所有预设服务
firewall-cmd --list-services 列出当前服务
firewall-cmd --permanent --zone=public --add-service=smtp 启用服务
firewall-cmd --permanent --zone=public --remove-service=smtp 禁用服务

firewall-cmd --zone=public --list-ports 
firewall-cmd --permanent --zone=public --add-port=8080/tcp 启用端口
firewall-cmd --permanent --zone=public --remove-port=8080/tcp 禁用端口
firewall-cmd --zone="public" --add-forward-port=port=80:proto=tcp:toport=12345 同服务器端口转发 80端口转发到12345端口
firewall-cmd --zone=public --add-masquerade 不同服务器端口转发，要先开启 masquerade
firewall-cmd --zone="public" --add-forward-port=port=80:proto=tcp:toport=8080:toaddr=192.168.1.1 不同服务器端口转发，转发到192.168.1.1的8080端口

firewall-cmd --get-zones 查看所有可用区域
firewall-cmd --get-active-zones 查看当前活动的区域,并附带一个目前分配给它们的接口列表
firewall-cmd --list-all-zones 列出所有区域的所有配置
firewall-cmd --zone=work --list-all 列出指定域的所有配置
firewall-cmd --get-default-zone 查看默认区域
firewall-cmd --set-default-zone=public 设定默认区域

firewall-cmd --get-zone-of-interface=eno222
firewall-cmd [--zone=<zone>] --add-interface=<interface> 添加网络接口
firewall-cmd [--zone=<zone>] --change-interface=<interface> 修改网络接口
firewall-cmd [--zone=<zone>] --remove-interface=<interface> 删除网络接口
firewall-cmd [--zone=<zone>] --query-interface=<interface> 查询网络接口

irewall-cmd --permanent --zone=internal --add-source=192.168.122.0/24 设置网络地址到指定的区域
firewall-cmd --permanent --zone=internal --remove-source=192.168.122.0/24 删除指定区域中的网路地址

firewall-cmd --get-icmptypes
```
##Rich Rules
- firewall-cmd –list-rich-rules 列出所有规则
- firewall-cmd [–zone=zone] –query-rich-rule=’rule’ 检查一项规则是否存在
- firewall-cmd [–zone=zone] –remove-rich-rule=’rule’ 移除一项规则
- firewall-cmd [–zone=zone] –add -rich-rule=’rule’ 新增一

##Direct Rules
- firewall-cmd –direct –add-rule ipv4 filter IN_public_allow 0 -p tcp –dport 80 -j ACCEPT 添加规则
- firewall-cmd –direct –remove-rule ipv4 filter IN_public_allow 10 -p tcp –dport 80 -j ACCEPT 删除规则
- firewall-cmd –direct –get-all-rules 列出规则




