---
title: "Mysql 主从配置"
date: 2021-12-10T15:13:28+08:00
author:      "pengjl"
categories:  ["mysql" ]
---

##### Master配置

- 创建用户

```shell
create user 'slave1_user'@'%' identified by '123456';
grant replication slave on *.* to 'slave1_user'@'%';
grant all privileges on *.* to "slave1_user"@"%";
alter user "slave_user"@"%" identified with mysql_native_password by "123456"; #后续有坑
flush privileges;
```

- 修改mysql配置

```shell
[mysqld]
server-id = 1　　　　　　　　//唯一id
log-bin=mysql-bin         //其中这两行是本来就有的，可以不用动，添加下面两行即可.指定日志文件
binlog-do-db = test　　　　 //记录日志的数据库
binlog-ignore-db = mysql    //不记录日志的数据库
```

- 重启Master节点，并查看状态

```shell
show master status\G;
*************************** 1. row ***************************
             File: mysql-bin.000012
         Position: 156
     Binlog_Do_DB: test
 Binlog_Ignore_DB: 
Executed_Gtid_Set: 
1 row in set (0.00 sec)
```



##### Slave 配置

- 修改mysql配置

```shell
[mysqld]
server-id = 2 #节点ID，不可重复
log-bin=mysql-bin # binlog日志文件名
replicate-do-db = test #需要同步的数据库
replicate-ignore-db = mysql,information_schema,performance_schema
```

- 停止 slave线程，设置Master节点，启动slave线程

```shell
stop slave;
change master to master_host='172.17.0.2',master_port=3306,master_user='slave1_user',master_password='123456',master_log_file='mysql-bin.000012',master_log_pos=156;
# master_log_file和master_log_pos 是master节点中 show master status 返回的数据；
start slave;
```

- 查看Sla ve状态

```shell
show slave status\G;

#两个状态为Yes 表示成功，进行测试
Slave_IO_Running: Yes
Slave_SQL_Running: Yes
```



#### binlog半同步配置

- master安装插件

```shell
install plugin rpl_semi_sync_master soname 'semisync_master.so';
set global rpl_semi_sync_master_enabled=on;
show plugins;
```

- slave安装插件

```shell
install plugin rpl_semi_sync_slave soname 'semisync_slave.so';
set global rpl_semi_sync_slave_enabled=on;
show plugins;
```

- slave重启同步服务

```shell
stop slave io_thread; 
start slave io_thread;
```



#### 主从延时检测及优化

- 检测

主库多线程并发写，从库单线程复制binlog日志，从库复制速度慢导致数据延时，通过`percona toolkit`工具检测

- 优化

从库配置并行同步`binlog`

```shell
show variables like 'slave_parallel_workers';
set global slave_parallel_workers = 10;
set global slave_parallel_type = LOGICAL_CLOCK;
```



#### 主从高可用架构

网上的也挺多的，比如：[MHA 参考链接](https://zhuanlan.zhihu.com/p/132508138)，暂时没有测试，后续再看




#### 踩坑

- master 节点重启后，slave节点连接不上，查看状态显示如下：

```shell
show slave status\G;

Slave_IO_Running	: 	Connecting
Slave_SQL_Running	: 	Yes
Last_IO_Error			: 	error reconnecting to master 'slave_user@172.17.0.2:3306' - retry-time: 60 retries: 2 message: Authentication plugin 'caching_sha2_password' reported error: Authentication requires secure connection.
```

解决办法：

```shell
#master 节点
alter user "slave_user"@"%" identified with mysql_native_password by "123456";

```

重启master节点，然后进入slave节点查看同步状态`show slave status\G;`

