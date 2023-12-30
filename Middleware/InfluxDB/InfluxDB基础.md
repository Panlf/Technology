# InfluxDB基础

## 简介
### 简述
InfluxDB是一个由InfluxData开发的开源时序型数据。它由Go写成，着力于高性能查询与存储时序型数据。InfluxDB被广泛应用于存储系统的监控数据，IoT行业的实时数据等场景。
时序：跟时间有关的数据，称之为时序
InfluxDB有三大特性

- Time Series（时间序列）：你可以使用与时间有关的相关函数（如最大、最小、求和等）
- Metrics（度量）：你可以实时对大量数据进行计算
- Events（事件）：它支持任意的事件数据

特点：

- 为时间序列数据专门编写的自定义高性能数据存储。TSM引擎具有高性能的写入和数据压缩
- Golang编写，没有其他的依赖
- 提供简单、高性能的写入，查询http api，Native Http API，内置http支持，使用http读写
- 插件支持其他数据写入协议，例如graphite、collected、OpenTSDB
- 支持类sql查询语句
- tags可以索引序列化，提供快速有效的查询
- Retention policies自动处理过期数据
- Continuous queries自动聚合，提高查询效率
- schemaless（无结构），可以是任意数量的列
- min，max，sum，count，mean，median一系列函数，方便统计
- Built-in Explorer自带管理工具
### 应用场景
时序数据以时间作为主要的查询纬度，通常会将连续的多个时序数据绘制成线，制作基于时间的多纬度报表，用于揭示数据背后的趋势、规律、异常，进行实时在线预测和预警，时序数据普遍存在于IT基础设施、运维监控系统和物联网中。如：监控数据统计。每毫秒记录一下电脑内存的使用情况，然后就可以根据统计的数据，利用图形化界面（InfluxDB V1一般配合Grafana）制作内存使用情况的折线图；可以理解为按时间记录一些数据（常用的监控数据、埋点统计数据等），然后制作图表做统计。

## 安装
### 传统安装

- 下载influxdb安装包
```
https://portal.influxdata.com/downloads
```

- 上传到linux系统中
- 解压缩安装包
```
tar -zxf influxdb-1.7.8_linux_arm64.tar.gz
```

- 进入解压缩目录查看目录结构
- 目录说明 
   - etc 主要用来存储 influxdb 系统配置信息
   - usr 主要用来存储 influxdb 操作相关脚本文件
   - var 主要用来存储 influxdb 运行日志、以及产生数据和依赖库文件
- 启动influxdb 进入usr/bin 目录执行
```
./influxd
```

- 客户端连接到influxdb
```
influx -database 'metrics' -host 'localhost' -port '8086' -username admin -password admin
```

### dokcer安装
```
version: '3.8'
volumes:
  influxdb: 

services:
  influxdb:
    image: influxdb:1.8.7
      ports: 
        - '8086:8086'
      volumes:
        - influxdb:/root/influxdb/data
      # - $PWD/influxdb.conf:/root/influxdb/influxdb.conf
      environment:
        - INFLUXDB_ADMIN_USER=root
        - INFLUXDB_ADMIN_PASSWORD=root
        - INFLUXDB_DB=sprixin
      restart: always
```

命令式
```
> docker pull influxdb:1.8.7
> docker run -d -p 8086:8086 --name influxdb -v /data/influxdb:/var/lib/influxdb --restart=always influxdb:1.8.7
```

