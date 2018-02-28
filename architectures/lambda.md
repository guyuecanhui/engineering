# Lambda架构

---

## 一、Lambda架构的背景
大数据处理技术需要解决大数据场景下可伸缩性与复杂性。Lambda架构的目标是设计出一个能满足实时大数据系统关键特性的架构，包括有：高容错、低延时和可扩展等。Lambda架构整合离线计算和实时计算，融合不可变性（Immunability），读写分离和复杂性隔离等一系列架构原则，可集成Hadoop，Kafka，Storm，Spark，Hbase等各类大数据组件。

### 1.1. 大数据系统关键特性
1. Robustandfault-tolerant（容错性和鲁棒性）：对大规模分布式系统来说，机器是不可靠的，可能会当机，但是系统需要是健壮、行为正确的，即使是遇到机器错误。除了机器错误，人更可能会犯错误。在软件开发中难免会有一些Bug，系统必须对有Bug的程序写入的错误数据有足够的适应能力，所以比机器容错性更加重要的容错性是人为操作容错性。对于大规模的分布式系统来说，人和机器的错误每天都可能会发生，如何应对人和机器的错误，让系统能够从错误中快速恢复尤其重要。
2. Lowlatency reads and updates（低延时）：很多应用对于读和写操作的延时要求非常高，要求对更新和查询的响应是低延时的。
3. Scalable（横向扩容）：当数据量/负载增大时，可扩展性的系统通过增加更多的机器资源来维持性能。也就是常说的系统需要线性可扩展，通常采用scale out（通过增加机器的个数）而不是scale up（通过增强机器的性能）。
4. General（通用性）：系统需要能够适应广泛的应用，包括金融领域、社交网络、电子商务数据分析等。
5. Extensible（可扩展）：需要增加新功能、新特性时，可扩展的系统能以最小的开发代价来增加新功能。
6. Allows ad hoc queries（方便查询）：数据中蕴含有价值，需要能够方便、快速的查询出所需要的数据。
7. Minimal maintenance（易于维护）：系统要想做到易于维护，其关键是控制其复杂性，越是复杂的系统越容易出错、越难维护。
8. Debuggable（易调试）：当出问题时，系统需要有足够的信息来调试错误，找到问题的根源。其关键是能够追根溯源到每个数据生成点。

### 1.2. 数据系统的本质
数据系统通过查询过去的（部分、全部）数据去回答问题。因此，DataSystem的通用定义为`query = function(alldata)`。对通用的表达式进行分解得到：`数据系统 = 数据 + 查询`，从而可以从数据和查询两个方面认识大数据系统的本质。

#### A. 数据本本质：When & What
数据是一个不可分割的单元，数据有两个关键的特性：When和What。

1. When是只数据是与时间相关的，也就是数据是在某个时间产生的。数据的时间性质决定了数据的全局发生先后，也就决定了数据的结果。
2. What是只数据的本身。由于数据跟某个时间点相关，所以数据的本身是不可变的(immutable)，这也就意味着对数据的操作其实只有两种：读取已存在的数据和添加更多的新数据。

#### B. 数据的存储：Store Everything Rawly and Immutably
根据上述对数据特性的分析，lambda架构中对数据的存储采用的方式是：数据不可变，存储所有数据。采用这两种方式存储的好处：

1. 简单。采用不可变的数据模型，存储数据时只需要简单的往主数据集后追加数据即可。相比于采用可变的数据模型，为了Update操作，数据通常需要被索引，从而能快速找到要更新的数据去做更新操作。
2. 应对人为和机器的错误。不可变和可重复计算是应对认为和机器错误的常用方法，因为所有的数据都在，引发错误的数据也在，修复的方法就可以简单的是遍历数据集上存储的所有的数据，丢弃错误的数据，重新计算得到Views。重新计算的关键点在于利用数据的时间特性决定的全局次序，依次顺序重新执行，必然能得到正确的结果。

## 二、Lambda架构

Lambda架构的主要思想是将大数据系统架构为多层个层次，分别为批处理层（batch layer）、实时处理层（speed layer）、服务层（serving layer），如下图所示。

