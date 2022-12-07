# 监听Binlog

1、业务场景

使用canal或者mysql-binlog-driver监听MySQL的日志，将SQL语句发送到Kafka，然后数据消费到ElasticSearch中做查询。

主要是数据量大，然后业务比较复杂，所以通过监听特定数据表的日志，然后进行关联查询数据，然后新增修改到ElasticSearch中，再进行查询。因为怕MySQL的bin-log增长太快，所以使用消息中间件进行削峰。

这里的消息中间件采用两种分支分别使用ActiveMQ和Kafka的进行开发。

2、技术要求

