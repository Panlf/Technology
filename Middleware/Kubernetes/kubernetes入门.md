# Kubernetes

## Kubernetes概述

### Kubernetes是什么

Kubernetes(K8s)用于自动化容器化应用程序的部署、扩展和管理。Kubernetes通常结合docker容器工作，并且整合多个运行者docker容器额主机集群。

**特性**

- 自动包装
- 自我修复
- 横向缩放
- 服务发现和负载均衡
- 自动部署和回滚
- 秘钥和配置管理
- 批处理

Kubernetes是一个基于容器技术的分布式架构领先方案，完备的分布式系统、完善的集群管理能力的支撑平台。

使用Kubernetes可以在物理或虚拟机的Kubernetes集群上运行容器化应用，Kubernetes能够提供一个以容器为中心的基础架构，满足在生产环境中运行应用的一些常见需求。

- 多个进程协同工作
- 存储系统挂载
- Distributing secrests
- 应用健康检测
- 应用实例的复制
- Pod自动伸缩/扩展
- Naming and discovering
- 负载均衡
- 滚动更新
- 资源监控
- 日志访问
- 调度应用程序
- 提供认证和授权

### Kubernetes快速入门

#### 环境准备

关闭Centos防火墙

```shell
systemctl disable firewalld
systemctl stop firewalld
```

安装etcd和kubernetes软件

```shell
yum install -y etcd kubernetes
```

启动服务

```shell
systemctl start etcd
systemctl start docker
```

> 如果docer启动失败，请参考（vi /etc/sysconfig/selinux 把selinux后面的改成disabled，重启机器，再重启docker就可以了）

​	

```shell
systemctl start kube-apiserver
systemctl start kube-controller-manager
systemctl start kube-seheduler
systemctl start kubelet
systemctl start kube-proxy
```

 #### 配置

Tomcat配置

- mytomcat.rc.yaml

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
	name: mytomcat
spec:
	replicas: 2
	selector:
		app: mytomcat
	template:
		metadata:
			labels:
				app: mytomcat
		spec:
			containers:
				- name: mytomcat
				  image: tomcat:7-jre7
				  ports:
				  - containerPort: 8080
```

kubectl create -f mytomcat.rc.yaml

- mytomcat.svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
	name: mytomcat
spec:
	type: NodePort
	ports:
		- port: 8080
		  nodePort: 30001
	selector:
		app: mytomcat
```

kubectl create -f mytomcat.svc.yaml

### Kubernetes基本架构

Kubernetes主要由以下几个核心组件组成

- etcd保存了整个集群的状态
- apiserver提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制
- controller manager 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等。
- scheduler负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上
- kubectl负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理
- Container runtime负责镜像管理以及Pod和容器的真正运行（CRI）
- kube-proxy 负责为Service提供cluster内部的服务发现和负载均衡

除了核心组件，还有一些推荐的Add-ons

- kube-dns负责为整个集群提供DNS服务
- Ingress Controller 为服务提供外网入口
- Heapster提供资源监控
- Dashboard提供GUI
- Federation提供跨可用区的集群
- Fluented-elasticsearch提供集群日志采集、存储与查询

Kubernetes设计理念和功能其实就是一个类似Linux的分层架构。

- 核心层 Kubernetes最核心的功能，对外提供API构建高层的应用，对内提供插件式应用执行环境
- 应用层 部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS解析等）
- 管理层 系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态Provision等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy等）
- 接口层 kubectl命令行工具、客户端SDK以及集群联邦
- 生态系统 在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴
  - Kubernetes外部：日志、监控、配置管理、CI、CD、Workflow、Faas、OTS应用、ChatOps等
  - Kubernetes内部：CRI、CNI、CVI、镜像仓库、Cloud Provider、集群自身的配置和管理等

#### Cluster

Cluster是计算、存储和网络资源的集合，Kubernetes利用这些资源运行各种基于容器的应用。

Kubernetes Cluster由Master和Node组成，节点上运行着若干个Kubernetes服务。

