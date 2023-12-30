# MongoDB基础

## 介绍
### 什么是MongoDB
文档数据库（以JSON为数据模型），由C++语言编写，旨在为WEB应用提供可扩展的高性能数据存储方案。MongoDB是一个介于关系数据库和非关系数据库之间的产品。数据格式为BSON，一种类似JSON的二进制形式存储格式，简称Binary JSON，和JSON一样支持内嵌的文档对象和数组对象，因此可以存储比较复杂的数据类型。

**概念对比**

| SQL概念 | MongoDB概念 |
| --- | --- |
| 数据库（database） | 数据库（database） |
| 表（table） | 集合（collection） |
| 行（row） | 文档（document） |
| 列（column） | 字段（field） |
| 索引（index） | 索引（index） |
| 主键（primary key） | _id（字段） |
| 视图（view） | 视图（view） |
| 表连接（table joins） | 聚合操作（$lookup） |

### MongoDB技术优势
MongoDB基于灵活的JSON文档模型，非常适合敏捷式快速开发。与此同时，其与生俱来的高可用、高水平扩展能力使得他在处理海量、高并发的数据应用时颇具优势。

- JSON结构和对象模型接近，开发代码量低
- JSON的动态模型意味着更容易响应新的业务需求
- 复制集提供99.999%高可用
- 分片架构支持海量数据和无缝扩容
### MongoDB应用场景

- 游戏场景，使用MongoDB存储游戏用户信息，用户的装备、积分等直接以内嵌文档形式存储，方便查询、更新；
- 物流场景，使用MongoDB存储订单信息，订单状态在运送过程中会不断更新，以MongoDB内嵌数组的形式来存储，一次查询就能将订单所有的变更读取出来；
- 社交场景，使用MongoDB存储用户信息，以及用户发表的朋友圈信息，通过地理位置索引实现附近的人、地点等功能；
- 物联网场景，使用MongoDB存储所有接入的智能设备信息，以及设备汇报的日志信息，并对这些信息进行多维度的分析；
- 视频直播，使用MongoDB存储用户信息、礼物信息等
- 大数据应用，使用云数据库MongoDB作为大数据 云存储系统，随时进行数据提取分析，掌握行业动态。

使用MongoDB的应用特征

- 应用不需要复杂/长事务及join支持
- 新应用，需求会变，数据模型无法确定，想快速迭代开发
- 应用需要2000-3000以上的读写QPS
- 应用需要TB甚至PB级别数据存储
- 应用发展迅速，需要能快速水平扩展
- 应用要求存储的数据不丢失
- 应用需要99.999%高可用
- 应用需要大量的地理位置查询、文本查询
## MongoDB实战
### 安装MongoDB
```shell
# 下载
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-rhel70-4.4.9.tgz
# 解压
tar -zxvf mongodb-linux-x86_64-rhel70-4.4.9.tgz
# 创建文件夹
mkdir -p /data/mongodb/data /data/mongodb/log /data/mongodb/conf
# 启动
./bin/mongod --port=27000 --dbpath=/data/mongodb/data --logpath=/data/mongodb/log/mongodb.log --bind_ip=0.0.0.0 --fork
# 关闭
./bin/mongod --port=27000 --dbpath=/data/mongodb/data --shutdown
```

- --dbpath 指定数据文件存放目录
- --logpath 指定日志文件，注意是指定文件不是目录
- --logappend 使用追加的方式记录日志
- --port 指定端口，默认27017
- --bind_ip 默认只监听localhost网卡
- --fork 后台启动
- --auth 开启认证模式

使用配置文件启动，yaml模式
```bash
systemLog:
  destination: file
  path: /data/mongodb/log/mongod.log
  logAppend: true
storage:
  dbPath: /data/mongodb/data
  engine: wiredTiger
  journal:
    enabled: true
net:
  bindIp: 0.0.0.0
  port: 27000
processManagement:
  fork: true
```
启动
```bash
./bin/mongod -f /data/mongodb/conf/mongo.conf
```
使用第二种方式关闭服务
```bash
./mongo --port=27000
use admin
db.shutdownServer()
```
### Mongo shell使用
| 命令 | 说明 |
| --- | --- |
| show dbs &#124; show databases | 显示数据库列表 |
| use 数据库名 | 切换数据库，如果不存在创建数据库 |
| show collections &#124; show tables | 显示当前数据库的集合列表 |
| db.集合名.stats() | 查看集合详情 |
| db.集合名.drop() | 删除集合 |
| show users | 显示当前数据库的用户列表 |
| show roles | 显示当前数据库的角色列表 |
| show profile | 显示最近发生的操作 |
| load("xxx.js") | 执行一个JavaScript脚本文件 |
| exit &#124; quit() | 退出当前shell |
| help | 查看mongodb支持哪些命令 |
| db.help() | 查询当前数据库支持的方法 |
| db.集合名.help() | 显示集合的帮助信息 |
| db.version() | 查看数据库版本 |

数据库操作
```bash
# 查看所有库
show dbs
# 切换到指定数据库，不存在则创建
use test
# 删除当前数据库
db.dropDatabase()
```
集合操作
```bash
# 查看集合
show collections
# 创建集合
db.createCollection("emp")
# 删除集合
db.emp.drop()
```
创建集合语法
```
db.createCollection(name,options)
```
options参数

| 字段 | 类型 | 描述 |
| --- | --- | --- |
| capped | 布尔 | （可选）如果为true，则创建固定集合。固定集合是指有着固定大小的集合，当达到最大值时，它会自动覆盖最早的文档。 |
| size | 数值 | （可选）为固定集合指定一个最大值（以字节计）。如果capped为true，也需要指定该字段。 |
| max | 数值 | （可选）指定固定集合中包含文档的最大数量。 |

