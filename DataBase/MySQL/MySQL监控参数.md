# MySQL监控参数

## 连接数（Connects）

- 最大使用连接数：`show status like 'Max_used_connections'`
- 当前打开的连接数：`show status like 'Threads_connected'`

## 缓存（bufferCache）

- 未从缓冲池读取的次数：`show status like 'Innodb_buffer_pool_reads'`
- 从缓冲池读取的次数：`show status like 'Innodb_buffer_pool_read_requests’`
- 缓冲池的总页数：`show status like 'Innodb_buffer_pool_pages_total’`
- 缓冲池空闲的页数：`show status like 'Innodb_buffer_pool_pages_free’`
- 缓存命中率计算：`（1-Innodb_buffer_pool_reads/Innodb_buffer_pool_read_requests）*100%`
- 缓存池使用率为：`((Innodb_buffer_pool_pages_total-Innodb_buffer_pool_pages_free）/Innodb_buffer_pool_pages_total）*100%`

## 锁（lock）

- 锁等待个数：`show status like 'Innodb_row_lock_waits'`
- 平均每次锁等待时间：`show status like 'Innodb_row_lock_time_avg'`
- 查看是否存在表锁：`show open TABLES where in_use>0;`有数据代表存在锁表，空为无表锁

## SQL

- 查看 mysql 开关是否打开：`show variables like 'slow_query_log'`，`ON` 为开启状态，如果为 `OFF`，`set global slow_query_log=1`进行开启
- 查看 mysql 阈值：`show variables like 'long_query_time’`，根据页面传递阈值参数，修改阈值 `set global long_query_time=0.1`
- 查看 mysql 慢 sql 目录：`show variables like 'slow_query_log_file'`
- 格式化慢 sql 日志：`mysqldumpslow -s at -t 10 /export/data/mysql/log/slow.log` 注：此语句通过 jdbc 执行不了，属于命令行执行。意思为：显示出耗时最长的 10 个 SQL 语句执行信息，10 可以修改为 TOP 个数。显示的信息为：执行次数、平均执行时间、SQL 语句

## statement

- insert 数量：`show status like 'Com_insert'`
- delete 数量：`show status like 'Com_delete'`
- update 数量：`show status like 'Com_update'`
- select 数量：`show status like 'Com_select'`

## 吞吐（Database throughputs）

- 发送吞吐量：`show status like 'Bytes_sent'`
- 接收吞吐量：`show status like 'Bytes_received'`
- 总吞吐量：`Bytes_sent+Bytes_received`

