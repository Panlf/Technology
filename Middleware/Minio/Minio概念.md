## Minio概念

### 使用场景

互联网海量非结构化数据的存储需求

- 电商网站	海量商品图片
- 视频网站    海量视频文件
- 网盘  海量文件
- 社交网站 海量图片

### Minion介绍

Minio是基于Apache License v2.0开源协议的对象存储服务。它兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。

Minio是一个非常轻量的服务，可以很简单和其他应用的结合，类似NodeJS，Redis或者MySQL。

官网：https://min.io/     https://www.minio.org.cn/

> 对象存储服务（Object Storage Service，OSS）是一种海量、安全、低成本、高可靠的云存储服务，适合存放任意类型的文件。容量和处理能力弹性扩展，多种存储类型供选择，全面优化存储成本。

#### Minio优点

- 部署简单：一个single二进制文件即是一切，还可支持各种平台
- minio支持海量存储，可按zone扩展（原zone不受任何影响），支持单个对象最大5TB
- 兼容Amazon S3接口，充分考虑开发人员的需求和体验
- 低冗余且磁盘损坏高容忍，标准且最高的数据冗余系数为2（即存储一个1M的数据对象，实际占用磁盘空间为2M）。但在任意n/2块disk损坏的情况下依然可以读出数据（n为一个纠删码集合（Erasure Coding Set）中的disk数量）。并且这种损坏恢复是基于单个对象的，而不是基于整个存储卷的。
- 读写性能优异

#### Minio的基础概念

- **Object** 存储到Minio的基本对象，如文件、字节流，Anything。。。
- **Bucket** 用来存储Object的逻辑空间。每个Bucket之间的数据是相互隔离的。对于客户端而言，就相当于一个存放文件的顶层文件夹
- **Drive** 即存储数据的磁盘，在Minio启动时，以参数的方式传入。Minio中所有的对象数据都会存储在Drive里。
- **Set** 即一组Drive的集合，分布式部署根据集群规模自动划分一个或多个Set，每个Set中的Drive分布在不同位置。一个对象存储在一个Set上。（For example:{1...64} is divided into 4 sets each of size 16.）
  - 一个对象存储在一个Set上
  - 一个集群划分为多个Set
  - 一个Set包含的Drive数量是固定的，默认由系统根据集群规模自动计算得出
  - 一个Set中的Drive尽可能分布在不同的节点上

### 纠删码EC (Erasure Code)

Minio使用纠删码机制来保证高可靠性，使用highwayhash来处理数据损坏（Bit Rot Protection）。关于纠删码，简单来说就是可以通过数学计算，把丢失的数据进行还原，它可以将n份原始数据，增加m份数据，并能通过n+m份中的任意n份数据，还原为原始数据。即如果有任意小于等于m份的数据失效，仍然能通过剩下的数据还原出来。

### 存储形式

文件对象上传到Minio，会在对应的数据存储磁盘中，以Bucket名称为目录，文件名称为下一级目录，文件名下是part.1和xl.meta(老版本)，前者是编码数据块及检验块，后者是元数据文件。

## Minio环境部署

### 单机部署

#### 基于Centos7

```shell
wget -q http://dl.minio.org.cn/server/minio/release/linux-amd64/minio
chmod +x minio
# 启动minio server服务，指定数据存储目录/mnt/data
./minio server /mnt/data
```

默认用户名和密码minioadmin:minioadmin，修改默认用户名密码可以使用

```shell
export MINIO_ROOT_USER=admin
export MINIO_ROOT_PASSWORD=12345678
```

默认的配置目录是${HOME}/.minio，可以通过--config-dir命令自定义配置目录

```shell
./minio server --config-dir /mnt/config /mnt/data
```

控制台监听端口是动态生成的，可以通过--console-address ":port"指定静态端口

```shell
./minio server --console-address ":50000" /mnt/data
```

#### 基于Docker

```shell
docker run -p 9000:9000 --name minio \
 -v /mnt/data:/data \
 -v /mnt/config:/root/.minio \
 minio/minio server /data
```

*存在问题：浏览器无法访问minio控制，因为没有对外暴露控制台端口*

对外暴露minio控制台的端口，通过--console-address ":50000"指定控制台端口为静态端口

```shell
docker run -p 9000:9000 -p 9001:9001 --name minio \
 -v /data/minio/data:/data \
 -v /data/minio/config:/root/.minio \
 minio/minio server --console-address ":9001" /data
```

自定义密码，后台启动

