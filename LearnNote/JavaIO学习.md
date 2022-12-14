# JavaIO

## Java的I/O演进之路

### I/O模型基本说明

I/O模型：就是用什么样的通道或者说是通信模式和架构进行数据的传输和接收，很大程度上决定了程序通信的性能，Java共支持3种网络编程的I/O模型：BIO、NIO、AIO

实际通信需求下，要根据不同的业务场景和性能需求决定选择不同的I/O模型。

### I/O模型

#### Java BIO

同步并阻塞（传统阻塞模型），服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销。

#### Java NIO

同步非阻塞，服务器实现模式为一个线程处理多个请求（连接），即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求就进行处理。

#### Java AIO

NIO.2  异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理，一般适用于连接数较多且连接时间较长的应用。

### 适用场景分析

- BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序简单易理解。
- NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器、弹幕系统，服务器间通讯等。
- AIO方式适用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK开始支持

## Java BIO深入剖析

### 基本介绍

- Java BIO就是传统的Java IO编程，其相关的类和接口在java.io
- BIO（blocking I/O）同步阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，可以通过线程池机制改善（实现多个客户连接服务器）

### 工作机制

Client客户端

1、Socket对象请求与服务的连接

2、从Socket管道中获取输入\输出流读写数据

Server服务端

1、ServerSocket注册端口

2、调用accept方法监听客户端Socket连接请求

3、从Socket管道中获取输入\输出流读写数据

## Java NIO深入剖析

### 基本介绍

- Java NIO （new IO）也有人称之为java non-blocking IO是从java1.4版本开始引入的一个新的IO API，可以替代标准的Java IO API。NIO与原来的IO有同样的作用和目的，但是使用的方式完全不同，NIO支持面向缓冲区的、基于通道的IO操作。NIO将以更加高效的方式进行文件的读写操作。NIO可以理解为非阻塞IO，传统的IO的read和write只能阻塞执行，线程在读写IO期间不能干其他事情，比如调用socket.read()时，如果服务器一直没有数据传输过来，线程就一直阻塞，而NIO中可以配置socket为非阻塞模式。
- NIO相关类都被放在java.nio包及子包下，并且对原java.io包中的很多类进行改写
- NIO有三大核心部分：Channel（通道）、Buffer（缓冲区）、Selector（选择器）
- Java NIO的非阻塞模式，使一个线程从某通道发送请求或者读取数据，但是它仅能得到目前可用的数据，如果目前没有数据可用时，就什么都不会获取，而不是保持线程阻塞，所以直至数据变的可以读取之前，该线程可以继续做其他事情。非阻塞写也是如此，一个线程请求写入一些数据到某通道，但不需要等待它完全写入，这个线程同时可以去做别的事情
- NIO是可以做到用一个线程来处理多个操作的。假设有1000个请求过来，根据实际情况，可以分配20或者80个线程来处理。不像之前的阻塞IO那样，非得分配1000个。

### NIO和BIO比较

- BIO以流的方式处理数据，而NIO以块的方式处理数据，块I/O的效率比流I/O高很多
- BIO是阻塞的，NIO则是非阻塞的
- BIO基于字节流和字符流进行操作，而NIO基于Channel和Buffer进行操作，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector用于监听多个通道的事件（比如：连接请求，数据到达等），因此使用单个线程就可以监听多个客户端通道

**Buffer缓冲区**

缓冲区本质上是一块可以写入数据，然后可以从中读取数据的内存。这块内存被包装成NIO Buffer对象，并提供了一组方法，用来方便的访问该块内存。相比较直接对数组的操作，Buffer API更加容易操作和管理。

**Channel通道**

Java NIO的通道类似流，但又有些不同：既可以从通道中读取数据，又可以写数据到通道。但流的（input或output）读写通常是单向的。通道可以非阻塞读取和写入通道，通道可以支持读取或写入缓冲区，也支持异步地读写。

**Selector选择器**

