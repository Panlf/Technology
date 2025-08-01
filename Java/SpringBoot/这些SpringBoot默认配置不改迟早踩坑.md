# 这些 SpringBoot 默认配置不改，迟早踩坑!

## 引言
彼时 SpringBoot 初兴，万象更新，号称“开箱即用”“约定优于配置”，一时间风靡四方。
开发者趋之若鹜，纷纷称快，仿佛自此架构之重可卸、配置之繁可省，一行`main()` 即可气定神闲、纵横沙场。
然则时光久远，方知此言非虚，却也未尽其真。所谓默认，不过是你未曾开口，框架自作主张。表面无碍，实则步步杀机，线上事故十有八九，皆因“未曾配置”的“默认”。
回首往昔，实堪自嘲。曾自诩熟稔底层、精通原理，然于这些藏于阴影处的默认设定，竟茫然不觉。故障一起，仓皇失措，耗时良久，方才发现，不过是框架做了一个并不适合的决定。
是以今日提笔，将过往种种记录于此，只盼后来者少走弯路。

## 正文

### Tomcat连接池
SpringBoot默认使用Tomcat作为Web容器，但默认的连接池配置在高并发场景下会成为瓶颈。
默认配置下，Tomcat的最大连接数只有200，最大线程数也只有200。这意味着当并发请求超过200时，后续请求就会排队等待。在生产环境中，这个配置明显不够用。

```yaml
server:
  tomcat:
    max-connections: 10000  # 最大连接数
    threads:
      max: 800              # 最大工作线程数
      min-spare: 100        # 最小空闲线程数
    accept-count: 100       # 等待队列长度
    connection-timeout: 20000
```

更坑的是，SpringBoot的默认超时时间是无限长。这会导致连接一直占用，直到客户端主动断开。

在网络不稳定的环境下，大量连接会一直挂着不释放，最终耗尽服务器资源。

### 数据库连接池
SpringBoot默认使用HikariCP作为数据库连接池，但默认的连接池配置在生产环境下会成为瓶颈。默认最大连接数只有10个，对于稍微复杂一点的应用来说根本不够用。

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 50
      minimum-idle: 10
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000
      leak-detection-threshold: 60000
```

特别要注意`leak-detection-threshold`这个配置。默认情况下这个检测是关闭的，如果代码中存在连接泄漏问题，根本发现不了。
开启后，HikariCP会监控连接的使用时间，超过阈值就会打印警告日志。

### JPA懒加载
SpringBoot集成JPA时，默认开启了懒加载。这个设计初衷是好的，但在实际使用中经常会导致N+1查询问题。

```java
@Entity
public class User {
    @Id
    private Long id;
    
    @OneToMany(fetch = FetchType.LAZY)  // 默认就是LAZY
    private List<Order> orders;
}
```

当查询用户列表时，每访问一次orders属性，就会触发一次数据库查询。
如果有100个用户，就会执行101次SQL。
这种情况下，要么使用`@EntityGraph`指定加载策略，要么在`Repository` 中使用 `JOIN FETCH`。

```java
@Query("SELECT u FROM User u LEFT JOIN FETCH u.orders")
List<User> findAllWithOrders();
```

### Jackson时区序列化
SpringBoot默认使用Jackson处理JSON序列化，但时区处理经常出问题。
默认情况下，Jackson会使用系统时区，这在分布式部署时会导致不一致的问题。

```yaml
spring:
  jackson:
    time-zone: GMT+8
    date-format: yyyy-MM-dd HH:mm:ss
    serialization:
      write-dates-as-timestamps: false
```

更要命的是，如果你的应用部署在不同时区的服务器上，同样的时间可能会被序列化成不同的值。
这个问题在国际化应用中特别突出。

### 日志配置
SpringBoot默认使用Logback，但默认配置下没有对日志文件进行滚动和清理。
长时间运行的应用会产生巨大的日志文件，最终占满磁盘空间。

```yaml
logging:
  file:
    name: app.log
  logback:
    rollingpolicy:
      max-file-size: 100MB
      max-history: 30
      total-size-cap: 3GB
```

另外，默认的日志级别是INFO，在生产环境中会产生大量不必要的日志。
合理设置日志级别可以显著提升性能。

### 缓存配置
SpringBoot的`@Cacheable`注解默认使用`ConcurrentHashMap`作为缓存实现，但这个实现没有过期机制，也没有大小限制。在高并发场景下，缓存会无限增长，最终导致内存溢出。

```yaml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=10000,expireAfterWrite=600s
```

可以考虑使用Caffeine替代默认实现，可以提供更好的性能和内存管理能力。

### 监控端点
SpringBoot Actuator默认暴露了很多监控端点，包括健康检查、配置信息、环境变量等。

这些信息在开发环境中很有用，但在生产环境中可能泄漏敏感信息。

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: when-authorized
```

只暴露必要的端点，并且配置适当的安全策略，避免信息泄漏。

