# Spring Boot 配置文件脱敏

## 添加依赖
```
<dependency>
    <groupId>com.github.ulisesbocchio</groupId>
    <artifactId>jasypt-spring-boot-starter</artifactId>
    <version>3.0.3</version>
</dependency>  
```

## 配置秘钥
```
jasypt:
  encryptor:
    password: Y6M9fAJQdU7jNp5MW
```

当然将秘钥直接放在配置文件中也是不安全的，我们可以在项目启动的时候配置秘钥，命令如下：
```
java -jar xxx.jar  -Djasypt.encryptor.password=Y6M9fAJQdU7jNp5MW
```

## 生成加密后的数据
```
@SpringBootTest
@RunWith(SpringRunner.class)
public class SpringbootJasyptApplicationTests {

    /**
     * 注入加密方法
     */
    @Autowired
    private StringEncryptor encryptor;

    /**
     * 手动生成密文，此处演示了url，user，password
     */
    @Test
    public void encrypt() {
        String url = encryptor.encrypt("jdbc\\:mysql\\://127.0.0.1\\:3306/test?useUnicode\\=true&characterEncoding\\=UTF-8&zeroDateTimeBehavior\\=convertToNull&useSSL\\=false&allowMultiQueries\\=true&serverTimezone=Asia/Shanghai");
        String name = encryptor.encrypt("root");
        String password = encryptor.encrypt("123456");
        System.out.println("database url: " + url);
        System.out.println("database name: " + name);
        System.out.println("database password: " + password);
        Assert.assertTrue(url.length() > 0);
        Assert.assertTrue(name.length() > 0);
        Assert.assertTrue(password.length() > 0);
    }
}
```

### 将加密后的密文写入配置
```
spring:
  datasource:
    #   数据源基本配置
    username: ENC(L8I2RqYPptEtQNL4x8VhRVakSUdlsTGzEND/3TOnVTYPWe0ZnWsW0/5JdUsw9ulm)
    password: ENC(EJYCSbBL8Pmf2HubIH7dHhpfDZcLyJCEGMR9jAV3apJtvFtx9TVdhUPsAxjQ2pnJ)
    driver-class-name: com.mysql.jdbc.Driver
    url: ENC(szkFDG56WcAOzG2utv0m2aoAvNFH5g3DXz0o6joZjT26Y5WNA+1Z+pQFpyhFBokqOp2jsFtB+P9b3gB601rfas3dSfvS8Bgo3MyP1nojJgVp6gCVi+B/XUs0keXPn+pbX/19HrlUN1LeEweHS/LCRZslhWJCsIXTwZo1PlpXRv3Vyhf2OEzzKLm3mIAYj51CrEaN3w5cMiCESlwvKUhpAJVz/uXQJ1spLUAMuXCKKrXM/6dSRnWyTtdFRost5cChEU9uRjw5M+8HU3BLemtcK0vM8iYDjEi5zDbZtwxD3hA=)
    type: com.alibaba.druid.pool.DruidDataSource
```

上述配置是使用默认的prefix=ENC(、suffix=)，当然我们可以根据自己的要求更改，只需要在配置文件中更改即可，如下：
```
jasypt:
  encryptor:
    ## 指定前缀、后缀
    property:
      prefix: 'PASS('
      suffix: ')'
```

那么此时的配置就必须使用PASS()包裹才会被解密，如下
```
spring:
  datasource:
    #   数据源基本配置
    username: PASS(L8I2RqYPptEtQNL4x8VhRVakSUdlsTGzEND/3TOnVTYPWe0ZnWsW0/5JdUsw9ulm)
    password: PASS(EJYCSbBL8Pmf2HubIH7dHhpfDZcLyJCEGMR9jAV3apJtvFtx9TVdhUPsAxjQ2pnJ)
    driver-class-name: com.mysql.jdbc.Driver
    url: PASS(szkFDG56WcAOzG2utv0m2aoAvNFH5g3DXz0o6joZjT26Y5WNA+1Z+pQFpyhFBokqOp2jsFtB+P9b3gB601rfas3dSfvS8Bgo3MyP1nojJgVp6gCVi+B/XUs0keXPn+pbX/19HrlUN1LeEweHS/LCRZslhWJCsIXTwZo1PlpXRv3Vyhf2OEzzKLm3mIAYj51CrEaN3w5cMiCESlwvKUhpAJVz/uXQJ1spLUAMuXCKKrXM/6dSRnWyTtdFRost5cChEU9uRjw5M+8HU3BLemtcK0vM8iYDjEi5zDbZtwxD3hA=)
    type: com.alibaba.druid.pool.DruidDataSource
```