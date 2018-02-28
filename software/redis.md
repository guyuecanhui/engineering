# Redis

---

<!--sec data-title="Redis安装" data-id="redis_0" data-show=true ces-->
**1. 解压后`make`**
``` shell
tar xvzf redis-3.2.3.tar.gz
cd redis-3.2.3
make
```

**2. 为所有需要起的node创建单独的路径，把redis.conf文件拷贝进去，需要修改以下项：**
``` shell
# 绑定本机地址，一定要在127之前
bind 10.179.75.155 127.0.0.1
# 同一节点上每个node的port不能一样
port 7000
maxmemory 1073741824
cluster-enabled yes
cluster-config-file conf/7000/nodes.conf
cluster-node-timeout 15000
cluster-slave-validity-factor 10
cluster-migration-barrier 1
cluster-require-full-coverage yes
```

**3. 创建`cmd`文件，辅助操作**
``` shell
if [ "$1" == "start" ]; then
    nohup conf/redis-server conf/7000/redis.conf > logs/out.log 2>&1 &
    echo "redis instance 7000 started"
    nohup conf/redis-server conf/7001/redis.geconf > logs/out.log 2>&1 &
    echo "redis instance 7001 started"
elif [ "$1" == "status" ]; then
    ps -ef|grep redis|grep -v 'grep'
elif [ "$1" == "stop" ]; then
    ps aux|grep redis|grep -v 'grep' | awk '{print $2}' | xargs kill -9
    echo "redis instances stopped"
elif [ "$1" == "cli" ]; then
    src/redis-cli -c -p 7000
else
    echo "$1 unrecognized parameter."
fi
```

**4、启动所有redis实例：**`./cmd start`

**5、下载安装rubygems：**`ruby setup.rb`

**6、下载安装redis gem，注意版本问题：**`gem install redis-3.2.2.gem`

**7、创建集群：**`src/redis-trib.rb create --replicas 1 10.179.141.60:7000 10.179.141.60:7001 10.179.143.18:7000 10.179.143.18:7001 10.179.75.155:7000 10.179.75.155:7001`

### Cluster手动配置集群
如果安装过程正常，到第7步就全结束了，但是也可能失败，这时候需要手动配置集群：

1. 停止所有redis实例，删除所有`nodes.conf`，重启所有redis实例
2. 编辑所有规划的主实例的`nodes.conf`，在第一行最后加上hash slot范围，例如：`fb04d1034c44e747c988d5cc44caf3ab6639fdd0 10.179.75.155:7000 master - 0 1501466893450 0 connected 10923-16383`
3. 随便登录个主实例，逐个添加其他主实例：`cluster meet 10.179.141.60 7000`
4. 依次登录从实例，添加集群中的某个实例后，选择一个node作为master：`cluster replicate fb04d1034c44e747c988d5cc44caf3ab6639fdd0`
5. 所有实例添加完毕后，集群的状态为OK：`cluster info`
<!--endsec-->

<!--sec data-title="Redis文章收集" data-id="redis_1" data-show=true ces-->
### Redis命令
* [Redis官网](http://redis.io/documentation)
* [Redis命令参考中文版](http://redisdoc.com/)

### Redis实践
* [唯品会大规模 Redis Cluster 的生产实践](http://mp.weixin.qq.com/s?__biz=MzA4Nzg5Nzc5OA==&mid=2651660079&idx=1&sn=bca50ad39792deadf167077308120264&scene=23&srcid=0603h3CpGqpli4btm4CAhwXC#rd)

## Redis故障及异常总结

### Redis Bug列表
* [Redis 3.0.X](https://raw.githubusercontent.com/antirez/redis/3.0/00-RELEASENOTES)

### Redis内存超限
* [Redis实际使用内存可能大于maxmemory限制](https://www.couyon.net/blog/using-redis-as-a-lru-cache-dont-do-it)
* [Redis Slave进行数据备份BGSAVE时可能内存突发跑满](http://chenzhenianqing.cn/articles/1069.html)
* [Redis 2.8.13 OOM crash even with maxmemory configured](https://github.com/antirez/redis/issues/2136)

### Redis数据问题
* [Redis从库备份和同步时小概率存在较严重数据错乱的Bug](http://www.tuicool.com/articles/EZruiu)
<!--endsec-->
