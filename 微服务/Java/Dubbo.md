## 一、什么是dubbo

### 1.概述

Apache Dubbo是一款易用、高性能的WEB和RPC框架,同时为构建企业级**微服务**提供服务发现、流量治理、可观测、认证鉴权等能力、工具与最佳实践。

阿里提出的DNS微服务解决方案：DNS =》 dubbo + nacos + sentinel

### 2.架构发展

SOA【 Service-oriented Architecture面向服务的架构】= RPC+服务治理

单体 =》水平扩展 =》垂直扩展 =》RPC =》SOA =》ESB（企业服务总线） =》 微服务

- 单体架构

  ![image-20240413142827477](../../../../picbed/store/picbed/img/image-20240413142827477.png)

- 水平架构：比较单体架构,水平分割架构的好处是

  1.提高了系统的稳定性一台节点出现问题,不会影响整个系统

  2.提高了系统的硬件支撑

  3.提高了系统并发的能力但是根本上没有解决单体架构的问题

  ![image-20240413142659050](../../../../picbed/store/picbed/img/image-20240413142659050.png)

- 垂直架构：把一个单体架构的应用，按照子系统进行了划分,每个子系统都独立部署在自己的 tomcat(JWM进程），多个子系统共享数据库等存储资源。

  ![image-20240413142619186](../../../../picbed/store/picbed/img/image-20240413142619186.png)

- RPC架构

  ![image-20240413142330190](../../../../picbed/store/picbed/img/image-20240413142330190.png)

- SOA架构

  ![image-20240413142305098](../../../../picbed/store/picbed/img/image-20240413142305098.png)

- **ESB架构** ：java消息转为 中间状态（grpc probuf/thrift idl） 给其他语言。

![image-20240413141513223](../../../../picbed/store/picbed/img/image-20240413141513223.png)

- **微服务架构**

  是SOA框架的升级，没有子系统了，全部服务化。代表框架：springcloud、DNS

  ![image-20240413142916310](../../../../picbed/store/picbed/img/image-20240413142916310.png)

## 二、Dubbo程序开发

### 1.代码结构与术语

1. provider           功能提供者
2. consumer        功能调用者【功能消费者】
3. commons-api  通用内容
4. registry            注册中心

![image-20240413143436317](../../../../picbed/store/picbed/img/image-20240413143436317.png)

### 2.序列化

![image-20240413144205428](../../../../picbed/store/picbed/img/image-20240413144205428.png)

#### 1.简介

序列化:是 Dubbo进行RPC中,非常重要的一个组成部分其核心作用就是把网络传输中的数据,按照特定的格进行传输。减小数据的体积,从而提高传输效率。

#### 2.实现方式

![image-20240413144307843](../../../../picbed/store/picbed/img/image-20240413144307843.png)





## 源码解读

![image-20240413150211454](../../../../picbed/store/picbed/img/image-20240413150211454.png)