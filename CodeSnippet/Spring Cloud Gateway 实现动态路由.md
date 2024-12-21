# Spring Cloud Gateway 实现动态路由
- 安装服务注册中心Nacos
## 依赖
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
    <version>3.1.7</version>
</dependency>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>2021.0.5.0</version>
</dependency>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>2021.0.5.0</version>
</dependency>

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-loadbalancer</artifactId>
    <version>3.1.6</version>
</dependency>

```

## 配置文件适配
### 适配前
```
server:
  port: 8080

spring:
  application:
    name: my-gateway

  cloud:
    nacos:
      # IP端口和用户名、密码
      server-addr: 127.0.0.1:8868
      username: nacos
      password: nacos

    gateway:
      routes:
        - id: api
          # 匹配后，转发到此域名
          uri: lb://api
          # 匹配路径
          predicates:
            - Path=/api/**
          # 过滤器配置
          filters:
            - StripPrefix=1
```
### 适配后
```
server:
  port: 8080

spring:
  application:
    name: my-gateway

  cloud:
    nacos:
      # IP端口和用户名、密码
      server-addr: 127.0.0.1:8868
      username: nacos
      password: nacos

+      config:
+        # 禁用Spring的配置导入检查
+        import-check:
+         enabled: false

    gateway:
+      discovery:
+        locator:
+        	# 开启通过服务中心动态更新配置的功能
+          enabled: true

#      routes:
#        - id: api
#          # 匹配后，转发到此域名
#          uri: lb://api
#          # 匹配路径
#          predicates:
#            - Path=/api/**
#          # 过滤器配置
#          filters:
#            - StripPrefix=1

```

## 监听变更与更新路由配置
我们需要写一点Java代码，监听Nacos上面的配置变更，然后更新路由信息
```java

/**
 * 路由信息变更监听
 */
@Component
@RefreshScope
public class OnRouteConfigChange implements ApplicationEventPublisherAware {

    private static final Logger LOGGER = LoggerFactory.getLogger(OnRouteConfigChange.class);

    /** 路由信息写入器 */
    @Resource
    private RouteDefinitionWriter routeDefinitionWriter;

    /** 获取yml文件里nacos的配置信息 */
    @Resource
    private NacosConfigProperties nacosConfigProperties;

    /** nacos 配置服务 */
    private ConfigService configService;

    /** 应用事件发布器 */
    private ApplicationEventPublisher publisher;


    /**
     * 获取 nacos 配置服务
     * @return
     * @throws NacosException
     */
    private ConfigService getConfigService() throws NacosException {
        // 懒汉加载，用到再创建这个服务
        if (null == configService){
            Properties properties = new Properties();
            properties.setProperty(PropertyKeyConst.SERVER_ADDR, nacosConfigProperties.getServerAddr());
            properties.setProperty(PropertyKeyConst.USERNAME, nacosConfigProperties.getUsername());
            properties.setProperty(PropertyKeyConst.PASSWORD, nacosConfigProperties.getPassword());
            configService = NacosFactory.createConfigService(properties);
        }
        return configService;
    }

    /**
     *  网关初始化
     */
    @PostConstruct
    public void initGatewayRoutes() {

        try {

            // 配置ID
            String dateId = "my_gateway_route_config";
            // 分组
            String group = "MY_GATEWAY_ROUTE";

            // 获取当前Nacos里，对应dataId的配置数据，随后开始监听对应dataId数据变更
            String configJson = getConfigService().getConfigAndSignListener(dateId, group, nacosConfigProperties.getTimeout(), new Listener() {
                @Override
                public Executor getExecutor() { return null; }

                /**
                 * 数据变更时触发
                 * @param configInfo 新的配置信息
                 */
                @Override
                public void receiveConfigInfo(String configInfo) {
                    LoggerUtils.info(LOGGER, "监听到网关路由配置变更：{0}", configInfo);
                    saveRoutesConfig(configInfo);
                }
            });

            LoggerUtils.info(LOGGER, "初始化网关路由配置：{0}", configJson);
            saveRoutesConfig(configJson);

        } catch (Exception e) {
            LoggerUtils.error(e, LOGGER, "初始化网关路由时发生错误", e);
        }
    }


    /**
     * 保存路由配置
     * @param configJson
     */
    private void saveRoutesConfig(String configJson){
        List<RouteDefinition> definitions = JSONArray.parseArray(configJson, RouteDefinition.class);
        if (!CollectionUtils.isEmpty(definitions)){
            // 遍历更新路由信息
            for (RouteDefinition definition : definitions){
                routeDefinitionWriter.save(Mono.just(definition)).subscribe();
            }
            // 发送刷新路由的事件通知
            publisher.publishEvent(new RefreshRoutesEvent(this));
        }
    }

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        publisher = applicationEventPublisher;
    }
}

```

## 在 Nacos 里配置路由信息
```
[
    {
        "id": "api",
        "uri": "lb://api",
        "predicates": [
            {
                "args": {
                    "pattern": "/api/**",
                },
                "name": "Path"
            }
        ],
        "filters": [
            {
                "args": {
                    "parts": 1
                },
                "name": "StripPrefix"
            }
        ],
        "order": 1
    }
]

```