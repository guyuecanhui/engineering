# JVM监控与分析产品

---

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

