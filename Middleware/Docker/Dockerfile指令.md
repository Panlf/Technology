## Dockerfile指令

### FROM

描述：FROM这个构建指令指定基本镜像，后续的指令运行于此基准镜像所提供的运行环境实践中，基准镜像可以是任何可用镜像文件，默认情况下，docker build会在docker主机上查找指定的镜像文件，在其不存在时，则会从Docker Hub Registry上拉取所需的镜像文件,如果找不到指定的镜像文件，docker build会返回一个错误信息

格式

```shell
FROM <image>[:<tag>] 或
FROM <image>@<digest>
		<image>：指定作为base image的名称；
		<tag>：base image的标签，为可选项，省略时默认为latest;
```

### COPY

描述：用于从宿主机复制文件至创建的新镜像文件

格式

```shell
COPY [--chown=<user>:<group>] <src>... <dest>
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]

    <src>：要复制的源文件或目录，支持使用通配符
           <src>必须是build上下文中的路径，不能是其父目录中的文件
          如果<src>是目录，则其内部文件或子目录会被递归复制，但<src>目录自身不会被复制
          如果指定了多个<src>，或在<src>中使用了通配符，则<dest>必须是一个目录，且必须以/结尾
   
    <dest>：目标路径，即正在创建的image的文件系统路径；建议为<dest>使用绝对路径，否则，COPY指定则以WORKDIR为其起始路径；
           如果<dest>事先不存在，它将会被自动创建，这包括其父目录路径
   --chown：仅在用于构建Linux容器的Dockerfiles上受支持，并且不适用于Windows容器。
```

指令示例

```shell
复制宿主机文件index.html到容器/data/html/index.html
COPY index.html /data/html/index.html   

复制宿主机data目录下文件（包括子目录）到容器/data/目录下，并不会复制目录本身
COPY data  /data/   
```

### ADD

描述：ADD指令类似于COPY指令，ADD支持使用TAR文件和URL路径，并且会将tar文件展开，如果指定的是url，会从指定的url下载文件放到目录中（ 如果url下载的文件为tar文件，则不会展开）

格式

```shell
ADD [--chown=<user>:<group>] <src>... <dest>
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]
```

示例

```shell
ADD /data/src/nginx-1.14.0.tar.gz  /data/src/
```

### WORKDIR

描述：用于为Dockerfile中所有的RUN、CMD、ENTRYPOINT、COPY和ADD指定设定工作目录，在Dockerfile文件中，WORKDIR指令可出现多次，其路径也可以为相对路径，不过，其是相对此前一个WORKDIR指令指定的路径。另外，WORKDIR也可调用由ENV指定定义的变量

格式

```shell
WORKDIR <dirpath>
```

示例

```shell
WORKDIR /var/log
WORKDIR $STATEPATH
```

### VOLUME

描述：用于在image中创建一个挂载点目录，以挂载Docker host上的卷或其它容器上的卷

格式

```shell
VOLUME <mountpoint> 
VOLUME ["<mountpoint>"]
```

示例

```shell
VOLUME /data
```

### EXPOSE

描述：用于为容器打开指定要监听的端口以实现与外部通信，这个指示声明，真正要暴露这个端口需要再构建容器的时候使用"-P"选项

格式

```shell
EXPOSE <port> [<port>/<protocol>...]
```

示例

```shell
EXPOSE 80/tcp
EXPOSE 80/udp
```

### ENV

描述：用于为镜像定义所需的环境变量，并可被Dockerfile文件中位于其后的其它指令（如ENV、ADD、COPY等）所调用
调用格式为$variable_name或${variable_name}

格式

```shell
ENV <key> <value> 或
ENV <key>=<value> ...

    如果<value>中包含空格，可以以反斜线(\)进行转义
    也可通过对<value>加引号进行标识；另外，反斜线也可用于续行；
    定义多个变量时，建议使用第二种方式，以便在同一层中完成所有功能
```



示例

```shell
ENV myName="John Doe" myDog=Rex\ The\ Dog \
    myCat=fluffy
ENV myName John Doe
ENV myDog Rex The Dog
ENV myCat fluffy
```

### RUN

描述：用于指定docker build过程中运行的程序，其可以是任何命令

格式

