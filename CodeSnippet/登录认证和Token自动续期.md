# 登录认证和Token自动续期

## 技术选型

要实现认证功能，很容易就会想到JWT或者session。

### 区别

基于session和基于JWT的方式的主要区别就是用户的状态保存的位置，session是保存在服务端的，而JWT是保存在客户端的。

### 认证流程

#### 基于session的认证流程

- 用户在浏览器中输入用户名和密码，服务器通过密码校验后生成一个session并保存到数据库
- 服务器为用户生成一个sessionId，并将具有sesssionId的cookie放置在用户浏览器中，在后续的请求中都将带有这个cookie信息进行访问
- 服务器获取cookie，通过获取cookie中的sessionId查找数据库判断当前请求是否有效。

#### 基于JWT的认证流程

- 用户在浏览器中输入用户名和密码，服务器通过密码校验后生成一个token并保存到数据库
- 前端获取到token，存储到cookie或者local storage中，在后续的请求中都将带有这个token信息进行访问
- 服务器获取token值，通过查找数据库判断当前token是否有效

### 优缺点

JWT保存在客户端，在分布式环境下不需要做额外工作。而session因为保存在服务端，分布式环境下需要实现多机数据共享 session一般需要结合Cookie实现认证，所以需要浏览器支持cookie，因此移动端无法使用session认证方案。

### 安全性

JWT的payload使用的是base64编码的，因此在JWT中不能存储敏感数据。而session的信息是存在服务端的，相对来说更安全，如果在JWT中存储了敏感信息，可以解码出来非常的不安全。

### 性能

经过编码之后JWT将非常长，cookie的限制大小一般是4k，cookie很可能放不下，所以JWT一般放在local storage里面。并且用户在系统中的每一次http请求都会把JWT携带在Header里面，HTTP请求的Header可能比Body还要大。而sessionId只是很短的一个字符串，因此使用JWT的HTTP请求比使用session的开销大得多。

### 一次性

无状态是JWT的特点，但也导致了这个问题，JWT是一次性的。想修改里面的内容，就必须签发一个新的JWT。

### 无法废弃

一旦签发一个JWT，在到期之前就会始终有效，无法中途废弃。若想废弃，一种常用的处理手段是结合redis。

### 续签

如果使用JWT做会话管理，传统的cookie续签方案一般都是框架自带的，session有效期30分钟，30分钟内如果有访问，有效期被刷新至30分钟。一样的道理，要改变JWT的有效时间，就要签发新的JWT。

最简单的一种方式是每次请求刷新JWT，即每个HTTP请求都返回一个新的JWT。这个方法不仅暴力不优雅，而且每次请求都要做JWT的加密解密，会带来性能问题。另一种方法是在redis中单独为每个JWT设置过期时间，每次访问时刷新JWT的过期时间。

## 功能实现

JWT所需依赖
```java
<dependency>
    <groupId>com.auth0</groupId>
    <artifactId>java-jwt</artifactId>
    <version>3.10.3</version>
</dependency>
```

JWT工具类
```java
public class JWTUtil {
    private static final Logger logger = LoggerFactory.getLogger(JWTUtil.class);

    //私钥
    private static final String TOKEN_SECRET = "123456";

    /**
     * 生成token，自定义过期时间 毫秒
     *
     * @param userTokenDTO
     * @return
     */
    public static String generateToken(UserTokenDTO userTokenDTO) {
        try {
            // 私钥和加密算法
            Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
            // 设置头部信息
            Map<String, Object> header = new HashMap<>(2);
            header.put("Type", "Jwt");
            header.put("alg", "HS256");

            return JWT.create()
                    .withHeader(header)
                    .withClaim("token", JSONObject.toJSONString(userTokenDTO))
                    //.withExpiresAt(date)
                    .sign(algorithm);
        } catch (Exception e) {
            logger.error("generate token occur error, error is:{}", e);
            return null;
        }
    }

    /**
     * 检验token是否正确
     *
     * @param token
     * @return
     */
    public static UserTokenDTO parseToken(String token) {
        Algorithm algorithm = Algorithm.HMAC256(TOKEN_SECRET);
        JWTVerifier verifier = JWT.require(algorithm).build();
        DecodedJWT jwt = verifier.verify(token);
        String tokenInfo = jwt.getClaim("token").asString();
        return JSON.parseObject(tokenInfo, UserTokenDTO.class);
    }
}
```