```
#先不启用密码验证, 创建用户和密码,启动后进入创建好用户和密码后,修改auth-enabled = true 重启容器生效,就必须要用户和密码

docker pull influxdb

docker run -d --name my-influxdb \
-p 8086:8086 \
-p 8083:8083 \
-p 2003:2003 \
-e INFLUXDB_GRAPHITE_ENABLED=true \
-v /data/influxdb/conf/influxdb.conf:/etc/influxdb/influxdb.conf \
-v /data/influxdb:/var/lib/influxdb \
-v /etc/localtime:/etc/localtime \
influxdb

#进入容器
docker exec -it my-influxdb /bin/bash
输入 influx
#创建用户和密码
create user "admin" with password 'admin' with all privileges
create user "admin" with password 'beyond_2021' with all privileges

auth admin admin 登录

show databases; 展示数据库

create database demo 创建表


#默认是不用用户密码的, 是否开启认证，默认值：false
cat /data/influxdb/conf/influxdb.conf 
[meta]
  dir = "/var/lib/influxdb/meta"

[data]
  dir = "/var/lib/influxdb/data"
  engine = "tsm1"
  wal-dir = "/var/lib/influxdb/wal"

[http]
  auth-enabled = false


#容器外面执行命令
curl -i -XPOST http://localhost:8086/query --data-urlencode "q=CREATE DATABASE testdb"

curl -XPOST http://localhost:8086/query --data-urlencode "q=create user "admin123" with password 'admin123' with all privileges"

./influx -database 'testdb' -execute 'auth admin123 admin123'

./influx -database 'testdb' -execute 'auth admin123 admin123'

show users; 启用用户密码后,会报错

输入 influx -username 'admin' -password 'beyond_2021'


# 保存策略
show retention policies on test 显示test数据库策略 如果没有指定策略默认是autogen

对test数据库创建一个策略,2小时之前数据删除,一个副本,设置为默认策略
create retention policy "abc" on "test" duration 2h replication 1 default

10天前数据删除  比如：h（小时），w（星期）
create retention policy "rp_10d" on "testdata" duration 10d replication 1 default

修改默认策略
alter retention policy "autogen" on "demo" duration 10d replication 1 default

alter retention policy "autogen" on "demo" duration 15d replication 1 default

修改策略
alter retention policy "rp_10d" ON "demo" duration 10d replication 1 default

插入数据不指定策略,按默认策略保存
insert into devops,host=server01 cpu=23.1,mem=0.61

指定策略保存数据
insert into "autogen" devops,host=server01 cpu=23.1,mem=0.71

查询时不指定策略,按默认策略查询
select * from "devdata"

指定策略查询数据
select * from "autogen"."devdata"

show tag keys from 表名

show field keys from 表名
```

## 相关概念
### 概念

- database：数据库；  用于针对不同应用进行数据隔离
- measurement：数据库中的表；  类似与关系型数据库table row记录
- points：表里面的一行数据；    point相当于关系库中表中一条记录
- influxDB中独有的一些概念：Point由时间戳（time）、数据（field）和标签（tags）组成
```
库 database
  表 measurement
    point = time（主键 可以自动生成，手动指定 必须存在）+field（普通字段 存储数据 必须存在）+ tags标签（可有可无 索引 用来加快查询速度 String）
      field：不经常查询数据，可以直接存储为field
      tags：索引字段，主要用来提高查询效率
```
### 与MySQL概念对比
| 概念 | MySQL | InfluxDB |
| --- | --- | --- |
| 数据库（同） | database | database |
| 表（不同） | table | measurement（测量；度量） |
| 列（不同） | column | point = tag（带索引的，非必须）、field（不带索引）、timestamp（唯一主键） |

point相当于传统数据库里的一行数据，如下表所示：

| point属性 | 传统数据库中的概念 |
| --- | --- |
| time（时间戳） | 每个数据记录时间，是数据库中的主索引（会自动生成） |
| fields（字段、数据） | 各种记录值（没有索引的属性）也就是记录的值：温度、湿度 |
| tags（标签） | 各种有索引的属性：地区、海拔 |

_注意，在influxdb中，字段必须存在。因为字段是没有索引的。如果使用字段作为查询条件，会扫描符合查询条件的所有字段值，性能不及tag。类比一下，fields相当于sql的没有索引的列。tags是可选的，但是强烈建议你用上它，因为tag是有索引的，tags相当于sql中的有索引的列。tag value只能是string类型。_
### 类型说明

- tag类型 
   - tag都是string类型
- field类型 
   - field支持四种类型 int，float，string，boolean
| 类型 | 方式 | 示例 |
| --- | --- | --- |
| float | 小数 | id=12.1 |
| int | 整数 | age=12 |
| boolean | true/false | boy=true |
| String | "" or '' | name="Alice" |

