## Java反射性能分析及优化

### 性能差的原因

性能差是相对的，相对于直接调用，直接调用的时候，是静态的，编译阶段编译器会做权限，可见性，参数等检验，加载阶段解析的时候，就会方法对应的符号引用转换为地址引用，到执行方法调用时，就可以直接新建栈帧进行方法调用了。但是反射调用的过程中，是动态的，在执行的时候才明确下来，所以存在一些验证以及一些安全机制的考虑，另外就是因为是动态的，所以可能存在一些JVM无法优化的因素。

### 原因总结

#### 获取Method对象慢

- 需要检查方法权限
- 需要遍历筛选递归
- 每一个Method都有一个root，不暴露给外部，而每次copy一个Method

#### 调用invoke方法慢

- Method#invoke方法会对参数做封装和解封操作
- 需要检查方法可见性
- 需要校验参数
- invoke调用逻辑是委托给MethodAccessor的，而accessor对象会在第一次invoke的时候才创建，是一种lazy init方式
- 反射方法难以内联，内联：把函数调用的方法直接内嵌到方法内部，减少函数调用的次数。native版的反射调用则无法被有效内联，因而调用开销无法随程序的运行而降低。
- JIT无法优化

### 分析获取Method对象

```java
@CallerSensitive
    public Method getMethod(String name, Class<?>... parameterTypes)
        throws NoSuchMethodException, SecurityException {
        Objects.requireNonNull(name);
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            //1. 检查方法权限
            checkMemberAccess(sm, Member.PUBLIC, Reflection.getCallerClass(), true);
        }
        //2. 获取方法
        Method method = getMethod0(name, parameterTypes);
        if (method == null) {
            throw new NoSuchMethodException(methodToString(name, parameterTypes));
        }
        //3. 返回方法的拷贝
        return getReflectionFactory().copyMethod(method);
    }

@CallerSensitive
    public Method getDeclaredMethod(String name, Class<?>... parameterTypes)
        throws NoSuchMethodException, SecurityException {
        Objects.requireNonNull(name);
        SecurityManager sm = System.getSecurityManager();
        if (sm != null) {
            //1. 检查方法权限
            checkMemberAccess(sm, Member.DECLARED, Reflection.getCallerClass(), true);
        }
        //2. 获取方法
        Method method = searchMethods(privateGetDeclaredMethods(false), name, parameterTypes);
        if (method == null) {
            throw new NoSuchMethodException(methodToString(name, parameterTypes));
        }
        //3. 返回方法的拷贝
        return getReflectionFactory().copyMethod(method);
    }
```

`getMethod`和`getDeclaredMethod`获取Method对象的过程大体差不多，都是

1、检查方法的权限

2、获取方法Method对象

3、返回方法的拷贝

这里有两个区别

1、getMethod中checkMemberAccess传入的是Member.PUBLIC，而getDeclaredMethod传入的Member.DECLARED

PUBLIC会包括所有的public方法，包括父类的方法，而DECLARED包括所有自己定义的方法，public 、 protected、private都在此，但是不包括父类的方法

2、getMethod中获取方法调用的是getMethod0，而getDeclaredMethod获取方法调用的是searchMethods

#### 检查方法权限checkMemberAccess方法

如果不允许调用线程访问成员，则该方法抛出SecurityException。默认策略是允许访问PUBLIC成员，以及访问与调用者具有相同类加载器的类。在所有其他情况下，此方法使用RuntimePermission("accessDeclaredMembers")权限调用checkPermission

方法整体是在检查是否可以访问对象成员。

#### 获取方法getMethod0方法

```java
private Method getMethod0(String name, Class<?>[] parameterTypes) {
        PublicMethods.MethodList res = getMethodsRecursive(
            name,
            parameterTypes == null ? EMPTY_CLASS_ARRAY : parameterTypes,
            /* includeStatic */ true);
        return res == null ? null : res.getMostSpecific();
    }
```

这是通过getMethodsRecursive获取到MethodList对象，然后通过getMostSpecificgetMostSpecific方法筛选出对应的方法。

getMostSpecificgetMostSpecific会筛选出返回值类型最为具体的方法。

