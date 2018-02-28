# 其他存储及缓存应用

---

<!--sec data-title="HDFS" data-id="storage_1" data-show=true ces-->
## 安装配置
要先将hadoop/bin加入到用户环境变量中：`.bashrc`
``` shell
# 查看某个目录下的文件大小
hadoop fs -du -h /druid/segments
# 删除某个目录下所有文件
hadoop fs -rmr /druid/segments/perflog-victor
hadoop fs -rmr /druid/indexing-logs/index_kafka_perflog-victor*
```
<!--endsec-->


<!--sec data-title="Elasticsearch" data-id="storage_2" data-show=true ces-->
## Curl命令说明
```
curl -X<VERB> '<PROTOCOL>://<HOST>:<PORT>/<PATH>?<QUERY_STRING>' -d '<BODY>'
```
其中：
* VERB HTTP方法：GET, POST, PUT, HEAD, DELETE
* PROTOCOL http或者https协议（只有在Elasticsearch前面有https代理的时候可用）
* HOST Elasticsearch集群中的任何一个节点的主机名，如果是在本地的节点，那么就叫localhost
* PORT Elasticsearch HTTP服务所在的端口，默认为9200，Vsearch默认为7300
* PATH API路径（例如_count将返回集群中文档的数量），PATH可以包含多个组件，例如_cluster/stats或者_nodes/stats/jvm
* QUERY_STRING 一些可选的查询请求参数，例如?pretty参数将使请求返回更加美观易读的JSON数据
* BODY 一个JSON格式的请求主体（如果请求需要的话）
<!--endsec-->