Selector是一个Java NIO组件，可以能够检查一个或多个NIO通道，并确定哪些通道已经准备好进行读取或写入。这样，一个单独的线程可以管理多个Channel，从而管理多个网络连接，提高效率。



- 每个channel都会对应一个Buffer
- 一个线程对应Selector，一个Selector对应多个channel（连接）
- 程序切换到哪个channel是由事件决定的
- Selector会根据不同的事件，在各个通道上切换
- Buffer就是一个内存块，底层是一个数组
- 数据的读取写入是通过Buffer完成的，BIO中要么是输入流，或者是输出流，不能双向，但是NIO的Buffer是可以读也可以写。
- Java NIO系统的核心在于：通道（Channel）和缓冲区（Buffer）。通道表示打开到IO设备（例如：文件、套接字）的连接。若需要使用NIO系统，需要获取用于连接IO设备的通道以及用于容纳数据的缓冲区。然后操作缓冲区，对数据进行处理。简而言之，Channel负责传输，Buffer负责存取数据。

### 缓冲区

一个用于特定基本数据类型的容器。由java.nio包定义的，所有缓冲区都是Buffer抽象类的子类。Java NIO中的Buffer主要用于与NIO通道进行交互，数据是从通道读入缓冲区，从缓冲区写入通道中

#### Buffer类及其子类

Buffer就像是一个数组，可以保存多个相同类型的数据。根据类型的不同

- ByteBuffer
- CharBuffer
- ShortBuffer
- IntBuffer
- LongBuffer
- FloatBuffer
- DoubleBuffer

#### 缓冲区的基本属性

- 容量（capacity） 作为一个内存块，Buffer具有一定的固定大小，也称为容量，缓冲区容量不能为负，并且创建后不能更改
- 限制（limit）表示缓冲区中可以操作数据的大小（limit后数据不能进行读写）。缓冲区的限制不能为负，并且不能大于其容量。写入模式，限制等于buffer的容量。读取模式下，limit等于写入的数据量
- 位置（position）下一个要读取或写入的数据的索引。缓冲区的位置不能为负，并且不能大于其限制
- 标记（mark）与重置（reset）标记是一个索引，通过Buffer中的mark()方法指定Buffer中一个特定的position，之后可以通过调用reset()方法恢复到这个position
- 标记、位置、限制、容量遵守以下不变式：0  <= mark <= position <= limit <= capacity

使用Buffer读写数据一般遵循以下四个步骤

- 1、写入数据到Buffer
- 2、调用flip()方法，转换为读取模式
- 3、从Buffer中读取数据
- 4、调用buffer.clear()方法或者buffer.compact()方法清除缓冲区

#### 直接与非直接缓冲区

byte buffer可以是两种类型，一种是基于直接内存（也就是非堆内存）；另一种是非直接内存（也就是堆内存）。对于直接内存来说，JVM将会在IO操作具有更高的性能，因为它直接作用于本地系统的IO操作。而非直接内存，也就是堆内存中的数据，如果要做IO操作，会先从本进程内存复制到直接内存，再利用本地IO处理。

从数据流的角度，非直接内存是下面这样的作用链

```
本地IO - 直接内存 - 非直接内存 - 直接内存 - 本地IO
```

而直接内存

```
本地IO - 直接内存 - 本地IO
```

很明显，在做IO处理时，比如网络发送大量数据时，直接内存会具有更高的效率。直接内存使用allocateDirect创建，但是它比申请普通的堆内存需要耗费更高的性能。不过，这部分的数据是在JVM之外的，因此它不会占用应用的内存。所以呢，当你有很大的数据要缓存，并且它的生命周期又很长，那么就比较适合使用直接内存。只是一般来说，如果不是能带来很明显的性能提升，还是推荐直接使用堆内存。字节缓冲区是直接缓冲区还是非直接缓冲区可通过调用其isDirect()方法来确定。

使用场景

- 有很大的数据需要存储，它的生命周期又很长
- 适合频繁的IO操作，比如网络并发场景

### 通道

	#### 概述