#### Master

Master主要职责是调度，即决定将应用放在哪运行。Master运行Linux系统，可以是物理机或虚拟机。Master是Kubernetes Cluster的大脑，运行着的Daemon服务包括kube-apiserver、kube-scheduler、kube-controller-manager、etcd和Pod网络

- API Server（kube-apiserver）

API Server提供HTTP/HTTPS RESTful API，即Kubernetes API，是Kubernetes里所有资源的CRUD等操作的唯一入口，也是集群控制的入口进程

- Scheduler（kube-scheduler）

Scheduler负责资源调度的里程，简单说，它决定将Pod放在哪个Node上运行

- Controller Manager（kube-controller-manager）

所有资源对象的自动化控制中心。Controller Manager负责管理Cluster各种资源，保证资源处于预期的状态。Controller Manager有多种，如replication controller、endpoints controller、namespace controller、serviceaccounts controller等。

不同的controller管理不同的资源，如replication controller管理Deployment、StatefulSet、DaemonSet的生命周期，namespace controller管理Namespace资源

- etcd

etcd负责保存Kubernetes  Cluster 的配置信息和各种资源的状态信息。当数据发生变化时，etcd会快速地通知Kubernetes 相关组件

- Pod网络

Pod要能够相互通信，Kubernetes Cluster必须部署Pod网络，flannel是其中一个可选方案。

#### Node

除了Master、Kubernets集群中的其他机器被称为Node节点。Node职责是运行容器应用，Node由Master管理，Node负责监控并汇报容器的状态，同时根据Master的要求管理容器的生命周期。Node也运行在Linux系统，可以是物理机或虚拟机。

每个Node节点上都运行着以下一组关键进程。

- kubelet

负责Pod对应的容器的创建、启动等任务，同时与Master节点密切协作，实现集群管理的基本功能。

- kube-proxy

实现Kubernetes Service的通信与负载均衡机制的重要组件。

- Docker Enginer

Docker引擎，负责本机的容器创建和管理工作。

#### Pod

Pod是Kubernetes的最小单元，也是最重要和最基本概念。每个Pod包含一个或多个容器，Pod的容器会作为一个整体被Master调度到一个Node上运行。Kubernetes为每个Pod都分配了唯一的IP地址，称为PodIP，一个Pod里的多个容器共享PodIP地址。在Kubernetes里，一个Pod里的容器与另外主机上的Pod容器能够直接通信。

#### Service

Kubernetes Service定义了外界访问一组特定的Pod方式，Service有自己的IP和端口，Service为Pod提供了负载均衡。他也是Kubernetes最核心的资源对象之一，每个Service其实就是我们经常提起的微服务架构中的一个微服务。

#### Replication Controller

Replication Controller是Kubernetes系统中的核心概念之一，它其实是定义了一个期望的场景，即声明某种Pod的副本数量在任意时刻都符合某个预期值，所以RC的定义包括如下几部分

- Pod期待的副本数（replicas）
- 用于筛选目标Pod的Label Selector
- 当Pod的副本数小于预期数量时，用于创建新Pod的Pod模板（template）

RC的特性与作用

- 在大多数情况下，我们通过定义一个RC实现Pod的创建过程及副本数量的自动控制
- RC里包括完整的Pod定义模板
- RC通过Label Selector机制实现对Pod副本的自动控制
- 通过改变RC里的Pod副本数量，可以实现Pod的扩容或缩容功能
- 通过改变RC里的Pod模板中镜像版本，可以实现Pod的滚动升级功能。

## Kubernetes集群

Kubernetes用于协调高度可用的计算机集群，这些计算机群集被连接作为单个单元工作。Kubernetes在一个集群上以更有效的方式自动分发和调度容器应用程序。Kubernetes集群由两种类型的资源组成：

- Master是集群的调度节点
- Nodes是应用程序实际运行的工作节点

Kubernetes集群部署的方式：kubeadm、minikube和二进制包。