注意：当集合不存在时，向集合中插入文档也会创建集合。

### 安全认证
创建管理员账号
```
# 设置管理员用户名密码需要切换到admin库
use admin
# 创建管理员
db.createUser({user:"fox",pwd:"fox",roles:["root"]})
# 查看所有用户
show users
# 删除用户
db.dropUser("fox")
```
常用权限

| 权限名 | 描述 |
| --- | --- |
| read | 允许用户读取指定数据库 |
| readWrite | 允许用户读写指定数据库 |
| dbAdmin | 允许用户在指定的数据库中执行管理函数，如创建索引、删除，查看统计或访问system.profile |
| dbOwner | 允许用户在指定的数据库中执行任意操作，增、删、改、查等 |
| userAdmin | 允许用户向system.users集合写入，可以在指定数据库里创建、删除和管理用户 |
| clusterAdmin | 只在admin数据库中可用，赋予用户所有分片和复制集相关函数的管理权限 |
| readAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的读权限 |
| readWriteAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的读写权限 |
| userAdminAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的userAdmin权限 |
| dbAdminAnyDatabase | 只在admin数据库中可用，赋予用户所有数据库的dbAdmin权限 |
| root | 只在admin数据库中可用，超级账号，超级权限 |

创建应用数据库用户
```
use appdb
db.createUser({user:"appdb",pwd:"fox",roles:["dbOwner"]})
```
默认情况下，MongoDB不会启用鉴权，以鉴权模式启动MongoDB
```
./bin/mongod -f /data/mongodb/conf/mongo.conf --auth
```
启用鉴权之后，连接MongoDB的相关操作都需要提供身份认证
```
./bin/mongod localhost:27000 -u fox -p fox --authenticationDatabase=admin
```
## MongoDB文档操作
### 插入文档
#### 新增单个文档

- insertOne：支持writeConcern
```
db.collection.insertOne(
	<document>,
	{
		writeConcern: <document>
	}
)
```
writeConcern决定一个写操作落到多少个节点上才算成功。writeConcern的取值包括：
0：发起写操作，不关心是否成功
1 - 集群最大数据节点数：写操作需要被复制到指定节点数才算成功
majority：写操作需要被复制到大多数节点上才算成功

- insert：若插入的数据主键已经存在，则会抛DuplicateKeyException异常，提示主键重复，不保存当前数据。
- save：如果_id主键存在则更新数据，如果不存在就插入数据。
#### 批量新增文档

- insertMany：向指定集合中插入多条文档数据
```
db.collection.insertMany(
	[<document 1>,<document 2>,...]
	{
  	writeConcern:<document>,
  	ordered: <boolean>
  }
)
```
writeConcern：写入策略，默认为1，即要求确认写操作，0是不要求
ordered：指定是否按顺序写入，默认true，按顺序写入

- insert和save也可以实现批量插入
### 查询文档
find查询集合中的若干文档。语法格式如下
```
db.collection.find(query,projection)
```

- query：可选，使用查询操作符指定查询条件
- projection：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值，只需省略该参数即可（默认省略）。投影时，id为1的时候，其他字段必须是1；id是0的时候，其他字段可以是0；如果没有_id字段约束，多个其他字段必须同为0或同为1。

如果查询返回的条目数量较多，mongo shell则会自动实现分批显示。默认情况下每次只显示20条，可以输入it命令读取下一批。
findOne查询集合中的第一个文档。语法格式如下：
```
db.collection.findOne(query,projection)
```
#### 条件查询
指定条件查询
```
# 查询带有nosql标签的book文档
db.books.find({tag:"nosql"})
# 按照id查询单个book文档
db.books.find({_id:ObjectId("")})
# 查询分类为“travel”、收藏数超过60个的book文档
db.books.find({type:"travel",favCount:{$gt:60}})
```
查询逻辑运算符

- $lt：存在并小于
- $lte：存在并小于等于
- $gt：存在并大于
- $gte：存在并大于等于
- $ne：不存在或存在但不等于
- $in：存在并在指定数组中
- $nin：不存在或不在指定数组中
- $or：匹配两个或多个条件中的一个
- $and：匹配全部条件
#### 排序&分页
指定排序
在MongoDB中使用sort()方法对数据进行排序
```
# 指定按收藏数（favCount）降序返回
db.books.find({type:"travel"}).sort({favCount:-1})
```
分页查询
skip用于指定跳过记录数，limit则用于限定返回结果数量。可以在执行find命令的同时指定skip、limit参数，以此实现分页的功能。比如，假定每页大小为8条，查询第3页的book文档
```
db.books.find().skip(8).limit(4)
```
#### 正则表达式匹配查询
**MongoDB使用$regex操作符来设置匹配字符串的正则表达式**
```
//使用正则表达式查找type包含so字符串的book
db.books.find({type:{$regex:"so"}})
//或者
db.books.find({type:/so/})
```
### 更新文档
可以用update命令对指定的数据进行更新，命令的格式如下：
```
db.collection.update(query,update,options)
```

- query：描述更新的查询条件
- update：描述更新的动作及新的内容
- options：描述更新的选项
   - upsert：可选，如果不存在update的记录，是否插入新的记录。默认false，不插入
   - multi：可选，是否按条件查询出的多条记录全部更新。默认false，只更新找到的第一条记录
   - writeConcern：可选，决定一个写操作落到多少个节点上才算成功。

**更新操作符**

