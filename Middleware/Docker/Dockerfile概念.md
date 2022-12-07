## Dockerfile

### Docker镜像原理

- Docker镜像是由特殊的文件系统叠加而成
- 最底端是bootfs，并使用宿主机的bootfs
- 第二层是root文件系统rootfs，称为base image
- 然后再往上可以叠加其他镜像文件
- 统一文件系统（Union File System）技术能够将不同的层整合成一个文件系统，为这些层提供了一个统一的视角，这样就隐藏了多层的存在，在用户的角度看来，只存在一个文件系统。
- 一个镜像可以放在另一个镜像的上面。位于下面的镜像称为父镜像，最底部的镜像成为基础镜像。
- 当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器

*思考*

1、Docker镜像本质是什么？

是一个分层文件系统

2、Docker中一个centos镜像为什么只有200MB，而一个centos操作系统的ios文件要几个G？

Centos的ios镜像文件包含bootfs和rootfs，而docker的centos镜像复用操作系统的bootfs，只有rootfs和其他镜像层

3、Docker中的一个tomcat镜像为什么有500MB，而一个tomcat安装包只有70多MB？

由于docker中镜像是分层的，tomcat虽然只有70多MB，但它需要依赖父镜像和基础镜像，所有整个对外暴露的tomcat镜像大小500多MB

### 镜像制作

Docker镜像如何制作

1、容器转为镜像

```shell
docker commit 容器id 镜像名称:版本号
```

```shell
# 将镜像压缩为压缩文件
docker save -o 压缩文件名称 镜像名称:版本号
```

```shell
#将压缩文件还原为镜像
docker load -i 压缩文件名称
```

2、dockerfile

Dockerfile概念

- Dockerfile是一个文本文件
- 包含一条条的指令
- 每一条指令构建一层，基于基础镜像，最终构建出一个新的镜像
- 对于开发人员：可以为开发团队提供一个完全一致的开发环境
- 对于测试人员：可以直接拿开发时所构建的镜像或者通过Dockerfile文件构建一个新的镜像开始工作了
- 对于运维人员：在部署时，可以实现应用的无缝移植