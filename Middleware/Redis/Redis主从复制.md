# Redis主从复制

## Redis持久化

Redis是一个内存数据库，为了保证数据的持久性，它提供了两种持久化方案

### RDB方式（默认）

RDB方式是通过快照（ snapshotting ）完成的，当符合一定条件时Redis会自动将内存中的数据进行快照并持久化到硬盘。

**触发快照的时机**

1. 符合自定义配置的快照规则 redis.conf
2. 执行 save 或者 bgsave 命令
3. 执行 flushall 命令
4. 第一次执行主从复制操作

#### 设置快照保存规则

快照规则是配置在 redis.conf 文件中的

```
#
# Save the DB on disk:
# 
# 持久化操作设置，下面的配置分别表示：900秒内至少一个键被修改则进行快照，5分钟内至少10个键被修改则进行快照，1分钟内10000个键被更改则进行快照

save 900 1
save 300 10
save 60 10000
```

注意事项

1. Redis在进行快照过程中不会修改RDB文件，只有快照结束后才会将旧的快照文件替换为新的，也就是说任何时候RDB文件都是完成的，不存在中间状态，保证了数据的完整性。
2. 我们可以通过定时备份RDB文件来实现Redis数据库的备份，RDB文件是经过`压缩的二进制文件` ,占用空间会小于内存中的数据，更加利于传输。

#### RDB优缺点

**缺点**：使用RDB方式进行持久化，如果看明白了其备份原理图，则很容易看出**Redis如果异常宕机或者重启**，就会丢失最后一次快照之后的所有数据修改。这个时候我们就需要根据具体的应用场景，通过**组合设置自动快照条件**的方式来将可能发生的数据损失控制在能够接受范围。如果数据相对来说比较重要，希望将损失降到最小，则可以使用 AOF 方式进行持久化，下面我们会聊到这种方式。

**优点：** RDB最大化了Redis性能，父进程在保存快照生成RDB文件时唯一要做的就是fork出一个子进程，然后这个子进程就会处理接下来的所有文件保存工作，父进程无需执行任何磁盘 I/O 操作。同时这也是一个缺点，如果数据集比较大的时候，fork可能比较耗时，造成服务器在一段时间内会停止处理客户端请求。

### AOF方式
默认情况下 Redis 没有开启 AOF （ append only file ）方式的持久化。

开启 AOF 持久化后，每执行一条会更改 Redis 中的数据的命令， Redis 就会将该命令写入硬盘中的 AOF 文件，这一过程显然会降低 Redis 的性能，但大部分情况下这个影响是能够接受的，另外使用较快的硬盘可以提高 AOF 的性能。

#### 开启AOF持久化模式
还是一样的，我们只需要去修改Redis安装目录中的 redis.conf 文件中下面三个属性值即可。

```
appendonly yes // 开启AOF

# The name of the append only file (default: "appendonly.aof")

appendfilename "appendonly.aof" //持久化文件

# The working directory.
#
# The DB will be written inside this directory, with the filename specified
# above using the 'dbfilename' configuration directive.
#
# The Append Only File will also be created inside this directory.
#
# Note that you must specify a directory here, not a file name.
dir ./ // 文件所在目录
```

这三个参数指定了开启AOF持久化，以及持久化文件名和文件所在目录。

#### 原理

**AOF同步和RDB类似之处在于都是采用fork进程来处理**

#### AOF重写（优化AOF文件）

```
set s1 11
set s1 22
```

上面的操作，如果没有优化之前AOF文件是会将这两个命令按照RESP序列化后存储，如果优化后，则只存储后面一条命令即 set s1 22，同一个key的值被覆盖了，只存储最终结果。

**重写过程分析**

> Redis 其实是会定期新创建一个 AOF 文件，然后做 AOF 文件的重写优化，在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。而一旦新 AOF 文件创建完毕， Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。

**优化的触发条件**

配置文件 redis.conf 

```
# 表示当前aof文件大小超过上一次aof文件大小的百分之多少的时候会进行重写。如果之前没有重写过，以启动时aof文件大小为准
auto-aof-rewrite-percentage 100
# 限制允许重写最小aof文件大小，也就是文件大小小于64mb的时候，不需要进行优化
auto-aof-rewrite-min-size 64mb
```

###  如何选择RDB和AOF

- **内存数据库，数据不能丢**：rdb（redis database）+aof
- **缓存服务器**：rdb
- 不建议只使用 aof (性能差)
- 恢复时：有aof就先选择aof恢复，没有的话选择rdb文件恢复

## Redis主从复制

简言之就是：

- 主对外从对内，主可写从不可写
- 主挂了，从不可为主

对，你没看错，Redis主从复制没有动态选举Master节点的能力，主挂了服务就不可以写数据了。仅仅就是增强了应用读数据的并发量同时做数据备份。

一般生产环境会采用 哨兵 或者 Redis Cluster 这种具备Master自动选举的方案。

**Redis 的全量同步过程主要分三个阶段：**

- **同步快照阶段：** Master 创建并发送快照给 Slave ， Slave 载入并解析快照。Master 同时将此阶段所产生的新的写命令存储到缓冲区。
- **同步写缓冲阶段**：Master 向 Slave 同步存储在缓冲区的写操作命令。
- **同步增量阶段**：Master 向 Slave 同步写操作命令。

**增量同步**

- Redis 增量同步主要指 **Slave 完成初始化后开始正常工作时， Master 发生的写操作同步到 Slave的过程**。
- 通常情况下， Master 每执行一个写命令就会向 Slave 发送相同的写命令，然后 Slave 接收并执行。