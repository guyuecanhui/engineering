# Druid查询语句

---

## 基本概念
### [查询基础](http://druid.io/docs/latest/querying/querying.html)
Druid所有查询都是通过HTTP POST实现，主要是通过JSON的不同properties来区分。统一查询语句：

`curl -X POST '<queryable_host>:<port>/druid/v2/?pretty' -H 'Content-Type:application/json' -d @<query_json_file>`

### [Datasources](http://druid.io/docs/latest/querying/datasource.html)
Druid中，一个Datasource类似于数据库里的一个表，其type主要为`table`，也可以是多个Table的`union`。对于GroupBy查询而言，它的结果也可以认为是后续查询的datasource。

### [Aggregation Granularity](http://druid.io/docs/latest/querying/granularities.html)
**Simple Granularities**是按自然天、小时等来划分的，当前支持`all`, `none`, `second`, `minute`, `fifteen_minute`, `thirty_minute`, `hour`, `day`, `week`, `month`, `quarter` and `year`.

**Duration Granularities**可以自己设定起始时间和时间任意毫秒的时间粒度，比如：`{"type": "duration", "duration": 3600000, "origin": "2012-01-01T00:30:00Z"}`

**Period Granularities**可以按ISO8601格式组合自然天、小时等，并且可以指定时区(主要影响`timestamp`的时区取值)、起始时间等，比如：`{"type": "period", "period": "P3Y6M4DT12H30M5S", "timeZone": "America/Los_Angeles", "origin": "2012-02-01T00:00:00-08:00"}`

### [Query Filters](http://druid.io/docs/latest/querying/filters.html)
filter相当于SQL里的`WHERE`语句。另外，这些filter还可以添加Extracion Function来改变列值，`extractionFun`在下一节介绍。

**Selector filter**：例如：`"filter": { "type": "selector", "dimension": <dimension_string>, "value": <dimension_value_string> }`，精确匹配`<dimension_string>=<dimension_value_string>`的行；

**Column Comparison filter**：例如：`"filter": { "type": "columnComparison", "dimensions": [<dimension_a>, <dimension_b>] }`，精确匹配`<dimension_a> = <dimension_b>`的行；

**Regular expression filter**：例如：`"filter": { "type": "regex", "dimension": <dimension_string>, "pattern": <pattern_string> }`，匹配`<dimension_string>`满足正则表达式`<pattern_string>`的行；

**Logical expression filters**：例如：`"filter": { "type": "and", "fields": [<filter>, <filter>, ...] }`，匹配满足所有子filter的行；`"filter": { "type": "or", "fields": [<filter>, <filter>, ...] }`，匹配满足任意子filter的行；`"filter": { "type": "not", "field": <filter> }`，匹配不满足子filter的行；

**JavaScript filter**：*默认是disabled*，例如：`"filter": { "type" : "javascript", "dimension" : "name", "function" : "function(x) { return(x >= 'bar' && x <= 'foo') }"}`，匹配function定义的`x`；

**Search filter**：主要用于字符串部分匹配，例如：`"filter": { "type": "search", "dimension": "product", "query": { "type": "insensitive_contains", "value": "foo"}}`，匹配满足`product=foo`（不区分大小写）的行；

**In filter**：例如：`"filter": { "type": "in", "dimension": "outlaw", "values": ["Good", "Bad", "Ugly"]}`，匹配所有`outlaw`在取值集合内的行；

**Like filter**：例如：`"filter": { "type": "like", "dimension": "last_name", "pattern": "D%"}`，匹配所有`last_name`以`D`开头的行；

**Bound filter**：用于过滤dimension取值范围（默认是`lexicographic`顺序排列，也可以指定其他排序方式），例如：`{"type": "bound", "dimension": "age", "lower": "21", "upper": "31" , "upperStrict": true, "ordering": "numeric"}`，匹配所有`21<=age<31`的行；