```shell
RUN <command> 或
RUN ["<executable>", "<param1>", "<param2>"]

第一种格式中，<command>通常是一个shell命令，且以“/bin/sh -c”来运行它，
这意味着此进程在容器中的PID不为1，不能接收Unix信号，因此，当使用docker stop <container>命令停止容器时，此进程接收不到SIGTERM信号；（部分有错）
第二种语法格式中的参数是一个JSON格式的数组，其中<executable>为要运行的命令,
后面的<paramN>为传递给命令的选项或参数；然而，此种格式指定的命令不会以“/bin/sh -c”来发起，
因此常见的shell操作如变量替换以及通配符(?,*等)替换将不会进行；不过，如果要运行的命令依赖于此shell特性的话，
可以将其替换为类似下面的格式。
RUN ["/bin/bash", "-c", "<executable>", "<param1>"]
```

### CMD

描述：类似于RUN指令，CMD指令也可用于运行任何命令或应用程序，不过，二者的运行时间点不同RUN指令运行于映像文件构建过程中，而CMD指令运行于基于Dockerfile构建出的新映像文件启动一个容器时CMD指令的首要目的在于为启动的容器指定默认要运行的程序，且其运行结束后，容器也将终止；不过，CMD指定的命令其可以被docker run的命令行选项所覆盖，在Dockerfile中可以存在多个CMD指令，但仅最后一个会生效

格式

```shell
CMD <command> 或
CMD [“<executable>”, “<param1>”, “<param2>”] 或
CMD ["<param1>","<param2>"]
```

### ENTRYPOINT

描述：类似CMD指令的功能，用于为容器指定默认运行程序，从而使得容器像是一个单独的可执行程序，与CMD不同的是，由ENTRYPOINT启动的程序不会被docker run命令行指定的参数所覆盖，而且，这些命令行参数会被当作参数传递给ENTRYPOINT指定指定的程序，不过，docker run命令的–entrypoint选项的参数可覆盖ENTRYPOINT指令，指定的程序 

### USER

描述：用于指定运行image时的或运行Dockerfile中任何RUN、CMD或ENTRYPOINT指令指定的程序时的用户名或UID
默认情况下，container的运行身份为root用户

格式

```shell
USER <user>[:<group>] 或
USER <UID>[:<GID>]
```

## 注意事项

- 因为每个指令都会在构建独立一层，所以最好同类指令想办法一起执行，比如RUN指令最好写一条而不是吧每个命令都用一次RUN
- 在构建的过程中，可能会产生很多不需要的文件，比如yum 了一个软件，这时候如果建立了缓存也是不必要的，因为这个缓存跟运行容器内的进程没有太大的关系。而且占用空间，所以最好吧这些非必要的东西做一些清理，比如yum结束之后，用yum clean all 清理一下 

## 简单案例

```dockerfile
FROM centos:latest
ADD  nginx-1.14.0.tar.gz  /data/
RUN cd /data/nginx-1.14.0 ; \
    useradd nginx ; \ 
    yum install gcc gcc-devel openssl openssl-devel gcc gcc-devel pcre pcre-devel -y ; \
    ./configure --prefix=/usr/local/nginx1.14 --user=nginx --group=nginx --with-http_ssl_module  --with-http_stub_status_module --with-http_flv_module --with-http_mp4_module --with-threads --with-file-aio; \
    make && make install ;\
    yum clean all; \
    make clean ;\
    ln -sv /usr/local/nginx1.14 /usr/local/nginx; \
    rm -fr /data
EXPOSE  80/tcp
ENTRYPOINT /usr/local/nginx/sbin/nginx -g "daemon off;"
```

```dockerfile
#解析
FROM # 基础镜像
MAINTAINER # 姓名+邮箱
RUN # 镜像构建的时候需要运行的命令
ADD # 添加内容
WORKDIR # 镜像的工作目录
VOLUME # 挂载的目录
EXPOSE # 暴露端口
CMD # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT # 指定这个容器启动的时候要运行的命令
ONBUILD # 当构建一个被继承DockerFile 这个时候就会运行ONBUILD的指令。
COPY # 类似ADD命令 将我们的文件拷贝到镜像中
ENV # 构建
```

```shell
CMD	# 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT  # 指定这个容器启动的时候要运行的命令，可以追加命令
```

