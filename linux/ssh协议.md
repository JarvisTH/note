## 一、概念

SSH——Secure-Shell的缩写，安全外壳协议。转为远程登录会话和其他网络服务提供安全性的协议。对数据包加密在传输。

基本框架：

![image-20230102170117125](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230102170117125.png)

## 二、SSH工作过程

- SSH版本协商：TCP连接后，协商版本号。
- 密钥和算法的协商：客户端与服务端支持的算法不完全相同，C/S各自发送自己支持的算法给对方进行协商。
- 认证阶段：进行请求的认证
- 会话请求阶段：服务端等待客户端的请求，然后进行处理返回
- 交互会话阶段：

## 三、shell中ssh的使用

### 1.SSH非免密登录执行——基于口令的验证

![image-20230102171116986](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230102171116986.png)



### 2.SSH免密登录执行——基于密钥的验证

![image-20230102171230433](https://cdn.jsdelivr.net/gh/JarvisTH/picbed/img/image-20230102171230433.png)