```java
// Returns a list of "root" Method objects. These Method objects must NOT
    // be propagated to the outside world, but must instead be copied
    // via ReflectionFactory.copyMethod.
    private PublicMethods.MethodList getMethodsRecursive(String name,
                                                         Class<?>[] parameterTypes,
                                                         boolean includeStatic) {
        
        //1.获取自己声明的public方法
        // 1st check declared public methods
        Method[] methods = privateGetDeclaredMethods(/* publicOnly */ true);
        //2.筛选符合条件的方法，根据方法名，参数类筛选，包括静态方法，构造MethodList对象
        PublicMethods.MethodList res = PublicMethods.MethodList
            .filter(methods, name, parameterTypes, includeStatic);
        // if there is at least one match among declared methods, we need not
        // search any further as such match surely overrides matching methods
        // declared in superclass(es) or interface(s).
        //3. 如果找到方法不为空，直接返回
        if (res != null) {
            return res;
        }

        // if there was no match among declared methods,
        // we must consult the superclass (if any) recursively...
        Class<?> sc = getSuperclass();
        if (sc != null) {
            //4. 没有找到方法，就获取其父类，递归调用getMethodsRecursive方法
            res = sc.getMethodsRecursive(name, parameterTypes, includeStatic);
        }

        // ...and coalesce the superclass methods with methods obtained
        // from directly implemented interfaces excluding static methods...
        //5. 遍历接口，获取接口中对应的方法并合并到methodList
        for (Class<?> intf : getInterfaces(/* cloneArray */ false)) {
            res = PublicMethods.MethodList.merge(
                res, intf.getMethodsRecursive(name, parameterTypes,
                                              /* includeStatic */ false));
        }

        return res;
    }


// Returns an array of "root" methods. These Method objects must NOT
    // be propagated to the outside world, but must instead be copied
    // via ReflectionFactory.copyMethod.
    private Method[] privateGetDeclaredMethods(boolean publicOnly) {
        Method[] res;
        //1. 通过缓存获取Method[]
        ReflectionData<T> rd = reflectionData();
        if (rd != null) {
            res = publicOnly ? rd.declaredPublicMethods : rd.declaredMethods;
            if (res != null) return res;
        }
        // No cached value available; request value from VM
        //2. 没有缓存，则通过JVM获取（getDeclaredMethods0是native方法，通过JVM实现）
        res = Reflection.filterMethods(this, getDeclaredMethods0(publicOnly));
        if (rd != null) {
            if (publicOnly) {
                rd.declaredPublicMethods = res;
            } else {
                rd.declaredMethods = res;
            }
        }
        return res;
    }
```

#### 返回方法的拷贝

```java
/**
     * Package-private routine (exposed to java.lang.Class via
     * ReflectAccess) which returns a copy of this Method. The copy's
     * "root" field points to this Method.
     */
    Method copy() {
        // This routine enables sharing of MethodAccessor objects
        // among Method objects which refer to the same underlying
        // method in the VM. (All of this contortion is only necessary
        // because of the "accessibility" bit in AccessibleObject,
        // which implicitly requires that new java.lang.reflect
        // objects be fabricated for each reflective call on Class
        // objects.)
        if (this.root != null)
            throw new IllegalArgumentException("Can not copy a non-root Method");

        Method res = new Method(clazz, name, parameterTypes, returnType,
                                exceptionTypes, modifiers, slot, signature,
                                annotations, parameterAnnotations, annotationDefault);
        res.root = this;
        // Might as well eagerly propagate this if already present
        //如果methodAccessor对象已经存在的话，那就设置，如果是从缓存里面获取的Method对象，则有值，如果是从JVM获取则为空
        res.methodAccessor = methodAccessor;
        return res;
    }
```

new 一个Method实例并返回。这里有两点要注意

1、设置root = this

2、会给Method设置MethodAccessor，用于后面的方法调用。也就是所有的Method的拷贝都会使用同一份methodAccessor

### 分析调用invoke方法

```java
 @CallerSensitive
    @ForceInline // to ensure Reflection.getCallerClass optimization
    @HotSpotIntrinsicCandidate
    public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException
    {
        if (!override) {
            Class<?> caller = Reflection.getCallerClass();
            //1.检查权限
            checkAccess(caller, clazz,
                        Modifier.isStatic(modifiers) ? null : obj.getClass(),
                        modifiers);
        }
         //2. 获取MethodAccessor
        MethodAccessor ma = methodAccessor;             // read volatile
        if (ma == null) {
            //创建 MethodAccessor
            ma = acquireMethodAccessor();
        }
         // 3. 调用MethodAccessor.invoke
        return ma.invoke(obj, args);
    }
```



