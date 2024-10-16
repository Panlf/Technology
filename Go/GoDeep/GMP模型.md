# GMP模型

GMP = foroutine + machine + processor (+ 一套有机组合的机制)

- G：goroutine协程
- P：processor处理器
- M：thread线程
- 全局队列：存放等待运行的G
- P的本地的队列：存放等待运行的G；优先将新创建的G放在P的本地队列中，如果满了会放在全局队列中
- P列表：程序启动时创建；最多有GOMAXPROCS个（可配置）
- M列表：当前操作系统分配到当前Go程序的内核线程数
- P和M的数量：
    - P的数量问题：环境变量$GOMAXPROCS；在程序中通过runtime.GOMAXPROCS()来设置
    - M的数量问题：Go语言本身是限定M的最大量是10000（忽略）；runtime/debug包中的SetMaxThreads函数来设置；有一个M阻塞，会创建一个新的M；如果有M空闲，那么就会回收或者睡眠

## 多线程/进程问题
### 设计复杂
- 进程/线程的数量越多，切换成本就越大，也就越浪费。
- 多线程随着同步竞争（如锁、竞争资源冲突等）

### 多进程、线程壁垒
- 高内存占用
- 高CPU调度消耗

## 协程引发问题
### N:1
- 无法利用多个CPU
- 出现阻塞的瓶颈

### 1:1
- 跟多线程/多进程模型无异
- 切换协程成本代价昂贵

### M:N
- 能够利用多核
- 过于依赖协程调度器的优化和算法

## 调度器的优化

### Goroutine的优化

- 内存占用
- 灵活调度，切换成本低

### 早期的GO的调度器

基本的全局Go队列和比较传统的轮询利用多个Thread去调度

弊端
- 1、创建、销毁、调度G都需要每个M获取锁，这就形成了激烈的锁竞争
- 2、M转移G会造成延迟和额外的系统负载
- 3、系统调用（CPU在M之间的切换）导致频繁的线程阻塞和取消阻塞操作增加了系统开销

## 调度器的设计策略
### 复用线程

避免频繁的创建、销毁线程，而是对线程的复用

- work stealing机制：当本线程无可运行的G时，尝试从其他线程绑定的P偷取G，而不是销毁线程。
- hand off机制：当本线程因为G进行系统调用阻塞时，线程释放绑定的P，把P转移给其他空闲的线程执行。

### 利用并行

GOMAXPROCS设置P的数量，最多有GOMAXPROCS个线程分布在多个CPU上同时运行

### 抢占

在coroutine中要等待一个协程主动让出CPU才执行下一个协程，在Go中，一个goroutine最多占用CPU 10ms，防止其他goroutine被饿死

### 全局G队列

当M执行work stealing 从其他P偷不到G时，它可以从全局G队列获取G

## “go func()”经历了什么过程

- 1、我们通过go func()来创建一个goroutine；
- 2、有两个存储G的队列，一个是局部调度器P的本地队列、一个是全局G队列。新创建的G会先保存在P的本地队列中，如果P的本地队列已经满了就会保存在全局队列中；
- 3、G只能运行在M中，一个M必须持有一个P，M与P是1 : 1的关系。M会从P的本地队列弹出一个可执行状态的G来执行，如果P的本地队列为空，就会想其他的MP组合偷取一个可执行的G来执行；
- 4、一个M调度G执行的过程是一个循环机制；
- 5、当M执行某一个G时候如果发生了syscall或则其余阻塞操作，M会阻塞，如果当前有一些G在执行，runtime会把这个线程M从P中摘除（detach），然后再创建一个新的操作系统的线程（如果有空闲的线程可用就复用空闲线程）来服务于这个P；
- 6、当M系统调用结束时候，这个G会尝试获取一个空闲的P执行，并放入到这个P的本地队列。如果获取不到P，那么这个线程M变成休眠状态，加入到空闲线程中，然后这个G会被放入全局队列中。

## 调度器的生命周期

### M0

- 启动程序后的编号为0的主线程
- 在全局变量runtime.m0中，不需要在heap上分配
- 负责执行初始化操作和启动第一个G
- 启动第一个G之后，M0就和其他的M一样了

### G0

- 每次启动一个M，都会第一个创建的goroutine，就是G0
- G0仅用于负责调度的G
- G0不指向任何可执行的函数
- 每个M都会有一个自己的G0
- 在调度或系统调用时会使用M会切换到G0来调度
- M0的G0会放在全局空间

## 通过Debug trace 查看GMP信息
- GODEBUG=schedtrace=1000 ./可执行文件
- SCHED：调试的信息
- 0ms：从程序启动到输出经历的时间
- gomaxprocs：P的数量，一般默认是和CPU的核心数一致的
- idleprocs：处理idle状态的P的数量，gomaxprocs-idleprocs=目前正在执行的P的数量
- spinningthreads：处于自旋状态的thread数量
- idlethread：处理idle状态的thread
- runqueue：全局G队列中的G数量
- [0,0]：每个P的local queue本地队列中，目前存在G的数量