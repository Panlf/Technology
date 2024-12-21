# Java日期格式化

在我们实际开发过程中，很多地方都需要用到日期格式转换。JDK提供了`SimpleDateFormat`，用于将日期类型格式化字符串，使用方法如下：
```
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

String str = sdf.format(new Date());
```

由于`SimpleDateFormat`并非是线程安全的，因此不能作为类变量使用，以下代码是错误的
```
static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");

public String formatDate(Date d){
    return sdf.format(d);
}
```

正确用法是每次调用`formatDate`都构造一个新的`SimpleDateFormat`对象，代码如下：
```
static String dateFormat = "yyyy-MM-dd";

public String formatDate(Date d){
    SimpleDateFormat sdf = new SimpleDateFormat(dateFormat);
    return sdf.format(d);
}
```

这段正确代码的性能非常糟糕，构造`SimpleDateFormat`会像所有其他格式化工具和模板引擎那样预先编译格式化字符串到一种中间结构。查看`SimpleDateFormat.compile`方法，大约有200行编译代码。

对于任何编译和格式化工具，预编译成中间格式是提升性能非常好的方法，不用每次都解析，但第一次预编译确实非常耗时，比如把JSP编译成Servlet，把Java源码编译成字节码，等等。

在做性能压测后监控虚拟机，发现`SimpleDateFormat`的构造总会出现在热点中，日期格式化时有三种方式可以增强性能。

预先构造`SimpleDateFormat`，放到`ThreadLocal`中，代码如下：
```
public class CommonUtil {
    private ThreadLocal<SimpleDateFormat> threadLocal = new ThreadLocal<>(){
        public SimpleDateFormat initialValue(){
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            return sdf;
        }
    }; 

    public String formatDate(Date d){
        SimpleDateFormat sdf = getDateFormat();
        return sdf.format(d);
    }

    private SimpleDateFormat getDateFormat(){
        return threadlocal.get();
    }
}

```

`formatDate`方法会从`ThreadLocal`中取出一个已经编译好的`SimpleDateFormat`，这样既保证了线程安全，又获取了高性能。

JDK8提供了线程安全的`DateTimeFormatter`，还可以这样做日期格式化：
```
DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy-MM-dd");
LocalDateTime now = LocalDateTime.now();
String str = format.format(now);
```

吞吐量`formatThreadLocal` > JDK8的`DateTimeFormatter` > 传统的`SimpleDateFormat`方式