### 文件上传大小限制
SpringBoot默认的文件上传限制非常小，单个文件只能上传1MB，整个请求大小限制10MB。

在实际业务中，这个限制经常不够用，用户上传稍大一点的文件就会报错。

这个属于是比较常见的问题，因为开发环境测试时通常用小文件，一切正常。等到用户上传几MB的PDF文档或者高清图片时，系统就开始报 `MaxUploadSizeExceededException` 异常。

这个异常往往还发生在文件传输完成之后，用户等了半天上传完毕，结果被告知文件过大，体验极差。

```yaml
spring:
  servlet:
    multipart:
      max-file-size: 100MB
      max-request-size: 100MB
      file-size-threshold: 2KB
      location: /tmp
      resolve-lazily: false
```

`file-size-threshold` 这个参数也很重要，它决定了多大的文件会直接写入内存。如果设置过大，大量并发上传会占用过多内存；设置过小，小文件也要写磁盘，影响性能。一般设置为几KB比较合适。

### 异步线程池配置
使用`@Async`注解时，SpringBoot默认使用`SimpleAsyncTaskExecutor`，这个执行器每次都会创建新线程，没有线程池复用机制。高并发情况下会创建大量线程，最终导致系统资源耗尽。

这个问题在开发阶段很难发现，因为异步任务通常不多。但在生产环境，如果有大量异步任务执行，比如发送短信、推送、记录日志等，系统会不断创建新线程。每个线程默认占用1MB的栈空间，创建几千个线程就是几GB内存。

更严重的是线程切换的开销，CPU大部分时间都在做上下文切换，真正的业务逻辑反而执行很慢。

```yaml
spring:
  task:
    execution:
      pool:
        core-size:8
        max-size:16
        queue-capacity:100
        keep-alive:60s
      thread-name-prefix:async-task-
    scheduling:
      pool:
        size:4
      thread-name-prefix: scheduling-
```

线程池大小的设置也有讲究。
如果是CPU密集型任务，线程数设置为CPU核心数就够了；如果是IO密集型任务，可以设置为CPU核心数的2-3倍。
`queue-capacity`设置了任务队列长度，当线程池满了之后，新任务会放到队列里等待执行。

### 静态资源缓存策略
SpringBoot默认不为静态资源设置HTTP缓存头，这意味着浏览器每次都会重新请求CSS、JS、图片等静态文件，严重影响页面加载性能。
用户每次访问页面，浏览器都要重新下载所有静态资源，即使这些文件根本没有变化。对于资源较多的单页应用来说，这个问题特别明显。用户看到的就是页面加载慢，特别是网络条件不好的时候，体验很差。

```yaml
spring:
  web:
    resources:
      cache:
        cachecontrol:
          max-age:365d
          cache-public:true
      chain:
        strategy:
          content:
            enabled:true
            paths:/**
        cache:true
      static-locations: classpath:/static/
```

开启内容版本化策略后，SpringBoot会根据文件内容生成MD5哈希值作为版本号，文件名变成`style-abc123.css`这样的格式。当文件内容发生变化时，哈希值也会变化，浏览器会认为这是新文件重新下载；如果文件没变化，浏览器就直接使用缓存，有效提升页面加载速度。

### 数据库事务超时
`@Transactional`注解默认没有设置超时时间，长时间运行的事务会一直持有数据库锁，影响其他操作的执行。特别是在批量数据处理时，很容易出现锁表问题。
这个问题在并发量不高的时候不明显，但随着业务增长，长事务的危害就暴露出来了。
比如一个数据导入任务需要处理几万条记录，如果放在一个事务里，可能要运行几分钟甚至更长时间。在这期间，相关的表都被锁住，其他用户的操作只能等待，系统响应变得很慢。

```java
@Transactional(timeout = 30, rollbackFor = Exception.class)
publicvoidbatchProcess(List<Data> dataList) {
    // 分批处理，避免长事务
    intbatchSize=100;
    for (inti=0; i < dataList.size(); i += batchSize) {
        List<Data> batch = dataList.subList(i, 
            Math.min(i + batchSize, dataList.size()));
        processBatch(batch);
    }
}
```

对于大批量数据处理，建议分成多个小事务，每个事务处理少量数据。这样即使某个小事务失败，也不会影响整体进度，而且可以及时释放数据库锁，提高系统并发性能。
同时再加上`rollbackFor = Exception.class`确保所有异常都会触发回滚，避免数据不一致。

## 写在最后
Spring Boot 的“约定优于配置”确实省心，但省的是开发者的心，不是系统的责任。每一项默认配置背后，其实都藏着设计者的假设和权衡，而这些假设，在我们的业务场景中也许未必成立。
这些坑我几乎都踩过，有些甚至反复踩了好几次。
愿你读到这里，能少走几步弯路，可不能拿生产事故去交学费。
提前优化配置，是对系统负责，也是对自己负责。