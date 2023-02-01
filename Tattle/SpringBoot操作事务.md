# SpringBoot操作事务

## 编程式事务

在 Spring Boot 中实现编程式事务又有两种实现方法：

- 使用 `TransactionTemplate` 对象实现编程式事务；
- 使用更加底层的 `TransactionManager` 对象实现编程式事务。

### TransactionTemplate 使用

要使用 `TransactionTemplate`  对象需要先将 `TransactionTemplate` 注入到当前类中 ，然后再使用它提供的 execute 方法执行事务并返回相应的执行结果，如果程序在执行途中出现了异常，那么就可以使用代码手动回滚事务，具体实现代码如下：

```java
@Resource
private TransactionTemplate transactionTemplate;

public int add(UserInfo userInfo){
    //...
    return transactionTemplate.execute(status ->{
        int result = 0;
        try {
            result = userService.add(userInfo);
        } catch(Exception e){
            status.setRollbackOnly();
        }
        return result;
    });
}
```

### TransactionManager 使用

`TransactionManager` `实现编程式事务相对麻烦一点，它需要使用两个对象：TransactionManager` 的子类，加上 `TransactionDefinition` 事务定义对象，再通过调用 `TransactionManager` 的 `getTransaction` 获取并开启事务，然后调用 `TransactionManager` 提供的 `commit` 方法提交事务，或使用它的另一个方法 `rollback` 回滚事务，它的具体实现代码如下：

```java
@Resource
private DataSourceTransactionManager dataSourceTransactionManager;

@Resource
private TransactionDefinition transactionDefinition;

public int add(UserInfo userInfo){
     TransactionStatus transactionStatus = dataSourceTransactionManager.getTransaction(transactionDefinition);

     try {
        //...
        dataSourceTransactionManager.commit(transactionStatus);
     }catch(Exception e){
        dataSourceTransactionManager.rollback(transactionStatus);
     }
    
}
```

## 声明式事务

声明式事务的实现比较简单，只需要在方法上或类上添加 `@Transactional` 注解即可，当加入了` @Transactional` 注解就可以实现在方法执行前，自动开启事务；在方法成功执行完，自动提交事务；如果方法在执行期间，出现了异常，那么它会自动回滚事务。


当然，@Transactional 支持很多参数的设置，它的参数设置列表如下：
|参数|类型|描述|
|--|--|--|
|`value`|String|指定要使用的事务管理器可选限定符|
|`propagation`|`enum`:`propagation`|可选传播设置|
|`isolation`|`enum`:`Isolation`|可选隔离级别。仅适用于传播值`REQUIRED`或`REQUIRES_NEW`|
|`timeout`|int（以秒为单位）|可选事务超时。仅适用于传播值`REQUIRED`或`REQUIRES_NEW`|
|`readOnly`|boolean|读写事务与只读事务。仅适用于以下值`REQUIRED`或`REQUIRES_NEW`|
|`rollbackFor`|数组Class对象，这些对象必须从Throwable|必须导致回滚的异常类的可选数组|
|`rollbackForClassName`|类名数组。这些类必须从Throwable|必须导致回滚的异常类名的可选数组|
|`noRollbackFor`|数组Class对象，这些对象必须从Throwable|不能导致回滚的异常类的可选数组|
|`noRollbackForClassName`|数组String类名，必须从Throwable|不能导致回滚的异常类名的可选数组|