# Hadoop技术

## Hadoop概述

**Hadoop 3.X 的新特性**

- Java的最低版本要求从Java 7 更改成Java 8
- HDFS 支持纠删码（Erasure Coding），从而将数据存储空间节省了50%
- 引入YARN的时间轴服务v.2（YARN TImeline Service v.2）
- 重写了Shell脚本
- 隐藏底层jar包
- 支持containers和分布式调度
- MapReduce任务级本地优化
- 支持多于两个的NameNodes
- 改变了多个服务的默认端口
- 用Intra解决DataNode宕机负载不均衡的问题
- 重写守护进程以及任务的堆内存管理
- 支持Microsoft Azure Data Lake文件系统
- 解决了AMAZON S3的数据一致性问题

## HDFS

### 原理

Hadoop分布式文件系统（Hadoop Distributed File System，HDFS）被设计成适合运行在通用硬件上的分布式文件系统。HDFS时一个高度容错性的系统，适合部署在廉价的机器上。HDFS提供高吞吐量的数据访问，非常适合大规模数据集上的应用

### HDFS的假设前提和设计目标

- 硬件错误
- 大规模数据集
- 简单的一致性模型
- 移动计算比移动数据更划算
- 异构软硬件平台间的可移植性

### HDFS的组件

HDFS包含Namenode、Datanode、Secondary Namenode三个组件。
- Namenode：HDFS的守护进程，用来管理文件系统的命名空间，负责记录文件是如何分割成数据块，以及这些数据块分别被存储到哪些数据节点上，它的主要功能是对内存及IO进行集中管理。
- Datanode：文件系统的工作节点，根据需要存储和检索数据块，并且定期向Namenode发送它们所存储的块的列表
- Secondary Namenode：辅助后台程序，与NameNode进行通信，以便定期保存HDFS元数据的快照，用以备份和恢复数据。

文件写入流程
- 客户端向Namenode发起文件写入的请求
- Namenode根据文件大小和文件快配置情况，返回给客户端所管理部分Datanode的信息
- 客户端将文件划分为多个块，根据Datanoode的地址信息，按顺序写入到每一个Datanode块中

文件读取流程
- 客户端向Namenode发起文件读取的请求
- Namenode返回文件存储的Datanode的信息
- 客户端读取文件信息

### HDFS数据复制

HDFS被设计能够在大集群中跨机器可靠地存储超大文件。它将每个文件存储成一系列地数据块，除了最后一个以外，所有的数据块都是同样大小的（Hadoop 1.X默认每个数据块大小为64MB，Hadoop 2.X和Hadoop 3.X默认每个数据块大小为128MB）。为了容错，文件的所有数据块都会有副本。每个文件的数据块大小和副本系数都市可配置的。应用程序可以指定某个文件的副本数量。副本系数可以在文件创建的时候指定，也可以在之后改变。HDFS中的文件都是一次性写入的，并且严格要求在任何时候都只能有一个写入者。

### HDFS Shell
- Hadoop文件操作命令
- Hadoop系统管理命令

### HDFS Java API

## 分布式计算框架 MapReduce

MapReduce是一种并行编程模型，用于大规模数据集的并行运算。“Map”（映射）和“Reduce”（归约）是它的主要思想，是从函数式编程语言里借来的，MapReduce还有从矢量编程语言里借来的特性。它极大地方便了编程人员在不会分布式并行编程地情况下，将自己的程序运行在分布式系统上。当前的软件实现是指定一个Map（映射）函数，实现任务的分配，指定并发的Reduce（归约）函数，用来任务的汇总。

## MapReduce 的主要任务
- 数据划分和计算任务调度
- 数据/代码互定位
- 系统优化
- 出错检测和恢复

## MapReduce 的处理流程

MapReduce中，Map阶段处理的数据如何传递给Reduce阶段，是整个MapReduce框架中最关键的一个流程，这个流程就叫Shuffle。它的核心机制包括数据分区、排序和缓存等。

Map是映射，负责数据的过滤分发，将原始数据转化为键值对；Reduce是合并，将具有相同key值的value进行处理后再输出新的键值对作为最终结果。为了Reduce可以并行处理Map的结果，必须对Map的输出进行一定的排序与分割，然后再将给对应的Reduce，这个将Map输出进一步整理并交给Reduce的过程就是Shuffle。

## MapReduce综合实例
- 数据去重
- 数据排序
- 求平均成绩
- WordCount单词统计排序