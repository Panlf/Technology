# 基于SpringBoot的延迟双删

## 业务场景
在多线程并发情况下，假设有两个数据库修改请求，为保证数据库与redis的数据一致性，修改请求的实现中需要修改数据库后，级联修改Redis中的数据。

## 解决方案
在使用Redis时，需要保持Redis和数据库数据的一致性，最流行的解决方案之一就是延时双删策略。

注意：要知道经常修改的数据表不适合使用Redis，因为双删策略执行的结果是把Redis中保存的那条数据删除了，以后的查询就都会去查询数据库。所以Redis使用的是读远远大于改的数据缓存。

延时双删方案执行步骤

- 删除缓存
- 更新数据库
- 延时500毫秒 (根据具体业务设置延时执行的时间)
- 删除缓存

### 为何要延时500毫秒？
这是为了我们在第二次删除Redis之前能完成数据库的更新操作。假象一下，如果没有第三步操作时，有很大概率，在两次删除Redis操作执行完毕之后，数据库的数据还没有更新，此时若有请求访问数据，便会出现我们一开始提到的那个问题。

### 为何要两次删除缓存？
如果我们没有第二次删除操作，此时有请求访问数据，有可能是访问的之前未做修改的Redis数据，删除操作执行后，Redis为空，有请求进来时，便会去访问数据库，此时数据库中的数据已是更新后的数据，保证了数据的一致性。

## 代码实践
### 引入依赖
```
<!-- redis使用 -->
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<!-- aop -->
<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

## 编写自定义aop注解和切面

ClearAndReloadCache延时双删注解

```
/**
 *延时双删
 **/
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Target(ElementType.METHOD)
public @interface ClearAndReloadCache {
    String name() default "";
}
```

ClearAndReloadCacheAspect延时双删切面

```
@Aspect
@Component
public class ClearAndReloadCacheAspect {

@Autowired
private StringRedisTemplate stringRedisTemplate;

/**
* 切入点
*切入点,基于注解实现的切入点  加上该注解的都是Aop切面的切入点
*
*/

@Pointcut("@annotation(com.pdh.cache.ClearAndReloadCache)")
public void pointCut(){

}
/**
* 环绕通知
* 环绕通知非常强大，可以决定目标方法是否执行，什么时候执行，执行时是否需要替换方法参数，执行完毕是否需要替换返回值。
* 环绕通知第一个参数必须是org.aspectj.lang.ProceedingJoinPoint类型
* @param proceedingJoinPoint
*/
@Around("pointCut()")
public Object aroundAdvice(ProceedingJoinPoint proceedingJoinPoint){
    System.out.println("----------- 环绕通知 -----------");
    System.out.println("环绕通知的目标方法名：" + proceedingJoinPoint.getSignature().getName());

    Signature signature1 = proceedingJoinPoint.getSignature();
    MethodSignature methodSignature = (MethodSignature)signature1;
    Method targetMethod = methodSignature.getMethod();//方法对象
    ClearAndReloadCache annotation = targetMethod.getAnnotation(ClearAndReloadCache.class);//反射得到自定义注解的方法对象

    String name = annotation.name();//获取自定义注解的方法对象的参数即name
    Set<String> keys = stringRedisTemplate.keys("*" + name + "*");//模糊定义key
    stringRedisTemplate.delete(keys);//模糊删除redis的key值

    //执行加入双删注解的改动数据库的业务 即controller中的方法业务
    Object proceed = null;
    try {
        proceed = proceedingJoinPoint.proceed();
    } catch (Throwable throwable) {
        throwable.printStackTrace();
    }

    //开一个线程 延迟1秒（此处是1秒举例，可以改成自己的业务）
    // 在线程中延迟删除  同时将业务代码的结果返回 这样不影响业务代码的执行
    new Thread(() -> {
        try {
            Thread.sleep(1000);
            Set<String> keys1 = stringRedisTemplate.keys("*" + name + "*");//模糊删除
            stringRedisTemplate.delete(keys1);
            System.out.println("-----------1秒钟后，在线程中延迟删除完毕 -----------");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }).start();

    return proceed;//返回业务代码的值
    }
}
```

### application.yml
```
server:
  port: 8082

spring:
  # redis setting
  redis:
    host: localhost
    port: 6379

  # cache setting
  cache:
    redis:
      time-to-live: 60000 # 60s

  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test
    username: root
    password: 1234

# mp setting
mybatis-plus:
  mapper-locations: classpath*:com/pdh/mapper/*.xml
  global-config:
    db-config:
      table-prefix:
  configuration:
    # log of sql
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    # hump
    map-underscore-to-camel-case: true
```


### user_db.sql脚本
```
DROP TABLE IF EXISTS `user_db`;
CREATE TABLE `user_db`  (
  `id` int(4) NOT NULL AUTO_INCREMENT,
  `username` varchar(32) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB AUTO_INCREMENT = 8 CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of user_db
-- ----------------------------
INSERT INTO `user_db` VALUES (1, '张三');
INSERT INTO `user_db` VALUES (2, '李四');
INSERT INTO `user_db` VALUES (3, '王二');
INSERT INTO `user_db` VALUES (4, '麻子');
INSERT INTO `user_db` VALUES (5, '王三');
INSERT INTO `user_db` VALUES (6, '李三');
```

### UserController
```
/**
 * 用户控制层
 */
@RequestMapping("/user")
@RestController
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("/get/{id}")
    @Cache(name = "get method")
    //@Cacheable(cacheNames = {"get"})
    public Result get(@PathVariable("id") Integer id){
        return userService.get(id);
    }

    @PostMapping("/updateData")
    @ClearAndReloadCache(name = "get method")
    public Result updateData(@RequestBody User user){
        return userService.update(user);
    }

    @PostMapping("/insert")
    public Result insert(@RequestBody User user){
        return userService.insert(user);
    }

    @DeleteMapping("/delete/{id}")
    public Result delete(@PathVariable("id") Integer id){
        return userService.delete(id);
    }
}
```

### UserService
```
/**
 * service层
 */
@Service
public class UserService {

    @Resource
    private UserMapper userMapper;

    public Result get(Integer id){
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getId,id);
        User user = userMapper.selectOne(wrapper);
        return Result.success(user);
    }

    public Result insert(User user){
        int line = userMapper.insert(user);
        if(line > 0)
            return Result.success(line);
        return Result.fail(888,"操作数据库失败");
    }

    public Result delete(Integer id) {
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
        wrapper.eq(User::getId, id);
        int line = userMapper.delete(wrapper);
        if (line > 0)
            return Result.success(line);
        return Result.fail(888, "操作数据库失败");
    }

    public Result update(User user){
        int i = userMapper.updateById(user);
        if(i > 0)
            return Result.success(i);
        return Result.fail(888,"操作数据库失败");
    }
}
```