#  CompletableFuture 详解

## 概述

我们知道Future的有局限性，它没法直接对多个任务进行链式、组合等处理，需要借助并发工具类才能完成，实现逻辑比较复杂。

而CompletableFuture是对Future的扩展和增强。CompletableFuture实现了Future接口，并在此基础上进行了丰富的扩展，完美弥补了Future的局限性，同时CompletableFuture实现了对任务编排的能力。借助这项能力，可以轻松地组织不同任务的运行顺序、规则以及方式。

从某种程度上说，这项能力是它的核心能力。而在以往，虽然通过`CountDownLatch`等工具类也可以实现任务的编排，但需要复杂的逻辑处理，不仅耗费精力且难以维护。

`CompletionStage`接口定义了任务编排的方法，执行某一阶段，可以向下执行后续阶段。异步执行的，默认线程池是`ForkJoinPool.commonPool()`，但为了业务之间互不影响，且便于定位问题，强烈推荐使用自定义线程池。

`CompletableFuture`中默认线程池如下
```
// 根据commonPool的并行度来选择,而并行度的计算是在ForkJoinPool的静态代码段完成的
private static final boolean useCommonPool =
    (ForkJoinPool.getCommonPoolParallelism() > 1);

private static final Executor asyncPool = useCommonPool ?
    ForkJoinPool.commonPool() : new ThreadPerTaskExecutor();
```

`ForkJoinPool`中初始化`commonPool`的参数
```
static {
    // initialize field offsets for CAS etc
    try {
        U = sun.misc.Unsafe.getUnsafe();
        Class<?> k = ForkJoinPool.class;
        CTL = U.objectFieldOffset
            (k.getDeclaredField("ctl"));
        RUNSTATE = U.objectFieldOffset
            (k.getDeclaredField("runState"));
        STEALCOUNTER = U.objectFieldOffset
            (k.getDeclaredField("stealCounter"));
        Class<?> tk = Thread.class;
        ……
    } catch (Exception e) {
        throw new Error(e);
    }

    commonMaxSpares = DEFAULT_COMMON_MAX_SPARES;
    defaultForkJoinWorkerThreadFactory =
        new DefaultForkJoinWorkerThreadFactory();
    modifyThreadPermission = new RuntimePermission("modifyThread");

    // 调用makeCommonPool方法创建commonPool,其中并行度为逻辑核数-1
    common = java.security.AccessController.doPrivileged
        (new java.security.PrivilegedAction<ForkJoinPool>() {
            public ForkJoinPool run() { return makeCommonPool(); }});
    int par = common.config & SMASK; // report 1 even if threads disabled
    commonParallelism = par > 0 ? par : 1;
}
```
## 功能
### 常用方法

依赖关系

- `thenApply()`：把前面任务的执行结果，交给后面的Function
- `thenCompose()`：用来连接两个有依赖关系的任务，结果由第二个任务返回


and集合关系

- `thenCombine()`：合并任务，有返回值
- `thenAccepetBoth()`：两个任务执行完成后，将结果交给`thenAccepetBoth`处理，无返回值
- `runAfterBoth()`：两个任务都执行完成后，执行下一步操作(Runnable类型任务)

or聚合关系

- `applyToEither()`：两个任务哪个执行的快，就使用哪一个结果，有返回值
- `acceptEither()`：两个任务哪个执行的快，就消费哪一个结果，无返回值
- `runAfterEither()`：任意一个任务执行完成，进行下一步操作(Runnable类型任务)

并行执行

- `allOf()`：当所有给定的`CompletableFuture` 完成时，返回一个新的`CompletableFuture`
- `anyOf()`：当任何一个给定的`CompletablFuture`完成时，返回一个新的`CompletableFuture`


结果处理

`whenComplete`：当任务完成时，将使用结果(或 null)和此阶段的异常(或 null如果没有)执行给定操作
- `exceptionally`：返回一个新的`CompletableFuture`，当前面的`CompletableFuture`完成时，它也完成，当它异常完成时，给定函数的异常触发这个`CompletableFuture`的完成

### 异步操作
`CompletableFuture`提供了四个静态方法来创建一个异步操作：

```
public static CompletableFuture<Void> runAsync(Runnable runnable)

public static CompletableFuture<Void> runAsync(Runnable runnable, Executor executor)

public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier)

public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier, Executor executor)
```

这四个方法的区别：

