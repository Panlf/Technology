## Docker应用部署

### MySQL部署

实现步骤

- 搜索MySQL镜像
- 拉取MySQL镜像
- 创建容器
- 操作容器中的MySQL

端口映射

- 容器内的网络服务和外部机器不能直接通信

- 外部机器和宿主机可以直接通信

- 宿主机和容器可以直接直接通信

- 当容器中的网络服务需要被外部机器访问时，可以将容器中提供服务的端口映射到宿主机的端口上。外部机器访问宿主机的该端口，从而间接访问容器的服务。

操作命令

1、搜索mysql镜像

```shell
docker search mysql
```

2、拉取镜像

```shell
docker pull mysql:8.0
```

3、创建容器，设置端口映射、目录映射

```shell
#在root目录下创建mysql目录用于存储mysql数据信息
mkdir ~/mysql
cd ~/mysql
```

```shell
docker run -id \
-p 3307:3306 \
--name=c_mysql \
-v $PWD/conf:/etc/mysql/conf.d \
-v $PWD/logs:/logs \
-v $PWD/data:/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=123456 \
mysql:8.0
```

参数说明：

- -p 3307:3306 将容器的3306端口映射到宿主机的3307端口
- -v $PWD/conf:/etc/mysql/conf.d  将主机当前目录下的conf/my.cnf挂载到容器的/etc/mysql/my.cnf，配置目录
- -v $PWD/data:/var/lib/mysql 将主机当前目录下的data目录挂载到容器的/var/lib/mysql。数据目录
- -e MYSQL_ROOT_PASSWORD=123456 初始化root密码

4、登录MySQL容器

```shell
docker exec -it c_mysql /bin/bash
```

### Tomcat部署

实现步骤

- 搜索tomcat镜像
- 拉取tomcat镜像
- 创建容器
- 部署项目
- 测试访问

1、搜索tomcat镜像

```shell
docker search tomcat
```

2、拉取tomcat镜像

```shell
docker pull tomcat
```

3、创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建tomcat目录用于存储tomcat数据信息
mkdir ~/tomcat
cd ~/tomcat
```

```shell
docker run -id --name=c_tomcat \
-p 8080:8080 \
-v $PWD:/usr/local/tomcat/webapps \
tomcat
```

参数说明

- -p 8080:8080 将容器的8080端口映射到主机的8080端口
- -v $PWD:/usr/local/tomcat/webapps 将主机中当前目录挂载到容器的webapps

### Nginx部署

实现步骤

- 搜索Nginx镜像
- 拉取Nginx镜像
- 创建容器
- 测试访问

1、搜索nginx镜像

```shell
docker search nginx
```

2、拉取nginx镜像

```shell
docker pull nginx
```

3、创建容器，设置端口映射、目录映射

```shell
# 在/root目录下创建nginx目录用于存储nginx数据信息
mkdir ~/nginx
cd ~/nginx
mkdir conf
cd conf
# 在~/nginx/conf下创建nginx.conf文件，粘贴下面内容
vim nginx.conf
```

```
user nginx;
worker_processes 1;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
    '$status $body_bytes_sent "$http_referer" '
    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    send_file on;

    keepalive_timeout 65;

    include /etc/nginx/conf.d/*/conf;
}
```

```shell
docker run -id --name=c_nginx \
-p 80:80 \
-v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf \
-v $PWD/logs:/var/log/nginx \
-v $PWD/html:/usr/share/nginx/html\
nginx
```

参数说明

- -v $PWD/conf/nginx.conf:/etc/nginx/nginx.conf 将主机当前目录下的/conf/nginx.conf挂载到容器/etc/nginx/nginx.conf。配置目录。
- -v $PWD/logs:/var/log/nginx 将主机当前目录下的logs目录挂载到容器的/var/log/nginx。日志目录。

### Redis部署

1、搜索redis镜像

```shell
docker search redis
```

2、拉取redis镜像

```shell
docker pull redis:6.0
```

3、创建容器，设置端口映射

```shell
docker run -id --name=c_redis -p 6379:6379 redis:6.0

# d
docker run -d --name redis -p 6379:6379 redis --requirepass "12345678"
```

### MongoDB

1、搜索mongo镜像

```shell
docker pull mongo
```

2、创建容器

```shell
docker run -p 27017:27017 -v /mongo/data:/data/db  \
-e MONGO_INITDB_ROOT_USERNAME=mongodb \
-e MONGO_INITDB_ROOT_PASSWORD=12345678 \
-d mongo
```