**Interval Filter**：可以用于`__time`列、long-value列以及其他可以被转换成long-value的列，其实也可以用Bound filter来替代，例如：`{"type" : "interval", "dimension" : "__time", "intervals" : [ "2014-10-01T00:00:00.000Z/2014-10-07T00:00:00.000Z", "2014-11-15T00:00:00.000Z/2014-11-16T00:00:00.000Z"]}`，匹配所有在interval列表范围内的行；

### [Aggregations](http://druid.io/docs/latest/querying/aggregations.html)
**Count aggregator**：单纯计数，例如：`{ "type" : "count", "name" : <output_name> }`；

**Sum aggregators**：包括`longSum`和`doubleSum`，例如：`{ "type" : "longSum", "name" : <output_name>, "fieldName" : <metric_name> }`

**Min / Max aggregators**：包括`doubleMin`、`doubleMax`、`longMin`和`longMax`，例如：`{ "type" : "longMin", "name" : <output_name>, "fieldName" : <metric_name> }`

**First / Last aggregator**：只能作为查询的部分条件，查询指定fieldName的最小/最大timestamp对应的metric value，如果一条都没有，返回0。支持`doubleFirst`、`doubleLast`、`longFirst`和`longLast`，例如：`{ "type" : "doubleLast", "name" : <output_name>, "fieldName" : <metric_name>}`

**JavaScript aggregators**：*默认是disabled*，可以对任意多列（metric或dimension）上构造JavaScript函数进行计算，计算的结果应该是floating-point值。例如：
``` json
{
  "type": "javascript",
  "name": "sum(log(x)*y) + 10",
  "fieldNames": ["x", "y"],
  "fnAggregate" : "function(current, a, b)      { return current + (Math.log(a) * b); }",
  "fnCombine"   : "function(partialA, partialB) { return partialA + partialB; }",
  "fnReset"     : "function()                   { return 10; }"
}
```

**Approximate Aggregations**：主要用于基数估计，待补充。

### [Post-Aggregations](http://druid.io/docs/latest/querying/post-aggregations.html)
主要用于查询后的数据再处理，支持以下aggregators：

**Arithmetic post-aggregator**：


## 查询类型
### Aggregation Queries
`GroupBy`命令是最灵活，但是也是性能最差的，主要用于在多个维度上执行group操作时使用，或者超大数据的精确查询时使用；`TopN`主要适用于在单个维度上的groupby和sort操作；`Timeseries`主要用于其他情况下的汇聚查询，特别是只把时间当作过滤或group的依据，但是`Timeseries`不包含`dimension`，也就是不能做`lookup`之类的降维操作。

### TopN queries
TopN查询返回按一定指标，对一个维度内的值进行排序后，前`threshold`个结果。然而在实现的时候，每个节点只返回k个结果，再进行归并排序（`k=max(1000,threshold)`），因此，当`threshold`过大时，前面的结果能够保证正确，但是后面的结果未必准确。如果对于大的N值要获得精确的结果，最好使用`GroupBy`以后再自己排序。

另外在实际使用的时候，需要注意的是**时间粒度和数据的分布**。对于某些数据，在任意的最小考察时间粒度（例如5分钟）内，它的数据都在TopN范围内，那么TopN总是能返回正确的结果；但是如果有些数据均匀分布在每个时间段内，因此在小的时间粒度内不属于TopN，但是在大的时间粒度内却属于TopN，那么TopN需要充分考虑这种情况。如果实际上并不需要精确的结果，使用近似的结果能够节省大量的资源开销。

