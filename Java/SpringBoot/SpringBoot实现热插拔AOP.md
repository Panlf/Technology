# SpringBoot实现热插拔AOP

> 现在有这么一个需求：就是我们日志的开与关是交给使用人员来控制的，而不是由我们开发人员固定写死的。大家都知道可以用aop来实现日志管理，但是如何动态的来实现日志管理呢？aop源码中的实现逻辑中有这么一个步骤，就是会依次扫描Advice的实现类，然后执行。我们要做的就是自定义一个advice的实现类然后，在用户想要开启日志的时候就把advice加到项目中来，关闭日志的时候就把advice剔除就行了。

## 核心实现代码
### 动态管理advice端点实现
```java
@RestControllerEndpoint(id = "proxy")
@RequiredArgsConstructor
public class ProxyMetaDefinitionControllerEndPoint {

    private final ProxyMetaDefinitionRepository proxyMetaDefinitionRepository;


    @GetMapping("listMeta")
    public List<ProxyMetaDefinition> getProxyMetaDefinitions(){
       return proxyMetaDefinitionRepository.getProxyMetaDefinitions();
    }

    @GetMapping("{id}")
    public ProxyMetaDefinition getProxyMetaDefinition(@PathVariable("id") String proxyMetaDefinitionId){
        return proxyMetaDefinitionRepository.getProxyMetaDefinition(proxyMetaDefinitionId);
    }

    @PostMapping("save")
    public String save(@RequestBody ProxyMetaDefinition definition){

        try {
            proxyMetaDefinitionRepository.save(definition);
            return "success";
        } catch (Exception e) {

        }
        return "fail";

    }

    @PostMapping("delete/{id}")
    public String delete(@PathVariable("id")String proxyMetaDefinitionId){
        try {
            proxyMetaDefinitionRepository.delete(proxyMetaDefinitionId);
            return "success";
        } catch (Exception e) {

        }
        return "fail";
    }

}
```

### 利用事件监听机制捕获安装或者卸载插件
```java
@RequiredArgsConstructor
public class ProxyMetaDefinitionChangeListener {

    private final AopPluginFactory aopPluginFactory;

    @EventListener
    public void listener(ProxyMetaDefinitionChangeEvent proxyMetaDefinitionChangeEvent){
        ProxyMetaInfo proxyMetaInfo = aopPluginFactory.getProxyMetaInfo(proxyMetaDefinitionChangeEvent.getProxyMetaDefinition());
        switch (proxyMetaDefinitionChangeEvent.getOperateEventEnum()){
            case ADD:
                aopPluginFactory.installPlugin(proxyMetaInfo);
                break;
            case DEL:
                aopPluginFactory.uninstallPlugin(proxyMetaInfo.getId());
                break;
        }

    }
}
```

### 安装插件
```java
public void installPlugin(ProxyMetaInfo proxyMetaInfo){
        if(StringUtils.isEmpty(proxyMetaInfo.getId())){
            proxyMetaInfo.setId(proxyMetaInfo.getProxyUrl() + SPIILT + proxyMetaInfo.getProxyClassName());
        }
        AopUtil.registerProxy(defaultListableBeanFactory,proxyMetaInfo);
    }
```

### 安装插件核心实现
```java
public static void registerProxy(DefaultListableBeanFactory beanFactory,ProxyMetaInfo proxyMetaInfo){
        AspectJExpressionPointcutAdvisor advisor = getAspectJExpressionPointcutAdvisor(beanFactory, proxyMetaInfo);
        addOrDelAdvice(beanFactory,OperateEventEnum.ADD,advisor);

    }

    private static AspectJExpressionPointcutAdvisor getAspectJExpressionPointcutAdvisor(DefaultListableBeanFactory beanFactory, ProxyMetaInfo proxyMetaInfo) {
        BeanDefinitionBuilder builder = BeanDefinitionBuilder.genericBeanDefinition();
        GenericBeanDefinition beanDefinition = (GenericBeanDefinition) builder.getBeanDefinition();
        beanDefinition.setBeanClass(AspectJExpressionPointcutAdvisor.class);
        AspectJExpressionPointcutAdvisor advisor = new AspectJExpressionPointcutAdvisor();
        advisor.setExpression(proxyMetaInfo.getPointcut());
        advisor.setAdvice(Objects.requireNonNull(getMethodInterceptor(proxyMetaInfo.getProxyUrl(), proxyMetaInfo.getProxyClassName())));
        beanDefinition.setInstanceSupplier((Supplier<AspectJExpressionPointcutAdvisor>) () -> advisor);
        beanFactory.registerBeanDefinition(PROXY_PLUGIN_PREFIX + proxyMetaInfo.getId(),beanDefinition);

        return advisor;
    }
```