### 环境准备

| 角色   | ip              | 组件                                                         |
| ------ | --------------- | ------------------------------------------------------------ |
| master | 192.168.126.140 | etcd、kube-apiserver、kube-controller-manager、kube-scheduler、docker |
| node01 | 192.168.126.141 | kube-proxy、kubelet、docker                                  |
| node02 | 192.168.126.142 | kube-proxy、kubelet、docker                                  |

- 查看默认防火墙状态（关闭后显示not running，开启后显示running）

```
firewall-cmd --state
```

- 关闭防火墙

```
systemctl stop firewalld.service
```

- 获取Kubernetes二进制包

https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG-1.9.md

### Master安装

#### Docker安装

设置yum源

```
vi etc/yum.repos.d/docker.repo

[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/$releaserver
enabled=1
gpcheck=1
gpgkey=https://yum.dockerproject.org/gpg
```

安装docker

```
yum install dokcer-engine
```

安装后查看docker版本

```
docker -v
```

#### etcd服务

etcd作为Kubernetes集群的主要服务，在安装Kubernetes服务前需要首先安装和启动

-  下载etcd二进制文件

https://github.com/etcd-io/etcd/releases

- 将文件上传至服务器，解压
- 将etcd和etcdctl文件夹复制到/usr/bin目录
- 配置systemd服务文件 /usr/lib/systemd/system/etcd.service

```
[unit]
Description=Etcd Server
After=network.target
[Service]
Type=simple
EnvironmentFile=/etc/etcd/etcd.conf
WorkingDirectory=/var/lib/etcd/
ExecStart=/usr/bin/etcd
Restart=on-failure
[Install]
WantedBy=multi-user.target
```

- 启动与测试etcd服务

```
systemctl daemon-reload
systemctl enable etcd.service
mkdir -p /var/lib/etcd/
systemctl start etcd.service
etcdctl cluster-health
```

#### kube-apiserver服务

解压后将kube-apiserver、kube-controller-manager、kube-scheduler以及管理要使用的kubectl二进制命令文件放到/usr/bin目录，即完成这几个服务安装

```
cp kube-apiserver kube-controller-manager kube-scheduler kubectl /usr/bin
```

下面是对kube-apiserver服务进行配置

编辑systemd服务文件 `vi /usr/lib/systemd/system/kube-apiserver.service`

```
[unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
After=etcd.service
wants=etcd.service
[Service]
EnvironmentFile=/etc/kubernetes/apiserver
ExecStart=/usr/bin/kube-apiserver $KUBE_API_ARGS
Restart=on-failure
Type=notify
[Install]
WantedBy=multi-user.target
```

配置文件

创建目录 mkdir /etc/kubernetes

vi /etc/kubernetes/apiserver

```
KUBE_API_ARGS="--storage-backend=etcd3 --etcd-servers=http://127.0.0.1:2379 --insecure-bind-address=0.0.0.0 --insecure-port=8080 --service-cluster-ip-range=169.169.0.0/16 --service-node-port-range=1-65535 --admission-control=NamespaceLifecycle,NamespaceExists,LimitRanger,SecurityContextDeny,ServiceAccount,DefaultStorageClass,ResourceQuota --logtostderr=true --log-dir=/var/log/kubernetes --v=2"
```

#### kube-controller-manager服务

kube-controller-manager服务依赖于kube-apiserver服务

配置systemd服务文件 `vi /usr/lib/systemd/system/kube-controller-manager.service`

```
[unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=kube-apiserver.service
Requires=kube-apiserver.service
[Service]
EnvironmentFile=/etc/kubernetes/controller-manager
ExecStart=/usr/bin/kube-controller-manager $KUBE_CONTROLLER_MANAGER_ARGS
Restart=on-failure
LimitMOFILE=65536
[Install]
WantedBy=multi-user.target
```

配置文件 `vi /etc/kubernetes/controller-manager`

