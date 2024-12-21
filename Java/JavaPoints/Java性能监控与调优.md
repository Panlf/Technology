# Java性能监控与调优

## 基于JDK命令行工具的监控
### JVM参数类型
#### 标准参数

- -help
- -server -client
- -version -showversion
- -cp -classpath
#### X参数

- 非标准参数
- -Xint：解释执行
- -Xcomp：第一次使用就编译成本地代码
- -Xmixed：混合模式，JVM自己来决定是否编译成本地代码
#### XX参数

- Boolean类型

格式：-XX:[+-]<name>表示启用或者禁用name属性
比如：-XX:+UseConcMarkSweepGC
    -XX:+UseG1GC

- 非Boolean类型

格式：-XX:<name>=<value>表示name属性的值是value
比如：-XX:MaxGCPauseMillis=500
          XX:GCTimeRatio=19

-Xmx -Xms
不是X参数，而是XX参数
-Xms等价于-XX:InitialHeapSize
-Xmx等价于-XX:MaxHeapSize
### 查看JVM运行时参数

- -XX:+PrintFlagsInitial
- -XX:+PrintFlagsFinal
- -XX:+UnlockExperimentalVMOptions解锁实验参数
- -XX:+UnlockDiagnosticVMOptions解锁诊断参数
- -XX:+PrintCommandLineFlags打印命令行参数
#### PrintFlagsFinal
=表示默认值
:=被用户或者JVM修改后的值
#### jps

- jps -help
- jps -l
#### jinfo

- 查看最大内存 jinfo -flag MaxHeapSize [id]
- 查看垃圾回收器 jinfo -flag UseConcMarkSweepGC [id] |  jinfo -flag UseG1GC [id] | jinfo -flag UseParallelGC [id] 
### jstat查看JVM统计信息
#### 类装载
 jstat -class [id] 1000 10
每隔1s执行10次
#### 垃圾收集

- -gc、-gcutil、-gccause、-gcnew、-gcold

jstat -gc [id] 1000 3

-gc输出结果

- S0C、S1C、S0U、S1U：S0和S1的总量与使用量
- EC、EU：Eden区总量与使用量
- OC、OU：Old区总量与使用量
- MC、MU：Metaspace区总量与使用量
- CCSC、CCSU：压缩类空间总量与使用量
- YGC、YGCT：YoungGC的次数与时间
- FGC、FGCT：FullGC的次数与时间
- GCT：总的GC时间
#### JIT编译

- -compiler、-printcompilation
### 如何导出内存映像文件

- 内存溢出自动导出

-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=./

- 使用jmap命令手动导出

jmap -dump:format=b,file=heap.hprof [id]

### jstack与线程状态
Jstack是一个Java工具，用于生成Java虚拟机当前线程的堆栈跟踪信息。它可以用来分析Java程序的线程状态和调试死锁问题。
jstack [id]
线程状态

- new
- runnable
- blocked
- waiting
- timed_waiting
- terminated
## Tomcat优化
### 线程优化

- docs/config/http.html

maxConnections: The maximum number of connections that the server will accept and process at any given time 
acceptCount: The maximum queue length for incoming connection requests when all possible request processing threads are in use
maxThreads: 工作线程, The maximum number of request processing threads to be created by this Connector
minSpareThreads: 最小空闲的工作线程。The minimum number of threads always kept running
### 配置优化

- docs/config/host.html

autoDeploy: This flag value is indicates if Tomcat should check periodically for new or updated web applications while Tomcat is running

- docs/config/http.html  

enableLookups: false
### Session优化

- 如果是JSP，可以禁用Session
## Nginx监控优化
### Nginx监控

- ngx_http_stub_status监控连接信息
- ngxtop监控请求信息
- nginx-rrd图形化监控
### Nginx优化
配置线程数和并发数
```
# CPU
worker_process 4;
events{
	# 每一个进程打开的最大连接数，包含了Nginx与客户端和nginx与upstream之间的连接
	work_connections 10240;
	# 可以一次建立多个连接	
	multi_accept on;
  use epoll;
}
```
配置后端Server的长连接
```
upstream server_pool{
	server localhost:8080 weight=1 max_fails=2 fail_timeout-30s;
	server localhost:8081 weight=1 max_fails=2 fail_timeout-30s;
	# 300个长连接
	keepalive 300;
}

location / {
	proxy_http_version 1.1;
	proxy_set_header Upgrade $http_upgrade;
	proxy_set_header Connection "upgrade";
	proxy_pass http://server_pool/;
}
```
配置压缩
```
gzip on;
gzip_http_version 1.1;
gzip_disable "MSIE[1-6]\.(?!.*SV1)";
gzip_proxied any;
gzip_types text/plain text/css application/javascript application/x-javascript application/json application/xml application/vnd.ms-fontobject application/x-font-ttf application/svg+xml application/x-icon;
gzip_vary on; #Vary: Accept-Encoding
gzip_static on;#如果有压缩好的直接使用

```
操作系统优化
```
配置文件/etc/sysctl.conf
#防止一个套接字在有过多试图连接到达时引起过载
sysctl -w net.ipv4.tcp_syncookies=1
#默认128，连接队列
sysctl-w net.core.somaxconn=1024
#timewait的超时时间
sysctl-w net.ipv4.tcp_fin_timeout=10
#os直接使用timewait的连接
sysctl -w net.ipv4.tcp_tw_reuse=1 
#回收禁用
sysctl -w net.ipv4.tcp_tw_recycle = 0

```
其他优化
```
# 减少文件在应用和内核之间拷贝
sendfile on;
# 当数据包达到一定大小再发送
tcp_nopush on;
# 有数据随时发送
tcp_nodelay off;
```
## 垃圾回收算法