## 基础操作
### 库 database
```
-- 库操作
- show databases; 查看所有库
- create database test; 创建一个库
- drop database test; 删除一个库
- use test; 选中一个库
- clear database|db;  清除当前上下文的库
```
### 表 measurement
注意：表不能显示创建，插入数据时自动插入表中数据
```
-- 表操作
- show measurements;  查看所有表
- drop measurement "test"; 删除一个表 # 注意：删除表
```
### 插入
```
-- 基本语法
- insert into <retention policy> measurement,tagKey=tagValue fieldKey=fieldValue timestamp
- 如:
insert user,name=blr,phone=110 id=20,email="11@qq.com"
从上面的输出，简单小结一下插入的语句写法:
1) insert + measurement + "," + tag=value,tag=value + + field=value,field=value
2) tag与tag之间用逗号分隔；field与field之间用逗号分隔
3) tag与field之间用空格分隔
4) tag都是string类型，不需要引导将value包裹
5) field如果是string类型，需要加引导
```
注意：更新：如果在插入数据时，插入的数据的时间和tags 与原有数据一致，则更新当前数据
### 查询
#### 普通查询
```
0、查询所有 # 注意：tags显示再查询结果最后
- select * from test;

1、查询所有的 tag 和 field
- select * from test where person_name='blr'
- select * from test where "name"='xiaochen' # name为influxdb关键字需要加入双引号区分

2、从单个measurement查询所有的field，不插tag
- select *::field from test

3、从单个measurement查询特定的field和tag # 注意：查询时至少要带上一个field key，如果只查询tag字段的话是查不到数据的
- select person_name,age from test

4、同时查询多张表 # 注意：返回是将每张表不同记录返回
- select * from test,student

5、模糊查询
# 前缀匹配，相当于 mysql 的 like  'blr%'
- select * from test where person_name = ~/^blr/

# 后缀匹配，相当于 mysql 的 like '%blr'
- select * from test where person_name = ~/blr$/

# 前后匹配，相当于 mysql 的 like '%blr%'
- select * from test where person_name = ~/blr/
```
#### 聚合函数
注意事项：聚合函数只能对field字段进行操作，不能对tag字段操作，否则查询出来的列表是空的
```
0、如果我就要对tag字段进行聚合函数计算怎么办？那我们可以通过子查询来实现
- select distinct(person_name) from (select * from test);

1、count() 统计
# 查询某个field字段中的非空数量
- select count(age) from test;

2、DISTINCT() 去重
- select distinct(age) from test;

3、MEAN() 求平均值，这个平均值必须是数字类型
- select mean(age) from test;

4、MEDIAN() 求中位数，从单个字段（field）中的排序值返回中间值
# 中位数统计学中的专有名词，是按顺序排列的一组数据中居于中间位置的数，代表一个样本、种群或概率分布中的一个数值，其可将数值集合划分为相等的上下两部分
- select median(age) from test;

5、SPREAD() 返回字段的最小值和最大值之间的差值。数据类型必须是长整型或者float64
- select spread(age) from test;

6、SUM() 求和
- select sum(age) from test;

7、BOTTOM() 返回一个字段中最小的N个值。字段类型必须是长整型或float64类型
- select bottom(age,3) from test;

8、FIRST() 返回一个字段中时间最早取值
- select first(age) from test;

9、LAST() 返回一个字段中时间最晚取值
- select last(age) from test

10、MAX() 求最大值
- select max(age) from test;
```
#### 分组聚合
```
1、基于时间分组
# 查询所有数据，并对其划分为每200毫秒一组
select count(age) from test group by time(200ms)
# 查询所有数据，并对其划分为每200秒一组
select count(age) from test group by time(200s)
# 查询所有数据，并对其划分为每12分钟一组
select count(age) from test group by time(12m)
# 查询所有数据，并对其划分为每12小时一组
select count(age) from test group by time(12h)
# 查询所有数据，并对其划分为每12天一组
select count(age) from test group by time(12d)
# 查询所有数据，并对其划分为每12周一组
select count(age) from test group by time(12w)
```
#### 分页查询
```
LIMIT 用法有2钟
1. limit 10 ：查询前10条数据
2. limit size offset N ：size表示每页大小，N表示第几条记录开始查询

# 查询前10条数据
select * from test limit 10

# 分页，pageSize 为每页显示大小 pageIndex 为查询的页数
pageIndex = 1
pageSize = 10
- select * from test limit pageSize offset (pageIndex - 1)*pageSize
```
#### 排序
```
# 升序
select * from test order by time asc

# 降序
select * from test order by time desc
```
#### in 查询
```
# 关系型数据库可以用in关键字来查询多个值
select * from test where person_name in ('','','')
# 但是时序数据库是没有 in 查询的，虽然in是保留的关键字，但是依然有办法解决，可用模糊查询

# 同时匹配123 和 thing
select * from test where person_name = ~/^123$|^thing$/
```

