# Java程序员的易错问题

## 使用Objects.equals比较对象

```java
Long longValue = 123L;
System.out.println(longValue==123); //true
System.out.println(Objects.equals(longValue,123)); //false
```

为什么替换`==`为`Objects.equals()`会导致不同的结果？这是因为使用`==`编译器会得到封装类型对应的基本数据类型`longValue`，然后与这个基本数据类型进行比较，相当于编译器会自动将常量转换为比较基本数据类型, 而不是包装类型。

使用该`Objects.equals()`方法后，编译器默认常量的基本数据类型为`int`。下面是源码`Objects.equals()`，其中`a.equals(b)`使用的是`Long.equals()`会判断对象类型，因为编译器已经认为常量是int类型，所以比较结果一定是false。

```java

public static boolean equals(Object a, Object b) {
    return (a == b) || (a != null && a.equals(b));
}
    
public boolean equals(Object obj) {
    if (obj instanceof Long) {
        return value == ((Long)obj).longValue();
    }
    return false;
}
```

知道了原因，解决方法就很简单了。直接声明常量的数据类型，如`Objects.equals(longValue,123L)`。其实如果逻辑严密，就不会出现上面的问题。我们需要做的是保持良好的编码习惯.

## 日期格式错误

```
Instant instant = Instant.parse("2021-12-31T00:00:00.00Z");
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("YYYY-MM-dd HH:mm:ss")
.withZone(ZoneId.systemDefault());
System.out.println(formatter.format(instant));//2022-12-31 08:00:00
```

以上用于`YYYY-MM-dd`格式化, 年从2021变成了2022。为什么？这是因为`java`的`DateTimeFormatter`模式`YYYY`和`yyyy`之间存在细微的差异。它们都代表一年，但是`yyyy`代表日历年，而`YYYY`代表星期。这是一个细微的差异，仅会导致一年左右的变更问题，因此您的代码本可以一直正常运行，而仅在新的一年中引发问题。12月31日按周计算的年份是2022年，正确的方式应该是使用yyyy-MM-dd格式化日期。

## 在 ThreadPool 中使用 ThreadLocal

如果创建一个`ThreadLocal`变量，访问该变量的线程将创建一个线程局部变量。合理使用`ThreadLocal`可以避免线程安全问题。

但是，如果在线程池中使用`ThreadLocal`，就要小心了。您的代码可能会产生意想不到的结果。举个很简单的例子，假设我们有一个电商平台，用户购买商品后需要发邮件确认。

```java
private ThreadLocal<User> currentUser = ThreadLocal.withInitial(() -> null);

private ExecutorService executorService = Executors.newFixedThreadPool(4);

public void executor() {
    executorService.submit(()->{
        User user = currentUser.get();
        Integer userId = user.getId();
        sendEmail(userId);
    });
}
```

如果我们使用`ThreadLocal`来保存用户信息，这里就会有一个隐藏的bug。因为使用了线程池，线程是可以复用的，所以在使用`ThreadLocal`获取用户信息的时候，很可能会误获取到别人的信息。您可以使用会话来解决这个问题。

## 使用HashSet去除重复数据

```
User user1 = new User();
user1.setUsername("test");

User user2 = new User();
user2.setUsername("test");

List<User> users = Arrays.asList(user1, user2);
HashSet<User> sets = new HashSet<>(users);
System.out.println(sets.size());// the size is 2
```

`HashSet`使用`hashcode`对哈希表进行寻址，使用`equals`方法判断对象是否相等。如果自定义对象没有重写`hashcode`方法和`equals`方法，则默认使用父对象的`hashcode`方法和`equals`方法。所以`HashSet`会认为这是两个不同的对象，所以导致去重失败。

## 线程池中的异常被吃掉

```
ExecutorService executorService = Executors.newFixedThreadPool(1);
executorService.submit(()->{
    //do something
    double result = 10/0;
});
```

上面的代码模拟了一个线程池抛出异常的场景。我们真正的业务代码要处理各种可能出现的情况，所以很有可能因为某些特定的原因而触发`RuntimeException。`

但是如果没有特殊处理，这个异常就会被线程池吃掉。这样就会导出出现问题你都不知道，这是很严重的后果。因此，最好在线程池中`try catch`捕获异常。