- `runAsync()`以Runnable函数式接口类型为参数，没有返回结果，`supplyAsync()`以Supplier函数式接口类型为参数，返回结果类型为U；Supplier接口的`get()`是有返回值的(会阻塞)
- 使用没有指定Executor的方法时，内部使用`ForkJoinPool.commonPool()`作为它的线程池执行异步代码。如果指定线程池，则使用指定的线程池运行。
- 默认情况下`CompletableFuture`会使用公共的`ForkJoinPool`线程池，这个线程池默认创建的线程数是 CPU 的核数（也可以通过 `JVM option:-Djava.util.concurrent.ForkJoinPool.common.parallelism`来设置ForkJoinPool线程池的线程数）。如果所有`CompletableFuture`共享一个线程池，那么一旦有任务执行一些很慢的 I/O 操作，就会导致线程池中所有线程都阻塞在 I/O 操作上，从而造成线程饥饿，进而影响整个系统的性能。所以，强烈建议你要根据不同的业务类型创建不同的线程池，以避免互相干扰.

异步操作

```
Runnable runnable = () -> System.out.println("无返回结果异步任务");
CompletableFuture.runAsync(runnable);

CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    System.out.println("有返回值的异步任务");
    try {
        Thread.sleep(5000);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "Hello World";
});
String result = future.get();
```

获取结果(join&get)

`join()`和`get()`方法都是用来获取`CompletableFuture`异步之后的返回值。`join()`方法抛出的是uncheck异常（即未经检查的异常),不会强制开发者抛出。get()方法抛出的是经过检查的异常，`ExecutionException`, `InterruptedException` 需要用户手动处理（抛出或者 try catch）

结果处理

当`CompletableFuture`的计算结果完成，或者抛出异常的时候，我们可以执行特定的 Action。主要是下面的方法：
```
public CompletableFuture<T> whenComplete(BiConsumer<? super T,? super Throwable> action)

public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action)

public CompletableFuture<T> whenCompleteAsync(BiConsumer<? super T,? super Throwable> action, Executor executor)
```

- Action的类型是`BiConsumer<? super T,? super Throwable>`，它可以处理正常的计算结果，或者异常情况。
- 方法不以Async结尾，意味着Action使用相同的线程执行，而Async可能会使用其它的线程去执行(如果使用相同的线程池，也可能会被同一个线程选中执行)。
- 这几个方法都会返回`CompletableFuture`，当Action执行完毕后它的结果返回原始的`CompletableFuture`的计算结果或者返回异常

```
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(1);
    } catch (InterruptedException e) {
    }
    if (new Random().nextInt(10) % 2 == 0) {
        int i = 12 / 0;
    }
    System.out.println("执行结束！");
    return "test";
});
// 任务完成或异常方法完成时执行该方法
// 如果出现了异常,任务结果为null
future.whenComplete(new BiConsumer<String, Throwable>() {
    @Override
    public void accept(String t, Throwable action) {
        System.out.println(t+" 执行完成！");
    }
});
// 出现异常时先执行该方法
future.exceptionally(new Function<Throwable, String>() {
    @Override
    public String apply(Throwable t) {
        System.out.println("执行失败：" + t.getMessage());
        return "异常xxxx";
    }
});

future.get();
```

上面的代码当出现异常时，输出结果如下
```
执行失败：java.lang.ArithmeticException: / by zero
null 执行完成！
```

## 应用场景

### 结果转换

将上一段任务的执行结果作为下一阶段任务的入参参与重新计算，产生新的结果。

#### thenApply

`thenApply`接收一个函数作为参数，使用该函数处理上一个`CompletableFuture`调用的结果，并返回一个具有处理结果的`Future`对象。

常用使用
```
public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn)

public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn)
```

具体使用
```
CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> {
    int result = 100;
    System.out.println("第一次运算：" + result);
    return result;
}).thenApply(number -> {
    int result = number * 3;
    System.out.println("第二次运算：" + result);
    return result;
});
```

#### thenCompose

`thenCompose`的参数为一个返回`CompletableFuture`实例的函数，该函数的参数是先前计算步骤的结果。

常用方法：
```
public <U> CompletableFuture<U> thenCompose(Function<? super T, ? extends CompletionStage<U>> fn);

public <U> CompletableFuture<U> thenComposeAsync(Function<? super T, ? extends CompletionStage<U>> fn) ;
```