### 保留策略
influxDB是没有提供直接删除数据记录的方法，但是提供数据保存策略，主要用于指定数据保留时间，超过指定时间，就删除这部分数据。（数据库过期策略至少一个小时），默认保存策略为永久保存。
查看某个库策略
```
# 查看某个库的策略
show retention policies on "数据库名称"
# 查看当前库下的策略，需要先用use database 命令指定库名
show retention policies
```

- name：策略名称
- duration：数据保存时间，超过这个时间自动删除，0表示永久保存
- shardGroupDuration：shardGroup的存储时间，shardGroup是InfluxDB的一个基本储存结构，在这个时间内插入的数据查询较快，数据存放大于168小时查询速度降低
- replicaN：全称是REPLICATION，副本个数
- default：是否默认策略

创建数据保留策略
```
# 1、创建 h （小时） d （天） w （星期）
- CREATE RETENTION POLICY "保留策略名称" ON "数据库名称" DURATION "该保留策略对应的数据过期时间" REPLICATION "复制因子，开源的InfluxDB单机环境永远为1" SHARD DURATION "分片组的默认时长" DEFAULT

- create retention policy "testpolicy" on myInfluxdb duration 72h replication 1 shard duration 1h default

# 2、修改保留策略
- alter retention policy "保留策略名称" on "数据库名称" duration 1d

# 3、修改默认保留策略
- alter retention policy "保留策略名称" on "数据库名称" default

# 4、删除保留策略
- drop retention policy "保留策略名称" on "数据库名称"
```
## 权限配置
### 开启权限
> 默认情况下 InfluxDB 是没有开启权限配置的，即默认情况下所有客户端可直接连接操作 InfluxDB 服务进行相关操作，但是这在生产环境中是不可取的，因此需要对 influxdb 服务加入权限相关配置。

#### 创建用户信息
首先通过客户端连接到服务，查看当前用户
```
- show users
```
创建用户
```
create user "用户名" with password '密码' with all privileges

# 注意：用户名必须双引号，密码必须单引号

# 创建指定库只读用户
create user "用户名" with password '密码'
grant read on 库名 to "用户名"

# 删除用户
drop user "用户名"
```
#### 配置文件中开启

- 进入influxdb安装目录中etc/influxdb
- 编辑influxdb.conf文件
```
[http]
  auth-enabled = false
```

