# RPC特点

## 概述

RPC的主要功能目标是让构建分布式计算（应用）更容易，在提供强大的远程调用能力时不损失本地调用的语义简洁性。为实现该目标，RPC框架需提供一种透明调用机制，让使用者不必显式区分本地调用和远程调用。

## 优势

- 分布式计算
- 部署灵活
- 解耦服务
- 扩展性强

## RPC框架
- Dubbo
- Motan
- Tars
- Spring Cloud
- gRPC
- Thrift

## RPC框架优点
- RPC框架一般使用长链接，不必每次通信都要3次握手，减少网络开销
- RPC框架一般都有注册中心，有丰富的监控管理；发布、下线接口、动态扩展等，对调用方来说是无感知、统一化的操作；协议私密，安全性较高
- RPC协议更简单内容更小，效率更高，服务化架构、服务化治理，RPC框架是一个强大的支撑

## 应用场景

特征
- 长链接通讯
- 注册发布机制
- 安全性，没有暴露资源操作
- 微服务的支持

应用例举
- 分布式操作系统的进程间通讯
- 构造分布式设计的软件环境
- 远程数据库服务
- 分布式应用程序设计
- 分布式程序的调试

### 设计架构

- 客户端
  - 服务调用（Service Client）
    - 负载均衡
    - 容错补偿
    - 调用透明
  - RPC协议（Protocol）
    - 序列化
    - 协议编码
    - 网络传输
      
- 服务端
  - 服务暴露
  - 连接管理
  - 线程池
  - 业务处理（Processor）
  - RPC协议（Protocol）
    - 序列化
    - 协议编码
    - 网络传输
      
## 应用架构

- 接入层
  - 动态代理
  - 链路追踪
  - 过滤器
- 治理层
  - 服务发现
  - 路由管理
  - 连接管理
  - 负载均衡
  - 配置管理
- 协议层
  - 协议解析
  - 压缩/解压缩
  - 序列化/反序列化
- 传输层
  - TCP/IP传输
  - HTTP传输
    

将每个功能点抽象成一个接口，把这个功能的接口与功能的实现分离并提供接口的默认实现，这样的架构相比单一耦合性的架构具有明显的优势。

## 调用流程

1、服务消费者（client客户端）通过本地调用的方式调用服务。
2、客户端存根（client stub）接收到请求后负责将方法、入参等信息序列化（组装）成能够进行网络传输的消息体。
3、客户端存根（client stub）找到远程的服务地址，并且将消息通过网络发送给服务端。
4、服务端存根（server stub）收到消息后进行解码（反序列化操作）。
5、服务端存根（server stub）根据解码结果调用本地的服务进行相关处理。
6、本地服务执行具体业务逻辑并将处理结果返回给服务端存根（server stub）
7、服务端存根（server stub）将返回结果重新打包成消息（序列化）并通过网络发送至消费方。
8、客户端存根（client stub）接收到消息，并进行解码（反序列化）。
9、服务消费方得到结果。

所涉及的技术
- 1、动态代理
  
生产Client Stub（客户端存根）和Server Stub（服务端存根）的时候需要用到java动态代理技术
- 2、序列化
  
在网络中，所有的数据都将会被转化为字节进行传送，需要对这些参数进行序列化和反序列化操作。
目前主流高效的开源序列化框架Kryo、fastjson、Hessian、Protobuf等。
- 3、NIO通信

Java提供了NIO的解决方案，Java 7也提供了更优秀的NIO.2支持。可以采用Netty或者mina框架来解决NIO数据传输的问题。开源的RPC框架Dubbo就是采用NIO通信，集成支持netty、mina、grizzly。

- 4、服务注册中心
  
通过注册中心，让客户端连接调用服务端所发布的服务。主流的注册中心组件：Redis、Zookeeper、Consul、Etcd。Dubbo采用的是Zookeeper提供服务注册和发现功能。
- 5、负载均衡
  
在高并发的场景下，需要多个节点或集群来提升整体吞吐能力。

- 6、健康检查
  
健康检查包括，客户端心跳和服务端主动探测两种方式。

## 深入解析
### 序列化技术
序列化处理要素

