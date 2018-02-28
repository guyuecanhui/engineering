# Kafka

---

<!--sec data-title="Kafka常用命令" data-id="kafka_0" data-show=true ces-->
```bash
# 后台启动kafka
bin/kafka-server-start.sh -daemon config/server.properties
# 停止kafka
bin/kafka-server-stop.sh
# 查看所有topic
bin/kafka-topics.sh --zookeeper 192.128.34.14:2181 --list
# 查看topic perflog的描述 
bin/kafka-topics.sh --zookeeper 192.128.34.14:2181 --describe --topic perflog
# 修改topic perflog的分区数为18
bin/kafka-topics.sh --zookeeper 192.128.34.14:2181 --alter --partitions 18 --topic perflog 
# 删除topic，最好在broker 0上删除
bin/kafka-topics.sh --zookeeper 192.128.34.14:2181 --delete --topic perflog
# 新建topic，并指定复本数1和分区数9
bin/kafka-topics.sh --zookeeper 192.128.34.14:2181 --create --replication-factor 1 --partitions 12 --topic perflog
# 从头查看所有perflog的数据
bin/kafka-console-consumer.sh --zookeeper 192.128.34.14:2181 --from-beginning --topic perflog
# 查看group perflog-final的topic消费情况
bin/kafka-consumer-groups.sh --zookeeper 192.128.34.14:2181 --describe --group perflog-new
# 通过console向kafka的topic写入数据
bin/kafka-console-producer.sh --broker-list localhost:9092 --topic test

bin/kafka-consumer-offset-checker.sh --zookeeper 10.137.32.26:24002,10.137.32.27:24002,10.137.32.28:24002/kafka --group jvmkpi_redis --topic jvmkpilog
```
<!--endsec-->

<!--sec data-title="Kafka问题合辑" data-id="kafka_1" data-show=true ces-->
1. [`java.nio.channels.ClosedChannelException`](https://my.oschina.net/chiyong/blog/522998)：Kafka server.properties中的advertised.host.name 属性没有配置；配置为kafka broker地址解决；
2. [`Kafka topic leader = -1 or leader = none`](http://ddmonk.github.io/2016/03/01/Kafka-topic-leader-1-or-leader-none/)：在创建topic时，该partition的所有Replica都宕机了；重启或设置rebanlance解决；
3. [`kafka.common.NotLeaderForPartitionException`](https://issues.apache.org/jira/browse/KAFKA-816)：正常的异常；
4. [`No broker partitions consumed by consumer thread`](https://discuss.elastic.co/t/kafka-consumer-rangeassignor-no-broker-partitions-consumed-by-consumer-thread-logstash-logstash-indexer/33012)：用`./kafka-consumer-offset-checker.sh --zookeeper *:*/kafka --topic * -- group *`看下lag是否为正，如果为负就删除数据；
5. [`kafka.common.ConsumerRebalanceFailedException`](http://blog.csdn.net/lizhitao/article/details/25301387)：同一个group的consumer先后启动，触发rebalance；通过增加consumer client的retry次数和backoff时间来解决；
<!--endsec-->