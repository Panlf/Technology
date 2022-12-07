# MyBatis

## 执行流程

- 动态代理    MapperProxy
- SQL会话     SqlSession 
- 执行器   Executor
- JDBC处理器   StatementHandler

SQLSession ==> Executor ==> StatementHandler

- SQLSession
  - 基本API 增、删、改、查
  - 辅助API 提交、关闭会话
- Executor
  - 基本功能    改、查、缓存维护事务管理
  - 辅助API   提交、关闭执行器、批处理刷新
- StatementHandler
  - SQL 声明 声明Statement、填充参数
  - SQL执行 改、查、批处理

## 缓存

### 一级缓存总结

- 与会话相关
- 参数条件有关
- 提交、修改都会清空

### 二级缓存

二级缓存也称作是应用级缓存，与一级缓存不同的是它的作用范围是整个应用，而且可以跨线程使用。所以二级缓存有更高的命中率，适合缓存一些修改。

#### 二级缓存配置

- cacheEnabled 全局缓存开关 默认 true
- useCache statement 缓存开关 true
- flushCache 清除默认 修改 true  查询false
- <cache /> 或者 @CacheNamespace  声明缓存空间
- <cache-ref> 或者  @CacheNamespaceRef 引用缓存空间

## StatementHandler

### 定义

JDBC处理器，基于JDBC构建Statement，并设置参数，然后执行Sql。每调用会话当中一次SQL，都会有与之相对应的且唯一的Statement实例。

- SimpleStatementHandler 简单处理器
- PreparedStatementHandler 预处理器
- CallableStatementHandler 存储过程

### 参数处理

#### 参数转换

ParamNameResolver

##### 单个参数

默认不作处理。除非设置@param

##### 多个参数

- 转换成Map
- 转换成param1、param2...
- 基于@param中name属性转换
- 基于反射转换成变量名，如果不支持转换成arg0、arg1

#### 参数映射

ParameterHandler

##### 单个原始类型

直接映射，勿略SQL中引用名称

##### Map类型

基于Map key映射

##### Object

基于属性名称映射，支持嵌套对象属性访问

#### 参数赋值

TypeHandler

### 结果集处理

- ResultSetHandler 将结果集行转换成对象
- ResultContext 存放当前对象，以及解析状态和控制解析数量
- ResultHandler 处理存入解析结果

## MyBatis映射体系

### MetaObject

- 查找属性
  - 勿略大小写、支持驼峰、支持子属性
- 获取属性值  
  - 基于点获取子属性 "user.name"
  - 基于索引获取列表值 "users[1].id"
  - 基于key获取map值 "user[name]" 
- 设置属性
  - 可设置子属性值
  - 支持自动创建子属性(必须带有空参构造方法，且不能是集合)

### BeanWrapper

- 查找属性
  - MetaObject中查找属性具体实现
- 获取属性值
  - 获取当前对象属性值
- 设置属性
  - 设置当前对象属性值
- 创建属性
  - 基于默认方法构建属性值

### MetaClass

- 基于属性名获取
  - get、set方法，支持子属性获取

### Reflector

- 基于属性名获取
  - get方法、set方法、属性类别（不支持子属性获取）
