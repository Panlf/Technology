# 图数据库Neo4j

图数据库是基于图论实现的一种NoSQL数据库，其数据存储结构和数据查询方式都是以图论为基础的，图数据库主要用于存储更多的连接数据。

NoSQL

- 键值数据库
- 列存储数据库
- 文档型数据库
- 图数据库

| 分类         | 数据类型           | 优势                                                 | 劣势                                           | 举例              |
| ------------ | ------------------ | ---------------------------------------------------- | ---------------------------------------------- | ----------------- |
| 键值数据库   | 哈希表             | 查找速度快                                           | 数据无结构化，通常只被当作字符串或者二进制数据 | Redis             |
| 列存储数据库 | 列式数据存储       | 查找速度快；支持分布横向扩展；数据压缩率高           | 功能相对受限                                   | HBase             |
| 文档型数据库 | 键值对扩展         | 数据结构要求不严格；表结构可变；不需要预先定义表结构 | 查询性能不高，缺乏统一的查询语法               | MongoDB           |
| 图数据库     | 节点和关系组成的图 | 利用图结构相关算法（最短路径、节点度关系查找等）     | 可能需要对整个图做计算，不利于图数据分布存储   | Neo4j、JanusGraph |



## 什么是Neo4j

Neo4j是一个开源的Neo4j图形数据库，2003年开始开发，使用scala和java语言，2007年开始发布。

- 是世界上最先进的图数据库之一，提供原生的图数据库存储，检索和处理
- 采用属性图模型，极大的完善和丰富图数据模型
- 专属查询语言Cypher，直观，高效

Neo4j的特性

- SQL就像简单的查询语言Neo4j CQL
- 它遵循属性图数据模型
- 它通过使用Apache Lucence支持索引
- 它支持UNIQUE约束
- 它包含一个用于执行CQL命令的UI：Neo4j数据浏览器
- 它支持完整的ACID（原子性、一致性、隔离性和持久性）规则
- 它采用原生图形库与本地GPE（图形处理引擎）
- 它支持查询的数据导出到JSON和XLS格式
- 它提供了REST API，可以被任何编程语言访问
- 它提供了可以通过任何UI MVC框架访问的Java脚本
- 它支持两种Java API: Cypher API和Native Java API来开发Java应用程序

## Neo4j的构建元素

- 节点
- 属性
- 关系
- 标签
- 数据浏览器

### 节点

节点（Node）是图数据库中的基本元素，用以表示一个实体记录，就像关系数据库中的一条记录一样。在Neo4j中节点可以包含多个属性（Property）和多个标签（Label）

### 关系

关系（Relationship）同样是图数据库中的基本元素。当数据库中已经存在节点后，需要使用关系将节点连接起来构成图。关系也称为图论的边，始端和末端必须是节点，不能指向空也不能从空发起。关系和节点一样可以包含多个属性，但关系只能有一个类型（Type）。关系是有方向的，在图的遍历操作中可以指定关系遍历的方向或者指定为无方向；在创建关系时不必为两个节点创建相互指向的关系，在遍历时不指定遍历方向即可。

### 属性

属性由键值对组成，就像java的哈希表一样，属性名类似变量名，属性值类似变量值。属性值没有null的概念，如果属性不需要直接移除即可，使用Cypher或Java API时，可用IS NULL关键字判断属性是否存在

### 路径

任意两个节点间都可能存在路径，路径的长度指路径中关系的条数。一个节点可以组成长度为0的路径。

### 标签

Lable 将一个公共名称与一组节点或关系相关联。节点或关系可以包含一个或多个标签。可以为现有节点或关系和创建新、删除标签。

### 数据浏览器

Neo4j 自带的数据浏览器，http://xxx.xxx.xxx.xxx:7474/browser

- 用于执行 CQL 命令并查询输入输出
- $ 提示符出执行所有 CQL 命令
  

## Neo4j使用场景

- 欺诈检测
- 实时推荐引擎
- 知识图谱
- 主数据管理
- 供应链管理
- 增强网络和IT运营管理能力
- 数据谱系
- 身份和访问管理
- 材料清单
- 社交网络

## Neo4j安装

```shell
docker run -d --name neo4j \  //-d表示容器后台运行 --name指定容器名字
	-p 7474:7474 -p 7687:7687 \  //映射容器的端口号到宿主机的端口号
	-v /neo4j/data:/data \  //把容器内的数据目录挂载到宿主机的对应目录下
	-v /neo4j/logs:/logs \  //挂载日志目录
	-v /neo4j/conf:/var/lib/neo4j/conf   //挂载配置目录
	-v /neo4j/import:/var/lib/neo4j/import \  //挂载数据导入目录
	-e NEO4J_AUTH=neo4j/12345678 \  //设定数据库的名字的访问密码
	neo4j //指定使用的镜像
```

```she
docker run -d --name neo4j \
	-p 7474:7474 -p 7687:7687 \
	-v /data/volumedata/neo4j/data:/data \
	-v /data/volumedata/neo4j/logs:/logs \
	-v /data/volumedata/neo4j/conf:/var/lib/neo4j/conf  \
	-v /data/volumedata/neo4j/import:/var/lib/neo4j/import \
	-e NEO4J_AUTH=neo4j/12345678 \
	neo4j 
```

## Neo4j - CQL使用

### Neo4j - CQL 简介

Neo4j的Cypher语言是为处理图形数据而构建的，CQL代表Cypher查询语言。

- 它是Neo4j图形数据库的查询语言
- 它是一种声明性模式匹配语言
- 它遵循SQL语法
- 它的语法是非常简单且人性化、可读的格式