#### 检查权限checkAccess

这里对override变量进行判断，如果override == true，就跳过检查 我们通常在Method#invoke之前，会调用Method#setAccessible(true)，就是设置override = true

#### 获取MethodAccessor或创建MethodAccessor

```java
 // NOTE that there is no synchronization used here. It is correct
    // (though not efficient) to generate more than one MethodAccessor
    // for a given Method. However, avoiding synchronization will
    // probably make the implementation more scalable.
    private MethodAccessor acquireMethodAccessor() {
        // First check to see if one has been created yet, and take it
        // if so
        MethodAccessor tmp = null;
        if (root != null) tmp = root.getMethodAccessor();
        if (tmp != null) {
            methodAccessor = tmp;
        } else {
            // Otherwise fabricate one and propagate it up to the root
            tmp = reflectionFactory.newMethodAccessor(this);
            setMethodAccessor(tmp);
        }

        return tmp;
    }

public MethodAccessor newMethodAccessor(Method method) {
        checkInitted();

        if (Reflection.isCallerSensitive(method)) {
            Method altMethod = findMethodForReflection(method);
            if (altMethod != null) {
                method = altMethod;
            }
        }

        // use the root Method that will not cache caller class
        Method root = langReflectAccess.getRoot(method);
        if (root != null) {
            method = root;
        }

        if (noInflation && !ReflectUtil.isVMAnonymousClass(method.getDeclaringClass())) {
            return new MethodAccessorGenerator().
                generateMethod(method.getDeclaringClass(),
                               method.getName(),
                               method.getParameterTypes(),
                               method.getReturnType(),
                               method.getExceptionTypes(),
                               method.getModifiers());
        } else {
            NativeMethodAccessorImpl acc =
                new NativeMethodAccessorImpl(method);
            DelegatingMethodAccessorImpl res =
                new DelegatingMethodAccessorImpl(acc);
            acc.setParent(res);
            return res;
        }
    }
```

#### 调用MethodAccessor.invoke方法

在生成MethodAccessor以后，就调用其invoke方法进行最终的放射调用。

### Java反射性能优化方案

#### 验证NativeMethodAccessorImpl及Java版的MethodAccessor

前面15次调用使用NativeMethodAccessorImpl，到第16次，超过阈值，调用MethodAccessorGenerator.generateMethod()来生成Java版的MethodAccessor的实现类，并且改变DelegatingMethodAccessorImpl所引用的MethodAccessor为Java版。

#### 缓存Method

```java
public void reflectCall1() throws Exception {
        Class<?> clazz = App.class;
        Object obj = clazz.getDeclaredConstructor().newInstance();
        Method method = clazz.getMethod("run",int.class);
        long start = System.nanoTime();
        for(int i = 0;i<10000;i++){
            method.invoke(obj,i);
        }
        //6414300ns
        System.out.println((System.nanoTime()-start)+"ns");
    }
```

#### 设置检查方法的可见性为true

```java
public void reflectCall2() throws Exception {
        Class<?> clazz = App.class;
        Object obj = clazz.getDeclaredConstructor().newInstance();
        Method method = clazz.getMethod("run",int.class);
        method.setAccessible(true);
        long start = System.nanoTime();
        for(int i = 0;i<10000;i++){
            method.invoke(obj,i);
        }
        System.out.println((System.nanoTime()-start)+"ns");
    }
```

#### 如果可以缓存Method的情况，提前设置好MethodAccessor为Java版MethodAccessor

#### 如果可以缓存Method的情况，绕过Method，直接使用Java版MethodAccessor

#### 使用AsmReflect包

### 结论

1、如果反射调用场景很少，则不需要太过纠结，直接反射调用就行

2、如果对性能要求较高，且无法缓存Method对象的情况下，尽量选择AsmReflect来进行反射调用。如果可以缓存，则也可以考虑使用Java版MethodAccessor，与AsmReflect差异并不是太大。
