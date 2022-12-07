# Disruptor 做内部高性能消息队列

## 案例

依赖
```
<dependency>
    <groupId>com.lmax</groupId>
    <artifactId>disruptor</artifactId>
    <version>3.3.4</version>
</dependency>
```

消息体
```
/**
 * 消息体
 */
@Data
public class MessageModel {
    private String message;
}
```

构造EventFactory
```
public class HelloEventFactory implements EventFactory<MessageModel> {
    @Override
    public MessageModel newInstance() {
        return new MessageModel();
    }
}
```

构造EventHandler-消费者
```
@Slf4j
public class HelloEventHandler implements EventHandler<MessageModel> {
    @Override
    public void onEvent(MessageModel event, long sequence, boolean endOfBatch) {
        try {
            //这里停止1000ms是为了确定消费消息是异步的
            Thread.sleep(1000);
            log.info("消费者处理消息开始");
            if (event != null) {
                log.info("消费者消费的信息是：{}",event);
            }
        } catch (Exception e) {
            log.info("消费者处理消息失败");
        }
        log.info("消费者处理消息结束");
    }
}
```

构造BeanManager
```
/**
 * 获取实例化对象
 */
@Component
public class BeanManager implements ApplicationContextAware {

    private static ApplicationContext applicationContext = null;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }

    public static ApplicationContext getApplicationContext() { return applicationContext; }

    public static Object getBean(String name) {
        return applicationContext.getBean(name);
    }

    public static <T> T getBean(Class<T> clazz) {
        return applicationContext.getBean(clazz);
    }
}
```

构造MQManager
```
@Configuration
public class MQManager {

    @Bean("messageModel")
    public RingBuffer<MessageModel> messageModelRingBuffer() {
        //定义用于事件处理的线程池， Disruptor通过java.util.concurrent.ExecutorSerivce提供的线程来触发consumer的事件处理
        ExecutorService executor = Executors.newFixedThreadPool(2);

        //指定事件工厂
        HelloEventFactory factory = new HelloEventFactory();

        //指定ringbuffer字节大小，必须为2的N次方（能将求模运算转为位运算提高效率），否则将影响效率
        int bufferSize = 1024 * 256;

        //单线程模式，获取额外的性能
        Disruptor<MessageModel> disruptor = new Disruptor<>(factory, bufferSize, executor,
                ProducerType.SINGLE, new BlockingWaitStrategy());

        //设置事件业务处理器---消费者
        disruptor.handleEventsWith(new HelloEventHandler());

        // 启动disruptor线程
        disruptor.start();

        //获取ringbuffer环，用于接取生产者生产的事件
        RingBuffer<MessageModel> ringBuffer = disruptor.getRingBuffer();

        return ringBuffer;
    }
}
```

构造Mqservice和实现类-生产者
```

public interface DisruptorMqService {

    /**
     * 消息
     * @param message
     */
    void sayHelloMq(String message);
}

@Slf4j
@Component
@Service
public class DisruptorMqServiceImpl implements DisruptorMqService {

    @Autowired
    private RingBuffer<MessageModel> messageModelRingBuffer;


    @Override
    public void sayHelloMq(String message) {
        log.info("record the message: {}",message);
        //获取下一个Event槽的下标
        long sequence = messageModelRingBuffer.next();
        try {
            //给Event填充数据
            MessageModel event = messageModelRingBuffer.get(sequence);
            event.setMessage(message);
            log.info("往消息队列中添加消息：{}", event);
        } catch (Exception e) {
            log.error("failed to add event to messageModelRingBuffer for : e = {},{}",e,e.getMessage());
        } finally {
            //发布Event，激活观察者去消费，将sequence传递给改消费者
            //注意最后的publish方法必须放在finally中以确保必须得到调用；如果某个请求的sequence未被提交将会堵塞后续的发布操作或者其他的producer
            messageModelRingBuffer.publish(sequence);
        }
    }
}
```

构造测试类及方法
```
@Slf4j
@RunWith(SpringRunner.class)
@SpringBootTest(classes = DemoApplication.class)
public class DemoApplicationTests {

    @Autowired
    private DisruptorMqService disruptorMqService;
    /**
     * 项目内部使用Disruptor做消息队列
     * @throws Exception
     */
    @Test
    public void sayHelloMqTest() throws Exception{
        disruptorMqService.sayHelloMq("消息到了，Hello world!");
        log.info("消息队列已发送完毕");
        //这里停止2000ms是为了确定是处理消息是异步的
        Thread.sleep(2000);
    }
}
```

## 总结

其实 生成者 -> 消费者 模式是很常见的，通过一些消息队列也可以轻松做到上述的效果。不同的地方在于，Disruptor 是在内存中以队列的方式去实现的，而且是无锁的。这也是 Disruptor 为什么高效的原因。