- 思想：枚举根节点，做可达性分析
- 根节点：类加载器、Thread、虚拟机栈的本地变量表、static成员、常量引用、本地方法栈的变量等等
### 标记清除
**算法**
算法分为“标记”和“清除”两个阶段：首先标记出所有需要回收的对象，在标记完成后统一回收所有。
**缺点**
效率不高。标记和清除两个过程的效率都不高。
产生碎片。碎片太多会导致提前GC。
### 复制
**算法**
它将可用内存按容量划分为大小相等的两块，每次只使用其中的一块。当这一块的内存用完了，就将还存活着的对象复制到另外一块上面，然后再把已使用过的内存空间一次清理掉。
**优缺点**
 实现简单，运行高效，但是空间利用率低。
### 标记整理
**算法**
标记过程仍然与“标记-清除”算法一样，但后续步骤不是直接对可回收对象进行清理，而是让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存
**优缺点**
没有了内存碎片，但是整理内存比较耗时。
### 分带垃圾回收

- Young区用复制算法
- Old区用标记清除或者标记整理
### 对象分配

- 对象优先在Eden区分配
- 大对象直接进入老年代：-XX:PretenureSizeThreshold
- 长期存活对象进入老年代：-XX:MaxTenuringThreshold、-XX:+PrintTenuringDistribution、-XX:TargetSurvivorRatio
## 垃圾收集器

- 串行收集器Serial：Serial、Serial Old
- 并行收集器Parallel：Parallel Scavenge、Parallel Old，吞吐量
- 并发收集器Concurrent：CMS、G1，停顿时间
### 并行VS并发

- 并行（Parallel）：指多条垃圾收集线程并行，但此时用户线程仍然处于等待状态。适合科学计算、后台处理等弱交互场景
- 并发（Concurrent）：指用户线程与垃圾收集线程同时执行（但不一定是并行的，可能会交替执行），垃圾收集线程在执行的时候不会停顿用户程序的运行。适合对响应时间有要求的场景，比如Web。
### 停顿时间VS吞吐量

- 停顿时间：垃圾收集器做垃圾回收中断应用执行的时间。-XX:MaxGCPauseMillis
- 吞吐量：花在垃圾收集的时间和花在应用时间的占比。-XX:GCTimeRatio=<n>，垃圾收集时间占：1/1+n
### 并行收集器

- 吞吐量优先
- -XX:+UseParallelGC、-XX:+UseParallelOldGC
- Server模式下的默认收集器
### 并发收集器

- 响应时间优先
- CMS：-XX:+UseConcMarkSweepGC、-XX:+UseParNewGC
- G1：-XX:+UseG1GC
### 如何选择垃圾收集器

- 优先调整堆的大小让服务器自己来选择
- 如果内存小于100M，使用串行收集器
- 如果是单核，并且没有停顿时间的要求，串行或者JVM自己选
- 如果允许停顿时间超过1秒，选择并行或者JVM自己选
- 如果响应时间最重要，并且不能超过1秒，使用并发收集器
### Parallel Collector

- -XX:+UseParallelGC 手动开启，Server默认开启
- -XX:ParallelGCThreads=<N>多少个GC线程
#### Parallel Collector Ergonomics

- -XX:MaxGCPauseMillis=<N>
- -XX:GCTimeRatio=<N>
- -Xmx<N>
#### 动态内存调整

- -XX:YoungGenerationSizelncrement=<Y>
- -XX:TenuredGenerationSizeIncrement=<T>
- -XX:AdaptiveSizeDecrementScaleFactor=<D>
### CMS
#### CMS垃圾收集过程

- CMS initial mark	初始标记Root，STW
- CMS concurrent mark	并发标记
- CMS-concurrent-preclean	并发预清理
#### CMS缺点

- CPU敏感
- 浮动垃圾
- 空间碎片
#### CMS的相关参数