### GroupBy queries
GroupBy查询是将目标`dimentions`按取值合并，与前两类查询比较，特殊的地方有两个，一个是可以指定结果数据的上限（通过[`limitSpec`](http://druid.io/docs/latest/querying/limitspec.html)语句指定），一个是可以指定最终展示的列（通过[`having`](http://druid.io/docs/latest/querying/having.html)语句过滤）。另外，GroupBy有两种策略：`v1`策略完全在堆内处理合并，而`v2`策略完全在堆外处理合并，默认使用`v2`策略，它们有不同的适用场景，调优方式也不相同。


### Metadata Queries

### Search Queries

## 常用查询语句
``` shell
# 发送kafka数据拉取请求，即创建supervisor
curl -X POST -H 'Content-Type: application/json' -d @supervisor-spec.json http://192.128.34.11:8090/druid/indexer/v1/supervisor

# 查看当前有哪些supervisor，即有什么datasource
curl -X GET http://192.128.34.11:8090/druid/indexer/v1/supervisor

# 查看当前某个kafka supervisor的状态
curl -X GET http://192.128.34.11:8090/druid/indexer/v1/supervisor/perflog-hc-try/status

# 停止kafka数据拉取请求
curl -X POST http://192.128.34.11:8090/druid/indexer/v1/supervisor/perflog-hwhc/shutdown

# 取消查询ID为abc123的查询
curl -X DELETE http://host:port/druid/v2/abc123

# 查看当前druid进程
jps -lvm |sed '/jps/d' |grep -Po '^[0-9]+|(?<=-Dcsc.home=/srv/druid/)rubik_druid_[a-zA-Z]+'|xargs -n 2
```

# 查询示例与说明

## TopN Queries
#### 示例1.1
1. `dataSource`为创建kafka的supervisor时指定的；
2. `intervals`为前闭后开区间；
3. `fieldName`为实际引用的数据列名，如在`aggregations`里，`count`为原始数据中实际存在的名为`count`的列，而在`postAggregations`里，`countn`为`aggregations`中输出的列；
4. 输出到结果中的为所有`name`和`metric`指定的字段，在这个例子里包括：`dn`、`countn`、`rate`和`another`；
``` http
POST /druid/v2/?pretty HTTP/1.1
Host: 192.128.34.11:8082
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: 89161812-f8e2-c3b6-694e-3b71a998aae5
{
    "queryType": "topN",
    "dataSource": "perflog-hc-try",
    "intervals": ["2017-05-17/2017-05-18"],
    "granularity": "hour",
    "dimension": "dn",
    "metric": "rate",
    "threshold": 100,
    "aggregations": [{
        "type": "longSum",
        "name": "countn",
        "fieldName": "count"
    }, {
        "type": "longMax",
        "name": "another",
        "fieldName": "max_succeedCount"
    }],
    "filter": {
        "type": "selector",
        "dimension": "motype",
        "value": "motype4"
    },
    "postAggregations": [{
        "type": "arithmetic",
        "name": "rate",
        "fn": "/",
        "fields": [{
            "type": "fieldAccess",
            "name": "countm",
            "fieldName": "countn"
        }, {
            "type": "fieldAccess",
            "name": "theother",
            "fieldName": "another"
        }]
    }]
}
```

#### 示例1.2
1. `TopN`也可以使用单个`dimension`的`extraction`，但是必须指定`metric`和`threshold`，也就是能精确查询的行数有限；
```http
POST /druid/v2/?pretty HTTP/1.1
Host: 192.128.34.11:8082
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: cae41e9b-de37-007a-96b4-996cf51fa29c
{
    "queryType": "topN",
    "dataSource": "perflog-hc-try",
    "granularity": "minute",
    "descending": "true",
    "metric": "total_conut",
    "threshold": 100,
    "filter": {
        "type": "and",
        "fields": [{
            "type": "selector",
            "dimension": "service",
            "value": "service2"
        }, {
            "type": "selector",
            "dimension": "measunit",
            "value": "mu2"
        }]
    },
    "dimension": {
        "type": "extraction",
        "dimension": "dn",
        "outputName": "cdn",
        "extractionFn": {
            "type": "lookup",
            "lookup": {
                "type": "map",
                "map": {
                    "dn0": "CDN0",
                    "dn1": "CDN0",
                    "dn10": "CDN0"
                }
            },
            "retainMissingValue": false,
            "injective": false,
            "replaceMissingValueWith": "MISSING"
        }
    },
    "intervals": ["2017-05-17T08:57:00/PT1M"],
    "aggregations": [{
        "type": "longSum",
        "fieldName": "sum_totalSuccessElapsedTime",
        "name": "sum_totalSuccessElapsedTime"
    }, {
        "type": "longSum",
        "fieldName": "sum_succeedCount",
        "name": "sum_succeedCount"
    }, {
        "type": "longMax",
        "fieldName": "max_totalSuccessElapsedTime",
        "name": "max_totalSuccessElapsedTime"
    }, {
        "type": "longMin",
        "fieldName": "min_totalSuccessElapsedTime",
        "name": "min_totalSuccessElapsedTime"
    }, {
        "type": "longSum",
        "fieldName": "sum_failedCount",
        "name": "sum_failedCount"
    }],
    "postAggregations": [{
        "type": "arithmetic",
        "name": "avg_totalSuccessElapsedTime",
        "fn": "/",
        "fields": [{
            "type": "fieldAccess",
            "name": "sum_totalSuccessElapsedTime",
            "fieldName": "sum_totalSuccessElapsedTime"
        }, {
            "type": "fieldAccess",
            "name": "sum_succeedCount",
            "fieldName": "sum_succeedCount"
        }]
    }, {
        "type": "arithmetic",
        "name": "total_conut",
        "fn": "+",
        "fields": [{
            "type": "fieldAccess",
            "name": "sum_succeedCount",
            "fieldName": "sum_succeedCount"
        }, {
            "type": "fieldAccess",
            "name": "sum_failedCount",
            "fieldName": "sum_failedCount"
        }]
    }, {
        "type": "arithmetic",
        "name": "success_rate",
        "fn": "/",
        "fields": [{
            "type": "fieldAccess",
            "name": "sum_succeedCount",
            "fieldName": "sum_succeedCount"
        }, {
            "type": "fieldAccess",
            "name": "total_conut",
            "fieldName": "total_conut"
        }]
    }],
    "context": {
        "groupByStrategy": "v2"
    }
}
```

## GroupBy Queries
#### 示例2.1
1. `dimensions`定义了`groupby`的对象，得到所有`beId`与`dn`列表的结果组合；
```http
POST /druid/v2/?pretty HTTP/1.1
Host: 192.128.34.11:8082
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: 212537e1-bc14-9274-1e81-58c03be316bd
{
    "queryType": "groupBy",
    "dataSource": "perflog-hc-try",
    "granularity": "all",
    "descending": "true",
    "dimensions": ["dn", "beId"],
    "filter": {
        "type": "and",
        "fields": [{
            "type": "selector",
            "dimension": "measunit",
            "value": "mu2"
        }, {
            "type": "in",
            "dimension": "dn",
            "values": ["dn0","dn1","dn2","dn3"]
        }]
    },
    "aggregations": [{
        "type": "longSum",
        "name": "total",
        "fieldName": "count"
    }],
    "intervals": ["2017-05-17T08:57:00/PT1M"],
    "context": {
        "groupByStrategy": "v2"
    }
}
```

#### 示例2.2
1. 与示例1.1相比，这个例子多输出了`outputName`的两列；
2. `extractionFn`定义了如何对`dimension`进行变换；
3. `lookup`定义了一种映射替换关系，例如将`dn0`替换为`CDN0`等；对于映射关系中不存在的key，将输出`replaceMissingValueWith`来填充；
```http
POST /druid/v2/?pretty HTTP/1.1
Host: 192.128.34.11:8082
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: a1361c5a-69b4-4e6c-67d2-4c83852440bc
{
    "queryType": "groupBy",
    "dataSource": "perflog-hc-try",
    "granularity": "minute",
    "descending": "true",
    "filter": {
        "type": "and",
        "fields": [{
            "type": "selector",
            "dimension": "service",
            "value": "service2"
        }, {
            "type": "selector",
            "dimension": "measunit",
            "value": "mu2"
        }]
    },
    "dimensions": [{
        "type": "extraction",
        "dimension": "dn",
        "outputName": "cdn",
        "extractionFn": {
            "type": "lookup",
            "lookup": {
                "type": "map",
                "map": {"dn0":"CDN0","dn1":"CDN0","dn10":"CDN0"}
            },
            "retainMissingValue": false,
            "injective": false,
            "replaceMissingValueWith": "MISSING"
        }
    }, {
        "dimension": "measunit",
        "outputName": "cmu",
        "extractionFn": {
            "type": "lookup",
            "lookup": {
                "type": "map",
                "map": {
                    "mu2": "cmu2"
                }
            },
            "retainMissingValue": false,
            "injective": false,
            "replaceMissingValueWith": "MISSING"
        }
    }],
    "intervals": ["2017-05-17T08:57:00/PT1M"],
    "aggregations": [{
        "type": "longSum",
        "fieldName": "sum_totalSuccessElapsedTime",
        "name": "sum_totalSuccessElapsedTime"
    }, {
        "type": "longSum",
        "fieldName": "sum_succeedCount",
        "name": "sum_succeedCount"
    }, {
        "type": "longMax",
        "fieldName": "max_totalSuccessElapsedTime",
        "name": "max_totalSuccessElapsedTime"
    }, {
        "type": "longMin",
        "fieldName": "min_totalSuccessElapsedTime",
        "name": "min_totalSuccessElapsedTime"
    }, {
        "type": "longSum",
        "fieldName": "sum_failedCount",
        "name": "sum_failedCount"
    }],
    "postAggregations": [{
        "type": "arithmetic",
        "name": "avg_totalSuccessElapsedTime",
        "fn": "/",
        "fields": [{
            "type": "fieldAccess",
            "name": "sum_totalSuccessElapsedTime",
            "fieldName": "sum_totalSuccessElapsedTime"
        }, {
            "type": "fieldAccess",
            "name": "sum_succeedCount",
            "fieldName": "sum_succeedCount"
        }]
    }, {
        "type": "arithmetic",
        "name": "total_count",
        "fn": "+",
        "fields": [{
            "type": "fieldAccess",
            "name": "sum_succeedCount",
            "fieldName": "sum_succeedCount"
        }, {
            "type": "fieldAccess",
            "name": "sum_failedCount",
            "fieldName": "sum_failedCount"
        }]
    }, {
        "type": "arithmetic",
        "name": "success_rate",
        "fn": "/",
        "fields": [{
            "type": "fieldAccess",
            "name": "sum_succeedCount",
            "fieldName": "sum_succeedCount"
        }, {
            "type": "fieldAccess",
            "name": "total_count",
            "fieldName": "total_count"
        }]
    }],
    "context": {
        "groupByStrategy": "v2"
    }
}
```

## Select Queries
#### 示例3.1
1. `dimensions`指定结果中返回的`dimensions`列表；
2. `metrics`指定结果中返回的`metrics`列表；
3. `pagingSpec`中，`fromNext=false`时，结果中`pagingIdentifiers`各segment的偏移量为正，否则为负；
4. 分页查询时，下一个查询的偏移量要为上一次查询结果中，`pagingIdentifiers`的偏移量+1；
``` http
POST /druid/v2/?pretty HTTP/1.1
Host: 192.128.34.11:8082
Content-Type: application/json
Cache-Control: no-cache
Postman-Token: 886accb1-7bbf-3caf-e08c-9d1a2283845f
{
    "queryType": "select",
    "dataSource": "perflog-hc-try",
    "descending": "false",
    "filter": {
        "type": "and",
        "fields": [{
            "type": "selector",
            "dimension": "motype",
            "value": "motype5"
        }, {
            "type": "selector",
            "dimension": "dn",
            "value": "dn3388"
        }, {
            "type": "selector",
            "dimension": "beId",
            "value": "beid3"
        }, {
            "type": "selector",
            "dimension": "measunit",
            "value": "mu68"
        }]
    },
    "dimensions": ["businessCode", "channelType", "service"],
    "metrics": ["count", "sum_succeedCount", "sum_failedCount"],
    "granularity": "all",
    "intervals": ["2017-05-17T06:00:00/PT1H"],
    "pagingSpec": {
        "fromNext": "false",
        "pagingIdentifiers": {"perflog-hc-try_2017-05-17T06:00:00.000Z_2017-05-17T06:05:00.000Z_2017-05-17T05:59:56.827Z": 6},
        "threshold": 100
    }
}
```