| 操作符 | 格式 | 描述 |
| --- | --- | --- |
| $set | {$set:{field:value}} | 指定一个键并更新值，若键不存在则创建 |
| $unset | {$unset:{field:1}} | 删除一个键 |
| $inc | {$inc:{field:value}} | 对数值类型进行增减 |
| $rename | {$rename:{old_field_name:new_field_name}} | 修改字段名称 |
| $push | {$push:{field:value}} | 将数值追加到数组中，若数组不存在则会进行初始化 |
| $pushAll | {$pushAll:{field:value_array}} | 追加多个值到一个数组字段内 |
| $pull | {$pull:{field:_value}} | 从数组中删除指定的元素 |
| $addToSet | {$addToSet:{field:value}} | 添加元素到数组中，具有排重功能 |
| $pop | {$pop:{field:1}} | 删除数组的第一个或最后一个元素 |

#### 更新单个文档
某个book文档被收藏了，则需要将该文档的favCount字段自增
```
db.books.update({_id:ObjectId("xxxx")},{$inc:{favCount:1}})
```
#### 更新多个文档
默认情况下，update命令只在更新第一个文档之后返回，如果需要更新多个文档，则可以使用multi选项。
将分类为“novel”的文档的增加发布时间（publishedDate）
```
db.books.update({type:"novel"},{$set:{publishedDate:new Date()}},{"multi":true})
```
> multi：可选，mongodb默认是false，只更新找到的第一条记录，如果这个参数为true，就把按条件查出来多条记录全部更新

update命令的选项配置较多，为了简化使用还可以使用一些快捷命令：

- updateOne：更新某个文档
- updateMany：更新多个文档
- replaceOne：替换单个文档
#### 使用upsert命令
upsert是一种特殊的更新，其表现为如果目标文档不存在，则执行插入命令。
```
db.books.update(
	{title:"my book"},
	{$set:{tags:["nosql","mongodb"],type:"none",author:"fox"}},
	{upsert:true}
)
```
nMatched、nModified都为0，表示没有文档被匹配及更新，nUpserted=1提示执行了upsert动作。
#### 实现replace语义
update命令中的更新描述（update）通常由操作符描述，如果更新描述中不包含任何操作符，那么MongoDB会实现文档的replace语义
```
db.books.update(
	{title:"my book"},
	{justTitle:"my first book"}
)
```
#### findAndModify命令
findAndModify兼容了查询和修改指定文档的功能，findAndModify只能更新单个文档
```
db.books.findAndModify({
	query:{_id:ObjectId("xxx")},
	update:{$inc:{favCount:1}}
})
```
该操作会返回符合查询条件的文档数据，并完成对文档的修改。
默认情况下，findAndModify会返回修改前的旧数据。如果希望返回修改后的数据，则可以指定new选项
```
db.books.findAndModify({
	query:{_id:ObjectId("xxx")},
	update:{$inc:{favCount:1}},
	new:true
})
```
与findAndModify语义相近的命令如下：

- findOneAndUpdate：更新单个文档并返回更新前（或更新后）的文档。
- findOneAndReplace：替换单个文档并返回替换前（或替换后）的文档。
### 删除文档
#### 使用remove删除文档

- remove命令需要配合查询条件
- 匹配查询条件的文档会被删除
- 指定一个空文档条件会删除所有文档
```
db.user.remove({age:28})
db.user.remove({age:{$lt:25}})
db.user.remove({})//删除全部记录
db.user.remove()//报错
```
remove命令会删除匹配条件的全部文档，如果希望明确限定只删除一个文档，则需要指定justOne参数，命令格式如下：
```
db.collection.remove(query,justOne)
```
例如：删除满足type:novel条件的首条记录
```
db.books.remove({type:"novel"},true)
```
#### 使用delete删除文档
官方推荐使用deleteOne()和deleteMany()方法删除文档，语法格式如下：
```
db.books.deleteMany({})
db.books.deleteMany({type:"novel"}) //删除type等于novel的全部文档
db.books.deleteOne({type:"novel"}) //删除type等于novel的一个文档
```
注意：remove、deleteMany等命令需要对查询范围内的文档逐个删除，如果希望删除整个集合，则使用drop命令会更加高效
#### 返回被删除文档
remove、deletelOne等命令在删除文档后只会返回确认性的信息，如果希望获得被删除的文档，则可以使用findOneAndDelete命令
```
db.books.findOneAndDelete({type:"novel"})
```
除了在结果中返回删除文档，findOneAndDelete命令还允许定义“删除的顺序”，即按照指定顺序删除找到的第一个文档
```
db.books.findOneAndDelete({type:"novel"},{sort:{favCount:1}})
```
remove、deleteOne等命令只能按默认顺序删除，利用这个特性，findOneAndDelete可以实现队列的先进先出。
### 聚合操作
聚合操作处理数据记录并返回计算结果（诸如统计平均值，求和等）。聚合操作组值来自多个文档，可以对分组数据执行各种操作以返回单个结果。聚合操作包含三类：单一作用聚合、聚合管道、MapReduce

- 单一作用聚合：提供了对常见聚合过程的简单访问，操作都从单个集合聚合文档。
- 聚合管道是一个数据聚合的框架，模型基于数据处理流水线的概念。文档进入多级管道，将文档转换为聚合结果。
- MapReduce操作具有两个阶段：处理每个文档并向每个输入文档发射一个或多个对象的map阶段，以及reduce组合map操作的输出阶段。
#### 单一作用聚合
MongoDB提供db.collection.estimatedDocumentCount(),db.collection.count(),db.collection.distinct()这类单一作用的聚合函数。所有这些操作都聚合来自单个集合的文档。虽然这些操作提供了对公共聚合过程的简单访问，但它们缺乏聚合管道和map-Reduce的灵活性和功能。

| 函数 | 描述 |
| --- | --- |
| db.collection.estimatedDocumentCount() | 忽略查询条件，返回集合或视图中所有文档的计数 |
| db.collection.count() | 返回与find()集合或视图的查询匹配的文档计数。等同于db.collection.find(query).count()构造 |
| db.collection.distinct() | 在单个集合或视图中查找指定字段的不同值，并在数组中返回结果 |