```shell
docker run -d -p 9000:9000 -p 50000:50000 --name minio \
 -e "MINIO_ROOT_USER=admin" \
 -e "MINIO_ROOT_PASSWORD=12345678" \
 -v /data/minio/data:/data \
 -v /data/minio/config:/root/.minio \
 minio/minio server --console-address ":50000" /data
```



```shell
docker run -d -p 9000:9000 -p 9001:9001 --name minio \
 -e "MINIO_ROOT_USER=admin" \
 -e "MINIO_ROOT_PASSWORD=12345678" \
 -v /data/minio/data:/data \
 -v /data/minio/config:/root/.minio \
 minio/minio server --console-address ":9001" /data
```



#### Minio纠删码模式

Minio使用纠删码`erasure code`和校验和`checksum`来保护数据免受硬件故障和无声数据损坏。即使您丢失一般数量(N/2)的硬盘，您仍然可以恢复数据。

> 纠删码是一种恢复丢失和损坏数据的数学算法，Minio采用Reed-Solomon code 将对象拆分为N/2数据和N/2奇偶校验块。这就意味着如果是12块盘，一个对象会被分为6个数据块，6个奇偶校验块，你可以丢失任意6块盘（不管其是存放的数据块还是奇偶校验块），你仍可以从剩下的盘中的数据进行恢复。

使用Minio Docker镜像，在8块盘中启动Minio服务

```shell
docker run -d -p 9000:9000 -p 50000:50000 --name minio \
-v /mnt/data1:/data1 \
-v /mnt/data2:/data2 \
-v /mnt/data3:/data3 \
-v /mnt/data4:/data4 \
-v /mnt/data5:/data5 \
-v /mnt/data6:/data6 \
-v /mnt/data7:/data7 \
-v /mnt/data8:/data8 \
minio/minio server /data{1...8} --console-address ":50000"
```

### 分布式集群部署

分布式Minio可以让你将多块硬盘（甚至在不同的机器上）组成一个对象存储服务。由于硬盘分布在不同的节点上，分布式Minio避免了单点故障。



#### 分布式存储可靠性常用方法

分布式存储，很关键的点在于数据的可靠性，即保证数据的完整，不丢失，不损坏。只有在可靠性实现的前提下，才有了追求一致性、高可用、高性能的基础。而对于在存储领域，一般对于保证数据可靠性的方法主要有两类，一类是冗余法，一类是校验法。

##### 冗余

冗余法最简单直接，即对存储的数据进行副本备份，当数据出现丢失，损坏，即可使用备份内容进行恢复，而副本备份的多少，决定了数据可靠性的高低。这其中会有成本的考量，副本数据越多，数据越可靠，但需要的设备就越多，成本就越高。可靠性是允许丢失其中一份数据。当前已有很多分布式系统是采用此种方式实现，如 Hadoop 的文件系统(3个副本)，Redis的集群，MySQL的主备模式等。

##### 校验

校验法即通过校验码的数学计算的方式，对出现丢失、损坏的数据进行校验、还原。注意，这里有两个作用，一个校验，通过对数据进行校验和( checksum )进行计算，可以检查数据是否完整，有无损坏或更改，在数据传输和保存时经常用到，如TCP协议;二是恢复还原,通过对数据结合校验码，通过数学计算，还原丢失或损坏的数据，可以在保证数据可靠的前提下，降低冗余，如单机硬盘存储中的RAID技术，纠删码(Erasure Code)技术等。Minio采用的就是纠删码技术。

#### 分布式Minio优势

##### 数据保护

分布式Minio采用纠删码来防范多个节点宕机和位衰减`	bit rot`。

分布式Minio至少需要4个硬盘，使用分布式Minio自动引入了纠删码功能。

##### 高可用

单机Minio服务存在单点故障，相反，如果是一个N块硬盘的分布式Minio，只要有N/2硬盘在线，你的数据就是安全的。不过你需要至少有N/2+1硬盘来创建新的对象。

例如，一个16节点的Minio集群，每个节点16块硬盘，就算8台服务器宕机，这个集群仍然可读的，不过你需要9台服务器才能写数据。

##### 一致性

Minio在分布式和单机模式下，所有读写操作都严格遵守read-after-write一致性模型。

#### 运行分布式Minio

启动一个分布式Minio实例，你只需要把硬盘位置作为参数传给minio server 命令即可，然后你需要在所有节点运行同样的命令。

##### 8个节点，每节点1块盘

启动分布式Minio实例，8个节点，每节点1块盘，需要在8个节点都运行下面的命令