- -XX:ConcGCThreads：并发的GC线程数
- -XX:+UseCMSCompactAtFullCollection： FullGC之后做压缩
- -XX:CMSFullGCsBeforeCompaction：多少次FullGC之后压缩一次
- -XX:CMSInitiatingOccupancyFraction：触发FullGC
- -XX:+UseCMSInitiatingOccupancyOnly：是否动态调
- -XX:+CMSScavengeBeforeRemark：FullGC之前先做YGC
- -XX:+CMSClassUnloadingEnabled：启用回收Perm区
### iCMS

- 适用于单核或者双核
### G1 Collector

- 新生代和老生代收集器
### YoungGC

- 新对象进入Eden区
- 存活对象拷贝到Survivor区
- 存活时间达到年龄阈值时，对象晋升到Old区
### MixedGC

- 不是FullGC，回收所有的Young和部分Old
- global concurrent marking
#### global concurrent marking

- Initial marking phase：标记GC Root，STW
- Root region scanning phase ：标记存活Region
- Concurrent marking phase：标记存活的对象
- Remark phase：重新标记，STW
- Cleanup phase ：部分STW
#### MixedGC时机

- InitiatingHeapOccupancyPercent :

堆占有率达到这个数值则触发global concurrent marking，默认45%

- G1HeapWastePercent :

在global concurrent marking结束之后，可以知道区有多少空间要被回收，在每次YGC之后和再次发生Mixed GC之前，会检查垃圾占比是否达到此参数，只有达到了，下次才会发生Mixed GC。
#### MixedGC相关参数

- G1MixedGCLiveThresholdPercent：Old区的region被回收时候的存活对象占比心
- G1MixedGCCountTarget：一次global concurrent marking之后，最多执行Mixed GC的次数
- G1OldCSetRegionThresholdPercent：一次Mixed GC中能被选入CSet的最多old区的region数量
#### 常用参数

- -XX:+UseG1GC开启G1
- -XX:G1HeapRegionSize=n，region的大小，1-32M，2048个
- -XX:MaxGCPauseMillis=200 最大停顿时间
- -XX:G1NewSizePercent、-XX:G1MaxNewSizePercent
- -XX:G1ReservePercent=10 保留防止to space溢出
- -XX:ParallelGCThreads=n SWT线程数
- -XX:ConcGCThreads=n 并发线程数=1/4*并行
#### 最佳实践

- 年轻代大小：避免使用-Xmn、-XX:NewRatio等显式设置Young区大小，会覆盖暂停时间目标。
- 暂停时间目标：暂停时间不要太严苛，其吞吐量目标是90%的应用程序时间和10%的垃圾回收时间，太严苛会直接影响到吞吐量
- 关于MixGC调优：-XX:InitiatingHeapOccupancyPercent、-XX:G1MixedGCLiveThresholdPercent、-XX:G1HeapWastePercent、-XX:G1MixedGCCountTarget、-XX:G1OldCSetRegionThresholdPercent
#### 是否需要切换到G1

- 50%以上的堆被存活对象占用
- 对象分配和晋升的速度变化非常大
- 垃圾回收时间特别长，超过了1秒
#### MixedGC时机

- InitiatingHeapOccupancyPercent：堆占有率达到这个数值则触发global concurrent marking，默认45%
- G1HeapWastePercent：在global concurrent marking结束之后，可以知道区有多少空间要被回收，在每次YGC之后和再次发生Mixed GC之前，会检查垃圾占比是否达到此参数，只有达到了，下次才会发生Mixed GC。
## GC日志
### 打印日志相关参数

- -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:$CATALINA_HOME/logs/gc.log -XX:+PrintHeapAtGC -XX:+PrintTenuringDistribution
## 常用代码优化方法

- 尽量重用对象，不要循环创建对象，比如：for循环字符串拼接
- 容器类初始化的时候指定长度
- ArrayList随机遍历快，LinkedList添加删除快
- 集合遍历尽量减少重复计算
- 使用Entry遍历Map
- 尽量使用基本类型而不是包装类型
- 不要手动调用System.gc()
- 及时消除过期对象的引用，防止内存泄露
- 尽量使用局部变量，减少变量的作用域
- 尽量使用非同步的容器ArrayList VS Vector
- 尽量减少同步作用范围，synchronized方法 VS 代码块
- ThreadLocal缓存线程不安全的对象，SimpleDateFormat
- 尽量使用延迟加载
- 尽量减少使用反射，加缓存
- 尽量使用连接池、线程池、对象池、缓存
- 及时释放资源，I/O流、Socket、数据库连接
- 慎用异常，不要用抛异常来表示正常的业务逻辑
- String操作尽量少用正则表达式
- 日志输出注意使用不同的级别
- 日志中参数拼接使用占位符