```
# 检索books集合中所有文档的计数
db.books.estimatedDocumentCount()
# 计算与查询匹配的所有文档
db.books.count({favCount:{$gt:50}})
# 返回不同type的数组
db.books.distinct("type")
# 返回收藏数大于90的文档不同type的数组
db.books.distinct("type",{favCount:{$gt:90}})
```
注意：在分片集群上，如果存在孤立文档或正在进行块迁移，则db.collection.count()没有查询谓词可能导致计数不准确。要避免这些情况，请在分片集群上使用db.collection.aggregate()方法。
#### 聚合管道
MongoDB聚合框架（Aggregation Framework）是一个计算框架，它可以

- 作用在一个或几个集合上
- 对集合中的数据进行一系列运算
- 将这些数据转化为期望的形式

从效果而言，聚合框架相当于SQL查询中的GROUP BY、LEFT OUTER JOIN、AS等
**管道（Pipeline）和阶段（Stage）**
整个聚合运算过程称为管道（Pipeline），它是由多个阶段（Stage）组成，每个管道：

- 接受一系列文档（原始数据）
- 每个阶段对这些文档进行一系列运算
- 结果文档输出给下一个阶段

聚合管道操作语法
```
pipeline=[$stage1,$stage2,...$stageN];
db.collection.aggregate(pipeline,{options})
```

- pipelines 一组数据聚合阶段。除$out、$Merge和$geonear阶段之外，每个阶段都可以在管道中出现多次。
- options 可选，聚合操作的其他参数。包含：查询计划、是否使用临时文件、游标、最大操作时间、读写策略、强制索引等等 

**常用的管道聚合阶段**

| 阶段 | 描述 | SQL等价运算符 |
| --- | --- | --- |
| $match | 筛选条件 | WHERE |
| $project | 投影 | AS |
| $lookup | 左外连接 | LEFT OUTER JOIN |
| $sort | 排序 | ORDER BY |
| $group | 分组 | GROUP BY |
| $skip/$limit | 分页 |  |
| $unwind | 展开数组 |  |
| $graphLookup | 图搜索 |  |
| $facet/$bucket | 分面搜索 |  |

##### 聚合操作示例
**统计每个分类的book文档数量**
```
db.books.aggregate([
	{$group:{_id:"$type",total:{$sum:1}}},
	{$sort:{total:-1}}
])
```
**标签的热度排行，标签的热度则按其关联book文档的收藏数（favCount）来计算**
```
db.books.aggregate({
	{$match:{favCount:{$gt:0}}},
	{$unwind:"$tag"},
	{$group:{_id:"$tag",total:{$sum:"$favCount"}}},
	{$sort:{total:-1}}
})
```

- $match阶段：用于过滤favCount=0的文档
- $unwind阶段：用于将标签数组进行展开，这样一个包含3个标签的文档会被拆解为3个条目
- $group阶段：对拆解后的文档进行分组计算，$sum:"$favCount"表示按favCount字段进行累加
- $sort阶段：接收分组计算的输出，按total得分进行排序

**统计book文档收藏数[0,10),[10,60),[60,80),[80,100),[100,-)**
db.books.aggregate([{
$bucket:{
groupBy:"$favCount",
boundaries:[0,10,60,80,100],
default:"other",
output:{"count":{$sum:1}}
}
}])
**统计分析数据**
导入邮政编码数据集：https://media.mongodb.org/zips.json
使用mongoimport工具导入数据
```
mongoimport -h localhost -d test -u fox -p fox 
 	--authenticationDatabase=admin -c zips --file D:\TempData\zips.json
```
> h --host：代表远程连接的数据库地址，默认连接本地MongoDB数据库
> --port ：代表远程连接的数据库的端口，默认连接的远程端口27017
> -u，--username：代表连接远程数据库的账号，如果设置数据库的认证，需要指定用户账号
> -p，--password：代表连接数据库的账号对应的密码
> -d，--db：代表连接的数据库
> -c，--collection：代表连接数据库中的集合
> -f，--fields：代表导入集合中的字段
> --type：代表导入的文件类型，包括csv和json，tsv文件，默认json格式
> --file：导入的文件名称
> --headerline：导入csv文件时，指明第一行是列名，不需要导入

**返回人口超过1000万的州**
```
db.zips.aggregate([
	{$group:{_id:"$state",totalPop:{$sum:"$pop"}}},
  {$match:{totalPop:{$gt:1000*10000}}}
])
```
这个聚合操作的等价SQL是：
```
select state,sum(pop) as totalPop
from zips
group by state
having totalPop >= (1000*10000)
```
**返回各州平均城市人口**
```
db.zips.aggregate([
  	{$group:{_id:{state:"$state",city:"$city"},pop:{$sum:"$pop"}}},
  	{$group:{_id:"$_id.state",avgCityPop:{$avg:"$pop"}}}
])
```
**按州返回最大和最小的城市**
```
db.zips.aggregate([
	 {
      $group:
      	{
        	_id: {state,"$state",city:"$city"},
        	pop: {$sum:"$pop"}
        }
   },
   {$sort:{pop:1}},
   {
    	$group:
    	{
        	_id:"$_id.state",
        	biggestCity:{$last:"$_id.city"},
        	biggestPop: {$last:"$pop"},
        	smallestCity:{$first:"$_id.city"},
        	smallestPop:{$first:"$pop"}
      }
   },
	 {
    	$porject:
    	{
      	_id:0,
      	state:"$_id",
      	biggestCity:{name:"$biggestCity",pop:"$biggestPop"},
      	smallestCity:{name:"$smallestCity",pop:"$smallestPop"}
      }
   }
])
```
#### MapReduce
MapReduce操作将大量的数据处理工作拆分成多个线程并行处理，然后将结果合并在一起。MongoDB提供的Map-Reduce非常灵活，对于大规模数据分析也相当实用。
MapReduce具有两个阶段：

