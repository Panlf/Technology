# SpringBoot+Nacos+Kafka 实现微服务流编排

## 准备工作

### Nacos 安装及使用入门

```
# 拉取镜像
docker pull nacos/nacos-server 

# 创建服务
docker run --env MODE=standalone --name nacos -d -p 8848:8848 nacos/nacos-server 

# ip:8848/nacos，账号 nacos；密码 nacos
```

## 代码案例

```
<parent>  
   <groupId>org.springframework.boot</groupId>  
   <artifactId>spring-boot-starter-parent</artifactId>  
   <version>2.1.0.RELEASE</version>  
</parent>  
  
<dependency>  
   <groupId>org.springframework.kafka</groupId>  
   <artifactId>spring-kafka</artifactId>  
</dependency>  
  
<dependency>  
   <groupId>com.alibaba.boot</groupId>  
   <artifactId>nacos-config-spring-boot-starter</artifactId>  
   <version>0.2.1</version>  
</dependency>  
```

配置文件
```
spring:  
  kafka:  
    bootstrap-servers: kafka-server:9092  
    producer:  
      acks: all  
    consumer:  
      group-id: node1-group #三个服务分别为node1 node2 node3  
      enable-auto-commit: false  
# 部署的nacos服务  
nacos:  
  config:  
    server-addr: nacos-server:8848  
```

读取配置
```
@Configuration  
@NacosPropertySource(dataId = "input", groupId = "node1-server", autoRefreshed = true)  
// autoRefreshed=true指的是nacos中配置发生改变后会刷新，false代表只会使用服务启动时候读取到的值  
@NacosPropertySource(dataId = "sink", groupId = "node1-server", autoRefreshed = true)  
public class NacosConfig {  
  
    @NacosValue(value = "${input:}", autoRefreshed = true)  
    private String input;  
  
    @NacosValue(value = "${sink:}", autoRefreshed = true)  
    private String sink;  
  
    public String getInput() {  
        return input;  
    }  
  
    public String getSink() {  
        return sink;  
    }  
} 
```

监听配置改变

服务的输入需要在服务启动时候创建消费者，在 topic 发生改变时候重新创建消费者，移除旧 topic 的消费者，输出是业务驱动的，无需监听改变，在每次发送时候读取到的都是最新配置的 topic。

因为在上面的配置类中 autoRefreshed = true，这个只会刷新 nacosConfig 中的配置值，服务需要知道配置改变去驱动消费的创建业务，需要创建 nacos 配置监听。
```
/**  
 * 监听Nacos配置改变，创建消费者，更新消费  
 */  
@Component  
public class ConsumerManager {  
  
    @Value("${spring.kafka.bootstrap-servers}")  
    private String servers;  
  
    @Value("${spring.kafka.consumer.enable-auto-commit}")  
    private boolean enableAutoCommit;  
  
    @Value("${spring.kafka.consumer.group-id}")  
    private boolean groupId;  
  
    @Autowired  
    private NacosConfig nacosConfig;  
  
    @Autowired  
    private KafkaTemplate kafkaTemplate;  
  
    // 用于存放当前消费者使用的topic  
    private String topic;  
  
    // 用于执行消费者线程  
    private ExecutorService executorService;  
  
    /**  
     * 监听input  
     */  
    @NacosConfigListener(dataId = "node1-server", groupId = "input")  
    public void inputListener(String input) {  
        // 这个监听触发的时候 实际NacosConfig中input的值已经是最新的值了 我们只是需要这个监听触发我们更新消费者的业务  
        String inputTopic = nacosConfig.getInput();  
        // 我使用nacosConfig中读取的原因是因为监听到内容是input=xxxx而不是xxxx,如果使用需要自己截取一下,nacosConfig中的内容框架会处理好，大家看一下第一张图的配置内容就明白了  
        // 先检查当前局部变量topic是否有值,有值代表是更新消费者，没有值只需要创建即可  
        if(topic != null) {  
            // 停止旧的消费者线程  
            executorService.shutdownNow();  
            executorService == null;  
        }  
        // 根据为新的topic创建消费者  
        topic = inputTopic;  
        ThreadFactory threadFactory = new ThreadFactoryBuilder().setNameFormat(topic + "-pool-%d").build();  
        executorService = new ThreadPoolExecutor(1, 1, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(2), threadFactory);  
        // 执行消费业务  
        executorService.execute(() -> consumer(topic));  
    }  
  
    /**  
     * 创建消费者  
     */  
    public void consumer(String topic) {  
        Properties properties = new Properties();  
        properties.put("bootstrap.servers", servers);  
        properties.put("enable.auto.commit", enableAutoCommit);  
        properties.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");  
        properties.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");  
        properties.put("group.id", groupId);  
        KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);  
        consumer.subscribe(Arrays.asList(topic));  
        try {  
            while (!Thread.currentThread().isInterrupted()) {  
                Duration duration = Duration.ofSeconds(1L);  
                ConsumerRecords<String, String> records = consumer.poll(duration);  
                for (ConsumerRecord<String, String> record : records) {  
                    String message = record.value();  
                    // 执行数据处理业务 省略业务实现  
                    String handleMessage =  handle(message);  
                    // 处理完成后发送到下一个节点  
                    kafkaTemplate.send(nacosConfig.getSink(), handleMessage);  
                }  
            }  
            consumer.commitAsync();  
        }  
        } catch (Exception e) {  
            LOGGER.error(e.getMessage(), e);  
        } finally {  
            try {  
                consumer.commitSync();  
            } finally {  
                consumer.close();  
            }  
        }  
    }  
}  

```