- 解析效率：序列化协议应该首要考虑的因素，像xml/json解析起来比较耗时，需要解析doom树，二进制自定义协议解析起来效率要快很多。
- 压缩率：同样一个对象，xml/json传输起来有大量的标签冗余对象信息，信息有效性低，二进制自定义协议占用的空间相对来说会小很多。
- 扩展性与兼容性：是否能够利于信息的扩展，并且增加字段后旧版客户端是否需要强制升级，这都是需要考虑的问题，在自定义二进制协议时候，要做好充分考虑设计。
- 可读性和可调试性：xml/json的可读性会比二进制协议好很多，并且通过网络抓包是可以直接读取，二进制规则则需要反序列化才能查看其内容。
- 跨语言：有些序列化协议是与开发语言紧密相关的，例如dubbo的hessian序列化协议就只能支持Java的RPC调用
- 通用性：xml/json非常通用，都有很好的第三方解析库，各个语言解析起来都十分方便，二进制数据的处理方面也有Protobuf和Hessian等插件，在做设计的时候尽量做到较好的通用性。

常用的序列化技术
- JDK原生序列化
- JSON序列化
- Hessian2序列化
- Protobuf序列化

### 动态代理

JDK动态代理的实现原理

```
代理的创建
构建代理器 -> 绑定实例 -> 绑定接口 -> 生成代理实例

代理的调用
读取缓存 -> 生成字节码 -> 反序列化class -> new创建实例
```

JDK内部如何处理？

代理类`$Proxy`里面会定义相同签名的接口，然后内部会定义一个变量绑定JDKProxy代理对象，当调用User.job接口方法，实质上调用的是JDKProxy.invoke()方法。

为什么要加入动态代理

- 不便于管理，不利于扩展维护。
- 可以做到拦截，添加其他额外功能

### 服务注册发现
- 服务注册发现的作用

感知服务端的变化，获取最新服务节点的连接信息。

- 服务注册发现的处理流程

服务注册：服务提供方将对外暴露的接口发布到注册中心内，注册中心为了检测服务的有效状态，一般会建立双向心跳机制。

服务订阅：服务调用方去注册中心查找并订阅服务提供方的IP，并缓存到本地用于后续调用。

- 如何实现服务的注册发现

基于Zookeeper的服务发现方式

A.在Zookeeper中创建一个服务根路径，可以根据接口命名，在这个路径再创建服务提供方与调用方目录，分别用来存储服务提供方和调用方的节点信息。

B.服务端发起注册时，会在服务提供方目录中创建一个临时节点，节点中存储注册信息。

C.客户端发起订阅时，会在服务调用方目录中创建一个临时节点，节点中存储调用方的信息，同时watch服务提供方的目录中所有的服务节点数据。当服务端产生变化时ZK就会通知订阅的客户端。

### 健康监测

健康检测实现分析，心跳检测的过程总共包含以下状态 

- 健康状态
- 波动状态
- 失败状态

完善的解决方案
  
- 阈值： 健康检测增加失败阈值记录。
- 成功率： 可以再追加调用成功率的记录（成功次数/总次数）
- 探针：对服务节点有一个主动的存活检测机制。
  
### 网络IO模型

#### 网络IO模型种类

- 同步阻塞IO（BIO）
- 同步非阻塞IO（NIO）
- IO多路复用
- 信号驱动IO
- 异步非阻塞IO（AIO）

为什么阻塞IO和IO多路复用最为常用？

在实际的网络IO的应用中，需要的是系统内核的支持以及编程语言的支持。现在大多数系统内核都会支持阻塞IO、非阻塞IO和IO多路复用，但像信号驱动IO、异步IO，只有高版本的Linux系统内核才会支持。

#### RPC框架采用的网络IO模型
1）IO多路复用应用特点

IO多路复用更适合高并发的场景，可以用较少的进程（线程）处理较多的socket的IO请求，但使用难度比较高。

2）阻塞IO应用特点

与IO多路复用相比，阻塞IO每处理一个socket的IO请求都会阻塞进程（线程），但使用难度较低。

3）RPC框架应用

RPC调用在大多数情况下，是一个高并发调用的场景，在RPC框架的实现中，一般会选择IO多路复用的方式。

### 零拷贝

系统内核处理IO操作分为两个阶段：等待数据和拷贝数据。
> 等待数据，就是系统内核在等待网卡接收到数据后，把数据写到内核中。
> 拷贝数据，就是系统内核在获取到数据后，将数据拷贝到用户进程的空间中。

