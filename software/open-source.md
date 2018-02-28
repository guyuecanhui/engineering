# 其他常用开源软件

<!--sec data-title="ZooKeeper" data-id="open_0" data-show=true ces-->
### [Zookeeper常用命令](http://blog.csdn.net/zljjava/article/details/50802756)
```shell
# 启动zk服务
./zkServer.sh start
# 连接zk客户端
./zkCli.sh -server 127.0.0.1:2181
```

### Zookeeper shell指令
``` shell
# 查看historical节点加载了哪个segment
ls /druid/segments/host347:8100
# 创建目录：
create /dir val
# 查看值：
get /name
# 删除节点：
delete /name
# 递归删除节点
rmr /druid
```
<!--endsec-->

<!--sec data-title="Jmeter" data-id="open_1" data-show=true ces-->
### 常用函数
``` shell
# 一天前的时间，注意：不适用于每月第一天
${__time(yyyy-MM,)}-${__intSum(${__time(dd,)}, -1)}T${__time(HH:mm,)}:00+08:00/P1D
# 一分钟前的时间，注意：不适用于每小时第1分钟之前的时间
${__time(yyyy-MM-dd,)}T${__time(HH,)}:${__intSum(${__time(mm,)}, -1)}:00+08:00/PT1M
```
<!--endsec-->