| CQL命令  | 用法                         |
| -------- | ---------------------------- |
| CREATE   | 创建节点，关系和属性         |
| MATCH    | 检索有关节点，关系和属性数据 |
| RETURN   | 返回查询结果                 |
| WHERE    | 提供条件过滤检索数据         |
| DELETE   | 删除节点和关系               |
| REMOVE   | 删除节点和关系的属性         |
| ORDER BY | 排序检索数据                 |
| SET      | 添加或更新标签               |

### 常用命令

#### CREATE创建

create语句是创建模型语句用来创建数据模型

##### 创建节点

```
# 创建简单节点
create (n)

# 创建多个节点
create (n),(m)

# 创建带标签和属性的节点并返回节点
# 这里n可以理解为person节点的别名
create (n:person {name:'如来'}) return n
```

##### 创建关系

Neo4j图数据库遵循属性图模型来存储和管理其数据。

根据属性图模型，关系应该是定向的。否则，Neo4j将抛出一个错误的消息。

基于方向性，Neo4j关系被分为两种主要类型

- 单向关系
- 双向关系

```
# 使用新节点创建关系
create (n:person {name:'杨戬'})-[r:师傅]->(m:person {name:'玉鼎真人'}) return type(r)

# 使用已知节点创建带属性的关系
match (n:person {name:'沙僧'}),(m:person {name:'唐僧'})
create (n)-[r:`师傅`{relation:'师傅'}]->(m) return r

# 检索关系节点的详细信息
match (n:person)-[r]-(m:person) return n,m
```

##### 创建全路径

```
create p=(:person {name:'蛟魔王'})-[:义兄]->(:person {name:'牛魔王'})<-[:义兄]-(:person {name:'鹏魔王'}) return p
```

#### RETURN返回

- 检索节点的某些属性
- 检索节点的所有属性
- 检索节点和关联关系的某些属性
- 检索节点和关联关系的所有属性

```
MATCH (n:`西游`) return id(n),n.name,n.tail,n.ralation
```

#### WHERE子句

像SQL一样，Neo4j CQL在CQL MATCH命令中提供了WHERE子句来过滤MATCH查询的结果

```
MATCH (n:person) where n.name ='孙悟空' RETURN n

match (n:person),(m:person) where n.name='孙悟空' and m.name='猪八戒'
create (n)-[r:师弟]->(m) return n.name,type(r),m.name
```

#### DELETE删除

Neo4j使用CQL DELETE子句

- 删除节点
- 删除节点及相关节点和关系

```
# 删除节点（前提：节点不存在关系）
MATCH (n:person {name:'白龙马'}) delete n

# 删除关系
MATCH (n:person {name:'沙僧'})<-[r]-(m) delete r return type(r)
```

#### REMOVE删除

- 删除节点或关系的标签
- 删除节点或关系的属性

```
# 删除属性
MATCH (n:role {name:'fox'}) remove n.age return n

# 创建节点
CREATE (m:role:person {name:'fox666'})

# 删除标签
match (m:role:person {name:'fox666'}) remove m:person return m
```

#### SET子句

- 向现有节点或关系添加属性
- 添加或更新属性值

```
MATCH (n:role {name:'fox'}) set n.age=32 return n
```

#### ORDER BY排序

对MATCH查询返回的结果进行排序

```
MATCH (n:`西游`) RETURN id(n),n.name order by id(n) desc
```

#### UNION子句

- union 它将两组结果中的公共行组合并返回到一组结果中。它不从两个节点返回重复的行。限制：结果列类型和来自两组结果的名称必须匹配，这意味着列名称应该相同，列的数据类型应该相同。
- union all 它结合并返回两个结果集的所有行成一个单一的结果集。它还返回由两个节点重复行。限制：结果列类型，并从两个结果集的名字必须匹配，这意味着列名称应该相同，列的数据类型应该相同。

```
MATCH (n:role) RETURN n.name as name
UNION
MATCH (m:person) RETURN m.name as name
```

#### LIMIT和SKIP子句

Neo4j CQL已提供LIMIT子句和SKIP来过滤或限制查询返回的行数。

LIMIT返回前几行，SKIP忽略前几行。

```
# 前两行	
MATCH (n:`西游`) RETURN n LIMIT 2
# 忽略前两行
MATCH (n:`西游`) RETURN n SKIP 2
```

#### NULL值

当我们创建一个具有现有节点标签名称但未指定其属性值的节点时，它将创建一个具有NULL属性值的新节点。

```
match (n:`西游`) where a.label is null return id(n),n.name,n.tail,n.label
```

#### IN操作符

```
match (n:`西游`) where n.name in ['孙悟空','唐僧'] return id(n),n.name,n.tail,n.label
```

#### INDEX索引

- create index 创建索引
- drop index 丢弃索引

```
# 创建索引
create index on :`西游`(name)

# 删除索引
drop index on :`西游`(name)	
```

#### UNIQUE约束

- 避免重复记录
- 强制执行数据完整性规则

```
# 创建唯一约束
create constraint on (n:xiyou) assert n.name is unique
# 删除唯一约束
drop constraint on (n:xiyou) assert n.name is unique
```

#### DISTINCT

```
match (n:`西游`) return distinct (n.name)
```

## Neo4j备份和恢复

```
cd %NEO4J_HOME%/bin

neo4j stop
# 数据备份
neo4j-admin dump --database=graph.db --to=/neo4j/backup/graph_backup.dump


# 数据恢复
neo4j-admin load --from=/neo4j/backup/graph_backup.dump --database=graph.db --force

neo4j start

```