所谓的零拷贝，就是取消用户空间与内核空间之间的数据拷贝操作，应用进程每一次的读写操作，都可以通过一种方式，让应用进程向用户空间写入或者读取数据，就如同直接向内核空间写入或者读取数据一样，再通过DMA将内核中的数据拷贝到网卡，或将网卡中的数据copy到内核。

#### RPC框架中的零拷贝应用

Netty框架是否也有零拷贝机制？

Netty的零拷贝则有些不一样，他完全站在了用户空间上，也就是基于JVM之上。

Netty当中的零拷贝是如何实现的？

RPC并不会把请求参数作为一个整体数据包发送到对端机器上，中间可能会拆分，也可能会合并其他请求，所以消息都需要有边界。接收到消息之后，需要对数据包进行处理，根据边界对数据包进行分割和合并，最终获得完整的消息。

Netty零拷贝主要体现在三个方面：
  
- Netty的接收和发送ByteBuffer是采用DIRECT BUFFERS，使用堆外的直接内存（内存对象分配在JVM中堆以外的内存）进行Socket读写，不需要进行字节缓冲区的二次拷贝。如果采用传统堆内存（HEAP BUFFERS）进行Socket读写，JVM会将堆内存Buffer拷贝一份到直接内存中，然后写入Socket中。
- Netty提供了组合Buffer对象，也就是CompositeByteBuf类，可以将ByteBuf分解为多个共享同一个存储区域的ByteBuf，避免了内存的拷贝。
- Netty的文件传输采用了FileRegion中包装NIO的FileChannel.transferTo()方法，它可以直接将文件缓冲区的数据发送到目标Channel，避免了传统通过循环write方式导致的内存拷贝问题。

零拷贝带来的作用就是避免没必要的CPU拷贝，减少了CPU在用户空间与内核空间之间的上下文切换，从而提升了网络通信效率与应用程序的整体性能。

### 时间轮

在Dubbo中，为增强系统的容错能力，会有相应的监听判断处理机制。在Dubbo最开始的实现中，是将所有的返回结果（DefaultFuture）都放入集合中，并且通过定时任务扫描所有的future，逐个判断是否超时。但是这样效率底下，Dubbo借鉴Netty，引入了时间轮算法，减少无意义的轮询判断操作。

Dubbo中的时间轮原理是如何实现？

主要是通过Timer、Timeout、TimerTask几个接口定义了一个定时器的模型，再通过HashedWheelTimer这个类实现了一个时间轮定时器。通过该定时器，Dubbo实现了高效的任务调度。

#### 时间轮在RPC的应用

- 调用超时：在高并发、高访问量的情况，时钟轮每次只轮询一个时间槽位中的任务，这样会节省大量的CPU。
- 启动加载：比如服务启动完成之后要去加载缓存，执行定时任务等，都可以放在时钟轮里。
- 定时心跳检测：可以将心跳的逻辑封装为一个心跳任务，放到时钟轮里。

## 高级运用

### 基于Dubbo分布式高并发场景下的运用要点

#### 异步处理机制

为了提升性能，连接请求与业务处理不会放在一个线程处理，这个就是服务端的异步化。服务端业务处理逻辑加入异步处理机制。

> 在RPC框架提供一种回调方式，让业务逻辑可以异步处理，处理完之后调用RPC框架的回调接口。

RPC框架的异步策略主要是调用端异步与服务端异步。调用端的异步就是通过Future方式。

服务端异步则需要一种回调的方式，让业务逻辑可以异步处理。这样就实现了RPC调用的全异步化。

#### 路由与负载均衡
##### 为什么要采用路由
真实的环境中一般以集群的方式提供服务，对于服务调用方来说，一个接口会有多个服务提供方同时提供服务，所以RPC在每次发起请求的时候，都需要从多个服务节点里面选取一个用于处理请求的服务节点。这就需要在RPC应用中增加路由功能。

##### 如何实现路由
RPC路由策略

- IP路由
- 参数化路由

负载均衡策略

RPC的负载均衡完全由RPC框架自身实现，通过所配置的负载均衡组件，自主选择合适服务节点。这个就是自适应的负载均衡策略。

这就需要判定服务节点的处理能力。

主要步骤

