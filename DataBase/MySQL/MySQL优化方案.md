# MySQL优化方案

## 如何定位有问题的SQL语句

开启mysql慢查询,从慢查询记录语句分析

```
show variables like "%query%";
#long_query_time 是查询超过该值就会记录
# show_query_log off关闭 on打开
# show_quert_log 慢查询语句的文件位置
#开启慢查询 命令修改重启会失效,再my.ini配置文件修改重启
set global show_query_log=on
#查询慢查询sql的条数
show status like "%slow_queries%";
```

使用`show processlist`

**id**	sql语句的一个进程标识 可以用kill id

**user**	当前用户,root用户可以查看所有sql,其余用户只能看自已的sql

**host**	sql语句来自哪个ip的哪个端口,用于追踪出问题语句的用户

**db**	显示这个进程查询的数据库

**command**	当前连接的执行命令,一般是休眠(sleep) 查询(query) 连接(connct)

**time**	这个状态持续的时间,单位为秒

**state**	显示使用当前连接的sql语句的状态(对分析很重要)，state只是语句执行中的某一个状态。

```
Checking table
　正在检查数据表（这是自动的）。
Closing tables
　正在将表中修改的数据刷新到磁盘中，
　同时正在关闭已经用完的表。这是一个很快的操作，
　如果不是这样的话，
　就应该确认磁盘空间是否已经满了或者磁盘是否正处于重负中。
Connect Out
　复制从服务器正在连接主服务器。
Copying to tmp table on disk
　由于临时结果集大于tmp_table_size，
　正在将临时表从内存存储转为磁盘存储以此节省内存。
Creating tmp table
　正在创建临时表以存放部分查询结果。
deleting from main table
　服务器正在执行多表删除中的第一部分，刚删除第一个表。
deleting from reference tables
　服务器正在执行多表删除中的第二部分，正在删除其他表的记录。
Flushing tables
　正在执行FLUSH TABLES，等待其他线程关闭数据表。
Killed
　发送了一个kill请求给某线程，那么这个线程将会检查kill标志位，
　同时会放弃下一个kill请求。MySQL会在每次的主循环中检查kill标志位，
　不过有些情况下该线程可能会过一小段才能死掉。如果该线程程被其他线程锁住了，那么kill请求会在锁释放时马上生效。
Locked
　被其他查询锁住了。
Sending data
　正在处理SELECT查询的记录，同时正在把结果发送给客户端。
Sorting for group
　正在为GROUP BY做排序。
　Sorting for order
　正在为ORDER BY做排序。
Opening tables
　这个过程应该会很快，除非受到其他因素的干扰。
　例如，在执ALTER TABLE或LOCK TABLE语句行完以前，
　数据表无法被其他线程打开。正尝试打开一个表。
Removing duplicates
　正在执行一个SELECT DISTINCT方式的查询，
　但是MySQL无法在前一个阶段优化掉那些重复的记录。
　因此，MySQL需要再次去掉重复的记录，然后再把结果发送给客户端。
Reopen table
　获得了对一个表的锁，但是必须在表结构修改之后才能获得这个锁。
　已经释放锁，关闭数据表，正尝试重新打开数据表。
Repair by sorting
　修复指令正在排序以创建索引。
Repair with keycache
　修复指令正在利用索引缓存一个一个地创建新索引。
　它会比Repair by sorting慢些。
Searching rows for update
　正在讲符合条件的记录找出来以备更新。
　它必须在UPDATE要修改相关的记录之前就完成了。
Sleeping
　正在等待客户端发送新请求.
System lock
　正在等待取得一个外部的系统锁。
　如果当前没有运行多个mysqld服务器同时请求同一个表，
　那么可以通过增加--skip-external-locking参数来禁止外部系统锁。
Upgrading lock
　INSERT DELAYED正在尝试取得一个锁表以插入新记录。
Updating
　正在搜索匹配的记录，并且修改它们。
User Lock
　正在等待GET_LOCK()。
Waiting for tables
　该线程得到通知，数据表结构已经被修改了，
　需要重新打开数据表以取得新的结构。
　然后，为了能的重新打开数据表，
　必须等到所有其他线程关闭这个表。
　以下几种情况下会产生这个通知：
　FLUSH TABLES tbl_name, ALTER TABLE, RENAME TABLE,
　REPAIR TABLE, ANALYZE TABLE,或OPTIMIZE TABLE。
waiting for handler insert
　INSERT DELAYED已经处理完了所有待处理的插入操作，正在等待新的请求。
```

## 优化SQL

### 索引设置

1.业务场景如果查询条件涉及到多个字段,建议用组合索引,而且组合索引遵循最左原则如index_a_b_c(a,b,c) a,ab,abc均可以使用,为了提高索引的命中率,把数据权重最大的字段放前面

2.多个单列索引只会生效一个

3.索引会占用开销,根据实际情况调整,如果线上环境正在跑,调优的时候记得选择空闲时间

### 小表驱动大表

减少表连接创建的次数,加快查询速度,A表:20W B表500W ,A letf join B 只需查询20w 如果是B left join A 需要500w

```
left join 左表驱动右表
right join 右表驱动左表
inner join 数据量小的作为驱动表
```

### 不使用子查询

```
SELECT FROM t1 WHERE id (SELECT id FROM t2 WHERE name='hechunyang');
```

子查询在`MySQL5.5`版本里，内部执行计划器是这样执行的：先查外表再匹配内表，而不是先查内表 t2，当外表的数据很大时，查询速度会非常慢。

在`MariaDB10/MySQL5.6`版本里，采用`join`关联方式对其进行了优化，这条SQL会自动转换为 `SELECT t1. FROM t1 JOIN t2 ON t1.id = t2.id;`

但请注意的是：优化只针对`SELECT`有效，对 `UPDATE/DELETE`子查询无效，生产环境尽量应避免使用子查询。

### 避免函数索引
```
SELECT FROM t WHERE YEAR(d) >= 2016;
```

由于`MySQL`不像`Oracle`那样⽀持函数索引，即使 d 字段有索引，也会直接全表扫描。

应改为
```
SELECT FROM t WHERE d >= '2016-01-01';
```

### 用 IN 来替换 OR 低效查询

慢
```
SELECT FROM t WHERE LOC_ID = 10 OR LOC_ID = 20 OR LOC_ID = 30;
```

高效查询
```
SELECT FROM t WHERE LOC_IN IN (10,20,30);
```

### LIKE 双百分号无法使用到索引
```
SELECT FROM t WHERE name LIKE '%de%';
```

使用
```
SELECT FROM t WHERE name LIKE 'de%';
```

### 分组统计可以禁止排序
```
SELECT goods_id,count() FROM t GROUP BY goods_id;
```
默认情况下，`MySQL`对所有`GROUP BY col1，col2… `的字段进⾏排序。如果查询包括`GROUP BY`，想要避免排序结果的消耗，则可以指定`ORDER BY NULL`禁止排序。

使用
```
SELECT goods_id,count () FROM t GROUP BY goods_id ORDER BY NULL;
```

### 禁止不必要的 ORDER BY 排序
```
SELECT count(1) FROM user u LEFT JOIN user_info i ON u.id = i.user_id WHERE 1 = 1 ORDER BY u.create_time DESC;
```

使用
```
SELECT count (1) FROM user u LEFT JOIN user_info i ON u.id = i.user_id;
```