```
KUBE_CONTROLLER_MANAGER_ARGS="--master=http://192.168.126.140:8080 --logtostderr=true --log-dir=/var/log/kubernetes --v=2"
```

#### kube-scheduler服务

kube-scheduler服务也依赖于kube-apiserver服务。

配置systemd服务文件，`vi /usr/lib/systemd/system/kube-scheduler.service`

```
[unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=kube-apiserver.service
Requires=kube-apiserver.service
[Service]
EnvironmentFile=/etc/kubernetes/scheduler
ExecStart=/usr/bin/kube-scheduler $KUBE_SCHEDULER_ARGS
Restart=on-failure
LimitMOFILE=65536
[Install]
WantedBy=multi-user.target
```

配置文件，`vi /etc/kubernetes/scheduler`

```
KUBE_SCHEDULER_ARGS="--master=http://192.168.126.140:8080 --logtostderr=true --log-dir=/var/log/kubernetes --v=2"
```

#### 启动

启动服务

```
systemctl deamon-reload
systemctl enable kube-api-server.service
systemctl start kube-api-server.service
systemctl enable kube-controller-manager.service
systemctl start kube-controller-manager.service
systemctl enable kube-scheduler.service
systemctl start kube-scheduler.service
```

检查每个服务的健康状态

```
systemctl status kube-api-server.service
systemctl status kube-controller-manager.service
systemctl status kube-scheduler.service
```

### Node1安装

在Node1节点上，以同样的方式把从压缩包中解压出的二进制文件kubelet kube-proxy放到/usr/bin目录中。

在Node1节点上需要预先安装docker，请参考Master上Docker的安装，并启动Docker

#### kubelet服务

配置systemd服务文件，`vi /usr/lib/systemd/system/kubelet.service`

```
[unit]
Description=Kubernetes kubelet Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=docker.service
Requires=docker.service
[Service]
WorkingDirectory=/var/lib/kubelet
EnvironmentFile=/etc/kubernetes/kubelet
ExecStart=/usr/bin/kubelet $KUBELET_ARGS
Restart=on-failure
killMode=process
[Install]
WantedBy=multi-user.target
```

mkdir -p /var/lib/kubelet

配置文件 `vi /etc/kubernetes/kubelet`

```
KUBELET_ARGS="-kubeconfig=/etc/kubernetes/kubeconfig --hostname-override=192.168.126.141 --logtostderr=false --log-dir=/var/log/kubernetes --v=2 --fail-swap-on=false"
```

用于kubelet连接Master Apiserver的配置文件

vi /etc/kubernetes/kubeconfig

```
apiVersion: v1
kind: Config
clusters:
	- cluster:
		server: http://192.168.126.140:8080
	name: local
contexts:
	- context:
		cluster: local
	name: mycontext
current-context: mycontext
```

#### kube-proxy服务

kube-proxy服务依赖于network服务，所以一定要保证network服务正常。

配置systemd服务文件，`vi /usr/lib/systemd/system/kube-proxy.service`

```
[unit]
Description=Kubernetes Kube-proxy Server
Documentation=https://github.com/GoogleCloudPlatform/kubernetes
After=network.service
Requires=network.service
[Service]
EnvironmentFile=/etc/kubernetes/proxy
ExecStart=/usr/bin/kube-proxy $KUBE_PROXY_ARGS
Restart=on-failure
LimitMOFILE=65536
killMode=process
[Install]
WantedBy=multi-user.target
```

配置文件 vi /etc/kubernetes/proxy

```
KUBE_PROXY_ARGS="--master=http://192.168.126.140:8080 --hostname-override=192.168.126.141 --logtostderr=true --log-dir=/var/log/kubernetes --v=2"
```

启动

```
systemctl daemon-reload
systemctl enable kubelet
systemctl start kubelet
systemctl status kubelet
systemctl enable kube-proxy
systemctl start kube-proxy
systemctl status kube-proxy
```



### Node2安装

同上

## 健康检查

- kubectl get nodes
- kubectl get cs
