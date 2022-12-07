# Kubernetes概念

## 概述

- k8s是谷歌在2014年开源的容器化集群管理系统
- 使用k8s进行容器化应用部署
- 使用k8s利于应用扩展
- k8s目标实施让部署容器化应用更加简洁和高效

## 集群架构组件

### Master 主控节点

#### apiserver

集群统一入口，以restful方式，交给etcd存储

#### scheduler

节点调度，选择node节点应用部署

#### controller-manager

处理集群中常规后台任务，一个资源对应一个控制器

#### etcd

存储系统，用于保存集群相关的数据

### Node 工作节点

#### kubelet

master排到node节点代表，管理本机容器

#### kube-proxy

提供网络代理，负载均衡等操作

## 核心概念

### Pod

- 最小部署单元
- 一组容器的集合
- 共享网络
- 生命周期是短暂的

### Controller

- 确保预期的pod副本数量
- 无状态应用部署/有状态应用部署
- 确保所有的node运行同一个pod
- 一次性任务/定时任务

### Service

- 定义一组pod的访问规则

### 

