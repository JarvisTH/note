# 使用Minikube创建集群

## 一、Kubernetes集群

**Kubernetes协调连接在一起作为一个单元工作的高可用性计算机集群**，包含两种类型的资源：

- Master节点：**负责管理群集**
- Node：运行应用程序的工作程序，**充当Kubernetes集群中的辅助计算机的VM或物理计算机。**每个节点都有一个Kubelet，Kubelet是用于管理节点并与Kubernetes主节点通信的代理。该节点还应该具有用于处理容器操作的工具，生产的Kubernetes集群至少应具有三个节点。

![img](https://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)

## 二、Minikube

Minikube是一种轻量级的Kubernetes实现，可在本地计算机上创建VM并部署仅包含一个节点的简单集群。Minikube CLI提供了用于引导群集的基本引导程序操作，包括启动，停止，状态和删除。