### 卸载插件
```java
public void uninstallPlugin(String id){
        String beanName = PROXY_PLUGIN_PREFIX + id;
        if(defaultListableBeanFactory.containsBean(beanName)){
           AopUtil.destoryProxy(defaultListableBeanFactory,id);
        }else{
            throw new NoSuchElementException("Plugin not found: " + id);
        }
    }
```

### 卸载插件核心实现
```
public static void destoryProxy(DefaultListableBeanFactory beanFactory,String id){
        String beanName = PROXY_PLUGIN_PREFIX + id;
        if(beanFactory.containsBean(beanName)){
            AspectJExpressionPointcutAdvisor advisor = beanFactory.getBean(beanName,AspectJExpressionPointcutAdvisor.class);
            addOrDelAdvice(beanFactory,OperateEventEnum.DEL,advisor);
            beanFactory.destroyBean(beanFactory.getBean(beanName));
        }
    }
```

### 操作advice实现
```java
public static void addOrDelAdvice(DefaultListableBeanFactory beanFactory, OperateEventEnum operateEventEnum,AspectJExpressionPointcutAdvisor advisor){
        AspectJExpressionPointcut pointcut = (AspectJExpressionPointcut) advisor.getPointcut();
        for (String beanDefinitionName : beanFactory.getBeanDefinitionNames()) {
            Object bean = beanFactory.getBean(beanDefinitionName);
            if(!(bean instanceof Advised)){
                if(operateEventEnum == OperateEventEnum.ADD){
                    buildCandidateAdvised(beanFactory,advisor,bean,beanDefinitionName);
                }
                continue;
            }
            Advised advisedBean = (Advised) bean;
            boolean isFindMatchAdvised = findMatchAdvised(advisedBean.getClass(),pointcut);
            if(operateEventEnum == OperateEventEnum.DEL){
                if(isFindMatchAdvised){
                    advisedBean.removeAdvice(advisor.getAdvice());
                    log.info("######################################### Remove Advice -->【{}】 For Bean -->【{}】 SUCCESS !",advisor.getAdvice().getClass().getName(),bean.getClass().getName());
                }
            }else if(operateEventEnum == OperateEventEnum.ADD){
                if(isFindMatchAdvised){
                    advisedBean.addAdvice(advisor.getAdvice());
                    log.info("######################################### Add Advice -->【{}】 For Bean -->【{}】 SUCCESS !",advisor.getAdvice().getClass().getName(),bean.getClass().getName());
                }
            }


        }
    }
```

## 热插拔AOP演示示例
### 创建一个service
```java
@Service
@Slf4j
public class HelloService implements BeanNameAware, BeanFactoryAware {
    private BeanFactory beanFactory;

    private String beanName;

    @SneakyThrows
    public String sayHello(String message) {
        Object bean = beanFactory.getBean(beanName);
        log.info("============================ {} is Advised : {}",bean, bean instanceof Advised);
        TimeUnit.SECONDS.sleep(new Random().nextInt(3));
        log.info("============================ hello:{}",message);

        return "hello:" + message;

    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
    }

    @Override
    public void setBeanName(String name) {
        this.beanName = name;
    }
}
```

### 创建一个controller
```
@RestController
@RequestMapping("hello")
@RequiredArgsConstructor
public class HelloController {

    private final HelloService helloService;

    @GetMapping("{message}")
    public String sayHello(@PathVariable("message")String message){
        return helloService.sayHello(message);
    }
}
```