通道（Channel），由java.nio.channels包定义的，Channel表示IO源与目标打开的连接。Channel类似于传统的“流”。只不过Channel本身不能直接访问数据，Channel只能与Buffer进行交互。

1、NIO的通道类似于流，但有些区别如下：

- 通道可以同时进行读写，而流只能读或者只能写
- 通道可以实现异步读写数据
- 通道可以从缓冲读数据，也可以写数据到缓冲

2、BIO中的stream是单向的，例如FileInputStream对象只能进行读取数据的操作，而NIO中的通道（Channel）是双向的，可以读操作，也可以写操作。

3、Channel在NIO中是一个接口

```
public interface Channel extends Closeable{}
```

#### 常用的Channel实现类

- FileChannel 用于读取、写入、映射和操作文件的通道
- DatagramChannel 通过UDP读写网络中的数据通道
- SocketChannel 通过TCP读写网络中的数据
- ServerSocketChannel 可以监听新进来的TCP连接，对每一个新进来的连接都会创建一个SocketChannel。【ServerSocketChannel类似ServerSocket，SocketChannel类似Socket】

### 选择器

#### 概述

选择器（Selector）是SelectableChannel对象的多路复用器，Selector可以同时监控多个SelectableChannel的IO状况，也就是说，利用Selector可使一个单独的线程管理多个Channel。Selector是非阻塞IO的核心

- Java的NIO，用非阻塞的IO方式。可以用一个线程，处理多个的客户端连接，就会使用到Selector（选择器）
- Selector能够检测多个注册的通道上是否有事件发生（注意：多个Channel以事件的方式可以注册到同一个Selector），如果有事件发生，便获取事件然后针对每个事件进行相应的处理。这样就可以只用一个单线程去管理多个通道，也就是管理多个通道，也就是管理多个连接和请求。
-  只有在连接/通道 真正有读写事件发生时，才会进行读写，就大大地减少了系统的开销，并且不必为每个连接都创建一个线程，不用去维护多个线程
- 避免了多线程之间的上下文切换导致的开销

## Java AIO深入剖析

Java AIO（NIO.2） 异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理。

与NIO不同，当进行读写操作时，只须直接调用API的read或write方法即可，这两种方法均为异步的，对于读操作而言，当有流可读取时，操作系统会将可读的流传入read方法的缓冲区，对于写操作而言，当操作系统将write方法传递的流写入完毕时，操作系统主动通知应用程序

即可以理解为，read/write方法都是异步的，完成后会主动调用回调函数。在JDK1.7中，这部分内容被称为NIO.2。	

## 总结

**BIO、NIO、AIO**

- Java BIO 同步并阻塞，服务器实现模式为一个连接一个线程，即客户端有连接请求时服务器端就需要启动一个线程进行处理，如果这个连接不做任何事情会造成不必要的线程开销，当然可以通过线程池机制改善。	 
- Java NIO 同步非阻塞，服务器实现模式为一个请求一个线程，即客户端发送的连接请求都会注册到多路复用器上，多路复用器轮询到连接有I/O请求时才启动一个线程进行处理。用户进程也需要时不时的轮询IO操作是否就绪，这就要求用户进程不停的去询问。
- Java AIO（NIO.2）异步非阻塞，服务器实现模式为一个有效请求一个线程，客户端的I/O请求都是由OS先完成了再通知服务器应用去启动线程进行处理。用户进程只需要对数据进行处理就好了，不需要进行实际的IO读写操作，因为真正的IO读取或者写入操作已经由OS完成了。

**BIO、NIO、AIO适用场景分析**

- BIO方式适用于连接数目比较小且固定的架构，这种方式对服务器资源要求比较高，并发局限于应用中，JDK1.4以前的唯一选择，但程序直观简单易理解。
- NIO方式适用于连接数目多且连接比较短（轻操作）的架构，比如聊天服务器，并发局限于应用中，编程比较复杂，JDK1.4开始支持
- AIO方式适用于连接数目多且连接比较长（重操作）的架构，比如相册服务器，充分调用OS参与并发操作，编程比较复杂，JDK7开始支持。

