# Lock4j分布式锁

Lock4j是一个分布式锁组件，它提供了多种不同的支持以满足不同性能和环境的需求，基于Spring AOP的声明式和编程式分布式锁，支持RedisTemplate、Redisson、Zookeeper。

## 引入依赖
```
<!-- Lock4j -->
<!-- 若使用redisTemplate作为分布式锁底层，则需要引入 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>lock4j-redis-template-spring-boot-starter</artifactId>
    <version>2.2.4</version>
</dependency>
<!-- 若使用redisson作为分布式锁底层，则需要引入 -->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>lock4j-redisson-spring-boot-starter</artifactId>
    <version>2.2.4</version>
</dependency>
```

##  添加redis配置
```
spring:
  redis:
    database: 0
    # Redis服务器地址 写你的ip
    host: 127.0.0.1
    # Redis服务器连接端口
    port: 6379
    # Redis服务器连接密码（默认为空）
    password:
    # 连接池最大连接数（使用负值表示没有限制  类似于mysql的连接池
    jedis:
      pool:
        max-active: 200
        # 连接池最大阻塞等待时间（使用负值表示没有限制） 表示连接池的链接拿完了 现在去申请需要等待的时间
        max-wait: -1
        # 连接池中的最大空闲连接
        max-idle: 10
        # 连接池中的最小空闲连接
        min-idle: 0
    # 连接超时时间（毫秒） 去链接redis服务端
    timeout: 6000
```

## 注解说明
|@Lock4j注解属性|说明|
|--|--|
|name|需要锁住的key名称|
|executor|可以通过该参数设置自定义特定的执行器|
|keys|需要锁住的keys名称，可以是多个|
|expire|锁过期时间，主要用来防止死锁|
|acquireTimeout|可以理解为排队等待时长，超过这个时长就退出排队，并排除获取锁超时异常|
|autoRelease|是否自动释放锁，默认是true|

## 简单使用
```
@RestController
@RequestMapping("/mock")
public class MockController {

    @GetMapping("/lockMethod")
    @Lock4j(keys = {"#key"}, acquireTimeout = 1000, expire = 10000)
    public Result lockMethod(@RequestParam String key) {
        ThreadUtil.sleep(5000);
        return Result.OK(key);
   
```

## 高级使用
### 自定义执行器Exector
```
@Component
public class CustomRedissonLockExecutor extends AbstractLockExecutor {
    
    @Override
    public Object acquire(String lockKey, String lockValue, long expire, long acquireTimeout) {
        return null;
    }

    @Override
    public boolean releaseLock(String key, String value, Object lockInstance) {
        return false;
    }
}
```
在注解上直接指定特定的执行器：@Lock4j(executor = CustomRedissonLockExecutor.class)。

### 自定义分布式锁key生成器
```
@Component
public class CustomKeyBuilder extends DefaultLockKeyBuilder {

    public CustomKeyBuilder(BeanFactory beanFactory) {
        super(beanFactory);
    }
}
```

### 自定义抢占锁失败执行策略
```
@Component
public class GrabLockFailureStrategy implements LockFailureStrategy {

    @Override
    public void onLockFailure(String key, Method method, Object[] arguments) {

    }
}
```
默认的锁获取失败策略为 com.baomidou.lock.DefaultLockFailureStrategy.

### 手动加锁释放锁
```
@Service
public class LockServiceImpl implements LockService {

    @Autowired
    private LockTemplate lockTemplate;

    @Override
    public void lock(String resourceKey) {

        LockInfo lock = lockTemplate.lock(resourceKey, 10000L, 2000L, CustomRedissonLockExecutor.class);
        if (lock == null) {
            // 获取不到锁
            throw new FrameworkException("业务处理中，请稍后再试...");
        }
        // 获取锁成功，处理业务
        try {
            doBusiness();
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            lockTemplate.releaseLock(lock);
        }
    }

    private void doBusiness() {
        // TODO 业务执行逻辑
    }
}
```