- 1、添加计分器和指标采集器
- 2、指标采集器收集服务节点CPU核数、CPU负载以及内存占用率等指标。
- 3、可以配置开启哪些指标采集器，并设置这些参考指标的具体权重。
- 4、通过对服务节点的综合打分，最终计算出服务节点的实际权重，选择合适的服务节点。
  
#### 熔断限流

限流

- 计数器
- 平滑限流的滑动窗口
- 漏斗算法
- 令牌桶算法

熔断机制

- 平均响应时间
- 异常比例
- 异步数

#### 优雅启动

##### 什么是启动预热

启动预热就是让刚启动的服务，不直接承担全部的流量，而是让它随着时间的移动慢慢增加调用次数，最终让流量缓和运行一段时间后达到正常水平。

##### 如何实现

首先要知道服务提供方的启动时间，有两种获取方法：

- 一种是服务提供方在启动的时候，主动将启动的时间发送给注册中心。
- 另一种就是注册中心来检测，将服务提供方的请求注册时间作为启动时间。

调用方通过服务发现获取服务提供方的启动时间，然后进行降权，减少被负载均衡选择的概率，从而实现预热的过程。

#### 优雅关闭

##### 为什么需要优雅关闭

调用方存在以下情况：

- 目标服务已经下线
- 目标服务正在关闭中

##### 如何实现优雅关闭

关闭的流程

设置请求“挡板”，挡板的作用就是告诉调用方，服务提供方已经开始进入关闭流程了，不能再处理其他请求了。

具体处理流程：

当服务提供方正在关闭，可以直接返回一个特定的异常给调用方。然后调用方把这个节点从健康列表挪出，并把其他请求自动重试到其他节点。如需更为完善，可以再加上主动通知机制。

如何捕获关闭事件呢？

Java应用程序，在接收到结束信号时，会调用Runtime.addShutdownHook方法触发关闭钩子。

在Dubbo框架中，在以下场景中触发优雅关闭：
> JVM主动关闭 System.exit(int)
> JVM由于资源问题退出(OOM)
> 应用程序接受到进程正常结束信号：SIGTERM或SIGINT信号

优雅停机是默认开启的，停机等待时间为10秒。可以通过配置`dubbo.service.shutdown.wait`来修改等待时间。

Dubbo推出了多段关闭的方式来保证服务完全无损。

### RPC线上实战实验

#### 流量复制回放

- 流量复制的作用

最好的测试方法就是采用线上流量来验证，通过流量复制，能够有效进行测试验证，最大限度的保障了服务的稳定性。

- RPC应用中如何支持流量复制

常见的方案有很多种，比如基于TcpCopy的Dubbo服务引流、或者通过Nginx的Mirror模块来实现流量复制。

我们在RPC通讯当中，将每次请求的出入参数，以异步线程方式旁录下来，做持久化处理，实现流量的录制功能。

### 动态分组

- 为什么需要动态分组？

因为非核心业务的调用量突然增长，导致整个集群变得不可用。把整个大集群根据不同的调用方来划分出不同的小集群，从而实现调用方流量隔离的效果，保障业务之间不会互相影响。

- 如何分组？

可以按照以下原则进行分组：

> 按照应用的重要级别来划分，让非核心业务应用跟核心业务应用划分在不同分组内。按照解耦，分离式的原则去做。

- 如何动态分组？

通过修改注册中心的数据来解决这个问题：

> 把注册中心里面的部分实例的别名做修改，然后通过服务发现功能去调用不同服务提供方的实例集合。

### 保障调用安全

- 为什么需要保障安全？

RPC的调用先由服务提供方定义好一个接口，通过RPC提供的API进行调用，这里面其实存在一个安全隐患问题，只要拿到了Jar包，就可以通过私服的Jar引入到项目中完成RPC的调用，所以需要有一个安全机制，能够确保调用方的合法身份。

- 如何解决调用安全问题？

解决办法只需要给每个调用方设定一个唯一的身份，只有登记过的调用方才能继续放行，没有登记过的调用方一律拒绝。

- 实现方案

采用类似JWT的签名方案或者采用加密算法约定私钥，对请求数据进行加密处理，由调用双方完自身完全鉴权处理。

- 解决伪造服务方问题

建立一个白名单机制，将开放合法的服务方IP和端口纳入白名单中，有两种处理方案：
> 1、白名单可以存放在注册中心中。
> 2、由服务端开辟一个白名单专用检测线程，如有不合法节点，发出预警并将其剔除。