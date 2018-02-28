# JVM监控与分析产品

---

<!--sec data-title="Garbage Collection 简介" data-id="jvm_00" data-show=true ces-->
### [Promotion Rate](https://plumbr.eu/blog/garbage-collection/what-is-promotion-rate)
* Promotion rate可以根据young generation的GC日志来分析，具体来说是GC后，年轻代减少的总量-堆空间减少的总量；
* Promotion rate越高，说明年轻代的GC频率越高，且潜在会导致老年代的GC频率增加，因此对应用的性能会产生较大的影响；
* 当Promotion rate接近allocation rate的时候，说明当前存在premature promotion现象，此时需要修改VM的参数，提高年轻代的大小，或更根本的，使用缓存等手段，减小allocation rate；
* 从年轻代promote到老年代，默认要经过15次复制，但是实际上，在年轻代没有足够空间，或者其他情况下，并不需要达到15次就会promote到老年代；

### GC时间
JVM内存越大，GC的时间也越长，有时候设置JVM的内存更大反而会导致性能下降，例如Plumbr提供的[这个例子](https://plumbr.eu/blog/garbage-collection/garbage-collection-increasing-the-throughput)；

### GC种类及比较

#### G1 — Garbage First
* In fact, Garbage-First is a soft real-time garbage collector, meaning that you can set specific performance goals to it. You can request the stop-the-world pauses to be no longer than x milliseconds within any given y-millisecond long time range,
* the heap does not have to be split into contiguous Young and Old generation. Instead, the heap is split into a number (typically about 2048) smaller heap regions that can house objects. Each region may be an Eden region, a Survivor region or an Old region. The logical union of all Eden and Survivor regions is the Young Generation, and all the Old regions put together is the Old Generation:
* only a subset of the regions, called the collection set will be considered at a time. All the Young regions are collected during each pause, but some Old regions may be included as well:
* Another novelty of G1 is that during the concurrent phase it estimates the amount of live data that each region contains. This is used in building the collection set: the regions that contain the most garbage are collected first. Hence the name: garbage-first collection.

#### GC性能比较
* 串行GC的性能除了在单核场景下较有优势，在多核场景下一般较差；
* 根据Plumbr提供的[实验数据](https://plumbr.eu/blog/garbage-collection/g1-vs-cms-vs-parallel-gc)，CMS的性能还是最靠谱的，其次是ParallelGC，G1虽然单次GC时延最小，但是总体吞吐量也最低，**但是注意：具体应用要具体验证！**；
* CMS特别适合[大内存](http://iamzhongyong.iteye.com/blog/1989829)、多核并且要求低时延的场景，[其工作过程可参考链接](http://www.cnblogs.com/zhangxiaoguang/p/5792468.html)；

### GC组合（Java 1.8）
| Young | Tenured | JVM options |
|:----- |:------- |:----------- |
| Increamental | Incremental | -Xincgc |
| Serial | Serial | -XX:+UseSerialGC |
| Parallel Scavenge | Serial | -XX:+UseParallelGC -XX:-UseParallelOldGC |
| Parallel New | Serial | N/A |
| Serial | Parallel Old | N/A |
| Parallel Scavenge | Parallel Old | -XX:+UseParallelGC -XX:+UseParallelOldGC |
| Parallel New | Parallel Old | N/A |
| Serial | CMS | -XX:-UseParNewGC -XX:+UseConcMarkSweepGC |
| Parallel Scavenge | CMS | N/A |
| Parallel New | CMS | -XX:+UseParNewGC -XX:+UseConcMarkSweepGC |
| G1 | G1 | -XX:+UseG1GC |

### GC日志Demo （Java 1.8）
``` shell
# CMS日志
2017-08-02T10:57:45.650+0800: 1.315: [Full GC (Allocation Failure) 1.315: [CMS: 51973K->51956K(68288K), 0.0029881 secs] 51973K->51956K(99008K), [Metaspace: 3324K->3324K(1056768K)], 0.0030612 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
# G1日志
2017-08-02T11:33:36.869+0800: 1.408: [Full GC (Allocation Failure)  50M->50M(100M), 0.0037933 secs]
   [Eden: 0.0B(24.0M)->0.0B(24.0M) Survivors: 0.0B->0.0B Heap: 50.7M(100.0M)->50.7M(100.0M)], [Metaspace: 3326K->3326K(1056768K)]
 [Times: user=0.00 sys=0.00, real=0.00 secs]
# Parallel日志
2017-08-02T11:17:23.961+0800: 1.357: [Full GC (Allocation Failure) [PSYoungGen: 0K->0K(29696K)] [ParOldGen: 51968K->51951K(68608K)] 51968K->51951K(98304K), [Metaspace: 3325K->3325K(1056768K)], 0.0176837 secs] [Times: user=0.02 sys=0.00, real=0.02 secs]
# Serial日志
2017-08-02T11:14:12.384+0800: 1.416: [Full GC (Allocation Failure) 1.416: [Tenured: 51968K->51951K(68288K), 0.0029273 secs] 51968K->51951K(99008K), [Metaspace: 3325K->3325K(1056768K)], 0.0029893 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
```
<!--endsec-->

<!--sec data-title="JClarity" data-id="jvm_0" data-show=true ces-->
### 简介

### 开源代码
[github - jclarity](https://github.com/jClarity?page=1)
<!--endsec-->

<!--sec data-title="Plumbr" data-id="jvm_1" data-show=true ces-->
## 题外话
* 性能调优是了解你有什么资源，哪些资源使用频率较低或过度使用。一旦你了解你的核心资源(CPU，内存，磁盘，网络，JVM线程等)，以及对于这些资源输入输出定时测量，你可以很快找出瓶颈在哪，如何减少这种依赖。

* [我们正在建立一个流的版本](http://www.infoq.com/cn/news/2016/03/jclarity-releases-censum-30)，可以监视多个JVM实时提供同样的详细指标和分析；而且，利用实时数据可以对要发生的危险事件进行警报，甚至预测可能发生的事件，类似于“在未来2分钟你的应用有95%的可能出现内存不足错误（OOME）。”这样的警报信息。

* Censum也将与我们的Illuminate产品集成。总之，Illuminate分析应用程序并指示什么类型的问题最影响性能，这是内存/ GC相关的或其他的东西（例如磁盘I / O，线程死锁，线程饥饿，等待外部系统响应）。如果问题是有关GC，那么你就可以深入了解Censum的功能特性，并获得如何解决问题的一些建议。
<!--endsec-->

