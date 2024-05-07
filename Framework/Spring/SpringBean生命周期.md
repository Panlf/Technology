# Spring Bean 生命周期
Bean生命周期大致分为以下几个阶段：
- Bean的实例化(Instantiation)：Spring框架会取出BeanDefinition的信息进行判断当前Bean的范围是否是singleton的，是否不是延迟加载的，是否不是FactoryBean等，最终将一个普通的singleton的Bean通过反射进行实例化
- Bean的属性赋值(Populate)：Bean实例化之后还仅仅是个"半成品"，还需要对Bean实例的属性进行填充，Bean的属性赋值就是指 Spring 容器根据BeanDefinition中属性配置的属性值注入到 Bean 对象中的过程。
- Bean的初始化(Initialization)：对Bean实例的属性进行填充完之后还需要执行一些Aware接口方法、执行BeanPostProcessor方法、执行InitializingBean接口的初始化方法、执行自定义初始化init方法等。该阶段是Spring最具技术含量和复杂度的阶段，并且Spring高频面试题Bean的循环引用问题也是在这个阶段体现的；
- Bean的使用阶段：经过初始化阶段，Bean就成为了一个完整的Spring Bean，被存储到单例池singletonObjects中去了，即完成了Spring Bean的整个生命周期，接下来Bean就可以被随心所欲地使用了。
- Bean的销毁(Destruction)：Bean 的销毁是指 Spring 容器在关闭时，执行一些清理操作的过程。在 Spring 容器中， Bean 的销毁方式有两种：销毁方法destroy-method和 DisposableBean 接口。

BeanDefinitionRegistryPostProcessor,BeanFactoryPostProcessor,BeanPostProcessor这三个后置处理器的调用时机都在Spring Bean生命周期中是不严谨的，按照上面我们对Bean生命周期的阶段划分，只有BeanPostProcessor作用于Bean的生命周期中，而BeanDefinitionRegistryPostProcessor,BeanFactoryPostProcessor是针对BeanDefinition的，所以不属于Bean的生命周期中。


Bean的创建过程核心步骤如下：

- createBeanInstance(BeanName, mbd, args) 进行Bean的实例化
- populateBean(BeanName, mbd, instanceWrapper)进行Bean的属性填充赋值
- initializeBean(BeanName, exposedObject, mbd)处理Bean初始化之后的各种回调事件
- registerDisposableBeanIfNecessary(BeanName, Bean, mbd)注册Bean的销毁接口
解决创建Bean过程中的循环依赖，Spring使用三级缓存解决循环依赖.

Bean的销毁流程与Bean初始化类似，当容器关闭时，才会对Bean销毁方法进行调用。销毁过程是这样的。顺着close()-> doClose() -> destroyBeans() -> destroySingletons() -> destroySingleton() -> destroyBean() -> Bean.destroy() ，会看到最终调用Bean的销毁方法。