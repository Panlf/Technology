## Docker网络

```shell
docker run -d -P --name tomcat01 tomcat

docker exec -it tomcat01 ip addr
# 发现容器启动时候会得到 eth0  ip地址，docker分配的

linux可以ping通docker容器内部
```

### 原理

1、我们每启动一个docker容器，docker就会给docker容器分配一个ip，我们只要安装了docker，就会有一个网卡docker0桥接模式，使用的技术是evth-pair技术

2、再启动一个容器，发现又多了一对网卡

```shell
# 我们发现这个容器带来网卡，都是一对对的
# evth-pair 就是一对的虚拟设备接口，他们都是成对出现的，一段连着协议，一段彼此相连
# 正因为有这个特性，evth-pair充当一个桥梁，连接各种虚拟网络设备
# OpenStac Docker容器之间的连接，OVS的连接，都是使用evth-pair技术
```

3、容器之间也是可以ping通的。

结论：tomcat01和tomcat02是公用的一个路由器，docker0

所有的容器不指定网络的情况下，都是docker0路由的，docker会给我们的容器分配一个默认的可用IP

Docker使用的是Linux的桥接，宿主机中是一个Docker容器的网桥docker0

Docker中的所有的网络接口都是虚拟。虚拟的转发效率高。

只要容器删除，对应网桥一对就没了。

### --link

```shell
docker exec -it tomcat02 ping tomcat01
ping: tomcat01:Name or service not know

docker run -d -P --name tomcat03 --link tomcat02 tomcat

docker exec -it tomcat03 ping tomcat02
```

### 自定义网络

```shell
# 查看所有的docker网络
docker network ls
```

网络模式

bridge 桥接

none 不配置网络

host 和宿主机共享网络

container 容器网络联通

测试

```shell
# --net bridge 
docker run -d -P --name tomcat01 tomcat
docker run -d -P --name tomcat01 --net bridge tomcat

# 自定义网络
docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
```

### 网络联通-Redis集群

```shell
# 创建网卡
docker network create redis --subnet 172.38.0.0/16

# 通过脚本创建6个redis配置
for port in $(seq 1 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
cat << EOF >/mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done

docker run -p 637${port}:6379 -p 1637${port}:16379 --name redis-${port} \
-v /mydata/redis/node-${port}/data:/data \
-v /mydata/redis/node-${port}/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.1${port} redis6.0 redis-server /etc/redis/redis.conf; \


# 创建集群
redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
```
