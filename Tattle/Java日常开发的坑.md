# Java日常开发的坑

## 六类典型空指针问题

- 包装类型的空指针问题
- 级联调用的空指针问题
- Equals方法左边的空指针问题
- ConcurrentHashMap这样的容器不支持Key和Value为null
- 集合，数组直接获取元素
- 对象直接获取属性

## 日期YYYY格式设置的坑

```
2019-12-31 转 YYYY-MM-dd 格式后 2020-12-31
```

## 金额数值计算精度的坑

建议使用`BigDecimal`，并构造方法使用字符串方式。

## FileReader默认编码导致乱码问题

## Integer缓存的坑

默认情况下，Integer的缓存区间就是[-128,127]，所以我们业务日常开发中，如果涉及Integer值的比较，需要注意整个坑，还有设置JVM参数加上`-XX:AutoBoxCacheMax=1000`，是可以调整这个区间参数的。

## static静态变量依赖Spring实例化变量，可能导致初始化出错

```java
private static SmsService smsService = SpringContextUtils.getBean(SmsService.class); 
```

## 使用ThreadLocal，线程重用导致信息错乱的坑

线程池会重用固定的几个线程，一旦线程重用，那么很可能首次从ThreadLocal获取的值是之前其他用户的请求遗留的值。这时，ThreadLocal中的用户信息就是其他用户的信息。

## 疏忽switch的return和break

## Arrays.asList的几个坑

- 基本类型不能作为Arrays.asList方法的参数，否则会被当做一个参数
```
int[] array = {1,2,3}
List list = Arrays.asList(array)
```
- Arrays.asList返回的List不支持增删操作
- 使用Arrays.asList的时候，对原始数组的修改会影响我们获得的那个List

## ArrayList.toArray()强转的坑

## 异常使用的几个坑

- 不要弄丢了你的堆栈异常信息 e
- 不要把异常定义为静态变量
- 生产环境不要使用`e.printStackTrace()`
- 处理线程池提交任务的异常
  - 在任务代码try/catch捕获异常
  - 通过Future对象的get方法接收抛出的异常，再处理
  - 为工作者线程设置UncaughtExceptionHandler，在uncaughtException方法中处理异常
  - 重写ThreadPoolExecutor的afterExecute方法，处理传递的异常引用
- finally重新抛出的异常也要注意

## JSON序列化，Long类型会被转成Integer类型

## 使用Executors声明线程池，newFixedThreadPool的OOM问题
newFixedThreadPool 使用的是无界队列

## 直接大文件或者一次性从数据库读取太多数据到内存，可能导致OOM问题

## 事务未生效的坑

- 底层数据库引擎不支持事务
- 在非public修饰的方法使用
- rollbackFor属性设置错误
- 本类方法直接调用
- 异常被try...catch吃了，导致事务失效

## 当反射遇到方法重载的坑