```shell
export MINIO_ROOT_USER=admin
export MINIO_ROOT_PASSWORD=12345678
minio server http://192.168.1.11/export1 http://192.168.1.12/export2 \
http://192.168.1.13/export3 http://192.168.1.14/export4 \
http://192.168.1.13/export5 http://192.168.1.14/export6 \
http://192.168.1.13/export7 http://192.168.1.14/export8 
```

##### 4个节点，每节点4块盘

启动分布式Minio实例，4个节点，每节点4块盘，需要在4个节点都运行下面的命令

```shell
export MINIO_ROOT_USER=admin
export MINIO_ROOT_PASSWORD=12345678
minio server http://192.168.1.11/export1 http://192.168.1.11/export2 \
http://192.168.1.11/export3 http://192.168.1.11/export4 \
http://192.168.1.12/export1 http://192.168.1.12/export2 \
http://192.168.1.12/export3 http://192.168.1.12/export4 \
http://192.168.1.13/export1 http://192.168.1.13/export2 \
http://192.168.1.13/export3 http://192.168.1.13/export4 \
http://192.168.1.14/export1 http://192.168.1.14/export2 \
http://192.168.1.14/export3 http://192.168.1.14/export4 \
```

### 使用Docker Compose部署Minio

https://docs.min.io/docs/deploy-minio-on-docker-compose.html

下载docker-compose.yaml和nginx.conf到当前的工作目录

```shell
docker-compose pull
docker-compose up
```

### 扩展现有的分布式集群

```shell
export MINIO_ROOT_USER=admin
export MINIO_ROOT_PASSWORD=12345678
minio server http://host{1...32}/export{1...32}
```

指定新的集群扩展现有集群

```shell
export MINIO_ROOT_USER=admin
export MINIO_ROOT_PASSWORD=12345678
minio server http://host{1...32}/export{1...32} http://host{33...64}/export{1...32}
```

## Minio客户端使用

#### 上传下载

```shell
# 查询minio服务上的所有buckets(文件和文件夹)
mc ls minio-server
# 下载
mc cp minio-server/tulingmall/fox/fox.jpg /tmp/
# 删除
mc rm minio-server/tulingmall/fox/fox.jpg
# 上传
mc cp zookeeper.out minio-server/tulingmall/ 
```

### Bucket管理

```shell
# 创建bucket
mc mb minio-server/bucket01
# 删除bucket
mc rb minio-server/bucket02
# bucket不为空，可以强制删除 慎用
mc rb --force minio-server/bucket01
# 查询bucket03磁盘使用情况
mc du minio-server/bucket03
```

### Minio Cilent admin的使用

Minio Cilent (mc)  提供了"admin"子命令来对您的Minio部署执行管理任务

```shell
service	服务重启并停止所有MinIo服务器
update	更新所有MinI0服务器
info 信息显示MinIo服务器信息
user	用户管理用户
group	小组管理小组
policy	MinIo服务器中定义的策略管理策略
config	配置管理MinIo服务器配置
heal	修复MinIo服务器上的磁盘，存储桶和对象
profile	概要文件生成概要文件数据以进行调试
top	顶部提供MinIo的顶部统计信息
trace	跟踪显示MinIO服务器的http跟踪
console	控制台显示MinIo服务器的控制台日志
prometheus Prometheus管理Prometheus配置
kms		kms执行KMS管理操作
```

#### 用户管理

```shell
mc admin user --help
# 新建用户
mc admin user add minio-server fox
mc admin user add minio-server fox02 12345678

# 查看用户
mc admin user list minio-server

# 禁用用户
mc admin user disable minio-server fox02

# 启用用户
mc admin user enable minio-server fox02

# 查看用户信息
mc admin user info minio-server fox

# 删除用户
mc admin user remove minio-server fox02

```

#### 策略管理

policy命令，用于添加、删除、列出策略，获取有关策略的信息并未MinIO服务器上的用户设置策略

```shell
mc admin policy --help

# 列出MinIO上的所有固定策略
mc admin policy list minio-server
# 查看policy信息
mc admin policy info minio-server readwrite
```

### 注意

- 用户名必须3个字符以上，密码设置必须8个字符以上。

### 开发的问题

服务器时间差过大

```
The difference between the request time and the current time is too large
```

服务器上面重新同步一下时间

```
1. 安装ntp ntpdate
yum -y install ntp ntpdate

2. 与时间服务器同步时间
ntpdate cn.pool.ntp.org
```