- 将具有相同Key的文档数据整合在一起的map阶段
- 组合map操作的结果进行统计输出的reduce阶段
```
db.collection.mapReduce(
	function(){emit(key,value);},//map函数
  function(key,values){return reduceFunction},//reduce函数	
	{
  	out:<collection>,
  	query:<document>,
  	sort:<document>,
  	limit:<number>,
  	finalize:<function>,
  	scope:<document>,
  	jsMode:<boolean>,
  	verbose:<boolean>,
  	bypassDocumentValidation:<boolean>
  }
)
```

- map 将数据拆分成键值对，交给reduce函数
- reduce 根据键将值做统计运算
- out 可选，将结果汇入指定表
- query 可选筛选数据的条件，筛选的数据送入map
- sort 排序完成后，送入map
- limit 限制送入map的文档数
- finalize 可选，修改reduce的结果后进行
- scope 可选，指定map、reduce、finalize的全局变量
- jsMode 可选，默认false。在mapreduce过程中是否将数据转换为json格式
- verbose 可选，是否在结果中显示时间，默认false
- bypassDocumentValidation 可选，是否略过数据校验

统计type为travel的不同作者的book文档收藏数
```
db.books.mapReduce(
	function(){emit(this.author.name,this.favCount)},
	function(key,values){return Arrays.sum(values)},
	{
  	query:{type:"travel"},
  	out:"book_favCount"
  }
)
```
### 视图
MongoDB视图是一个可查询的对象，它的内容由其他集合或视图上的聚合管道定义。MongoDB不会将视图内容持久化到磁盘。当客户端查询视图时，视图的内容按需计算。MongoDB可以要求客户端具有查询视图的权限。MongoDB不支持对视图进行写操作。
作用

- 数据抽象
- 保护敏感数据的一种方法
- 将敏感数据投影到视图之外
- 只读
- 结合基于角色的授权，可按角色访问信息
#### 创建视图
基本语法格式
```
db.createView(
	"<viewName>",
	"<source>",
	[<pipeline>],
  {
  	"collation":{<collation>}
  }
)
```

- viewName：必须，视图名称
- source：必须，数据源，集合/视图
- [<pipeline>]：可选，一组管道
- collation 可选，排序规则
##### 单个集合创建视图
假设现在查看当天最高的10笔订单视图，例如需要实时显示金额最高的订单
```
db.createView(
	"orderInfo", //视图名称
	"order",//数据源
	[
  	//筛选条件
  	{$match:{"orderTime":{$gte:ISODate("2022-01-26T00:00:00.000Z")}}},
  	//按金额排序
  	{$sort:{"price":-1}},
  	//限制10个文档
  	{$limit:10},
    //选择要显示的字段
   	// 0 排他字段，若字段上使用(_id除外)，就不能有其他包含字段
  	// 1 包含字段
  	{$project:{_id:0,orderNo:1,price:1,orderTime:1}}
  ]
)
```
##### 多个集合创建视图
跟单个集合是一样，只是多个了$lookup连接操作符，视图根据管道最终结果显示，所以可以关联多个集合
```
db.orderDetail.drop()
db.createView(
	"orderDetail",
	"order",
	[
  	{$lookup:{from:"shipping",localField:"orderNo",foreignField:"orderNo",as:"shipping"}},
  	{$porject:{"orderNo":1,"price":1,"shipping.address":1}}
  ]
)
```
#### 修改视图
```
db.runCommand({
  collMod:"orderInfo",
	viewOn:"order",
	pipeline:[
  	{$match:{"orderTime":{$gte:ISODate("2022-01-26T00:00:00.000Z")}}},
	  {$sort:{"price":-1}},
    {$limit:10},
    {$project:{_id:0,orderNo:1,price:1,qty:1,orderTime:1}}
  ]
})
```
#### 删除视图
```
db.orderInfo.drop();
```
### 索引
#### 索引介绍
索引是一种用来快速查询数据的数据结构。B+Tree就是一种常用的数据库索引数据结构，MongoDB采用B+Tree做索引，索引创建在collections上。MongoDB不使用索引的查询，先扫描所有的文档，再匹配符合条件的文档。使用索引的查询，通过索引找到文档，使用索引能够极大的提升查询效率。

注意：MongoDB具体使用的是B+Tree，因为B+Tree是B-Tree的子集。所以叫B-Tree也对，但容易产生误导。
**索引分类**

- **按照索引包含的字段数量，可以分为单键索引和组合索引（或符合索引）**
- **按照索引字段的类型，可以分为主键索引和非主键索引**
- **按照索引节点与物理记录的对应方式来分，可以分为聚簇索引和非聚簇索引，其中聚簇索引是指索引节点上直接包含了数据记录，而后者则仅仅包含一个指向数据记录的指针。**
- **按照索引的特性不同，又可以分为唯一索引、稀疏索引、文本索引、地理空间索引等**
#### 索引操作
##### 创建索引
创建索引语法格式
```
db.collection.createIndex(keys,options)
```

- key值为你要创建的索引字段 1 按升序创建索引 -1 按降序创建索引
- 可选参数列表如下：
| Parameter | Type | Description |
| --- | --- | --- |
| background | boolean | 建索引过程会阻塞其他数据库操作，background可指定以后台方式创建索引，即增加“background”可选参数，默认为false |
| unique | boolean | 建立的索引是否唯一。指定为true创建唯一索引。默认值为false |
| name | string | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称 |
| dropDups | boolean | 3.0+版本已废弃。在建立唯一索引时是否删除重复记录，指定true创建唯一索引。默认值为false |
| sparse | boolean | 在文档中不存在的字段数据不启用索引。这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档。默认值为false |
| expireAfterSeconds | integer | 指定一个以秒为单位的数值，完成TTL设定，设定集合的生存时间 |
| v | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本 |
| weights | document | 索引权重值，数值在1到99999之间，表示该索引相对于其他索引字段的得分权重。 |
| default_language | string | 对于文本索引，该参数决定了停用词及词干和词器的规则列表。默认为英语。 |
| language_override | string | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为language |

