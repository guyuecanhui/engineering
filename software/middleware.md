# 消息中间件

---

<!--sec data-title="Nats环境安装与性能测试（非Git版本）" data-id="mid_0" data-show=true ces-->
## 安装Go语言环境
1. 下载Go：<https://golang.org/dl/>；
2. 将Go安装到`/usr/local`下：`tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz`；
3. 编辑`/etc/profile`，增加`export PATH=$PATH:/usr/local/go/bin`；
4. 发布修改：`source /etc/profile`；

## 安装启动Nats
1. 下载Nats服务端：<http://nats.io/download/nats-io/gnatsd/>
2. 解压到`/home/hc/nats`，进入`nats`目录；
3. [启动Nats Server](https://nats.io/documentation/tutorials/gnatsd-install/)：`./gnatsd [-m 8222: http monitor] []`

## 性能测试
1. 下载`go-nats`工程：<https://github.com/nats-io/go-nats>，解压工程，并将根目录重命名为：`/usr/local/go/src/github.com/nats-io/go-nats`；
2. 下载`nuid`工程：<https://github.com/nats-io/nuid>，解压工程，并将根目录重命名为：`/usr/local/go/src/github.com/nats-io/nuid`；
3. 到`go-nats`工程的`examples`目录下，执行`nats-bench.go`进行测试：`go run nats-bench.go -np 5 -ns 5 -n 10000000 -ms 16 foo`；
4. `np`表示publisher的个数，`ns`表示subscriber的个数，`n`表示消息总数，`ms`表示消息大小，`foo`指定csvfile。
<!--endsec-->

