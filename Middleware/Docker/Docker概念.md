## Docker概念

- Docker是一个开源的应用容器引擎
- 基于Go语言实现
- Docker可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的容器 中，然后发布到任何流行的Linux机器上。
- 容器是完全使用沙箱机制，相互隔离
- Docker从17.03版本之后分为CE（Community Edition社区版）和EE（Enterprise Edition 企业版）

文档地址：https://docs.docker.com

仓库地址：https://hub.docker.com

**小结** 容器化技术不是模拟的一个完整的操作系统，docker是一种容器技术，解决软件跨环境迁移的问题

### Docker与虚拟机比较

容器就是将软件打包成标准化单元，以用于开发、交付和部署

- 容器镜像是轻量的、可执行的独立软件包，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置。
- 容器化软件在任何环境中都能够始终如一的运行。
- 容器赋予了软件独立性，使其免受外在环境差异的影响，从而有助于减少团队间在相同的基础设施上运行不同软件时的冲突。

相同

- 容器和虚拟机具有相似的资源隔离和分配优势

不同

- 容器虚拟化的是操作系统，虚拟机虚拟化的是硬件
- 传统虚拟机可以运行不同的操作系统，容器只能运行同一类型操作系统
- 传统虚拟机，虚拟出一条硬件，运行一个完整的操作系统，然后在这个系统上安装和运行软件
- 容器内的应用直接运行在宿主机的内容，容器是没有自己的内核的，也没有虚拟我们的硬件，所以轻便
- 每个容器是互相隔离的，每个容器内都有一个属于自己的文件系统，互不影响。

### Docker为什么比虚拟机快

- Docker有着比虚拟机更少的抽象层
- Docker利用的是宿主机的内核，vm需要是Guest OS
- 新建一个容器的时候，Docker不需要像虚拟机一样重新加载一个操作系统的内核，避免引导。虚拟机是加载Guest OS，分钟级别的，而Docker是利用宿主机的操作系统，省略了这个复杂的过程，秒级。

## 安装Docker

Centos

```shell
# 1. yum 包更新到最新
yum update
# 2. 安装需要的软件包，yum-util提供yum-config-manager功能，另外两个是devicemapper驱动依赖的
yum install -y yum-utils device-mapper-persistent-data lvm2
# 3.设置yum源
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
# 4.安装docker
# yum install -y docker-ce docker-ce-cli containerd-io

# 自动选择y，全自动
yum install -y docker-ce

# 5.查看docker版本，验证是否安装成功
docker -v
```

### 卸载Docker

```shell
# yum remove docker-ce docker-ce-cli containerd-io
yum remove docker-ce

rm -rf /var/lib/docker
```

## Docker架构

- Clients
- Hosts
- Registries

基本名词

- 镜像（Image） Docker镜像（Image），就相当于一个root文件系统。比如官方镜像ubuntu:16.04就包含了完整的一套Ubuntu16.04最小系统的root文件系统。
  
  Docker镜像就好比一个模板，可以通过这个模板来创建容器服务。通过这个镜像可以创建多个容器（最终服务运行或者项目运行就是在容器中）。

- 容器（Container）镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和对象一样，镜像是静态的的定义，容器是镜像运行的实体。容器可以被创建、启动、停止、删除、暂停等。
  
  Docker利用容器技术，独立运行一个或者一个组应用，通过镜像创建。

- 仓库（Repository）仓库可看成一个代码控制中心，用来保存镜像
  
  分为公有仓库和私有仓库

Docker是一个Client - Server 结构的系统，Docker的守护进程运行在主机上，通过Socket从客户端访问。DockerServer接收到Docker - Client的指令，就会执行这个命令。

## 镜像加速器

默认情况下，将来从docker hub（https://hub.docker.com/）上下载docker镜像，太慢。一般都会配置镜像加速器

- USTC 中科大镜像加速器（https://docker.mirrors.ustc.edu.cn）
- 阿里云
- 网易云
- 腾讯云

配置阿里云的镜像加速器，先登录阿里云的镜像加速服务，然后里面就可以直接粘贴语句。

```shell
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://5d4jvnbw.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## Docker命令

### Docker帮助命令

- docker -version # 显示docker的版本信息
- docker info  # 显示docker的系统信息
- docker 命令 --help # 帮助命令

### Docker服务相关的命令

- 启动服务 systemctl start docker
- 查看服务状态 systemctl status docker
- 停止服务 systemctl stop docker
- 重启服务 systemctl restart docker
- 开机启动 systemctl enable docker

### Docker镜像相关命令

- 查看镜像 docker images
- 查看所有镜像ID docker images -q
- 搜索镜像 docker search redis
- 拉取镜像 docker pull redis

拉取镜像也可以指定版本拉取，比如`docker pull redis:5.0` ，但是具体可以支持什么版本需要去`https://hub.docker.com`去查找对应的版本。