实例
```
db.values.createIndex({open:1,close:1},{background:true})
db.values.createIndex({title:1},{unique:true})
```
##### 查看索引
```
# 查看索引信息
db.books.getIndexes()
# 查看索引键
db.books.getIndexKeys()
```
查看索引占用空间
```
db.collection.totalIndexSize([is_detail])
```

- is_detail：可选参数，传入除0或false外的任意数据，都会显示该集合中每个索引的大小及总大小。如果传入0或false则只显示该集合中所有索引的总大小。默认值为false。
##### 删除索引
```
# 删除集合指定索引
db.col.dropIndex("索引名称")
# 删除集合所有索引
db.col.dropIndexes()
```
#### 索引类型
##### 单键索引
Signle Field Indexes
在某一个特定的字段上建立索引mongoDB在ID上建立唯一的单键索引，所以经常会使用id来进行查询；在索引字段上进行精确匹配、排序以及范围查找都会使用此索引。
```
db.books.createIndex({title:1})

# 对内嵌文档字段创建索引
db.books.createIndex({"author.name":1})
```
##### 复合索引
Compound Index
复合索引是多个字段组合而成的索引，其性质和单字段索引类似。但不同的是，复合索引中字段的顺序、字段的升降对查询性能有直接的影响，因此在设计复合索引时则需要考虑不同的查询场景。
```
db.books.createIndex({type:1,favCount:1})
```
##### 多键索引
Multikey Index
在数组的属性上建立索引。针对这个数组的任意值的查询都会定位到这个文档，既多个索引入口或者键值引用同一个文档
准备inventory集合：
```
db.inventory.insertMany([
	{_id:5,type:"food",item:"aaa",ratings:[5,8,9]},
	{_id:6,type:"food",item:"aaa",ratings:[5,9]},
	{_id:7,type:"food",item:"aaa",ratings:[5,8]}
])
```
创建多键索引
```
db.inventory.createIndex({ratings:1})
```
多键索引很容易与复合索引产生混淆，复合索引是多个字段的组合，而多键索引则仅仅是在一个字段上出现了多键（multikey）。而实质上，多键索引也可以出现在复合字段上。
```
# 创建复合多键索引
db.inventory.createIndex({item:1,ratings:1})
```
注意：MongoDB并不支持一个复合索引中同时出现多个数组字段。

在包含嵌套对象的数组字段上创建多键索引
```
db.inventory.createIndex({"stock.size":1,"stock.quantity":1})
```
##### 地理空间索引
Geospatial Index
在移动互联网时代，基于地理位置的检索（LBS）功能几乎是所有应用系统的标配。MongoDB为地理空间检索提供了非常方便的功能。地理位置索引（2dsphereindex）就是专门用于实现位置检索的一种特殊索引。
案例：MongoDB如何实现“查询附近商家”
假如商家的数据模型如下：
```
db.restaurant.insert({
	restaurantId:0,
	restaurantName:"兰州牛肉面",
 	location:{
  	type:"Point",
  	coordinates:[-73.97,40.77]
  }
})
```
创建2dsphere索引
```
db.restaurant.createIndex({location:"2dsphere"})
```
查询附近1000米商家信息
```
db.restaurant.find({
	location:{
  	$near:{
    	$geometry:{
      	  type:"Point",
        	coordinates:[-73.88,40.78]
      },
    	$maxDistance:1000
    }
  }
})
```

- $near查询操作符，用于实现附近商家的检索，返回数据结果会按距离排序
- $geometry操作符用于指定一个GeoJSON格式的地理空间对象，type=Point表示地理坐标点，coordinates则是用户当前所在的经纬度位置，$maxDistance限定了最大距离，单位是米
##### 全文索引
Text Indexes
MongoDB支持全文检索功能，可通过建立文本索引来实现简易的分词检索。
```
db.reviews.createIndex({comments:"text"})
```
$text操作符可以在有text index的集合上执行文本检索。$text将会使用空格和标点符合作为分隔符对检索字符串进行分词，并且对检索字符串中所有的分词结果进行一个逻辑上的OR操作。
全文索引能解决快速文本查找的需求，比如有一个博客文章集合，需要根据博客的内容来快速查找，则可以针对博客内容建立文本索引。
##### Hash索引
Hashed Indexes 
不同于传统的B-Tree索引，哈希索引使用hash函数来创建索引。在索引字段上进行精确匹配，但不支持范围查询，不支持多键hash，Hash索引上的入口是均匀分布的，在分片集合中非常有用。
```
db.users.createIndex({username:'hashed'})
```
##### 通配符索引
Wildcard Indexes
MongoDB的文档模式动态变化的，而通配符索引可以建立在一些不可预知的字段上，以此实现查询的加速。MongoDB4.2引入了通配符索引来支持对未知或任意字段的查询。
创建通配符索引
```
db.products.createIndex({"product_attributes.$**":1})
```
注意事项

- 通配符索引不兼容的索引类型或属性
   - Compound
   - TTL
   - Text
   - 2d(Geospatial)
   - 2dsphere(Geospatial)
   - Hashed
   - Unique
