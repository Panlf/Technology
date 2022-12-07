## Docker内核理解

### 容器技术原理

#### chroot

chroot是在Unix和Linux系统的一个操作，针对正在运行的软件行程和它的子进程，改变它外显的根目录。一个运行在这个环境下，经由chroot设置根目录的程序，它不能够对这个指定根目录之外的文件进行访问动作，不能读取，也不能更改它的内容。

#### Namespace

Namespace对内核资源进行隔离，使得容器中的进程都可以在单独的命名空间中运行并且只可以访问当前容器命名空间的资源

Namespace可以隔离进程ID、主机名、用户ID、文件名、网络访问和进程间通信等相关资源

命名空间

- pid namespace 隔离进程 ID
- net namespace 隔离网络接口
- mnt namespace 文件系统挂载点隔离
- ipc namespace 信号量，消息队列和共享内存的隔离
- uts namespace 主机名和域名的隔离

#### Cgroup

一种Linux内核功能，可以限制和隔离进程资源使用情况（CPU、内存、磁盘I/O、网络等）

#### 联合文件系统

又叫UnionFS，是一种通过创建文件层进程操作的文件系统

常见的联合文件系统有AUFS、Overlay和Devicemapper等

### 核心概念

#### 镜像

镜像是一个只读的文件和文件夹组合，是Docker容器启动的先决条件

#### 容器

容器是镜像的运行实体

容器运行着真正的应用进程

容器有初建、运行、停止、暂停和删除五种状态

在容器内部，无法看到主机上的进程、环境变量、网络等信息

#### 仓库

Docker的镜像仓库用来存储和分发Docker镜像

公共镜像仓库和私有镜像仓库

### 镜像使用

#### 拉取镜像

`docker pull [registry]/[repository]/[image]:[tag]`

registry 注册服务器

repository 镜像仓库

image 镜像名称

tag 镜像的标签

#### 查看镜像

`doker images`命令列出本地所有的镜像