- 指定配置文件重启InfluxDB服务
```
./influxd -config=/root/influxdb-1.7.8-1/etc/influxdb/influxdb.conf
```
## 集群
influxdb 开源 单机环境 1.x
cluster 集群环境收费
参考：[https://github.com/chengshiwen/influxdb-cluster/wiki](https://github.com/chengshiwen/influxdb-cluster/wiki)
### 简介

- InfluxDB CLuster是一个开源的 时间序列数据库，没有外部依赖。它对于记录指标、事件和执行分析很有用。
- InfluxDB CLuster启发于InfluxDB Enterprise、InfluxDB v1.8.10 和 InfluxDB v0.11.1，旨在替代InfluxDB Enterprise。
- InfluxDB CLuster易于维护，可以与上游InfluxDB 1.x保持实时更新
### 特性

- 内置 HTTP API ，无需编写任何服务端代码即可启动和运行
- 数据可以被标记 tag ，允许非常灵活的查询
- 类似 SQL 的查询语言
- 集群支持开箱即用，因此处理数据可以水平扩展。集群目前处于生产就绪状态
- 易于安装和管理，数据写入查询速度快
- 旨在实时应答查询。这意味着每个数据点在到来时都会被计算索引，并且在 < 100 毫秒内返回的查询中立即可用
### 架构
InfluxDB Cluster 安装由两组独立的进程组成：Data 节点和 Meta 节点。
Meta节点：
元节点持有以下所有的元数据

- 集群中的所有节点和它们的角色
- 集群中存在的所有数据库和保留策略
- 所有分片和分片组，以及它们存在于哪些节点上
- 集群用户和他们的权限
- 所有的连续查询

Data节点：
数据节点持有所有的原始时间序列数据和元数据，包括：

- 测量值
- 标签键和值
- 字段键和值

Meta 节点通过 TCP 协议和 Raft 共识协议相互通信，默认都使用端口 8089，此端口必须在 Meta 节点之间是可访问的。默认 Meta 节点还将公开绑定到端口 8091 的 HTTP API，influxd-ctl 命令使用该 API。

Data 节点通过绑定到端口 8088 的 TCP 协议相互通信。Data 节点通过绑定到 8091 的 HTTP API 与 Meta 节点通信。这些端口必须在 Meta 节点和 Data 节点之间是可访问的。

在集群内，所有 Meta 节点都必须与所有其它 Meta 节点通信。所有 Data 节点必须与所有其它 Data 节点和所有 Meta 节点通信。
### 集群搭建
参考：[https://github.com/chengshiwen/influxdb-cluster/wiki](https://github.com/chengshiwen/influxdb-cluster/wiki)
## 配置&优化
### 数据导入导出
```
# 导出
influx_inspect export -compress -datadir "/var/lib/influxdb/data" -waldir "/var/lib/influxdb/wal" -out "/home/myDB" -database myDB

# 导入
influx -import -database myDB -path=/home/myDB -precision=ns

# 数据压缩导入
influx -import -database myDB -path=/home/myDB -compress -precision=ns -username=root -password=root

# 数据未压缩导入
influx -import -database myDB -path=/home/myDB -compress -precision=ns -username=root -password=root
```
### 配置调优
对InfluxDB的配置优化，主要从一些配置参数出发提高InfluxDB的性能
#### 性能优化
InfluxDB的存储引擎是TSM Tree（Time-Structured Merge Tree），基本上整体思想和LSM Tree（Log-Structured-Tree）类似，做了一些时序场景下数据存储结构上的建模优化。`cache-snapshot-memory-size`值需要调试
```
[data]
cache-snapshot-memory-size=562144000
```
cache-snapshot-memory-size 这个大小控制的是 LSM 中的 cache 的大小，当 cache 达到一定阈值后，cache会落盘生成tsm file，此时的tsm file的level为 level 0，两个相同level的 tsm file 会进行 compact 生成一个level + 1的 tsm file，既两个 level 0 的tsm file会生成一个 level 1的tsm file，这种设计既TSM tree的写入放大问题。
由于Influxdb是固定两个低level文件compact成一个高一级level的tsm file，所以如果cache size越小,dump成tsm file的频率越高，进而做compact的频率也越高，造成写入放大越显著，当写入的频率很高的场景下，会导致influxdb的吞吐下降非常明显。
compact频率变高后，Influxdb写入放大很重要一个原因是TSH file数据做了编码压缩磁盘占用空间，当compact时,需要对数据decode，会带来明显的性能损耗。
Influxdb为了优化这个问题，在做compact时分为optimize compact和full compact两种类型。在full compact场景下，首先会对tsm file中的block做decode，然后按照每个Block存储的point数量，将decode的Point value按照时间顺序重新encode成Block，然后写入到新的TSM file中，其中的性能损耗会非常显著。而在optimize场景下，不会读取block内部的数据，会对多个block拼接，减少性能消耗。
目前部分场景下Influxdb做compact还是会选择full compact。optimize compact虽然会提升compact速度和减少compact的资源消耗，但是会引起查询放大问题:需要从多个block中才能获取到需要返回的数据。

总结

- cache-snapshot-memory-size 值理论上是越大越好，但是需要关注你的硬件配置
- cache-snapshot-memory-size 值跟当前并发写入tags数量有关系，如果你的tags数很大的情况下，一定要调大这个值，如果tags数不多，只是少数tags的数据写入频率很高，那么这个值稍低也不会对性能有太大的影响。

#### 数据层面
表 point = [time + tags] (series 100w tags 10w) + fields
max-series-per-databse 可调整为0，如注释所示：该参数控制每个db的最大的series数量
max-values-per-tag可调整为0，如注释所示：该参数控制每个tag的tag_value数量
```
[data]
max-series-per-databse=0
max-values-per-tag=0
```
#### 查询超时
```
query-timeout="100"
```
