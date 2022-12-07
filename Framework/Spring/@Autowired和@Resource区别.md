# @Autowired和@Resource
## @Autowired
@Autowired为Spring提供的注解，需要导入org.springframework.beans.factory.annotation.Autowired

@Autowired采取的策略为按照类型注入。

当一个类型有多个bean值的时候，会造成无法选择具体注入哪一个情况，这个时候需要配合着@Qualifier使用。

## @Resource
@Resource注解是由J2EE提供，需要导入javax.annotation.Resource

@Resource默认按照ByName自动注入

- 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常
- 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常
- 如果指定了Type，则从上下文中找到类似匹配的唯一bean进行装配，找不到或者找到了多个，就会抛出异常
- 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配。

