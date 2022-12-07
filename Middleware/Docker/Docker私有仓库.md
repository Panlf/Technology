## Docker私有仓库

### 搭建私有仓库

```shell
# 拉取私有仓库镜像
docker pull registry

# 启动私有仓库
docker run -id --name=registry -p 5000:5000 registry

# 打开浏览器，输入地址http://私有仓库服务器ip:5000/v2/_catalog，看到{"repositories":[]}表示私有仓库 搭建成功

# 修改daemon.json
vim /etc/docker/daemon.json

# 在上述文件中添加一个key，保存退出。此步用于让docker信任私有仓库地址；注意将私有仓库服务器ip修改为自己私有仓库服务器真实ip
{"insecure-registries":["私有仓库服务器ip:5000"]}
# 重启docker服务
systemctl restart docker
docker start registry
```

### 上传镜像到私有仓库

```shell
# 标记镜像为私有仓库的镜像
docker tag centos:7 私有仓库服务器IP:5000/centos:7

# 上传标记的镜像
docker push 私有仓库服务器IP:5000/centos:7
```

### 从私有仓库拉取镜像

```shell
# 拉取镜像
docker pull 私有仓库服务器ip:5000/centos:7
```
