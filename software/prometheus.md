# Prometheus简介

---

## 一、[Prometheus是什么](http://www.cnblogs.com/davygeek/p/6668706.html)

Prometheus是一个开源的系统监控和报警工具，特点是：

* 多维数据模型（时序列数据由metric名和一组key/value组成）
* 在多维度上灵活的查询语言(PromQl)
* 不依赖分布式存储，单主节点工作.
* 通过基于HTTP的pull方式采集时序数据
* 可以通过push gateway进行时序列数据推送(pushing)，主要用于短生命期的采集任务将数据缓存在gateway上
* 可以通过服务发现或者静态配置去获取要采集的目标服务器
* 多种可视化图表及仪表盘支持

### 1.1. pull方式

Prometheus采集数据是用的pull也就是拉模型,通过HTTP协议去采集指标，只要应用系统能够提供HTTP接口就可以接入监控系统，相比于私有协议或二进制协议来说开发、简单。

### 1.2. push方式

对于定时任务这种短周期的指标采集，如果采用pull模式，可能造成任务结束了，Prometheus还没有来得及采集，这个时候可以使用加一个中转层，客户端推数据到Push Gateway缓存一下，由Prometheus从push gateway pull指标过来。(需要额外搭建Push Gateway，同时需要新增job去从gateway采数据)

## 二、组成及架构

* Prometheus server 主要负责数据采集和存储，提供PromQL查询语言的支持
* 客户端sdk 官方提供的客户端类库有go、java、scala、python、ruby，其他还有很多第三 方开发的类库，支持nodejs、php、erlang等
* Push Gateway 支持临时性Job主动推送指标的中间网关
* exporters 支持其他数据源的指标导入到Prometheus，支持数据库、硬件、消息中间件、存储系统、http服务器、jmx等
* alertmanager 实验性组件、用来进行报警
* prometheus_cli 命令行工具
* PromDash 使用rails开发的dashboard，用于可视化指标数据
* 其他辅助性工具

### 1. 整体架构
![架构图](http://rnd-isourceb.huawei.com/images/SZ/20170801/bbb57128-748d-41c4-9702-2a6f2130b1d9/image.png)
Prometheus通过instrumented jobs抓取数据，可以是直接通过http接口，也可以间接的通过pushgateway抓取。它将所有采集的数据保存在本地，通过运行一系列规则，决定生成新的时序记录，或者上报告警。这些数据可以通过grafana等其他API consumers消费来进行可视化。

### 2. 适用场景
Prometheus最适合记录纯指标时序数据，无论是机器性能监控，还是动态服务监控。对于微服务架构，它能够支持多维指标数据采集和一定强度的查询。

Prometheus具有高可靠性，当系统出现问题时，可以支持快速的故障诊断。每个Prometheus server都是独立工作，不需要依赖三方组件。

### 3. 不适用场景
Prometheus会对原始数据进行一定的转换和摘要，而不是全部保存原始的数据，因此不适用于原始数据的全文查询。对于这种场景，只能通过其他系统来支持全文检索，而使用Prometheus进行指标监控。