# 什么时候触发 GC

## 什么时候触发Young GC----针对年轻代
当Eden区满了的时候，会触发Young GC

## 什么时候触发 Full GC----针对整个堆

在发生Young GC的时候，虚拟机会检测之前每次晋升到老年代的平均大小是否大于年老代的剩余空间，如果大于，则直接进行Full GC；如果小于，但设置了Handle PromotionFailure，那么也会执行Full GC。

```
-XX:HandlePromotionFailure：是否设置空间分配担保

JDK7及以后这个参数就失效了.

只要老年代的连续空间大于新生代对象的总大小或者历次晋升到老年代的对象的平均大小就进行MinorGC，否则FullGC
```

- 永久代空间不足，会触发Full GC
- System.gc()也会触发Full GC
- 堆中分配很大的对象

所谓大对象，是指需要大量连续内存空间的java对象，例如很长的数组，此种对象会直接进入老年代，而老年代虽然有很大的剩余空间，但是无法找到足够大的连续空间来分配给当前对象，此种情况就会触发JVM进行Full GC。

```
-XX:+UseCMSCompactAtFullCollection：设置CMS收集器在完成垃圾收集后是否要进行一次内存碎片的整理
```

为了解决大对象这个问题，CMS垃圾收集器提供了一个可配置的参数，即-XX:+UseCMSCompactAtFullCollection开关参数，用于在“享受”完Full GC服务之后额外免费赠送一个碎片整理的过程

```
-XX:CMSFullGCsBeforeCompaction：设定进行多少次CMS垃圾回收后，进行一次内存压缩
```

内存整理的过程无法并发的，空间碎片问题没有了，停顿时间不得不变长了，JVM设计者们还提供了另外一个参数 -XX:CMSFullGCsBeforeCompaction,这个参数用于设置在执行多少次不压缩的 Full GC 后,跟着来一次带压缩的。

**CMS GC concurrent mode failure 问题**

concurrent mode failure是在执行CMS GC的过程中同时业务线程将对象放入老年代，而此时老年代空间不足，这时 CMS 还没有机会回收老年带产生的，或者在做 Minor GC的时候，新生代救助空间放不下，需要放入老年代，而老年代也放不下而产生的。

## CMS GC----针对年老代
```
配置了-XX:CMSInitiatingOccupancyFraction=75和-XX:+UseCMSInitiatingOccupancyOnly，设定CMS在对内存占用率达到75％的时候开始GC

配置了-XX:+CMSClassUnloadingEnabled，CMSInitiatingPermOccupancyFraction=80%，即：Perm Gen的使用达到一定的比率，默认为92%

配置了-XX:+ExplicitGCInvokesConcurrent，且未配置-XX:+DisableExplicitGC的情况下，显示调用了System.gc()
```

CMS在并发模式工作的时候是只收集老年代的。但一旦并发模式失败（发生concurrent mode failure）就有选择性的会进行全堆收集，也就是退回到Full GC。