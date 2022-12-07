## Docker命令解析

### 镜像命令

docker images 查看所有本地的主机上的镜像

```shell
> docker images

# 结果解释
REPOSITORY 镜像的仓库源
TAG 镜像的标签
IMAGE ID 镜像的id
CREATED 镜像的创建时间
SIZE 镜像的大小

# 可选项
-a  --all  # 列出所有镜像
-q  --quiet # 只显示镜像的id
```

docker search 搜索镜像

```shell
> docker search mysql

# 可选项
--filter=STARS=3000 # 搜索出来的镜像就是STARS大于3000的

> docker search mysql --filter=STARS=3000
```

docker pull 下载镜像

```shell
# 下载镜像 docker pull 镜像名[:版本]
> docker pull mysql # 默认就是最新版本

#分层下载 docker image的核心  联合文件系统

Digest  # 签名
docker.io/library/mysql:latest # 真实地址


# 等价与
docker pull docker.io/library/mysql:latest
```

docker rmi 删除镜像

```shell
docker rmi -f IMAGE-ID # 删除指定的镜像
docker rmi -f IMAGE-ID IMAGE-ID IMAGE-ID # 删除多个镜像
docker rmi -f $(docker images -aq) # 删除全部的镜像
```

docker查看容器日志

```shell
docker logs 容器ID
```

### 容器命令

新建容器并启动

```shell
docker run [可选参数] image

# 参数说明
--name="Name" 容器名字 tomcat01 tomcat02 用来区分容器
-d 后台方式运行
-it 使用交互方式运行，进入容器查看内容
-p     指定容器的端口 -p 8080:8080
    -p ip:主机端口:容器端口
    -p 主机端口:容器端口    （常用）
    -p 容器端口
    容器端口
-P  随机指定端口

# 启动并进入容器
> docker run -it centos /bin/bash

# 从容器中退回主机
exit
```

​    列出所有的运行的容器

```shell
docker ps 命令
-a # 列出当前正在运行的容器 + 带出历史运行过的容器
-n=? # 显示最近创建的容器
-q # 只显示容器的编号
```

退出容器

```shell
exit # 直接容器停止并退出
Ctrl + P + Q # 容器不停止退出
```

删除容器

```shell
docker rm 容器id # 删除指定容器 不能删除正在运行的容器，除非rm -f    
docker rm -f $(docker ps -aq) # 删除所有容器
docker ps -a -q | xargs docker rm # 删除所有容器
```

启动和停止容器

```shell
docker start 容器id
docker restart 容器id
docker stop 容器id
docker kill 容器id # 强制停止当前容器
```

其他命令

```shell
# 后台启动容器
docker run -d 镜像名

# 查看日志
-tf # 显示日志
--tail number # 要显示日志条数
docker logs -tf --tail 10 容器id


# 查看容器中进程信息
docker top 容器id

# 查看镜像的元数据
docker inspect 容器id

# 进入当前正在运行的容器
docker exec -it 容器id /bin/bash # 进入容器后开启一个新的终端，可以在里面操作

docker attach 容器id  # 进入容器正在执行的终端，不会启动新的进程

# 从容器内拷贝文件到主机上
docker cp 容器id:容器内路径 目的地主机路径

# 拷贝是一个手动的过程，可以使用数据卷方式实现
```

### 命令练习

#### Nginx部署

```shell
docker search
docker pull nginx
docker run -d --name nginx01 -p 3344:80 nginx
curl loaclhost:3344
docker exec -it nginx01 /bin/bash
```

#### Tomcat部署

```shell
docker run -it  --rm tomcat:9.0
## 一般用作测试，用完就删除

docker pull tomcat
docker run -d -p 3355:8080 --name tomcat01 tomcat
docker exec -it tomcat01 /bin/bash
```
