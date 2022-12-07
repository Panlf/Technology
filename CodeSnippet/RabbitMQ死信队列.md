# SpringBoot+RabbitMQ死信队列

死信：无法被消费的消息，称为死信。

## 死信来源
- 消息 TTL 过期（time to live，存活时间，可以用在限时支付消息）
- 队列达到最大长度（队列满了，无法路由到该队列）
- 消息被拒绝（ basic.reject / basic.nack ），并且 requeue = false

## 代码案例

### 主要配置
```

@Configuration
public class DeadConfig {

    /* 正常配置 **********************************************************************************************************/

    /**
     * 正常交换机，开启持久化
     */
    @Bean
    DirectExchange normalExchange() {
        return new DirectExchange("normalExchange", true, false);
    }

    @Bean
    public Queue normalQueue() {
        // durable: 是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
        // exclusive: 默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
        // autoDelete: 是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
        Map<String, Object> args = deadQueueArgs();
        // 队列设置最大长度
        args.put("x-max-length", 5);
        return new Queue("normalQueue", true, false, false, args);
    }

    @Bean
    public Queue ttlQueue() {
        Map<String, Object> args = deadQueueArgs();
        // 队列设置消息过期时间 60 秒
        args.put("x-message-ttl", 60 * 1000);
        return new Queue("ttlQueue", true, false, false, args);
    }

    @Bean
    Binding normalRouteBinding() {
        return BindingBuilder.bind(normalQueue()).to(normalExchange()).with("normalRouting");
    }

    @Bean
    Binding ttlRouteBinding() {
        return BindingBuilder.bind(ttlQueue()).to(normalExchange()).with("ttlRouting");
    }

    /* 死信配置 **********************************************************************************************************/

    /**
     * 死信交换机
     */
    @Bean
    DirectExchange deadExchange() {
        return new DirectExchange("deadExchange", true, false);
    }

    /**
     * 死信队列
     */
    @Bean
    public Queue deadQueue() {
        return new Queue("deadQueue", true, false, false);
    }

    @Bean
    Binding deadRouteBinding() {
        return BindingBuilder.bind(deadQueue()).to(deadExchange()).with("deadRouting");
    }

    /**
     * 转发到 死信队列，配置参数
     */
    private Map<String, Object> deadQueueArgs() {
        Map<String, Object> map = new HashMap<>();
        // 绑定该队列到私信交换机
        map.put("x-dead-letter-exchange", "deadExchange");
        map.put("x-dead-letter-routing-key", "deadRouting");
        return map;
    }

}
```

arguments 具体参数

|参数名|作用|
|--|--|
|x-message-ttl| 发送到队列的消息在丢弃之前可以存活多长时间（毫秒）|
|x-max-length|队列最长长度|
|x-expires|队列被自动删除（毫秒）之前可以使用多长时间|
|x-max-length|队列在开始从头部删除之前可以包含就绪消息的总体大小|
|x-max-length-bytes|队列在开始从头部删除之前可以包含的就绪消息的总体大小|
|x-dead-letter-exchange|设置队列溢出行为。这决定了在达到队列的最长长度时消息会发生什么。有效值为drop-head或reject-publish。交换的可选名称，如果消息被拒绝或过期，将重新发布这些名称。|
|x-dead-letter-routing-key|可选的替换路由密钥，用于在消息以字母为单位时使用。如果未设置，将使用消息的原始路由密钥|
|x-max-priority|队列支持的最大优先级数，如果未设置，队列将不支持消息优先级|
|x-queue-mode|将队列设置为延迟模式，在磁盘上保留尽可能多的消息以减少内存使用；如果未设置，队列将保留内存缓存以尽快传递消息|
|x-queue-master-locator|将队列设置为主位置，确定在节点集群上声明时队列主机所在的规则|
|x-overflow|队列达到最大长度时，可选模式包括：drop-head、reject-publish、reject-publish-dlx|

### 调用配置

调用6次，队列为5，多的那次进入死信队列
```
 /**
  * 正常消息队列，队列最大长度5
  */
 @GetMapping("/normalQueue")
 public String normalQueue() {

     Map<String, Object> map = new HashMap<>();
     map.put("messageId", String.valueOf(UUID.randomUUID()));
     map.put("data", System.currentTimeMillis() + ", 正常队列消息，最大长度 5");

     rabbitTemplate.convertAndSend("normalExchange", "normalRouting", map, new CorrelationData());
     return JSONObject.toJSONString(map);
 }
```

消息 TTL 过期

消息的TTL 指的是消息的存活时间，我们可以通过设置消息的TTL或者队列的TTL来实现。

- 消息的TTL ：对于设置了过期时间属性(expiration)的消息，消息如果在过期时间内没被消费，会过期
- 队列的TTL ：对于设置了过期时间属性(x-message-ttl)的队列，所有路由到这个队列的消息，都会设置上这个过期时间
  
两种配置都行，一般都用在定时任务，限时支付这种地方。
```
 /**
  * 消息 TTL, time to live
  */
 @GetMapping("/ttlToDead")
 public String ttlToDead() {

     Map<String, Object> map = new HashMap<>();
     map.put("messageId", String.valueOf(UUID.randomUUID()));
     map.put("data", System.currentTimeMillis() + ", ttl队列消息");

     rabbitTemplate.convertAndSend("normalExchange", "ttlRouting", map, new CorrelationData());
     return JSONObject.toJSONString(map);
 }
```

代码中尽量使用 消息TTL，不要用 队列TTL

拒绝消息

正常队列消费后拒绝消息，并且不进行重新入队
```
@Component
@RabbitListener(queues = "normalQueue")
public class NormalConsumer {
    @RabbitHandler
    public void process(Map<String, Object> message, Channel channel, Message mqMsg) throws IOException {
        System.out.println("收到消息，并拒绝重新入队 : " + message.toString());
        channel.basicReject(mqMsg.getMessageProperties().getDeliveryTag(), false);
    }
}
```

### 死信队列消费
```
@Component
@RabbitListener(queues = "deadQueue")
public class DeadConsumer {
    @RabbitHandler
    public void process(Map<String, Object> message, Channel channel, Message mqMsg) throws IOException {
        System.out.println("死信队列收到消息 : " + message.toString());
        channel.basicAck(mqMsg.getMessageProperties().getDeliveryTag(), false);
    }
}
```