![lambda_architecture.png](http://rnd-isourceb.huawei.com/images/SZ/20170801/2f45f1ed-03df-4251-aa8c-aa5b59e366b4/lambda_architecture.png)

理想状态下，任何数据访问都可以从表达式`query = function(alldata)`开始，但是，若数据达到相当大的一个级别（例如PB），且还需要支持实时查询时，就需要耗费非常庞大的资源。一个解决方式是预运算查询函数（precomputedquery funciton），称之为Batch View，即`（A）batchview = function(alldata)`，这样当需要执行查询时，可以从BatchView中读取结果。

此外，为了处理实时新增的数据，可以增加SpeedLayer，可总结为以`（B）realtimeview = function(realtimeview, newdata)`；Lambda Architecture将数据处理分解为BatchLayer和SpeedLayer有如下优点：

1. 容错性：SpeedLayer中处理的数据不断写入BatchLayer，当BatchLayer中重新计算数据集包含SpeedLayer处理的数据集后，当前的RealtimeView就可以丢弃，这就意味着SpeedLayer处理中引入的错误，在BatchLayer重新计算时都可以得到修证。这点也可以看成时CAP理论中的最终一致性（EventualConsistency）的体现。
2. 复杂性隔离。BatchLayer处理的是离线数据，可以很好的掌控。Speed Layer采用增量算法处理实时数据，复杂性比Batch Layer要高很多。通过分开BatchLayer和Speed Layer，把复杂性隔离到Speed Layer，可以很好的提高整个系统的鲁棒性和可靠性。

最终系统通过ServingLayer用于响应用户的查询请求，合并BatchView和Realtime View中的结果数据集到最终的数据集，即`（C）query = function(batchview, realtimeview)`。因此，ServingLayer的职责包含：

1. 对batchView和RealTimeView的随机访问。
2. 更新BatchVeiw和RealTimeView，并负责结合两者的数据，对用户提供统一的接口。


### 2.1. Batch Layer
在Lambda架构中，实现`（A）batchview = function(alldata)`的部分称之为BatchLayer。他承担两个职责：

1. 存储MasterDataset，这是一个不变的持续增长的数据集
2. 针对这个MasterDataset进行预运算

在全体数据集上在线运行查询函数得到结果的代价太大，同时处理查询时间过长，导致用户体验不好。如果我们预先在数据集上计算并保存预计算的结果，查询的时候直接返回预计算的结果，而无需重新进行复制耗时的计算。显然，batchview是一个批处理过程，如采用Hadoop或spark支持的map－reduce方式。采用这种方式计算得到的每个view都支持再次计算，且每次计算的结果都相同。

### 2.2. Speed Layer
BatchLayer能够很好的处理离线数据，但是在很多场景数据不断产生，并且业务场景需要实时查询。SpeedLayer就是设计用来处理增量实时数据。SpeedLayer和BatchLayer比较类似，对数据进行计算并生成RealtimeView，其主要的区别在于：

1. SpeedLayer处理的数据是最近的增量数据流，BatchLayer处理的是全体数据集。
2. SpeedLayer为了效率，接收到新数据及时更新RealtimeView，而BatchLayer根据全体离线数据直接得到BatchView。SpeedLayer是一种增量计算，而非重新计算（recomputation）。
3. SpeedLayer因为采用增量计算，所以延迟小，而BatchLayer是全数据集的计算，耗时比较长。

### 2.3. Lambda架构的评估

#### 优点：
1. 数据的不可变性。里面给出的数据传输模型是在初始化阶段对数据进行实例化，这样的做法是能获益良多的。能够使得大量的MapReduce工作变得有迹可循，从而便于在不同阶段进行独立调试。
2. 强调了数据的重新计算问题。在流处理中重新计算是个主要挑战，但是经常被忽视。比方说，某工作流的数据输出是由输入决定的，那么一旦代码发生改动，我们将不得不重新计算来检视变更的效度。什么情况下代码会改动呢？例如需求发生变更，计算字段需要调整或者程序发出错误，需要进行调试。

#### 缺点：
Jay Kreps认为Lambda包含固有的开发和运维的复杂性。Lambda需要将所有的算法实现两次，一次是为批处理系统，另一次是为实时系统，还要求查询得到的是两个系统结果的合并。由于存在以上缺点，Linkedin的Jaykreps提出了Kappa架构如下图所示：

![kappa_architecture.png](http://rnd-isourceb.huawei.com/images/SZ/20170801/d708ff96-e2ca-4682-967c-6a174d15e971/kappa_architecture.png)

1. 使用Kafka或其它系统来对需要重新计算的数据进行日志记录，以及提供给多个订阅者使用。例如需要重新计算30天内的数据，我们可以在Kafka中设置30天的数据保留值。
2. 当需要进行重新计算时，启动流处理作业的第二个实例对之前获得的数据进行处理，之后直接把结果数据放入新的数据输出表中。
3. 当作业完成时，让应用程序直接读取新的数据记录表。
4. 停止历史作业，删除旧的数据输出表。