具体使用：
```
CompletableFuture<Integer> future = CompletableFuture
    .supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(30);
            System.out.println("第一次运算：" + number);
            return number;
        }
    })
    .thenCompose(new Function<Integer, CompletionStage<Integer>>() {
        @Override
        public CompletionStage<Integer> apply(Integer param) {
            return CompletableFuture.supplyAsync(new Supplier<Integer>() {
                @Override
                public Integer get() {
                    int number = param * 2;
                    System.out.println("第二次运算：" + number);
                    return number;
                }
            });
        }
    });
```

`thenApply`和`thenCompose`的区别：

- `thenApply`转换的是泛型中的类型，返回的是同一个`CompletableFuture`；
- `thenCompose`将内部的`CompletableFuture`调用展开来并使用上一个`CompletableFutre`调用的结果在下一步的`CompletableFuture`调用中进行运算，是生成一个新的`CompletableFuture`。

### 结果消费

与结果处理和结果转换系列函数返回一个新的`CompletableFuture`不同，结果消费系列函数只对结果执行`Action`，而不返回新的计算值。

根据对结果的处理方式，结果消费函数又可以分为下面三大类：

- `thenAccept()`：对单个结果进行消费
- `thenAcceptBoth()`：对两个结果进行消费
- `thenRun()`：不关心结果，只对结果执行Action

#### thenAccept

观察该系列函数的参数类型可知，它们是函数式接口Consumer，这个接口只有输入，没有返回值。

常用方法：
```
public CompletionStage<Void> thenAccept(Consumer<? super T> action);

public CompletionStage<Void> thenAcceptAsync(Consumer<? super T> action);
```

具体使用：
```
CompletableFuture<Void> future = CompletableFuture
    .supplyAsync(() -> {
        int number = new Random().nextInt(10);
        System.out.println("第一次运算：" + number);
        return number;
    }).thenAccept(number ->
                  System.out.println("第二次运算：" + number * 5));
```

#### thenAcceptBoth

`thenAcceptBoth`函数的作用是，当两个`CompletionStage`都正常完成计算的时候，就会执行提供的`action`消费两个异步的结果。

常用方法：
```
public <U> CompletionStage<Void> thenAcceptBoth(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);

public <U> CompletionStage<Void> thenAcceptBothAsync(CompletionStage<? extends U> other,BiConsumer<? super T, ? super U> action);
```

具体使用：
```
CompletableFuture<Integer> futrue1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
    @Override
    public Integer get() {
        int number = new Random().nextInt(3) + 1;
        try {
            TimeUnit.SECONDS.sleep(number);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("任务1结果：" + number);
        return number;
    }
});

CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
    @Override
    public Integer get() {
        int number = new Random().nextInt(3) + 1;
        try {
            TimeUnit.SECONDS.sleep(number);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("任务2结果：" + number);
        return number;
    }
});

futrue1.thenAcceptBoth(future2, new BiConsumer<Integer, Integer>() {
    @Override
    public void accept(Integer x, Integer y) {
        System.out.println("最终结果：" + (x + y));
    }
});
```

#### thenRun

`thenRun`也是对线程任务结果的一种消费函数，与`thenAccept`不同的是，`thenRun`会在上一阶段 `CompletableFuture`计算完成的时候执行一个`Runnable`，而`Runnable`并不使用该`CompletableFuture`计算的结果。

常用方法：
```
public CompletionStage<Void> thenRun(Runnable action);

public CompletionStage<Void> thenRunAsync(Runnable action);
```

具体使用：
```
CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> {
    int number = new Random().nextInt(10);
    System.out.println("第一阶段：" + number);
    return number;
}).thenRun(() ->
           System.out.println("thenRun 执行"));
```

### 结果组合

#### thenCombine

合并两个线程任务的结果，并进一步处理。

常用方法：
```
public <U,V> CompletableFuture<V> thenCombine(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);

public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn);

public <U,V> CompletableFuture<V> thenCombineAsync(CompletionStage<? extends U> other,BiFunction<? super T,? super U,? extends V> fn, Executor executor);
```

具体使用：
```
CompletableFuture<Integer> future1 = CompletableFuture
    .supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(10);
            System.out.println("任务1结果：" + number);
            return number;
        }
    });
CompletableFuture<Integer> future2 = CompletableFuture
    .supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(10);
            System.out.println("任务2结果：" + number);
            return number;
        }
    });
CompletableFuture<Integer> result = future1
    .thenCombine(future2, new BiFunction<Integer, Integer, Integer>() {
        @Override
        public Integer apply(Integer x, Integer y) {
            return x + y;
        }
    });
System.out.println("组合后结果：" + result.get());
```