说明：

- 生成的token中不带有过期时间，token的过期时间由redis进行管理
- UserTokenDTO中不带有敏感信息，如password字段不会出现在token中

Redis工具类
```java
public final class RedisServiceImpl implements RedisService {
    /**
     * 过期时长
     */
    private final Long DURATION = 1 * 24 * 60 * 60 * 1000L;

    @Resource
    private RedisTemplate redisTemplate;

    private ValueOperations<String, String> valueOperations;

    @PostConstruct
    public void init() {
        RedisSerializer redisSerializer = new StringRedisSerializer();
        redisTemplate.setKeySerializer(redisSerializer);
        redisTemplate.setValueSerializer(redisSerializer);
        redisTemplate.setHashKeySerializer(redisSerializer);
        redisTemplate.setHashValueSerializer(redisSerializer);
        valueOperations = redisTemplate.opsForValue();
    }

    @Override
    public void set(String key, String value) {
        valueOperations.set(key, value, DURATION, TimeUnit.MILLISECONDS);
        log.info("key={}, value is: {} into redis cache", key, value);
    }

    @Override
    public String get(String key) {
        String redisValue = valueOperations.get(key);
        log.info("get from redis, value is: {}", redisValue);
        return redisValue;
    }

    @Override
    public boolean delete(String key) {
        boolean result = redisTemplate.delete(key);
        log.info("delete from redis, key is: {}", key);
        return result;
    }

    @Override
    public Long getExpireTime(String key) {
        return valueOperations.getOperations().getExpire(key);
    }
}
```

登陆功能
```java
public String login(LoginUserVO loginUserVO) {
    //1.判断用户名密码是否正确
    UserPO userPO = userMapper.getByUsername(loginUserVO.getUsername());
    if (userPO == null) {
        throw new UserException(ErrorCodeEnum.TNP1001001);
    }
    if (!loginUserVO.getPassword().equals(userPO.getPassword())) {
        throw new UserException(ErrorCodeEnum.TNP1001002);
    }

    //2.用户名密码正确生成token
    UserTokenDTO userTokenDTO = new UserTokenDTO();
    PropertiesUtil.copyProperties(userTokenDTO, loginUserVO);
    userTokenDTO.setId(userPO.getId());
    userTokenDTO.setGmtCreate(System.currentTimeMillis());
    String token = JWTUtil.generateToken(userTokenDTO);

    //3.存入token至redis
    redisService.set(userPO.getId(), token);
    return token;
}
```

- 判断用户名密码是否正确
- 用户名密码正确则生成token
- 将生成的token保存至redis

拦截器类
```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                             Object handler) throws Exception {
    String authToken = request.getHeader("Authorization");
    String token = authToken.substring("Bearer".length() + 1).trim();
    UserTokenDTO userTokenDTO = JWTUtil.parseToken(token);
    //1.判断请求是否有效
    if (redisService.get(userTokenDTO.getId()) == null 
            || !redisService.get(userTokenDTO.getId()).equals(token)) {
        return false;
    }

    //2.判断是否需要续期
    if (redisService.getExpireTime(userTokenDTO.getId()) < 1 * 60 * 30) {
        redisService.set(userTokenDTO.getId(), token);
        log.error("update token info, id is:{}, user info is:{}", userTokenDTO.getId(), token);
    }
    return true;
}
```

拦截器配置类
```java
@Configuration
public class InterceptorConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(authenticateInterceptor())
                .excludePathPatterns("/logout/**")
                .excludePathPatterns("/login/**")
                .addPathPatterns("/**");
    }

    @Bean
    public AuthenticateInterceptor authenticateInterceptor() {
        return new AuthenticateInterceptor();
    }
}
```