- 通配符索引是稀疏的，不索引空字段。因此，通配符索引不能支持查询字段不存在的文档
- 通配符索引为文档或数组的内容生成条目，而不是文档/数组本身。因此通配符索引不能支持精确的文档/数组相等匹配。通配符索引可以支持查询字段等于空文档{}的情况。
#### 索引属性
##### 唯一索引
Unique Indexes
在现实场景中，唯一性是很常见的一种索引约束需求，重复的数据记录会带来许多处理上的麻烦。通过建立唯一性索引，可以保证集合中文档的指定字段拥有唯一值。

```
# 创建唯一索引
db.values.createIndex({title:1},{unique:true})
# 复合索引支持唯一性约束
db.values.createIndex({title:1,type:1},{unique:true})
# 多键索引支持唯一性约束
db.inventory.createIndex({ratings:1},{unique:true})
```

- 唯一性索引对于文档中缺失的字段，会使用null值代替，因此不允许存在多个文档缺失索引字段的情况
- 对于分片的集合，唯一性约束必须匹配分片规则。换句话说，为了保证全局唯一性，分片键必须作为唯一性索引的前缀字段
##### 部分索引
Partial Indexes
部分索引仅对满足指定过滤器表达式的文档进行索引。通过在一个集合中为文档的一个子集建立索引，部分索引具有更低的存储需求和更低的索引创建和维护的性能成本。3.2新版功能。
部分索引提供了稀疏索引功能的超集，应该优先于稀疏索引。
```
db.restaurants.createIndex(
	{cuisine:1,name:1},
	{partialFilterExpression:{rating:{$gt:5}}}
)
```
partialFilterExpression选项接受指定过滤条件的文档

- 等式表达式（例如：field:value或使用$eq操作符）
- $exists:true
- $gt $gte $lt $lte
- $type
- 顶层的$and
```
# 符合条件，使用索引
db.restaurants.find(cuisine:"Italian",rating:{$gte:8})
# 不符合条件，不能使用索引
db.restaurants.find(cuisine:"Italian")
```
** 唯一约束结合部分索引使用导致唯一约束失效的问题**
注意：如果同时指定partialFilterExpression和唯一约束，那么唯一约束只适用于满足筛选器表达式的文档。如果文档不满足筛选条件，那么带有唯一约束的部分索引不会阻止插入不满足唯一约束的文档。
##### 稀疏索引
Sparse Indexes
索引的稀疏属性确保索引只包含具有索引字段的文档的条目，索引将跳过没有索引索引字段的文档。
特性：只对存在字段的文档进行索引（包括字段值为null的文档）
```
# 不索引不包含xmpp_id字段的文档
db.addresses.createIndex({"xmpp_id":1},{sparse:true})
```
如果稀疏索引会导致查询和排序操作的结果集不完整，MongoDB将不会使用该索引，除非hint()明确指定索引
##### TTL索引
在一般的应用系统中，并非所有的数据都需要永久存储。例如一些系统事件、用户消息等，这些数据随着时间的推移，其重要程度逐渐降低。更重要的是，存储这些大量的历史数据需要花费较高的成本，因此项目中通常会对过期且不再使用的数据进行老化处理。
通常的做法是：

- 方案一：为每个数据记录一个时间戳，应用侧开启一个定时器，按时间戳定期删除过期的数据
- 方案二：数据按日期进行分表，同一天的数据归档到同一张表，同样使用定时器删除过期的表

对于数据老化，MongoDB提供了一种更加简便的做法：TTL（Time To Live）索引。TTL索引需要声明在一个日期类型的字段中，TTL索引是特殊的单字段索引。MongoDB可以使用它在一定时间或特定时钟时间后自动从集合中删除文档。
```
# 创建TTL索引，TTL值为3600秒
db.eventlog.createIndex({"lastModifiedDate":1,{expireAfterSeconds:3600}})
```
对集合创建TTL索引之后，MongoDB会在周期性运行的后台线程中对该集合进行检查及数据清理工作。除了数据老化功能，TTL索引具有普通索引的功能，同样可以用于加速数据的查询。
TTL索引不保证过期数据会在过期过期后立即被删除。文档过期和MongoDB从数据库中删除文档的时间之间可能存在延迟。删除过期文档的后台任务每60秒运行一次。因此，在文档到期和后台任务运行之间的时间段内，文档可能会保留在集合中。
**可变的过期时间**
TTL索引在创建之后，仍然可以对过期时间进行修改。这需要使用callMod命令对索引的定义进行变更
```
db.runCommand({
	collMod:"log_events",
	index:{keyPattern:{createdAt:1},expireAfterSeconds:600}
})
```
**使用约束**
TTL索引的确可以减少开发的工作量，而且通过数据库自动清理的方式会更加高效、可靠，但是在使用TTL索引时需要注意以下的限制：