### 任务交互

线程交互指将两个线程任务获取结果的速度相比较，按一定的规则进行下一步处理。

#### applyToEither

两个线程任务相比较，先获得执行结果的，就对该结果进行下一步的转化操作。

常用方法：
```
public <U> CompletionStage<U> applyToEither(CompletionStage<? extends T> other,Function<? super T, U> fn);

public <U> CompletionStage<U> applyToEitherAsync(CompletionStage<? extends T> other,Function<? super T, U> fn);
```

具体使用：
```
CompletableFuture<Integer> future1 = CompletableFuture
    .supplyAsync(new Supplier<Integer>() {
        @Override
        public Integer get() {
            int number = new Random().nextInt(10);
            try {
                TimeUnit.SECONDS.sleep(number);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("任务1结果:" + number);
            return number;
        }
    });
CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
    @Override
    public Integer get() {
        int number = new Random().nextInt(10);
        try {
            TimeUnit.SECONDS.sleep(number);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("任务2结果:" + number);
        return number;
    }
});

future1.applyToEither(future2, new Function<Integer, Integer>() {
    @Override
    public Integer apply(Integer number) {
        System.out.println("最快结果：" + number);
        return number * 2;
    }
});
```

#### acceptEither
两个线程任务相比较，先获得执行结果的，就对该结果进行下一步的消费操作。

常用方法：
```
public CompletionStage<Void> acceptEither(CompletionStage<? extends T> other,Consumer<? super T> action);

public CompletionStage<Void> acceptEitherAsync(CompletionStage<? extends T> other,Consumer<? super T> action);
```

具体使用：
```
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
    @Override
    public Integer get() {
        int number = new Random().nextInt(10) + 1;
        try {
            TimeUnit.SECONDS.sleep(number);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("第一阶段：" + number);
        return number;
    }
});

CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
    @Override
    public Integer get() {
        int number = new Random().nextInt(10) + 1;
        try {
            TimeUnit.SECONDS.sleep(number);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("第二阶段：" + number);
        return number;
    }
});

future1.acceptEither(future2, new Consumer<Integer>() {
    @Override
    public void accept(Integer number) {
        System.out.println("最快结果：" + number);
    }
});
```

#### runAfterEither

两个线程任务相比较，有任何一个执行完成，就进行下一步操作，不关心运行结果。

常用方法：
```
public CompletionStage<Void> runAfterEither(CompletionStage<?> other,Runnable action);

public CompletionStage<Void> runAfterEitherAsync(CompletionStage<?> other,Runnable action);
```

具体使用：

```
CompletableFuture<Integer> future1 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
    @Override
    public Integer get() {
        int number = new Random().nextInt(5);
        try {
            TimeUnit.SECONDS.sleep(number);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("任务1结果：" + number);
        return number;
    }
});

CompletableFuture<Integer> future2 = CompletableFuture.supplyAsync(new Supplier<Integer>() {
    @Override
    public Integer get() {
        int number = new Random().nextInt(5);
        try {
            TimeUnit.SECONDS.sleep(number);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("任务2结果:" + number);
        return number;
    }
});

future1.runAfterEither(future2, new Runnable() {
    @Override
    public void run() {
        System.out.println("已经有一个任务完成了");
    }
}).join();
```

#### anyOf

`anyOf()`的参数是多个给定的 `CompletableFuture`，当其中的任何一个完成时，方法返回这个`CompletableFuture`。

常用方法：
```
public static CompletableFuture<Object> anyOf(CompletableFuture<?>... cfs)
```

具体使用：
```
Random random = new Random();
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(random.nextInt(5));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "hello";
});

CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(random.nextInt(1));
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return "world";
});
CompletableFuture<Object> result = CompletableFuture.anyOf(future1, future2);
```

#### allOf

allOf方法用来实现多`CompletableFuture`的同时返回。

常用方法：
```
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> {
    try {
        TimeUnit.SECONDS.sleep(2);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    System.out.println("future1完成！");
    return "future1完成！";
});

CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> {
    System.out.println("future2完成！");
    return "future2完成！";
});

CompletableFuture<Void> combindFuture = CompletableFuture.allOf(future1, future2);

try {
    combindFuture.get();
} catch (InterruptedException e) {
    e.printStackTrace();
} catch (ExecutionException e) {
    e.printStackTrace();
}
```