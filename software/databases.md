# 常用数据库

---

<!--sec data-title="MySQL" data-id="db_0" data-show=true ces-->
## MySQL安装
**1、解压文件**
``` shell
tar -xvf mysql-budle.tar
rpm -ivh mysql-server.rpm mysql-client.rpm mysql-devel.rpm
```

**2、查看初始密码：**`grep "temporary password" /var/log/mysql/mysqld.log`

**3、登录mysql：**`mysql -u root -p`

**4、执行以下SQL设置权限：**
```sql
SET PASSWORD = PASSWORD('Huawei@123');
ALTER USER 'root'@'localhost' PASSWORD EXPIRE NEVER;
flush privileges;
exit
```

## 创建用户和数据库
``` sql
--创建数据库，采用create schema和create database创建数据库的效果一样。
create database [数据库名称] default character set utf8 collate utf8_general_ci;
--创建用户，密码8位以上，包括：大写字母、小写字母、数字、特殊字符
create user '[用户名称]'@'%' identified by '[用户密码]';
--用户授权数据库，*代表整个数据库
grant select,insert,update,delete,create on [数据库名称].* to [用户名称];
--立即启用修改
flush  privileges ;
--取消用户所有数据库（表）的所有权限
revoke all on *.* from tester;
--删除用户
delete from mysql.user where user='tester';
--删除数据库
drop database [schema名称|数据库名称];
```

## 控制命令
``` shell
# 查看mysql安装路径
whereis mysql
# 查看安装了哪些mysql服务
rpm -qa | grep -i mysql
# 登录MySQL客户端
mysql -h 192.128.34.28 -u root
# 查看所有database
show databases;
# 查看druidperf下所有的表
use druidperf;
show tables;
```

### 查询`druid`表
```sql
-- 查看perflog-test中最近10条数据
select * from druid_pendingSegments where dataSource = 'perflog-test' order by start desc limit 10;
-- 查看perflog-liz的segment加载规则
select * from druid_rules where dataSource = 'perflog-liz';
```
<!--endsec-->

<!--sec data-title="Oracle" data-id="db_1" data-show=true ces-->
## 登录Oracle
``` shell
# 进入Oracle用户：
su - oracle
# 以管理员用户进入Oracle：
sqlplus / as sysdba
```

## Oracle控制台命令
``` sql
# 启动/停止Oracle：
startup / shutdown immediate
# 启动/停止/查看监听：
lsnrctl start / stop / status
# 添加新用户：
create user cmpadmin identified by "Huawei123";
# 修改用户权限：
grant connect,resource,dba to cmpadmin;
grant create any sequence to cmpadmin;
grant create any table to cmpadmin;
grant delete any table to cmpadmin;
grant insert any table to cmpadmin;
grant select any table to cmpadmin;
grant unlimited tablespace to cmpadmin;
grant execute any procedure to cmpadmin;
grant update any table to cmpadmin;
grant create any view to cmpadmin;
# 修改用户密码：
alter user cmpadmin identified by "Padmin12!";
# 解除用户锁定：
alter user system account unlock;
# 删除用户：
drop user cmpadmin cascade;
# 删除CMP表空间供新建：
drop user cmpadmin_test cascade;
drop user cmpapp_test cascade;
drop tablespace cmp_index_test including contents and datafiles; 
drop tablespace cmp_data_test including contents and datafiles; 
```
<!--endsec-->

<!--sec data-title="PostgreSQL" data-id="db_2" data-show=true ces-->
## SUSE 11.x下安装PostgreSQL 9.4
1. 下载安装包：http://www.enterprisedb.com/products/pgdownload.do#linux
2. 文件上传到目标主机上，执行：`chmod +x postgresql-9.4.14-1-linux-x64.run`
3. 安装PostgreSQL：`./postgresql-9.4.14-1-linux-x64.run`
4. 选择安装目录：`Installation Directory [/opt/PostgreSQL/9.4]: /home/PostgreSQL/9.4`
5. 选择数据目录：`Data Directory [/home/PostgreSQL/9.4/data]: /opt/hucheng/PostgreSQL/9.4/data`
6. 输入密码、端口，选择语言，确认安装；
7. 切换到postgres用户：`su - postgres`
8. 设置环境变量：`vi .bash_profile`，然后`source .bash_profile`
```
export PGHOME=/home/PostgreSQL/9.4
export PGDATA=/opt/hucheng/PostgreSQL/9.4/data
export PATH=$PATH:$HOME/bin:$PGHOME/bin
```
9. 初始化数据库：`initdb -E UTF-8 -D data7 --locale=zh_CN.UTF-8`
10. 在data7目录下，在`data7/pg_hba.conf`文件的IPv4 local connections下添加`host    all             all             10.134.20.99/32         trust`
11. 在`data7/postgresql.conf`中，将`#listen_addresses = 'localhost'`修改为`listen_addresses = '*'`；
12. 启动数据库

## 常用命令
### 管理命令
* 切换用户：`su - postgres`
* 启动数据库：`pg_ctl -D data7 -l logfile start`
* 停止数据库：`pg_ctl stop`
* 使用命令行登录：`psql -h 127.0.0.1 -d postgres -U postgres`

### 操作命令
* 创建数据库：`create database tools;`
* 删除数据库：`drop database testdb;`
* 查看数据库：`\l`
<!--endsec-->