- TTL索引只能支持单个字段，并且必须是非_id字段
- TTL索引不能用于固定集合
- TTL索引无法保证及时的数据老化，MongoDB会通过后台的TTLMonitor定时器来清理老化数据，默认的间隔时间是1分钟，当然如果在数据库负载过高的情况下，TTL的行为则会进一步受到影响。
- TTL索引对于数据的清理仅仅使用了remove命令，这种方式并不是很高效。因此TTL Monitor在运行期间对系统CPU、磁盘都会造成一定的压力。相比之下，按日期分表的方式操作会更加高效。
##### 隐藏索引
Hidden Indexes
隐藏索引对查询规划器不可见，不能用于支持查询。通过对规划器隐藏索引，用户可以在不实际删除索引的情况下评估删除索引的潜在影响。如果影响是负面的，用户可以取消隐藏索引，而不必重新创建已删除的索引。4.4新版功能。
```
# 创建隐藏索引
db.restaurants.createIndex({borough:1},{hidden:true})
# 隐藏现有索引
db.restaurants.hideIndex({borough:1})
db.restaurants.hideIndex("索引名称")
# 取消隐藏索引
db.restaurants.unhideIndex({borough:1})
db.restaurants.unhideIndex("索引名称")
```
#### 索引使用建议
**为每一个查询建立合适的索引**
这是针对于数据量较大比如超过几十上百万（文档数目）数量级的集合。如果没有索引MongoDB需要把所有的Document从盘上读到内存，这会对MongoDB服务器造成较大的压力并影响到其他请求的执行。
**创建合适的复合索引，不要依赖于交叉索引**
如果你的查询会使用多个字段，MongoDB有两个索引技术可以使用：交叉索引和复合索引。交叉索引就是针对每个字段单独建立一个单字段索引，然后在查询执行时候使用相应的单字段索引进行索引交叉而得到查询结果。交叉索引目前触发率较低，所以如果你有一个多字段查询的时候，建议使用复合索引能够保证索引正常的使用。
```
# 查找所有年龄小于30岁的深圳市马拉松运动员
db.athelets.find({sport:"marathon",location:"sz",age:{$lt:30}})
# 创建符合索引
db.athelets.createIndex({sport:1,location:1,age:1})
```
**符合索引字段顺序：匹配条件在前，范围条件在后（Equality First， Range After）**
前面的例子，在创建复合索引时如果条件有匹配和范围之分，那么匹配条件（sport:"marathon"）应该在复合索引的前面。范围条件(age:<30)字段应该放在复合索引的后面。
**尽可能使用覆盖索引（Covered Index）**
**建索引要在后台运行**
在对一个集合创建索引时，该集合所在数据库将不接受其他读写操作。对大数据量的集合建索引，建议使用后台运行选项{background:true}
#### explain执行计划详解
通常我们需要关心的问题

- 查询是否使用了索引
- 索引是否减少了扫描的记录数量
- 是否存在低效的内存排序

MongoDB提供了explain命令，它可以帮助我们评估指定查询模型（querymodel）的执行计划，根据实际情况进行调整，然后提高查询效率。
explain()方法的形式如下：
```
db.collection.find().explain(<verbose>)
```
verbose可选参数，表示执行计划的输出模式，默认queryPlanner

| 模式名称 | 描述 |
| --- | --- |
| queryPlanner | 执行计划的详细信息，包括查询计划、集合信息、查询条件、最佳执行计划、查询方式和MongoDB服务信息等 |
| exectionStats | 最佳执行计划的执行和被拒绝的计划等信息 |
| allPlansExecution | 选择并执行最佳执行计划，并返回最佳执行计划和其他执行计划的执行情况 |

##### queryPlanner
```
# 未创建title的索引
db.books.find({title:"book-1"}).explain("queryPlanner")
```
| 字段名称 | 描述 |
| --- | --- |
| plannerVersion | 执行计划的版本 |
| namespace | 查询的集合 |
| indexFilterSet | 是否使用索引 |
| parsedQuery | 查询条件 |
| winningPlan | 最佳执行计划 |
| stage | 查询方式 |
| filter | 过滤条件 |
| direction | 查询顺序 |
| rejectedPlans | 拒绝的执行计划 |
| serverInfo | mongodb服务器信息 |

##### executionStats
executionStats模式的返回信息中包含了queryPlanner模式的所有字段，并且还包含了最佳执行计划的执行情况
```
db.books.createIndex({title:1})
db.books.find({title:"book-1"}).explain("executionStats")
```
| 字段名称 | 描述 |
| --- | --- |
| winningPlan.inputStage | 用来描述子stage，并且为其父stage提供文档和索引关键字 |
| winningPlan.inputStage.stage | 子查询方式 |
| winningPlan.inputStage.keyPattern | 所扫描的inex内容 |
| winningPlan.inputStage.indexName | 索引名 |
| winningPlan.inputStage.isMultiKey | 是否是multiKey。如果索引建立在array上，将是true |
| executionStats.executionSuccess | 是否执行成功 |
| executionStats.nReturned | 返回的个数 |
| executionStats.executionTimeMillis | 这条语句执行时间 |
| executionStats.executionStages.executionTimeMillisEstimate | 检索文档获取数据的时间 |
| executionStats.executionStages.inputStage.executionTimeMillisEstimate | 扫描获取数据的数据 |
| executionStats.totalKeysExamined | 索引扫描次数 |
| executionStats.totalDocsExamined | 文档扫描次数 |
| executionStats.executionStages.isEOF | 是否到达steam结尾，1或者true代表已到达结尾 |
| executionStats.executionStages.works | 工作单元数，一个查询会分解成小的工作单元 |
| executionStats.executionStages.advanced | 优先返回的结果数 |
| executionStats.executionStages.docsExamined | 文档检查数 |

##### allPlansExecution
allPlansExecution返回的信息包含executionStats模式的内容，且包含allPlansExecution:[]块

| 状态 | 描述 |
| --- | --- |
| COLLSCAN | 全表扫描 |
| IXSCAN | 索引扫描 |
| FETCH | 根据索引检索指定文档 |
| SHARD_MERGE | 将各个分片返回数据进行合并 |
| SORT | 在内存中进行了排序 |
| LIMIT | 使用了limit限制了返回数 |
| SKIP | 使用了skip进行跳过 |
| IDHACK | 对_id进行查询 |
| SHARDING_FILTER | 通过mongos对分片数据进行查询 |
| COUNTSCAN | count不使用index进行count时的stage返回 |
| COUNT_SCAN | count使用了index进行count时的stage返回 |
| SUBPLA | 未使用到索引的$or查询的stage返回 |
| TEXT | 使用全文索引进行查询的stage返回 |
| PROJECTION | 限定返回字段时候stage返回 |

执行计划的返回结果中尽量不要出现以下stage

- COLLSCAN（全表扫描）
- SORT（使用sort但是无index）
- 不合理skip
- SUBPLA（未用到index的$or）
- COUNTSCAN（不使用index进行count）
