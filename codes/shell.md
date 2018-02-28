# Shell Cookbook

---

<!--sec data-title="编写带参数的启停脚本" data-id="shell_0" data-show=true ces-->
```bash
# ./cmd start 启动所有redis实例
if [ "$1" == "start" ]; then
    nohup conf/redis-server conf/7000/redis.conf > logs/out.log 2>&1 &
    echo "redis instance 7000 started"
    nohup conf/redis-server conf/7001/redis.conf > logs/out.log 2>&1 &
    echo "redis instance 7001 started"
# ./cmd status 查看当前运行的redis实例
elif [ "$1" == "status" ]; then
    ps -ef|grep redis|grep -v 'grep'
# ./cmd stop 停止所有redis实例
elif [ "$1" == "stop" ]; then
    ps aux|grep redis|grep -v 'grep' | awk '{print $2}' | xargs kill -9
    echo "redis instances stopped"
# ./cmd cli 打开本地7000的redis实例客户端
elif [ "$1" == "cli" ]; then
    src/redis-cli -c -p 7000
else
    echo "$1 unrecognized parameter."
fi
```
<!--endsec-->

<!--sec data-title="常用Linux命令" data-id="shell_1" data-show=true ces-->
```bash
# 查看物理CPU个数：
cat /proc/cpuinfo| grep "physical id"| sort| uniq| wc -l

# 查看每个物理CPU中core的个数(即核数)：
cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
cat /proc/cpuinfo| grep "processor"| wc -l

# 读取服务器的网络接口和IP地址：
Ipl=`/sbin/ifconfig eth1|grep "inet addr:"|awk -F ":" '{print $2}'|awk '{print $1}'`

# 获取服务器监听的TCP/UDP端口：
Tport="${Tport} `ss -tln|grep -v "127.0.0.1"|awk '{print $3}'|awk -F ":" '{print $NF}'|sort -u|grep ^[0-9]`"
Uport="${Uport} `ss -uan|grep -v "127.0.0.1"|awk '{print $4}'|awk -F ":" '{print $NF}'|sort -u|grep ^[0-9]`"

# 把占用526号端口的所有进程关掉：
kill -9 `lsof -i:526 | grep IPv4 | awk '{print $2}'`

# 修改系统时间：
date -s "2016-10-18 16:30:00"

# 将文件夹`diagnose`下所有文件权限降为`user:group`：
chown -R user:group diagnose

# 查找当前文件夹下文件名包含`hc`的所有文件：
find . -name '*hc*'

# 查看当前文件夹下所有子文件占用磁盘大小：
du -ah --max-depth=1

# 查看磁盘使用量和占用率：
df -m

# 查看Linux版本：
more /etc/issue

# 查看Linux内核版本：
uname -a

# 安装python 3，并设置成hc用户下的默认python
# 1. download and install python 3.x.x
tar -zxvf Python-3.x.x.tgz
cd Python-3.x.x
./configure
make
make install
# 2. create user hc
useradd -u 544 -d /home/hc -g users -m hc
# 3. add user profile
su - hc
vi .bash_profile
# 4. add "export PATH=$HOME/bin:/opt/python/Python-3.6.1:usr/local/bin:$PATH" to the end of bash_profile
source bash_profile

# 终止某个名称的进程：
ps aux|grep [j]ava | awk '{print $2}' | xargs kill -9

# 压缩/解压文件/文件夹：
tar -cvf target.tar sourceFiles

# 搜索当前文件夹下所有文件中包含`text`的文件及其内容：
grep -R "text"

# 远程拷贝文件夹到本地目录：
scp -r root@10.6.159.147:/opt/soft/test /opt/soft/

# 创建用户：
useradd -d /srv/druid druid

# Vim字符串全部替换：
:%s/original/replaced/g

# Python将print重定向到文件时，禁用缓存
python -u ingest-delay.py ddsLongSql 392400 > ingest-delay.log &

# 从字符串中获取匹配正则的子串
echo "jdbc_url=jdbc:oracle:thin:@10.137.33.169:1536:ora11g"|grep -Eo "([0-9]+\.){3}[0-9]+\:[0-9]+"
```
<!--endsec-->