- 删除镜像 docker rmi [IMAGE ID]
- 删除所有的镜像 docker rmi \`docker iamges -q\`

### Docker容器相关命令

- 查看正在运行容器  docker ps     
  
  查看所有容器 docker ps -a

- 创建并启动容器 docker run 参数

参数说明：

​            -i 保持容器运行，通常与-t同时使用。加入it这两个参数后，容器创建后自动进入容器中，退出容器后，容器自动关闭。

​           -P 随机端口映射

​            -t 为容器重新分配一个伪输入终端，通常与-i同时使用

​            -d 以守护（后台）模式运行容器。创建一个容器在后台运行，需要使用docker exec进入容器。退出后，容器不会关闭。

​            -it 创建的容器一般称为交互式容器，-id创建的容器一般称为守护式容器

​            --name 为创建的容器命名

- 进入容器  docker exec 参数 
- 启动容器 docker start 容器名
- 停止容器 docker stop 容器名
- 删除容器 docker rm [容器ID|容器名]
- 删除所有容器 docker rm \`docker ps -aq\`  

如果容器是运行状态则删除失败，需要停止容器才能删除

- 查看容器信息 docker inspect 容器名

## 容器数据卷

将应用和环境打包成一个镜像，如果数据都在容器中，那么我们容器删除，数据就会丢失！需求：数据可以持久化

容器之间可以有一个数据共享的技术！Docker容器中产生的数据，同步到本地！

这就是卷技术！目录的挂载，将我们容器内的目录，挂载到Linux上面！

- 数据卷是宿主机中的一个目录或者文件
- 当容器目录和数据卷目录绑定后，对方的修改会立即同步
- 一个数据卷可以被多个容器同时挂载
- 一个容器也可以被挂载多个数据卷

数据卷作用

- 容器数据持久化
- 外部机器和容器间通信
- 容器之间数据交换

### 配置数据卷

创建启动容器时，使用-v参数设置数据卷
`docker run ... -v 宿主机目录(文件):容器内目录(文件) -v 宿主机目录(文件):容器内目录(文件) ...`

注意事项

- 1、目录必须是绝对路径
- 2、如果目录不存在会自动创建
- 3、可以挂载多个数据卷

### 数据卷容器

多容器进行数据交换

1、多个容器挂载同一个数据卷

2、数据卷容器

总结：容器的持久化和同步操作！容器间也是可以数据共享的！

#### 数据卷容器设置

1、创建启动c3数据卷容器，使用-v参数设置数据卷容器

`docker run -it --name=c3 -v /volume centos:7 /bin/bash`

volume 可以自定义

2、创建启动c1 c2容器，使用--volumes-from 参数 设置数据卷 ，容器间的数据共享

`docker run -it --name=c1 --volumes-from c3 centos:7 /bin/bash`

`docker run -it --name=c2 --volumes-from c3 centos:7 /bin/bash`

### 具名和匿名挂载

1、匿名挂载

```shell
# 匿名挂载
# -v 容器内路径
docker run -d -P --name nginx01 -v /etc/nginx nginx

# 查看所有的volume的情况
docker volume ls

# 这种就是匿名挂载，我们在-v只写了容器内的路径，没有写容器外的路径
```

2、具名挂载

```
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx nginx 
```

```shell
-v 容器内路径 # 匿名挂载
-v 卷名:容器内路径 # 具名挂载
-v /宿主机路径:/容器内路径 # 指定路径挂载
```

拓展

```shell
# 通过 -v 容器内路径 ro rw 改变读写权限
ro readonly
rw readwrite

# 一旦设置了容器权限，容器对我们挂载出来的内容就有限定了
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:ro nginx
docker run -d -P --name nginx02 -v juming-nginx:/etc/nginx:rw nginx

# ro 只能通过宿主机来操作，容器内部是无法操作的
```

### 容器卷实战

MySQL数据同步

```shell
docker pull mysql:8.0

docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d \
-v /home/mysql/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:8.0
```

多个MySQL实现数据共享

```shell
docker run -d -p 3310:3306 -v /etc/mysql/conf/d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 --name mysql01 mysql:8.0 
docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=123456 --name mysql02 --volumes-from mysql01 mysql:8.0 
```

## 可视化

- portainer

```
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock \
--privileged=true portainer/portainer
```

- Rancher
